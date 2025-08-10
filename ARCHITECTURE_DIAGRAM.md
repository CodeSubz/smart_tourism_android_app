# 🏛️ Smart Tourism App - Architecture Overview

## System Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           CONTENT CREATION PHASE                           │
└─────────────────────────────────────────────────────────────────────────────┘
                                       │
┌──────────────────────┐    ┌──────────────────────┐    ┌──────────────────────┐
│   Monument Images    │    │    Tourist Guides    │    │   Category Images    │
│  (JPEG/PNG files)    │    │   (Markdown files)   │    │   (Classification)   │
│                      │    │                      │    │                      │
│ Python/imageDatasets │    │  Python/guides/      │    │  Python/categories/  │
│    /CITYNAME/        │    │    CITYNAME/         │    │    CITYNAME/         │
└──────────┬───────────┘    └──────────┬───────────┘    └──────────┬───────────┘
           │                           │                           │
           └─────────────────┬─────────────────┬─────────────────────┘
                             │                 │
                    ┌────────▼─────────┐      │
                    │  Python Scripts  │      │
                    │                  │      │
                    │ • build_sqlite.py│      │
                    │ • processing.py  │      │
                    │ • TensorFlow     │      │
                    └────────┬─────────┘      │
                             │                │
                    ┌────────▼─────────┐      │
                    │  SQLite Database │      │
                    │                  │      │
                    │ • Features       │      │
                    │ • Metadata       │      │
                    │ • Guides Content │      │
                    └────────┬─────────┘      │
                             │                │
                             └────────┬───────┘
                                      │
┌─────────────────────────────────────▼───────────────────────────────────────┐
│                              RUNTIME PHASE                                 │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                           ANDROID APPLICATION                              │
│                                                                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐             │
│  │ LoadingActivity │  │  MainActivity   │  │ CameraActivity  │             │
│  │                 │  │                 │  │                 │             │
│  │ • Permission    │  │ • Monument List │  │ • TensorFlow    │             │
│  │ • Database Load │  │ • Categories    │  │ • Recognition   │             │
│  │ • Preferences   │  │ • Preferences   │  │ • Live Preview  │             │
│  └─────────┬───────┘  └─────────┬───────┘  └─────────┬───────┘             │
│            │                    │                    │                     │
│            └──────────┬─────────┴──────────┬─────────┘                     │
│                       │                    │                               │
│            ┌──────────▼──────────┐  ┌──────▼──────────┐                    │
│            │   GuideActivity     │  │ PreferencesActivity │                │
│            │                     │  │                    │                │
│            │ • Markdown Viewer   │  │ • Categories Setup │                │
│            │ • Recommendations   │  │ • Attributes Config│                │
│            │ • Map Integration   │  │ • Language Settings│                │
│            └─────────────────────┘  └────────────────────┘                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                              DATA LAYER                                    │
│                                                                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐             │
│  │ SQLite Database │  │    Location     │  │  Preferences    │             │
│  │                 │  │                 │  │                 │             │
│  │ • Monument Data │  │ • GPS Tracking  │  │ • User Settings │             │
│  │ • Guide Content │  │ • Proximity     │  │ • Categories    │             │
│  │ • AI Features   │  │ • Notifications │  │ • Language      │             │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## User Journey Flow

```
👤 User Opens App
       │
       ▼
🔄 LoadingActivity
   │ • Check permissions
   │ • Load database
   │ • First-time setup
       │
       ▼
🏠 MainActivity
   │ • Browse monuments
   │ • View by categories
   │ • Access preferences
       │
       ├─────────────────────┐
       ▼                     ▼
📷 Camera Recognition    ⚙️ Preferences
   │ • Point at monument   │ • Set categories
   │ • AI identification   │ • Choose attributes
   │ • Instant results     │ • Language settings
       │                     │
       ▼                     │
📖 Guide Viewer ◄────────────┘
   │ • Read monument info
   │ • View recommendations
   │ • Open maps
   │ • Share content
       │
       ▼
🗺️ Navigation
   • Maps integration
   • Walking directions
   • Distance tracking
```

## Technology Stack

```
┌─────────────────────────────────────────────────────────────────┐
│                        FRONTEND (Android)                      │
│                                                                 │
│  UI Framework: Android SDK + Material Design                   │
│  Languages: Java + Kotlin                                      │
│  Architecture: MVVM Pattern                                    │
│  Navigation: Android Navigation Component                      │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│                        AI/ML LAYER                             │
│                                                                 │
│  Framework: TensorFlow Lite                                    │
│  Computer Vision: OpenCV                                       │
│  Model Type: Image Classification                              │
│  Inference: On-device (offline)                               │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│                        DATA LAYER                              │
│                                                                 │
│  Database: SQLite (local storage)                             │
│  Content Format: Markdown                                      │
│  Location: Android Location Services                          │
│  Preferences: SharedPreferences                               │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│                        BACKEND (Python)                        │
│                                                                 │
│  Processing: Computer Vision Pipeline                          │
│  ML Training: TensorFlow                                       │
│  Database Generation: SQLite creation                          │
│  Content Management: Markdown processing                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Key Components Interaction

```
┌─────────────┐    recognizes    ┌─────────────┐    queries    ┌─────────────┐
│   Camera    │ ──────────────► │  AI Model   │ ────────────► │  Database   │
│             │                 │ (TensorFlow │               │  (SQLite)   │
└─────────────┘                 │    Lite)    │               └─────────────┘
                                └─────────────┘                       │
                                                                      │ fetches
                                                                      ▼
┌─────────────┐    displays     ┌─────────────┐    loads     ┌─────────────┐
│    User     │ ◄────────────── │ Guide View  │ ◄─────────── │ Guide Data  │
│ Interface   │                 │             │               │ (Markdown)  │
└─────────────┘                 └─────────────┘               └─────────────┘
       │                                │
       │ user input                     │ recommendations
       ▼                                ▼
┌─────────────┐    influences   ┌─────────────┐
│Preferences  │ ──────────────► │Recommendation│
│  Engine     │                 │   Algorithm  │
└─────────────┘                 └─────────────┘
```

This architecture provides:
- **Offline-first experience**: All data stored locally
- **Real-time AI recognition**: TensorFlow Lite on-device inference
- **Scalable content management**: Easy to add new cities and monuments
- **Personalized experience**: User preferences drive recommendations
- **Cross-platform potential**: Python backend can serve multiple frontends