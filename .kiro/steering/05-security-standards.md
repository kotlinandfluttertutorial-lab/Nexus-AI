# 05 — Security Standards

## Secrets and Credentials

**Never hardcode in source code:**
- API keys
- OAuth client secrets
- JWT signing secrets
- Encryption keys
- Passwords or PINs
- Firebase service account credentials
- Production database credentials
- Any token or secret value

Secrets must be supplied via:
- `local.properties` (git-ignored, developer machines only)
- Environment variables injected at build time via CI/CD
- Encrypted secure storage on-device (EncryptedSharedPreferences or Android Keystore)
- A secrets management system (e.g., Google Secret Manager) for server-side secrets

`local.properties` and any file containing secrets must be listed in `.gitignore`. Verify before every commit.

---

## Secure Storage

- Use `EncryptedSharedPreferences` (Jetpack Security) for sensitive on-device key-value data
- Use Android Keystore for cryptographic key storage
- Never store sensitive data in plain SharedPreferences, plain DataStore, or plain Room without encryption
- Never write sensitive data to external storage
- Never log sensitive data (see Coding Standards §Logging)

---

## Network Security

- All network communication in Staging and Production must use HTTPS
- Do not allow cleartext HTTP in production network security config
- Pin certificates where appropriate for high-sensitivity endpoints
- Define explicit timeouts on all OkHttp clients:
  - Connect timeout
  - Read timeout
  - Write timeout
- Do not disable SSL certificate verification under any circumstances in production builds
- Use a dedicated `NetworkSecurityConfig` that restricts cleartext and enforces HTTPS

---

## Input Validation

- Validate all external input before processing: user input, AI model output, MCP tool parameters, document content
- Validate tool input schemas before execution — reject invalid input with a structured error, not a crash
- Sanitize file paths, URIs, and any input that touches the filesystem
- Set maximum size limits on documents, prompts, and context windows to prevent memory exhaustion

---

## Authentication and Authorization

- Handle token expiration gracefully — refresh silently where possible, re-authenticate when required
- Invalidate tokens on logout — clear from memory and secure storage
- Do not cache credentials in memory longer than necessary
- Implement authorization checks at the use case layer, not only at the UI layer
- Do not trust client-supplied identity claims without verification

---

## Android Component Security

- Set `android:exported="false"` on all Activities, Services, BroadcastReceivers, and ContentProviders that do not need to be accessible from other apps
- Only export components that explicitly require external access; document why
- Validate all `Intent` extras received from external sources before use
- Use explicit intents for internal app communication where possible

---

## WebView Security (if used)

- Disable JavaScript unless explicitly required for the feature
- Disable `allowFileAccess` and `allowContentAccess` unless required
- Never load untrusted URLs in a WebView
- If JavaScript is required, restrict the origin with `setWebViewClient` and validate navigation
- Do not expose sensitive `JavascriptInterface` methods without strict input validation

---

## Backup and Data Protection

- Set `android:allowBackup="false"` or define an explicit `BackupRules` file that excludes sensitive files (databases, encrypted prefs, tokens)
- Ensure `fullBackupContent` rules exclude all sensitive data directories

---

## Error Handling and Information Exposure

- Never expose raw exception messages, stack traces, or internal system details to the user interface
- Map all errors to domain error types with user-safe messages before surfacing to UI
- Do not include server-side error details (e.g., SQL errors, internal paths) in user-facing messages
- Log detailed errors server-side or to a crash reporting tool; show only safe summaries to users

---

## Release Build Security

- Enable ProGuard / R8 code shrinking and obfuscation in release builds
- Disable all debug logging in release builds (`BuildConfig.DEBUG` guard or ProGuard rules)
- Disable `android:debuggable` in release builds (Gradle sets this automatically; never force-enable in release)
- Remove all development-only endpoints, hardcoded test credentials, and debug utilities before release
- Sign release builds with a properly secured keystore — keystore file and credentials must not be committed to Git

---

## Dependency Security

- Pin dependency versions — never use open ranges (`+` or `latest`) in production builds
- Review dependency licenses before inclusion
- Monitor for known CVEs in dependencies; update promptly when security patches are released
- Prefer well-maintained, widely-used libraries over obscure alternatives for security-sensitive operations (cryptography, networking, authentication)

---

## AI-Specific Security

- Never include raw user credentials or secrets in AI prompts
- Sanitize user input before including it in prompts to reduce prompt injection risk
- Validate and sanitize AI model output before using it as input to tools or system operations
- Apply size and content limits to documents and prompts passed to AI providers
- Do not expose internal tool schemas, system prompts, or architecture details to end users
