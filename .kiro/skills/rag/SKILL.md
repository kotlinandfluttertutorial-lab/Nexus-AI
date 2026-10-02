# Skill: RAG (Retrieval-Augmented Generation)

## Purpose

Provides reusable expertise for implementing the RAG pipeline in Nexus AI: document ingestion, extraction, normalization, chunking, embedding, vector storage, similarity search, context assembly, and integration with AI generation.

## When to Use

Activate this skill when:
- Implementing any stage of the RAG pipeline
- Designing document and chunk domain models
- Integrating embedding providers
- Implementing or replacing the vector store
- Assembling retrieval context for AI generation
- Handling pipeline failure modes (extraction, embedding, vector store, empty retrieval)
- Writing tests for any RAG stage
- Reviewing RAG code for UI coupling or missing error handling

## Core Rules

Refer to `01-architecture.md` (RAG Architecture) and `06-ai-architecture.md` (RAG + AI) for the authoritative rules. This skill provides implementation patterns.

1. **RAG is invisible to Compose and ViewModel.** Features call an orchestration use case; RAG is opaque to them.
2. **Preserve document and chunk identity** throughout the entire pipeline.
3. **Bound all context sizes.** Never pass unbounded text to an AI provider.
4. **Every stage handles its own failure.** No silent skips.
5. **Empty retrieval is handled explicitly.** Do not generate with hallucinated context.
6. **Use fake embeddings and fake vector stores in tests.** Never real embedding API calls in unit tests.

## Pipeline Stages

```
Document (URI / bytes)
    ↓
Extraction          — extract raw text from PDF, DOCX, TXT, images (OCR), etc.
    ↓
Normalization       — clean whitespace, encoding issues, header/footer noise
    ↓
Chunking            — split into bounded, overlapping chunks with metadata
    ↓
Embedding           — generate vector embeddings per chunk
    ↓
Vector Store        — persist chunk + embedding with searchable metadata
    ↓
Similarity Search   — top-K cosine/dot-product search given a query embedding
    ↓
Context Assembly    — rank, deduplicate, format retrieved chunks into a context block
    ↓
AI Generation       — inject assembled context into AIRequest via Orchestration
```

## Domain Models (Domain layer)

```kotlin
data class Document(
    val id: String,
    val title: String,
    val source: DocumentSource,
    val mimeType: String,
    val sizeBytes: Long,
    val createdAt: Instant,
    val status: DocumentStatus,
)

data class Chunk(
    val id: String,
    val documentId: String,
    val content: String,
    val pageNumber: Int?,      // null for formats without pages
    val chunkIndex: Int,       // position within the document
    val startChar: Int,        // character offset in normalized text
    val endChar: Int,
)

data class Embedding(
    val chunkId: String,
    val vector: FloatArray,
    val dimension: Int,
    val modelId: String,       // which embedding model produced this
)

data class ScoredChunk(
    val chunk: Chunk,
    val score: Float,          // similarity score [0.0, 1.0]
    val document: Document,    // denormalized for context assembly
)

sealed interface DocumentStatus {
    data object Pending : DocumentStatus
    data object Indexing : DocumentStatus
    data object Ready : DocumentStatus
    data class Failed(val stage: PipelineStage, val reason: String) : DocumentStatus
}

enum class PipelineStage { Extraction, Normalization, Chunking, Embedding, VectorStore }
```

## Repository Interfaces (Domain layer)

```kotlin
interface DocumentRepository {
    fun getDocuments(): Flow<List<Document>>
    suspend fun getDocument(id: String): Result<Document>
    suspend fun saveDocument(document: Document): Result<Unit>
    suspend fun updateStatus(id: String, status: DocumentStatus): Result<Unit>
    suspend fun deleteDocument(id: String): Result<Unit>
}

interface ChunkRepository {
    suspend fun saveChunks(chunks: List<Chunk>): Result<Unit>
    suspend fun getChunks(documentId: String): Result<List<Chunk>>
    suspend fun deleteChunks(documentId: String): Result<Unit>
}

interface VectorStore {
    suspend fun upsert(chunk: Chunk, embedding: Embedding): Result<Unit>
    suspend fun search(queryEmbedding: Embedding, topK: Int, filter: VectorFilter? = null): Result<List<ScoredChunk>>
    suspend fun delete(documentId: String): Result<Unit>
}

interface EmbeddingProvider {
    val dimension: Int
    val modelId: String
    suspend fun embed(text: String): Result<Embedding>
    suspend fun embedBatch(texts: List<String>): Result<List<Embedding>>
}
```

