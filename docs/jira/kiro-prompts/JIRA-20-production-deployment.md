# Deployment & Production Settings Screen

> **Roadmap module:** 20 — Deployment & Production AI Applications
> **Epic:** Agentic AI Full Stack — Android Client
> **Backend dependency:** NAI-AI-20 (Cloud Run deployed FastAPI; /health + /readiness must be live)

---

You are implementing JIRA-20 — Deployment & Production Settings Screen for Nexus AI.

## Jira Title

Deployment & Production Settings Screen

## Description

Deliver the production settings screen where users configure the active backend environment
(Local / Stage / Production), manage API keys, toggle theme, and view app/build information.
Also completes production hardening: R8/ProGuard configuration, GitHub Actions CI/CD pipeline,
and release build validation.

## Required Skills

- android
- compose
- security
- testing

## Pre-Implementation Checklist

Before making any changes:

1. Read and understand:
   - This ticket and all Acceptance Criteria below
   - `.kiro/steering/05-security-standards.md` — Release Build Security section (mandatory)
   - `.kiro/steering/07-git-standards.md` — Release Tagging section
   - `.kiro/skills/android/SKILL.md`, `.kiro/skills/security/SKILL.md`
2. Inspect all previous tickets — this ticket completes the production configuration
   for the entire app.
3. API keys must use `EncryptedSharedPreferences` — never plain storage or BuildConfig literals.
4. R8/ProGuard rules must not strip Hilt, Retrofit, Room, or serialization classes.

## Out of Scope

- Push notifications or remote configuration
- In-app update mechanism

## Architecture Rules

- Environment selection stored in `DataStore<Preferences>` via `AppConfigRepository`.
- Each environment has its own `baseUrl` and `apiKey` entry — stored as a map keyed by env name.
- `AppConfig` domain model: activeEnvironment, environmentMap (Map<String, EnvironmentConfig>).
- `EnvironmentConfig` data class: name, baseUrl (stored in DataStore), apiKey (in EncryptedSharedPrefs).
- Theme preference stored in DataStore — system / light / dark.
- Release build: `minifyEnabled = true`, `shrinkResources = true`, R8 full mode.

## Implementation Task

### Feature: Settings Screen (`feature/settings/`)
Grouped settings sections:

#### Backend Environment
- Environment selector row: Local / Stage / Production (radio buttons / segmented control)
- Selected environment base URL display + Edit action → URL edit bottom sheet
- Health Check button for selected environment → calls `/health`, shows result chip

#### API Keys
- API key management cards per provider (OpenAI, Gemini, etc.)
- Each card: provider name, masked key preview ("sk-…4a2f"), Edit / Clear actions
- Keys stored in `EncryptedSharedPreferences` — masked in all displays and logs

#### Appearance
- Theme selector: System / Light / Dark (segmented control)
- Theme change applies immediately without restart

#### About
- App version (from `BuildConfig.VERSION_NAME`)
- Build type badge (DEBUG / RELEASE)
- Active environment badge
- Architecture documentation link (opens `docs/` in a WebView or browser)

### Production Hardening
- `build.gradle` release variant: `minifyEnabled true`, `shrinkResources true`
- `proguard-rules.pro`: keep rules for Hilt, Retrofit, Kotlin serialization, Room entities
- `.github/workflows/android-ci.yml`: triggers on push/PR, runs `testDebugUnitTest` + `assembleDebug`
- `local.properties.example` documenting all required keys

## Acceptance Criteria

| # | Criterion |
|---|---|
| AC1 | Environment selector switches between Local, Stage, and Production; base URL updates immediately. |
| AC2 | Selected environment URLs are persisted in DataStore per environment. |
| AC3 | API keys are stored in `EncryptedSharedPreferences` and shown only as masked previews. |
| AC4 | Theme selector changes the app theme immediately without restart. |
| AC5 | About section shows app version, build type, and active environment. |
| AC6 | Health check button calls `/health` for the selected environment and shows the result. |
| AC7 | Release build succeeds with R8 enabled and no critical class stripping. |
| AC8 | R8/ProGuard rules preserve Hilt, Retrofit, Kotlin serialization, and Room. |
| AC9 | GitHub Actions CI pipeline builds debug and runs unit tests successfully. |
| AC10 | No API keys or base URLs are hardcoded anywhere in source code. |
| AC11 | Debug and release builds both succeed. |

## Security Requirements (Non-Negotiable)

- No API keys, base URLs, or credentials in `build.gradle`, `strings.xml`, or any source file.
- API keys supplied at runtime via the settings screen and stored in `EncryptedSharedPreferences`.
- Keystore credentials never committed to Git.
- Release builds have no `android:debuggable="true"`.
- No `DEBUG` or `VERBOSE` log statements in release builds.

## Workflow

1. Implement `AppConfigRepository` with DataStore for environment selection.
2. Update primary Retrofit client to source base URL reactively from DataStore.
3. Implement `SettingsViewModel` and settings screen.
4. Implement theme DataStore preference with immediate application.
5. Configure R8/ProGuard in release variant.
6. Write `proguard-rules.pro` with required keep rules.
7. Create `.github/workflows/android-ci.yml`.
8. Run: `./gradlew testDebugUnitTest assembleDebug assembleRelease lint`.
9. Verify every AC individually with build output evidence.

## Final Response Format

Provide:
- Implementation Summary
- Files Created / Modified
- Environment Switch Evidence (base URL changes on switch)
- Health Check Evidence (for each environment)
- Release Build Output (R8 stats — classes removed/kept)
- GitHub Actions Workflow Evidence (workflow file + successful run)
- Tests Executed and Results
- AC1–AC11 PASS/FAIL with evidence
- Known Limitations

> Never claim release readiness without actual build output evidence.
> Never claim an AC is PASS without evidence.
