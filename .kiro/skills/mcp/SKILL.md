# Skill: MCP (Model Context Protocol)

## Purpose

Provides reusable expertise for integrating MCP servers into Nexus AI: connection lifecycle, capability discovery, tool discovery, tool invocation, converting MCP tools to application-level tools, error handling, timeout, cancellation, and testing.

## When to Use

Activate this skill when:
- Implementing MCP server connection management
- Discovering and registering MCP tools
- Converting MCP tool definitions to application-level `ApplicationTool` models
- Invoking MCP tools from agent or orchestration code
- Handling MCP connection errors, timeouts, or reconnects
- Writing tests for MCP integration
- Reviewing MCP-related code for boundary violations

## Core Rules

Refer to `01-architecture.md` (MCP Dependency Chain) and `06-ai-architecture.md` (MCP + AI section) for the authoritative rules. This skill provides implementation patterns.

1. **MCP transport never reaches Compose or ViewModel.** The boundary is the MCP Adapter in the Data layer.
2. **MCP tools become `ApplicationTool` models** before being used by orchestration or agents.
3. **Validate all MCP tool inputs** before invocation — never pass raw AI-provided parameters without validation.
4. **Every invocation has a timeout.** No open-ended MCP calls.
5. **Handle all MCP error states** — connection failure, tool not found, invocation error, timeout — and map to domain error types.
6. **Use `FakeMCPClient` in all tests** above the transport layer.

## Architecture Boundary

```
Agent / AI Orchestration  (Domain — calls ApplicationTool)
    ↓
ApplicationTool           (Domain — common tool abstraction)
    ↓
MCPToolAdapter            (Data — implements ApplicationTool, wraps MCPClient)
    ↓
MCPClient                 (Data — transport abstraction interface)
    ↓
MCPTransport              (Data — actual stdio/SSE/WebSocket connection)
```

Orchestration and agents interact with `ApplicationTool` only. They have no imports from MCP packages.

## Implementation Guidance

### MCP Client Interface (Data layer boundary)

```kotlin
// Domain-facing MCP abstraction — defined in domain or at the data/domain boundary
interface MCPClient {
    suspend fun connect(): Result<Unit>
    suspend fun disconnect()
    val isConnected: Flow<Boolean>
    suspend fun discoverTools(): Result<List<MCPToolDefinition>>
    suspend fun invokeTool(name: String, input: MCPToolInput): Result<MCPToolOutput>
}

data class MCPToolDefinition(
    val name: String,
    val description: String,
    val inputSchema: JsonObject,   // JSON Schema for parameter validation
    val requiredParameters: List<String>,
)
```

### MCP Tool Adapter — Converting to ApplicationTool

```kotlin
// Data layer — bridges MCPClient and the ApplicationTool domain abstraction
class MCPToolAdapter(
    private val definition: MCPToolDefinition,
    private val client: MCPClient,
    private val timeout: Duration = 30.seconds,
) : ApplicationTool {

    override val name: String = definition.name
    override val description: String = definition.description
    override val inputSchema: ToolSchema = definition.inputSchema.toDomainSchema()

    override suspend fun execute(input: ToolInput): Result<ToolOutput> {
        // Validate required parameters before sending to MCP
        val missingParams = definition.requiredParameters.filter { input.getString(it) == null }
        if (missingParams.isNotEmpty()) {
            return Result.failure(ToolError.MissingParameters(missingParams))
        }

        return withTimeout(timeout) {
            client.invokeTool(name, input.toMCPInput())
                .map { it.toDomainOutput() }
                .mapFailure { MCPError.InvocationFailed(name, it).toDomainToolError() }
        }
    }
}
```

### MCP Server Manager (Data layer)

```kotlin
class MCPServerManager @Inject constructor(
    private val clientFactory: MCPClientFactory,
    private val toolRegistry: ToolRegistry,
    private val scope: CoroutineScope,
) {
    private val connectedClients = mutableMapOf<String, MCPClient>()

    /** Connect to a configured MCP server and register its tools. */
    suspend fun connectServer(config: MCPServerConfig): Result<Unit> = runCatching {
        val client = clientFactory.create(config)
        client.connect().getOrThrow()

        val tools = client.discoverTools().getOrThrow()
        val adapters = tools.map { MCPToolAdapter(it, client) }
        adapters.forEach { toolRegistry.register(it) }

        connectedClients[config.id] = client
        observeConnectionHealth(config.id, client)
    }

    /** Disconnect a server and unregister its tools. */
    suspend fun disconnectServer(serverId: String) {
        connectedClients[serverId]?.let { client ->
            toolRegistry.unregisterBySource(serverId)
            client.disconnect()
            connectedClients.remove(serverId)
        }
    }

    private fun observeConnectionHealth(serverId: String, client: MCPClient) {
        scope.launch {
            client.isConnected.collect { connected ->
                if (!connected) {
                    // Mark tools from this server as unavailable
                    toolRegistry.markUnavailable(serverId)
                    Log.w(TAG, "MCP server $serverId disconnected")
                }
            }
        }
    }

    private companion object { const val TAG = "MCPServerManager" }
}
```

