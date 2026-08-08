# AniVerse — عالم الأنمي 🎌

A gamified Android streaming app for anime and drama series, where watching content earns you points, levels, and achievements.

## Overview

AniVerse is a native Android app for discovering and watching anime and drama series, built around a short-form vertical video experience (think TikTok-style reels) alongside full episode playback. It targets Arabic-speaking viewers who want a lightweight, mobile-first way to binge series — but with a twist: instead of a flat paywall, access to episodes is gated by an in-app points economy. Users earn points by watching content, completing daily tasks, and watching rewarded ads, or they can subscribe for unlimited access. It's designed to keep casual viewers engaged through progression (XP, levels, achievements) while giving the app sustainable monetization through ads and subscriptions.

## My Role

I designed and built the entire application solo, end to end:

- **App architecture** — organized the codebase into clear layers (activities, fragments, managers, database, ads, gamification, backend clients) and designed the navigation flow across Home, Reels, Library, Store, and Profile.
- **Dual-backend integration** — content catalog (series/episodes) served from **Supabase** via a Retrofit REST layer, while user identity, points, and cross-device sync run through **Firebase Firestore** — kept intentionally separate so content updates never touch user data.
- **Gamification engine** — designed the XP/leveling system, points economy (points-per-episode, points-per-reel), daily tasks, and unlockable achievements from scratch (`GameEngine`, `UserGameStats`, `AchievementDef`, `TaskDef`).
- **Monetization stack** — integrated the full Google AdMob suite (App Open, Interstitial, Rewarded, Banner) and Google Play Billing subscriptions, and later re-architected the rewarded-ad flow after diagnosing a production bug that was blocking the app's core reward loop (see Technical Highlights).
- **Local + remote data layer** — Room database for offline watch history and continue-watching, synced against the remote catalog and user profile.
- **Release engineering** — set up R8/ProGuard shrinking, resolved Play Console native-symbol warnings, and hardened the build for Play Store submission.

## Key Features

- 🏠 **Home feed** — browse curated anime & drama series by category
- 📱 **Reels player** — TikTok-style vertical swipe feed for short-form episode clips
- 🎬 **Full series player** — dedicated episode-by-episode player with continue-watching support
- ⭐ **Library** — favorites and continue-watching history, persisted locally with Room
- 🎮 **Gamification** — XP, levels, and unlockable achievements for watching content
- ✅ **Daily tasks** — recurring point-earning objectives to drive daily retention
- 🎁 **Rewarded ads** — watch an ad to earn points instantly, with a resilient always-ready ad cache
- 💳 **Subscriptions & point packs** — weekly/monthly/VIP plans and one-time point purchases via Google Play Billing
- 👤 **Profile** — personal stats, level progress, and point balance
- 🔔 **Daily notifications** — scheduled re-engagement reminders via WorkManager
- 🌐 **Full Arabic RTL UI**

## Tech Stack