## Pipeline Use Case (Domain)

```kotlin
class IndexDocumentUseCase @Inject constructor(
    private val extractor: DocumentExtractor,
    private val normalizer: TextNormalizer,
    private val chunker: DocumentChunker,
    private val embeddingProvider: EmbeddingProvider,
    private val vectorStore: VectorStore,
    private val chunkRepository: ChunkRepository,
    private val documentRepository: DocumentRepository,
) {
    suspend operator fun invoke(documentId: String): Result<Unit> {
        documentRepository.updateStatus(documentId, DocumentStatus.Indexing).getOrElse { return Result.failure(it) }

        // Stage 1: Extraction
        val rawText = extractor.extract(documentId)
            .getOrElse {
                documentRepository.updateStatus(documentId, DocumentStatus.Failed(PipelineStage.Extraction, it.message ?: ""))
                return Result.failure(RAGError.ExtractionFailed(documentId, it))
            }

        // Stage 2: Normalization
        val normalizedText = normalizer.normalize(rawText)

        // Stage 3: Chunking
        val chunks = chunker.chunk(documentId, normalizedText)
            .getOrElse { return Result.failure(RAGError.ChunkingFailed(documentId, it)) }

        chunkRepository.saveChunks(chunks).getOrElse { return Result.failure(it) }

        // Stage 4: Embedding + Stage 5: Vector Store (batched)
        chunks.chunked(EMBEDDING_BATCH_SIZE).forEach { batch ->
            val texts = batch.map { it.content }
            val embeddings = embeddingProvider.embedBatch(texts)
                .getOrElse {
                    documentRepository.updateStatus(documentId, DocumentStatus.Failed(PipelineStage.Embedding, it.message ?: ""))
                    return Result.failure(RAGError.EmbeddingFailed(documentId, it))
                }

            batch.zip(embeddings).forEach { (chunk, embedding) ->
                vectorStore.upsert(chunk, embedding)
                    .getOrElse {
                        documentRepository.updateStatus(documentId, DocumentStatus.Failed(PipelineStage.VectorStore, it.message ?: ""))
                        return Result.failure(RAGError.VectorStoreFailed(documentId, it))
                    }
            }
        }

        documentRepository.updateStatus(documentId, DocumentStatus.Ready)
        return Result.success(Unit)
    }

    companion object { private const val EMBEDDING_BATCH_SIZE = 20 }
}
```

## Retrieval + Context Assembly Use Case (Domain)

```kotlin
class RetrieveContextUseCase @Inject constructor(
    private val embeddingProvider: EmbeddingProvider,
    private val vectorStore: VectorStore,
    private val config: RAGConfig,
) {
    /**
     * Retrieves relevant chunks for a query and assembles a bounded context block.
     * Returns an empty context block if no results are found — never fabricates context.
     */
    suspend operator fun invoke(
        query: String,
        filter: VectorFilter? = null,
    ): Result<AssembledContext> {
        val queryEmbedding = embeddingProvider.embed(query)
            .getOrElse { return Result.failure(RAGError.EmbeddingFailed("query", it)) }

        val results = vectorStore.search(queryEmbedding, config.topK, filter)
            .getOrElse { return Result.failure(RAGError.RetrievalFailed(it)) }

        if (results.isEmpty()) {
            return Result.success(AssembledContext.Empty)
        }

        val context = assembleContext(results, config.maxContextChars)
        return Result.success(context)
    }

    private fun assembleContext(results: List<ScoredChunk>, maxChars: Int): AssembledContext {
        val builder = StringBuilder()
        val sources = mutableListOf<ChunkSource>()
        var charCount = 0

        for (result in results.sortedByDescending { it.score }) {
            val block = formatChunk(result)
            if (charCount + block.length > maxChars) break
            builder.append(block)
            charCount += block.length
            sources.add(ChunkSource(result.chunk.id, result.document.title, result.chunk.pageNumber))
        }

        return AssembledContext.WithContent(
            text = builder.toString(),
            sources = sources,
            retrievedCount = results.size,
        )
    }

    private fun formatChunk(result: ScoredChunk): String = buildString {
        result.chunk.pageNumber?.let { append("[Page $it] ") }
        append(result.chunk.content)
        append("\n\n")
    }
}

data class RAGConfig(
    val topK: Int = 5,
    val maxContextChars: Int = 8_000,
    val minScore: Float = 0.0f,
)

sealed interface AssembledContext {
    data object Empty : AssembledContext
    data class WithContent(
        val text: String,
        val sources: List<ChunkSource>,
        val retrievedCount: Int,
    ) : AssembledContext
}
```