### MCP Server Configuration

```kotlin
// Domain entity — describes an MCP server configuration
data class MCPServerConfig(
    val id: String,
    val name: String,
    val transport: MCPTransportConfig,
    val enabled: Boolean = true,
    val timeoutMs: Long = 30_000,
)

sealed interface MCPTransportConfig {
    data class StdIO(val command: String, val args: List<String>) : MCPTransportConfig
    data class SSE(val url: String, val headers: Map<String, String> = emptyMap()) : MCPTransportConfig
    data class WebSocket(val url: String) : MCPTransportConfig
}
```

### Error Mapping

```kotlin
sealed interface MCPError {
    data class ConnectionFailed(val serverId: String, val cause: Throwable) : MCPError
    data class ToolNotFound(val toolName: String) : MCPError
    data class InvocationFailed(val toolName: String, val cause: Throwable) : MCPError
    data class TimeoutError(val toolName: String, val timeoutMs: Long) : MCPError
    data class InvalidResponse(val toolName: String, val details: String) : MCPError
    data object NotConnected : MCPError
}
```

All `MCPError` types are mapped to `ToolError` domain types before reaching orchestration or agents. Orchestration never imports `MCPError`.

### Input Validation Pattern

```kotlin
object MCPInputValidator {

    fun validate(input: MCPToolInput, schema: JsonObject): Result<MCPToolInput> {
        // Validate required fields
        val required = schema.getArray("required") ?: JsonArray()
        val missingFields = required.filterIsInstance<String>()
            .filter { !input.contains(it) }

        if (missingFields.isNotEmpty()) {
            return Result.failure(ToolError.MissingParameters(missingFields))
        }

        // Validate no oversized string values
        input.entries.forEach { (key, value) ->
            if (value is String && value.length > MAX_PARAM_LENGTH) {
                return Result.failure(ToolError.InvalidParameter(key, "exceeds max length"))
            }
        }

        return Result.success(input)
    }

    private const val MAX_PARAM_LENGTH = 10_000
}
```

## Common Mistakes

| Mistake | Correct Approach |
|---|---|
| Calling MCP transport from a ViewModel | MCP is Data-layer only; ViewModel calls use case → tool abstraction |
| Importing MCP SDK types in Domain layer | MCP types stay in Data; domain sees only `ApplicationTool` |
| Not validating required parameters before invocation | Always validate against `inputSchema.required` first |
| No timeout on MCP tool invocation | Wrap every invocation with `withTimeout()` |
| Swallowing `MCPError.ConnectionFailed` | Propagate as `ToolError`; surface to orchestration to decide retry |
| Registering MCP tools without capability check | Check server capabilities before registering unsupported tools |
| Reconnect logic in a coroutine without cancellation handling | Always respect `CancellationException` in reconnect loops |
| Exposing `JsonObject` / raw MCP types to Orchestration | Convert to typed `ToolInput` / `ToolOutput` at the adapter boundary |

## Testing Guidance

- Use `FakeMCPClient` from `core/testing/` for all tests above the transport layer
- Configure `FakeMCPClient.availableTools` per test to simulate different server capabilities
- Configure `FakeMCPClient.invocationResults` to simulate success and failure per tool
- Test `MCPToolAdapter` with a `FakeMCPClient` — cover: missing params, timeout, invocation error, success
- Test `MCPServerManager` connection and disconnection with fake clients
- Never use real MCP server connections in unit tests
- For integration tests, use a local MCP server fixture or a recorded response replay

## Relationship to Steering

This skill applies `01-architecture.md` (MCP Dependency Chain) and `06-ai-architecture.md` (MCP + AI). Security rules from `05-security-standards.md` apply to MCP server credentials and connection strings. Steering takes precedence.