![Java](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![Android](https://img.shields.io/badge/Android-3DDC84?style=for-the-badge&logo=android&logoColor=white)
![Firebase](https://img.shields.io/badge/Firebase-FFCA28?style=for-the-badge&logo=firebase&logoColor=black)
![Supabase](https://img.shields.io/badge/Supabase-3FCF8E?style=for-the-badge&logo=supabase&logoColor=white)
![Retrofit](https://img.shields.io/badge/Retrofit-48B983?style=for-the-badge&logo=square&logoColor=white)
![Room](https://img.shields.io/badge/Room%20Database-4285F4?style=for-the-badge&logo=sqlite&logoColor=white)
![ExoPlayer](https://img.shields.io/badge/ExoPlayer-FF0000?style=for-the-badge&logo=youtubeplayer&logoColor=white)
![Google AdMob](https://img.shields.io/badge/Google%20AdMob-4285F4?style=for-the-badge&logo=googleads&logoColor=white)
![Google Play Billing](https://img.shields.io/badge/Play%20Billing-01875F?style=for-the-badge&logo=googleplay&logoColor=white)
![Glide](https://img.shields.io/badge/Glide-5C2D91?style=for-the-badge&logo=android&logoColor=white)
![Gradle](https://img.shields.io/badge/Gradle-02303A?style=for-the-badge&logo=gradle&logoColor=white)
![R8](https://img.shields.io/badge/R8%2FProGuard-black?style=for-the-badge&logo=android&logoColor=white)

| Layer | Technology |
|---|---|
| Language | Java |
| UI | Android Views, ViewPager2, Material Components, ViewBinding |
| Video playback | ExoPlayer 2 |
| Content backend | Supabase (Postgres + REST via Retrofit/OkHttp/Gson) |
| User data & sync | Firebase Firestore, Firebase Analytics |
| Local storage | Room (watch history, continue-watching) |
| Ads | Google AdMob (App Open, Interstitial, Rewarded, Banner) |
| Payments | Google Play Billing Library 9.x |
| Background work | WorkManager (daily notifications) |
| Images | Glide |
| Build & release | Gradle, R8/ProGuard, App Bundle (AAB) |

## Screenshots

> 📸 Drop your screenshot files into a `screenshots/` folder at the repo root using the filenames below, and they'll render automatically.

### Splash Screen
![Splash Screen](screenshots/splash.png)
_App launch screen shown while initial data and ad SDKs are loading._

### Home Feed
![Home Feed](screenshots/home.png)
_Main discovery screen with curated anime & drama series, categories, and a continue-watching banner._

### Reels Player
![Reels Player](screenshots/reels.png)
_Vertical swipe feed for short-form episode clips, styled after TikTok/Reels-style browsing._

### Series Details
![Series Details](screenshots/series_details.png)
_Series overview page with synopsis, episode list, and entry point into the full player._

### Library
![Library](screenshots/library.png)
_Favorites and continue-watching history, synced from local watch history._

### Store — Point Packs & Subscriptions
![Store](screenshots/store.png)
_In-app store with point packs, weekly/monthly/VIP subscription plans, achievements, and daily tasks._

## Technical Highlights & Challenges Solved

- **Root-caused and fixed a production-blocking rewarded-ads bug.** Users were hitting "ad not ready" errors on the app's core reward loop. Traced it to five compounding issues — per-screen ad caches with no shared state, a hard cap on retry attempts, a loading UI that unlocked the watch button before the ad actually finished loading, no handling of AdMob's ~1-hour ad expiry, and a callback registered after `show()` was called. Redesigned it as a single app-wide `RewardedAdManager` singleton that preloads from app startup, retries indefinitely with backoff while foregrounded, tracks ad expiry, and replaces the error toast with a "preparing ad" wait state that auto-shows the ad the moment it's ready.
- **Migrated Google Play Billing from v7 to v9.1.0** across a breaking API surface (`PendingPurchasesParams` builder replacing the no-arg call, `QueryProductDetailsResult` wrapper replacing the raw list). Verified the real method signatures directly against the shipped `.aar` via `javap` rather than relying on documentation that lagged the release.
- **Enabled R8/ProGuard shrinking for release** on a codebase with a large reflection surface — Gson-deserialized models, a Retrofit dynamic-proxy interface, Room entities/DAOs, and `Serializable` payloads passed between screens — without breaking any of it. Wrote targeted `-keep` rules instead of blanket keeps for third-party libraries (which already ship their own consumer ProGuard rules), then verified correctness by diffing `mapping.txt`: sensitive classes stayed unobfuscated while internal implementation classes were renamed and stripped, cutting the shipped `.dex` size measurably.
- **Diagnosed a Play Console native-symbols warning** down to its actual cause instead of guessing: the only native library in the bundle (pulled in transitively via Firebase/AndroidX DataStore) ships pre-stripped by Google with no debug sections at all — confirmed by inspecting its ELF sections directly, which made clear the warning wasn't fixable from the app side.
- **Built a self-contained gamification engine** (XP curve, leveling, achievements, daily tasks) that gates premium content behind an in-app points economy, giving the ad and subscription monetization a clear, motivating reason to exist rather than acting as a blunt paywall.

## Status

**✅ Published on Google Play**
