# Skill: Security

## Purpose

Provides reusable expertise for applying Nexus AI security standards to every implementation: secrets management, secure storage, network security, input validation, Android component hardening, and AI-specific security.

## When to Use

Activate this skill when:
- Adding or modifying API key / credential handling
- Implementing authentication or authorization flows
- Configuring network clients (OkHttp, Retrofit)
- Handling AI prompts, tool inputs, or document content
- Configuring AndroidManifest components (exported flags, permissions)
- Setting up ProGuard / R8 rules for release
- Reviewing any PR that touches credentials, network, storage, or AI input/output

## Core Rules

Refer to `05-security-standards.md` for the authoritative rules. This skill provides implementation patterns.

1. **No secrets in source code.** Ever. Not even in comments.
2. **HTTPS only** in staging and production.
3. **Validate all external input** before processing — user input, AI output, MCP parameters, document content.
4. **Never log sensitive data.** API keys, tokens, prompts, personal data.
5. **Minimum export surface** on Android components.

## Implementation Guidance

### Reading API Keys Safely

Inject at build time via `local.properties` → `BuildConfig`. Never hardcode.

`local.properties` (git-ignored):
```properties
GEMINI_API_KEY=your_key_here
OPENAI_API_KEY=your_key_here
```

`build.gradle.kts`:
```kotlin
import java.util.Properties

val localProps = Properties().apply {
    val file = rootProject.file("local.properties")
    if (file.exists()) load(file.inputStream())
}

android {
    defaultConfig {
        buildConfigField("String", "GEMINI_API_KEY",
            "\"${localProps.getProperty("GEMINI_API_KEY", "")}\"")
        buildConfigField("String", "OPENAI_API_KEY",
            "\"${localProps.getProperty("OPENAI_API_KEY", "")}\"")
    }
}
```

Inject via Hilt — never access `BuildConfig` directly outside the DI module:

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object ApiKeyModule {

    @Provides
    @Named("gemini_api_key")
    fun provideGeminiApiKey(): String = BuildConfig.GEMINI_API_KEY.also {
        check(it.isNotBlank()) { "GEMINI_API_KEY is not configured. See local.properties." }
    }
}
```

### Secure On-Device Storage

```kotlin
// Use EncryptedSharedPreferences for sensitive key-value data
val masterKey = MasterKey.Builder(context)
    .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
    .build()

val encryptedPrefs = EncryptedSharedPreferences.create(
    context,
    "nexusai_secure_prefs",
    masterKey,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM,
)
```

Always wrap behind a data source abstraction — never use directly in ViewModel or use case.

### OkHttp Security Configuration

```kotlin
val okHttpClient = OkHttpClient.Builder()
    .connectTimeout(30, TimeUnit.SECONDS)
    .readTimeout(60, TimeUnit.SECONDS)   // Longer for AI streaming
    .writeTimeout(30, TimeUnit.SECONDS)
    .addInterceptor(AuthInterceptor(apiKeyProvider))
    // Never log request/response bodies in production
    .apply {
        if (BuildConfig.DEBUG) {
            addInterceptor(HttpLoggingInterceptor().apply {
                level = HttpLoggingInterceptor.Level.HEADERS // Not BODY — may contain sensitive data
            })
        }
    }
    .build()
```

### Network Security Config

`res/xml/network_security_config.xml`:
```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
    <!-- Allow cleartext for local development emulator only -->
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="true">10.0.2.2</domain>
    </domain-config>
</network-security-config>
```

`AndroidManifest.xml`:
```xml
<application
    android:networkSecurityConfig="@xml/network_security_config"
    android:allowBackup="false"
    ... >
```

### Input Validation Before AI Processing

```kotlin
object PromptValidator {
    private const val MAX_PROMPT_LENGTH = 32_000

    fun validate(input: String): Result<String> {
        if (input.isBlank()) return Result.failure(ValidationError.BlankInput)
        if (input.length > MAX_PROMPT_LENGTH) return Result.failure(
            ValidationError.InputTooLong(input.length, MAX_PROMPT_LENGTH)
        )
        return Result.success(input.trim())
    }
}
```

### Tool Input Validation

```kotlin
// Validate before execution — never trust AI-provided tool parameters
class SearchTool : ApplicationTool {
    override val name = "search"
    override val description = "Search the web for information"

    override suspend fun execute(input: ToolInput): Result<ToolOutput> {
        val query = input.getString("query")
            ?: return Result.failure(ToolError.MissingParameter("query"))
        if (query.isBlank() || query.length > 500)
            return Result.failure(ToolError.InvalidParameter("query", "must be 1-500 chars"))

        // Validated — proceed
        return performSearch(query)
    }
}
```

### Secure Logging

```kotlin
object SecureLogger {
    private val sensitiveKeyPattern = Regex(
        "(?i)(api_?key|token|password|secret|auth|bearer|key=)[^\\s&\"']*"
    )

    fun redact(message: String): String =
        message.replace(sensitiveKeyPattern) { match ->
            "${match.groupValues[1]}[REDACTED]"
        }
}

// Always guard debug logs
if (BuildConfig.DEBUG) {
    Log.d(TAG, SecureLogger.redact(debugMessage))
}
```

### AndroidManifest Hardening Checklist

```xml
<!-- Activities: exported only if needed for deep links / external launch -->
<activity
    android:name=".MainActivity"
    android:exported="true"   <!-- Only main entry; all others false -->
    android:launchMode="singleTop" />

<!-- Workers, Services: never exported unless explicitly required -->
<service
    android:name=".DocumentIndexingService"
    android:exported="false" />
```

### Release ProGuard Rules

```proguard
# Keep domain entities (used with serialization)
-keep class com.nexusai.domain.entity.** { *; }

# Keep Hilt generated code
-keep class dagger.hilt.** { *; }
-keep class **_HiltModules* { *; }

# Remove all logging in release
-assumenosideeffects class android.util.Log {
    public static int v(...);
    public static int d(...);
}
```

## Security Review Checklist

Before marking any implementation complete, verify:

- [ ] No API keys, tokens, or secrets in source code or committed files
- [ ] All network clients have explicit timeouts configured
- [ ] HTTPS enforced in network security config
- [ ] No sensitive data in logs (use `BuildConfig.DEBUG` guards)
- [ ] All Android components have correct `exported` flags
- [ ] `allowBackup="false"` or backup rules exclude sensitive data
- [ ] AI prompt inputs are validated for length and content before sending
- [ ] Tool inputs are validated against schema before execution
- [ ] Error messages surfaced to UI contain no internal system details
- [ ] R8/ProGuard configured for release builds
- [ ] `local.properties` is in `.gitignore`

## Common Mistakes

| Mistake | Correct Approach |
|---|---|
| `BuildConfig.API_KEY` accessed in a ViewModel | Inject via Hilt `@Named` from a DI module |
| Logging full AI prompt/response body | Log only request ID and status; never content |
| `cleartextTrafficPermitted="true"` globally | Allow only for `10.0.2.2` in debug; enforce HTTPS everywhere else |
| `android:exported` omitted on component | Always set explicitly; defaults changed in Android 12+ |
| Raw exception message shown in UI snackbar | Map to `UiText` with a user-safe string |
| `allowBackup="true"` default | Set `false` or define explicit backup rules |
| Storing tokens in plain DataStore | Use `EncryptedSharedPreferences` for sensitive values |
| Trust user-supplied tool parameters without validation | Always validate against tool schema first |

## Relationship to Steering

This skill applies `05-security-standards.md`. It also intersects with `06-ai-architecture.md` for AI-specific security. Steering files take precedence over any guidance here.