## Background Indexing (WorkManager)

Document indexing runs as a `CoroutineWorker` — it survives UI lifecycle changes and can be retried:

```kotlin
@HiltWorker
class DocumentIndexingWorker @AssistedInject constructor(
    @Assisted context: Context,
    @Assisted params: WorkerParameters,
    private val indexDocumentUseCase: IndexDocumentUseCase,
) : CoroutineWorker(context, params) {

    override suspend fun doWork(): Result {
        val documentId = inputData.getString(KEY_DOCUMENT_ID)
            ?: return Result.failure()

        return indexDocumentUseCase(documentId).fold(
            onSuccess = { Result.success() },
            onFailure = {
                if (runAttemptCount < MAX_RETRIES) Result.retry() else Result.failure()
            },
        )
    }

    companion object {
        const val KEY_DOCUMENT_ID = "document_id"
        const val MAX_RETRIES = 3
    }
}
```

## Error Taxonomy

```kotlin
sealed interface RAGError {
    data class ExtractionFailed(val documentId: String, val cause: Throwable) : RAGError
    data class ChunkingFailed(val documentId: String, val cause: Throwable) : RAGError
    data class EmbeddingFailed(val documentId: String, val cause: Throwable) : RAGError
    data class VectorStoreFailed(val documentId: String, val cause: Throwable) : RAGError
    data class RetrievalFailed(val cause: Throwable) : RAGError
    data object EmptyRetrieval : RAGError   // Explicit — callers must decide how to handle
}
```

## Common Mistakes

| Mistake | Correct Approach |
|---|---|
| Passing full document text to AI without chunking | Always chunk and embed; retrieve only relevant chunks |
| Unbounded context assembly | Enforce `maxContextChars` in `assembleContext()` |
| Silent empty retrieval (no context passed, no indication to caller) | Return `AssembledContext.Empty`; caller decides to proceed without context or surface to user |
| Skipping failed chunks and continuing silently | Fail fast with structured `RAGError`; let the caller decide on retry |
| Embedding provider logic in Domain | `EmbeddingProvider` interface in Domain; implementation in Data |
| Vector store logic in Domain | `VectorStore` interface in Domain; Room-Vec / in-memory implementation in Data |
| RAG context assembly in a composable | Composable calls ViewModel → use case; context assembly is opaque |
| Indexing on the main thread | Always dispatch via WorkManager `CoroutineWorker` |

## Testing Guidance

- Use `FakeEmbeddingProvider` — deterministic fixed vectors — for all tests above the embedding boundary
- Use `FakeVectorStore` — in-memory, deterministic search results — for retrieval tests
- Test each pipeline stage as an independent unit with fakes for adjacent stages
- Test `IndexDocumentUseCase` with controlled extraction, chunking, and embedding fakes covering: success, extraction failure, embedding failure, vector store failure
- Test `RetrieveContextUseCase` with: normal results, empty results, embedding failure, vector store failure
- Test context assembly bounds: verify that `maxContextChars` is respected
- Test `DocumentIndexingWorker.doWork()` directly — inject use case fake

## Relationship to Steering

This skill applies `01-architecture.md` (RAG Architecture) and `06-ai-architecture.md` (RAG + AI Integration). Background processing rules from `02-android-development.md` apply to WorkManager usage. Testing rules from `04-testing-standards.md` govern fake usage. Steering takes precedence over any guidance here.
