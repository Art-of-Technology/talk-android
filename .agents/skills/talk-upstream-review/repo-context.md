<!--
  - SPDX-FileCopyrightText: 2026 Nextcloud GmbH and Nextcloud contributors
  - SPDX-License-Identifier: GPL-3.0-or-later
-->
# Android upstream review context

- Fork: `Art-of-Technology/talk-android`; source: `nextcloud/talk-android`, reference branch `master`.
- Verify the fork integration branch from live branch/PR evidence before preparing any future PR; do not infer it from upstream `master` or local checkout names. Never target upstream.
- Follow root `AGENTS.md`, `CONTRIBUTING.md`, and `SETUP.md`; preserve the existing human contribution restrictions and AI disclosure requirements.
- This map identifies sensitive surfaces, not a claim that they contain fork customizations. Establish the actual fork delta and candidate overlap before assessing compatibility.

## Source and compatibility map

Paths below are relative to `app/src/main/java/com/nextcloud/talk/` unless qualified.

| Surface | Review focus and evidence |
| --- | --- |
| Authentication and accounts | `account/`, `login/`, `data/user/`, API and cookie handling: login callbacks, session expiry, account switching, credential isolation and logout cleanup. |
| Native notifications and push | `models/json/notifications/`, `models/json/push/`, `callnotification/`, workers and notification handlers; `app/src/gplay/` Firebase token worker. Trace server payloads, registration, decryption, actions, background restrictions and permission changes. |
| Chat, bots and buttons | `chat/`, rich message models and action handlers: trace any changed bot/action payload end to end; verify native rendering, capabilities, permissions, fallback for unsupported actions, and correct account/room routing. Do not assume web-only bot buttons exist natively. |
| Calls | `call/`, `webrtc/`, `signaling/`, foreground services: verify MCU and no-MCU paths, reconnection, incoming calls, audio routing, microphone/camera permissions and background behavior. |
| Deep links and sharing | `app/src/main/AndroidManifest.xml`, account callbacks and URI/share utilities: cold/warm launch, selected account, room authorization, malformed links and safe external navigation. |
| Build, privacy and release | `app/build.gradle.kts`, flavor manifests, dependency versions, database migrations and permissions: preserve generic/gplay separation, application IDs, signing compatibility, encrypted storage and minimal sensitive logging. Keep deployment branding and signing material in ignored configuration. |

## Validation map for a future authorized implementation

- Use the checked-in Gradle wrapper and `SETUP.md`; Windows uses `gradlew.bat`. CI currently selects JDK 21; inspect current SDK requirements in `app/build.gradle.kts` rather than relying on historical prose.
- Run `./gradlew detekt ktlintCheck` (root guidance and `.github/workflows/check.yml`).
- Unit workflow: `./gradlew --no-daemon testGplayDebugUnit`; select related tests under `app/src/test/`, including account/login, WebPush encryption, chat actions, account cookie isolation and both signaling modes.
- Build affected flavors with `./gradlew assembleGenericDebug assembleGplayDebug`; ensure Play-only dependencies do not enter generic paths. Include lint/check tasks when candidate changes require them.
- Instrumented tests: `./gradlew connectedAndroidTest`; `app/src/androidTest/` includes login, URI/share and database coverage. Follow repository server/test credential setup; never commit credentials or report unrun tests as passing.
- Device smoke checks: notification receive/open/reply, revoked permission, expired login, multi-account routing, bot/action payload fallback, deep links and incoming calls in foreground/background. Cover both supported push flavor behaviors and MCU/no-MCU calls.
- For candidate protocol changes, test against the intended server/Talk capabilities and relevant older supported combinations; record versions and unavailable hardware/services explicitly.
- A documentation-only review does not require an Android build. Recommendations must list candidate-specific validation and any remaining device, signing or server compatibility uncertainty.
