# Audio Quote -
### Share just the part of a long voice message that matters

Android Audio Quotes — Select and share only the relevant part of a long voice message instead of sending the entire recording.

## Platform - Android

<img src=docs/images/short.png width=70% >

[Demo Video](docs/videos/demo.mp4)
## Flow

1. Open Calls → Scheduled Calls to see all upcoming calls sorted by date.

2. Tap a call to open its group or the call icon to open the scheduled call.

3. Long-press a scheduled call message → Reschedule Call.

4. Select a new date and time → Confirm.


# How to apply

Based on official drklo/telegram client
```
git clone --recursive --shallow-submodules https://github.com/DrKLO/Telegram.git Telegram
cd telegram
git checkout --detach 3de37c8cfb039452be7398909461da6288babf15
```

Download
```
https://github.com/col3name/timequoute-voice-message/blob/main/timequote_changes.patch
```

Apply patches
```
git submodule update --init --recursive

git apply --check ./timequote_changes.patch
git apply ./timequote_changes.patch

git diff --stat
```


# How to apply

Based on official drklo/telegram client
```
git clone --recursive --shallow-submodules https://github.com/DrKLO/Telegram.git Telegram
cd telegram
git checkout --detach 3de37c8cfb039452be7398909461da6288babf15

git submodule update --init --recursive
```


Apply patches
```
git apply --check ./timequote_changes.patch
git apply ./timequote_changes.patch
```
