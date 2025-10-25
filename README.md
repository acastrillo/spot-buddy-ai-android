# Spot Buddy - Native Android App

**AI-Powered Fitness Tracking with Instagram Share Sheet Integration**

[![Android](https://img.shields.io/badge/Android-8.0%2B-green.svg)](https://developer.android.com)
[![Kotlin](https://img.shields.io/badge/Kotlin-2.1.0-blue.svg)](https://kotlinlang.org)
[![Jetpack Compose](https://img.shields.io/badge/Jetpack%20Compose-1.7.0-orange.svg)](https://developer.android.com/jetpack/compose)

## Overview

Spot Buddy is a native Android fitness tracking application that lets users save Instagram workouts with a simple 2-tap workflow. Share a workout screenshot from Instagram, and Spot Buddy automatically extracts exercises using OCR and AI enhancement.

**Key Features**:
- 📸 **Instagram Share Sheet Integration** - 2-tap workflow to save workouts
- 🤖 **AI-Powered Parsing** - Amazon Bedrock (Claude Sonnet 4.5) for smart workout extraction
- 💪 **Exercise Tracking** - Track PRs, body metrics, and progress over time
- 📅 **Calendar & Scheduling** - Plan workouts and mark completions
- ⏱️ **Smart Timers** - Interval, HIIT, and rest timers
- 🔄 **Cross-Platform Sync** - Shared AWS backend with Web app
- 📊 **Advanced Stats** - PR tracking, 1RM calculations, volume analysis
- 🌙 **Dark Theme** - Fitness-focused UI with custom color palette

## Architecture

This is a **native Kotlin Android app** that shares the same AWS backend infrastructure with the [Spot Buddy Web App](https://github.com/acastrillo/spot-buddy-web).

```
┌─────────────────────────────────────────┐
│       Native Android App (Kotlin)      │
│     Jetpack Compose + Material 3       │
└─────────────────┬───────────────────────┘
                  │
                  │ REST API (Retrofit)
                  ↓
┌─────────────────────────────────────────┐
│       Shared AWS Backend               │
│  • DynamoDB (workouts, users)          │
│  • Cognito (authentication)            │
│  • Textract (OCR)                      │
│  • Bedrock (AI features)               │
│  • Stripe (subscriptions)              │
└─────────────────────────────────────────┘
                  ↑
                  │ Same APIs
                  │
┌─────────────────┴───────────────────────┐
│      Next.js Web App (Existing)        │
│  https://spotter.cannashieldct.com     │
└─────────────────────────────────────────┘
```

## Tech Stack

- **Language**: Kotlin 2.1.0
- **UI Framework**: Jetpack Compose (declarative UI)
- **Architecture**: MVVM with Clean Architecture
- **Navigation**: Jetpack Navigation Compose
- **Networking**: Retrofit + OkHttp
- **Local Database**: Room (offline-first)
- **Dependency Injection**: Hilt
- **Image Loading**: Coil
- **Authentication**: AWS Cognito + Google Sign-In
- **OCR**: AWS Textract
- **AI**: Amazon Bedrock (Claude Sonnet 4.5)
- **Payments**: Stripe Android SDK
- **Background Sync**: WorkManager

## Project Status

**Current Phase**: 🚧 **Project Setup**

This repository is being set up to begin native Android development. See [DEPLOYMENT-PLAN.md](./docs/DEPLOYMENT-PLAN.md) for the complete development roadmap.

### Development Timeline (90 Days)

| Phase | Duration | Status |
|-------|----------|--------|
| **Phase 1**: Project Setup | 1 week | 🚧 In Progress |
| **Phase 2**: Authentication | 2 weeks | ⏳ Pending |
| **Phase 3**: Instagram Share | 2 weeks | ⏳ Pending |
| **Phase 4**: Workout Features | 4 weeks | ⏳ Pending |
| **Phase 5**: AI Features | 3 weeks | ⏳ Pending |
| **Phase 6**: Polish & Testing | 2 weeks | ⏳ Pending |
| **Phase 7**: Google Play Launch | 1 week | ⏳ Pending |

## Getting Started

### Prerequisites

- **Android Studio**: Ladybug | 2024.2.1 or later
- **JDK**: 17 or later
- **Android SDK**: API 26+ (Android 8.0+)
- **Kotlin**: 2.1.0+

### Setup Instructions

1. **Clone the repository**
   ```bash
   git clone https://github.com/acastrillo/spot-buddy-ai-android.git
   cd spot-buddy-ai-android
   ```

2. **Open in Android Studio**
   - Open Android Studio
   - Click "Open an Existing Project"
   - Select the `spot-buddy-ai-android` directory

3. **Configure environment**
   - Copy `.env.example` to `.env`
   - Add your AWS credentials and API keys

4. **Build and Run**
   - Wait for Gradle sync to complete
   - Connect an Android device or start an emulator
   - Click the green **Run** button (Shift+F10)

## Key Features

### Instagram Share Sheet Integration

The killer feature of the Android app is native Instagram integration:

1. **User sees workout on Instagram** → Screenshot
2. **User clicks Share button** → Select "Spot Buddy"
3. **App automatically**:
   - Extracts text using AWS Textract OCR
   - Parses exercises with AI (Bedrock Claude Sonnet 4.5)
   - Saves to DynamoDB with cross-device sync
   - Shows editable workout in app

**Implementation**: See [docs/INSTAGRAM-INTEGRATION.md](./docs/INSTAGRAM-INTEGRATION.md)

### Offline-First Architecture

- **Room Database**: Local SQLite database for offline access
- **Background Sync**: WorkManager syncs with DynamoDB when online
- **Conflict Resolution**: Last-write-wins with timestamp-based merging

### Subscription Tiers

- **Free**: 15 workouts max, 1 OCR/week, 30-day history
- **Starter** ($7.99/mo): Unlimited workouts, 5 OCR/week, 10 AI enhancements/month
- **Pro** ($14.99/mo): Unlimited OCR, 30 AI enhancements/month, 30 AI generations/month
- **Elite** ($34.99/mo): 100 AI enhancements/month, 100 AI generations/month, AI Coach

## Documentation

- [**DEPLOYMENT-PLAN.md**](./docs/DEPLOYMENT-PLAN.md) - Complete Android deployment guide (copied from web repo)
- [**PROJECT-STRUCTURE.md**](./docs/PROJECT-STRUCTURE.md) - Android project structure and organization
- [**INSTAGRAM-INTEGRATION.md**](./docs/INSTAGRAM-INTEGRATION.md) - Instagram share intent implementation
- [**API-REFERENCE.md**](./docs/API-REFERENCE.md) - Shared backend API documentation
- [**ARCHITECTURE.md**](./docs/ARCHITECTURE.md) - System architecture and design decisions

## Related Repositories

- **Web App**: [spot-buddy-web](https://github.com/acastrillo/spot-buddy-web) - Next.js web application
- **iOS App**: Coming soon (native Swift + SwiftUI)

## Contributing

This is a personal project, but feedback and suggestions are welcome! Please open an issue to discuss any changes you'd like to see.

## License

MIT License - See [LICENSE](./LICENSE) for details

## Contact

- **Developer**: acastrillo
- **Web App**: https://spotter.cannashieldct.com
- **GitHub**: https://github.com/acastrillo

---

**Built with ❤️ for the fitness community**
