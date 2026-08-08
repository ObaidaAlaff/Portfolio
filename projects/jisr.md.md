<h1 align="center">LinguaLink</h1>

<p align="center"><b>Speak your language, hear theirs — real-time translated chat, calls, and media.</b></p>

<p align="center">
  <img src="https://img.shields.io/badge/Flutter-3.7-02569B?style=flat-square&logo=flutter&logoColor=white" alt="Flutter">
  <img src="https://img.shields.io/badge/Dart-0175C2?style=flat-square&logo=dart&logoColor=white" alt="Dart">
  <img src="https://img.shields.io/badge/Supabase-3ECF8E?style=flat-square&logo=supabase&logoColor=white" alt="Supabase">
  <img src="https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white" alt="PostgreSQL">
  <img src="https://img.shields.io/badge/Firebase-FFCA28?style=flat-square&logo=firebase&logoColor=black" alt="Firebase">
  <img src="https://img.shields.io/badge/Agora_RTC-099DFD?style=flat-square" alt="Agora">
  <img src="https://img.shields.io/badge/FFmpeg-007808?style=flat-square&logo=ffmpeg&logoColor=white" alt="FFmpeg">
  <img src="https://img.shields.io/badge/status-in%20progress-F59E0B?style=flat-square" alt="Status">
</p>

## Overview

LinguaLink removes the language barrier from everyday communication. Two people chat, call, and exchange voice notes while each side reads and hears their own language — translation happens inside the pipeline, not as a manual step the user has to trigger.

It's built for people who work, study, or stay in touch across a language gap. Beyond messaging, it includes a pronunciation trainer and a studio that translates uploaded audio and video files. Arabic and English are supported end to end, with full RTL layout and a light/dark theme.

## Features

- **Auto-translated messaging** — each participant reads in their own language; the original is one tap away
- **Voice notes with spoken translation** — the recipient hears the message in their language, with transcript and translation inline
- **Video messages with burned-in subtitles** — timed transcription produces an SRT that ffmpeg renders onto the video
- **Live translated calls** — audio and video over Agora with real-time captions streamed via WebSocket
- **Translation Studio** — upload any audio or video file, pick source and target language, download the translated result
- **Pronunciation practice** — scored exercises with word-by-word accuracy, fluency, and completeness feedback
- **Subscription tiers** — free through unlimited, with usage metering for messages and minutes
- **Bilingual and bidirectional** — Arabic and English with correct RTL mirroring throughout

## Screenshots

> Add images to `screenshots/` using the filenames below.

### Messaging

| Chats | Conversation | Voice pipeline |
|:---:|:---:|:---:|
| ![Chats](screenshots/chats-list.png) | ![Conversation](screenshots/conversation.png) | ![Voice pipeline](screenshots/voice-processing.png) |
| Unread threads raised onto cards, each showing its language pair | Translated message with the original text beneath it | A voice note moving through transcription, translation, and synthesis |

### Calls

| Incoming call | Live translated call |
|:---:|:---:|
| ![Incoming call](screenshots/incoming-call.png) | ![Live call](screenshots/live-call.png) |
| Full-screen ring with expanding ripple and accept/reject actions | Real-time captions translated in both directions |

### Practice & Account

| Practice session | Settings | Subscription |
|:---:|:---:|:---:|
| ![Practice](screenshots/practice-session.png) | ![Settings](screenshots/settings.png) | ![Subscription](screenshots/subscription.png) |
| Speaking exercise with reference audio and recording control | Grouped preferences for translation, notifications, and theme | Plan comparison with store-provided pricing |

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Flutter 3.7 · Dart |
| State / DI / Routing | GetX |
| Backend | Supabase — PostgreSQL, Auth, Storage, Realtime, Edge Functions |
| Authentication | Supabase Auth · Google Sign-In (OAuth ID token) |
| Speech-to-text | OpenAI Whisper (via Edge Function) |
| Machine translation | Google Gemini (via Edge Function) |
| Text-to-speech | ElevenLabs (via Edge Function) |
| Voice & video calls | Agora RTC Engine · WebSocket caption relay |
| Media processing | ffmpeg_kit_flutter_new — audio extraction, subtitle burn-in |
| Push notifications | Firebase Cloud Messaging · flutter_local_notifications |
| Local persistence | SQFlite · SharedPreferences |
| Payments | in_app_purchase — Google Play Billing & StoreKit |

## My Role

I designed and built the entire application — there was no starter codebase beyond `flutter create`.

**Architecture** — GetX for state, routing, and DI, with a strict controller/view split and eleven injectable services resolved at startup. That separation is what later let me re-skin all 18 screens without touching business logic.

**Backend** — PostgreSQL schema on Supabase (users, conversations, messages, reactions, call invitations, call logs, subscriptions) with row-level security on every table, plus a `SECURITY DEFINER` trigger that provisions profiles on signup including OAuth claims. Five Edge Functions keep every third-party API key off the client.

**Translation pipeline** — the multi-stage flow turning a voice note or video into a translated artifact: ffmpeg extracts audio → Whisper transcribes with per-segment timings → Gemini translates → ElevenLabs synthesizes speech, or an SRT is burned back onto the video.

**Live calls** — Agora RTC for transport alongside a WebSocket channel carrying translated captions, with a full invite/ring/accept/reject/missed lifecycle over Supabase Realtime and FCM.

**Design system & monetization** — a token-based theme layer driving the whole UI from one file, an animation library that respects the OS reduce-motion setting, and subscriptions via `in_app_purchase` with store pricing and the restore flow Apple requires.

## Technical Highlights

**Keeping API keys out of the binary.** Whisper, Gemini, and ElevenLabs credentials never reach the client. All three run behind Supabase Edge Functions invoked with an authenticated session, so decompiling the APK yields nothing useful.

**Resumable multi-stage media pipeline.** Translating a video chains five fallible operations across the device and three external services. Each job is a state machine caching every intermediate result — extracted audio, timed segments, translated strings — so a network failure during translation retries from that step instead of re-running ffmpeg extraction. The same job model backs both chat video messages and the Studio.

**Preserving subtitle timing across translation.** Whisper returns timed segments, but translating the transcript as one block destroys the mapping between text and timestamps. Translating segment by segment and reassembling against the original timings keeps subtitles synchronized even when word order differs substantially between languages.

**Fixing a silent authentication failure.** Profile loading swallowed all exceptions, so a failed read left the user ID unpersisted — the user reached the home screen with an empty session and every downstream feature failed with no error shown. The ID now persists unconditionally, falling back to auth metadata when the profile row can't be read.

**RTL-correct visual design.** The reference design was left-to-right and single-theme. Directional geometry — a chat bubble's tail, spacing between elements — uses `BorderRadiusDirectional` and `EdgeInsetsDirectional` so it mirrors automatically in Arabic, and negative letter-spacing is applied only to Latin text since it breaks Arabic letterform joining.

## Status

Feature-complete and building cleanly on Android — `flutter analyze` reports no issues and the test suite passes. Remaining pre-release work:

- Register in-app purchase products in Google Play Console and App Store Connect
- Replace the placeholder `com.example.*` identifiers and add release signing
- Complete Google Sign-In setup (SHA-1 fingerprint, iOS `REVERSED_CLIENT_ID`)
- Add server-side receipt verification before accepting live purchases
- Verify the iOS build on macOS — not yet compiled

## Running Locally

Requires a Supabase project with `supabase_schema.sql` applied and the functions in `supabase/functions/` deployed, plus a Firebase project for push notifications.

```bash
flutter pub get && flutter run
```
