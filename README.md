# Conditional Send — send large videos when Wi-Fi is available

Conditional Send is a Telegram Android feature concept and patch for deferring large video uploads until the device has an unmetered Wi-Fi connection. It is designed to save mobile data while keeping the original Telegram send flow intact: the user sends a video once, and Telegram resumes the upload when Wi-Fi becomes available.

> **Status:** Patch-backed concept. This repository contains `conditional-send.patch`, not a complete Telegram Android checkout. The patch has not been applied or built in this repository, so the behavior below describes the intended flow and the parts visible in the patch; it is not a claim that the feature is production-ready.

▶︎ [`conditional-send.patch`](conditional-send.patch) — the current Telegram Android integration patch.

## Demo and screenshots

▶︎ [Watch the demo video](docs/videos/demo.mp4)

<video controls width="772" src="docs/videos/demo.mp4">
  Your browser does not support embedded video. [Open the demo video](docs/videos/demo.mp4).
</video>

| Deferred message | Send over Wi-Fi dialog | Large video sending settings |
|---|---|---|
| ![Deferred video message](docs/images/1.png) | ![Send over Wi-Fi dialog](docs/images/2.png) | ![Large video sending settings](docs/images/3.png) |

## Repository

| Path | What |
|---|---|
| `conditional-send.patch` | Adds Conditional Send hooks to Telegram Android's sending flow, chat UI, data settings, localized strings, and WorkManager dependency. |
| `README.md` | Feature flow, integration notes, and known limitations. |
| `docs/videos/demo.mp4` | Short screen recording of the Conditional Send flow. |
| `docs/images/1.png` | Deferred video message with the Wi-Fi waiting status. |
| `docs/images/2.png` | Confirmation dialog for sending a large video over Wi-Fi. |
| `docs/images/3.png` | Large video sending mode settings. |

## Feature

| Target file in the Telegram Android checkout | What |
|---|---|
| `TMessagesProj/src/main/java/org/telegram/messenger/ConditionalSend.java` | Stores the Wi-Fi wait flag, reads the mode and threshold, checks the active network, and schedules resume work. |
| `TMessagesProj/src/main/java/org/telegram/messenger/SendMessagesHelper.java` | Detects large videos, preserves deferred messages, and resumes the existing unsent-message path. |
| `TMessagesProj/src/main/java/org/telegram/ui/ChatActivity.java` | Shows the decision dialog and provides Send now / Cancel actions for a waiting message. |
| `TMessagesProj/src/main/java/org/telegram/ui/Cells/ChatMessageCell.java` | Displays the local waiting status in the outgoing message cell. |
| `TMessagesProj/src/main/java/org/telegram/ui/DataSettingsActivity.java` | Adds the large-video sending mode and threshold controls under Data and Storage. |
| `TMessagesProj/src/main/res/values/strings.xml` | Adds English UI strings for the dialog, settings, status, and actions. |
| `TMessagesProj/build.gradle` | Adds the AndroidX WorkManager dependency. |

## How it works

* **Detection.** When the user sends a video above the configured size threshold over a metered connection, Conditional Send can intervene. Videos already on Wi-Fi continue through the normal Telegram send path.
* **Decision dialog.** In Ask before sending mode, Telegram shows **Send over Wi-Fi?** with two choices: **Wait for Wi-Fi** places the video in the local outgoing queue; **Send now** uploads it over the current connection.
* **Local message.** A deferred video appears immediately in the originating chat on the sender's device. Its status reads **Deferred until Wi-Fi**, while the upload is still waiting.
* **Automatic resume.** A WorkManager job requires an unmetered network. When the constraint is met, it asks Telegram's existing unsent-message flow to retry the waiting upload.
* **Message actions.** Tapping a waiting message exposes **Send now** and **Cancel sending**. Send now removes the Wi-Fi gate and retries the same outgoing message; Cancel removes it through Telegram's normal cancellation path.
* **Delivery.** After the upload is handed back to Telegram's normal sender, the message is expected to behave like an ordinary outgoing video with the usual upload and delivery indicators.

## Settings

The target UX adds a **Settings → Data and Storage → Large video sending** section.

| Mode | Behavior |
|---|---|
| **Always send immediately** | Preserve standard Telegram behavior, regardless of network type. |
| **Ask before sending** | Ask when a video exceeds the configured threshold and the current connection is metered. This is the intended default. |
| **Automatically wait for Wi-Fi** | Put qualifying videos into the local queue without showing the dialog when Wi-Fi is unavailable. |

The target threshold values are **50 MB, 100 MB, 200 MB, and 500 MB**. The intended default is **100 MB**.

## User flow

1. The user selects a large video and taps **Send**.
2. If the connection is metered, Telegram either shows the decision dialog or applies the selected automatic mode.
3. Choosing **Wait for Wi-Fi** adds the video to the outgoing queue and keeps a local message placeholder in the chat.
4. The user can continue using Telegram and switch between chats.
5. When an eligible Wi-Fi connection becomes available, the queued upload resumes automatically.
6. Until then, the user can tap the waiting message to send it immediately or cancel it.

A separate manual shortcut is also part of the target UX: a long press on the Send button should expose **Send when on Wi-Fi**, allowing a single video to be deferred regardless of the global mode.

## Queue and background behavior

The patch stores the Wi-Fi requirement on the outgoing Telegram message using the `conditional_send_wifi` parameter instead of creating a separate upload object. This keeps the queue entry attached to the message that the user sees in the chat.

WorkManager schedules unique work per account with `NetworkType.UNMETERED`. The worker checks the connection and re-enters Telegram's existing `checkUnsentMessages()` path. This is intended to provide:

* persistence across app process restarts and normal device reboots supported by WorkManager;
* retry after a temporary network failure;
* a single message identity for retrying, rather than creating duplicate outgoing messages;
* explicit user overrides for Send now and Cancel sending.

Android may delay background work because of system scheduling, battery restrictions, permissions, or the device's current state. Automatic sending therefore occurs when Android allows the worker to run.

## Integration into Telegram Android

The patch targets the official Telegram Android sources. No base commit is recorded in this repository, so use a compatible checkout and inspect the patch before applying it.

```sh
git clone --recursive --shallow-submodules https://github.com/DrKLO/Telegram.git Telegram
cd Telegram

git checkout --detach 3de37c8cfb039452be7398909461da6288babf15

git apply --check /path/to/timequoute-voice-message/conditional-send.patch
git apply /path/to/timequoute-voice-message/conditional-send.patch
```

Then build the Telegram Android project using its normal Gradle or Android Studio workflow. A successful build and runtime behavior have not been verified here.

## Notes and limits

* The local repository is a patch container, not a checked-out Telegram Android source tree; the patch must be applied to the target checkout before compilation.
* The inspected patch currently defines a `20 MB` default and a slider containing `20/50/100/200/500 MB`. That does not match this feature specification's `100 MB` default and `50/100/200/500 MB` choices; the implementation should be aligned before release.
* The inspected patch contains the settings, dialog, waiting status, actions, message flag, and WorkManager resume path. A long-press hook for **Send when on Wi-Fi** was not found in the patch and remains a target UX requirement.
* The no-duplicate and restart-resilience behavior is based on reusing Telegram's existing unsent-message retry path; it has not been independently tested in this repository.
* No automated tests or pinned upstream base revision is included yet; screenshots and a demo recording are included under `docs/`.
