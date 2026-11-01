# Fast Search by Sender

### Find all messages from a group member directly from the avatar context menu

![Fast search by sender preview](docs/images/fast-search-user-messages.png)

## Android

[Watch the demo](docs/videos/fast-search-user-messages.mp4)

## Task description

In a group chat, long-press a member's avatar next to any message to open the avatar context menu. Select **Search Messages** to immediately open message search with that sender selected.

The user does not need to open the search filters and select the sender manually. The search request uses the existing `messages.search` API and passes the selected sender through `from_id`.

## User flow

1. Open a group chat.
2. Long-press a member's avatar next to one of their messages.
3. Select **Search Messages**.
4. Telegram opens the search UI with the sender filter already applied.

## Implementation

- `ChatActivity` adds the sender search action to the avatar context menu.
- `AvatarPreviewer` supports the same action in the avatar preview menu.
- `MediaDataController` reuses `messages.search` with `from_id`.
- `MessagesSearchAdapter` displays the resulting messages.
- The Russian localization uses **Найти все сообщения**.

## Patch

The ready-to-apply patch is available at [`fast-search-user-messages.patch`](fast-search-user-messages.patch).

## How to apply

Based on the official DrKLO/Telegram client:

```bash
git clone --recursive --shallow-submodules https://github.com/DrKLO/Telegram.git Telegram
cd Telegram
git checkout --detach 3de37c8cfb039452be7398909461da6288babf15
git submodule update --init --recursive
git apply --check /path/to/fast-search-user-messages.patch
git apply /path/to/fast-search-user-messages.patch
```
