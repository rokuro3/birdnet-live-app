# Adapting BirdNET Live Into Another Flutter App

This note is for a developer who is new to Flutter and wants to turn BirdNET
Live into a different app rather than only making small changes.

## 1. Flutter Basics You Need First

### Flutter in one sentence

Flutter is a UI framework where you build screens by composing widgets in Dart.
The same codebase can target Android, iOS, and desktop platforms.

### The files that matter most at the start

- `pubspec.yaml`:
  package metadata, dependencies, assets, and app version
- `lib/main.dart`:
  startup logic and dependency initialization
- `lib/app.dart`:
  root `MaterialApp`, theme, localization, and first screen selection
- `lib/`:
  almost all application logic and UI

### How Flutter apps are usually structured

- `main()` boots the app
- `runApp()` mounts the root widget
- Widgets describe the UI
- State drives what the UI shows
- Rebuilding widgets is normal and expected

In this repository, `main()` initializes platform services first, then starts a
Riverpod `ProviderScope`, and finally loads `App`.

### Widget basics

- **StatelessWidget**: UI from fixed input
- **StatefulWidget**: UI with local mutable state
- **ConsumerWidget**: Riverpod-aware widget that reads providers

This app uses `ConsumerWidget` heavily because Riverpod is the main state
management approach.

### State management basics

BirdNET Live uses **Riverpod**.

The common pattern is:

1. Define a provider for a value or service
2. Read it in a widget with `ref.watch(...)`
3. Update it through a notifier or service

That means when you want to change app behavior, you often need to inspect both
the screen widget and the provider behind it.

### Assets and configuration

Flutter apps often declare static assets in `pubspec.yaml`. This project uses
that for images, model files, and species metadata.

Important examples:

- `assets/images/`
- `assets/models/`
- `assets/species_data/`

### Localization basics

User-facing strings should not be hardcoded in widgets. This repository keeps
them in ARB files under:

- `lib/l10n/`

If you change text shown in the app, this is one of the first places to check.

### Build and iteration basics

Common commands:

```bash
flutter pub get
flutter analyze
flutter test
flutter run
```

- **Hot reload** updates code without restarting the whole app
- **Hot restart** restarts Dart state but is still faster than a full rebuild
- **flutter analyze** is the basic static check you should run often

## 2. What This Repository Is

BirdNET Live is a Flutter app for real-time bird species identification using
on-device ONNX inference. The app listens to microphone audio, runs a model
locally, and shows detections together with a live spectrogram and survey tools.

This is not a small demo app. It already contains:

- real-time audio capture
- on-device model inference
- GPS-based workflows
- session persistence and export
- localization
- multiple operating modes

That makes it a strong base if you want to build a different field app, but it
also means there is more to understand than in a typical beginner Flutter app.

## 3. High-Level Repository Map

### Core project files

- `pubspec.yaml`:
  dependencies, assets, version
- `analysis_options.yaml`:
  analyzer and lint rules
- `mkdocs.yml`:
  documentation site configuration
- `README.md`:
  project overview and common commands

### Application code

- `lib/core/`:
  app-wide constants, theme, and foundational utilities
- `lib/shared/`:
  shared models, services, providers, and reusable widgets
- `lib/features/`:
  feature-oriented modules

### Feature modules

- `lib/features/live/`:
  live identification mode
- `lib/features/point_count/`:
  timed survey sessions
- `lib/features/survey/`:
  long-running GPS survey workflow
- `lib/features/file_analysis/`:
  offline audio file analysis
- `lib/features/explore/`:
  location-based species exploration
- `lib/features/inference/`:
  ONNX model loading and prediction
- `lib/features/audio/`:
  audio capture and buffering
- `lib/features/history/`:
  saved sessions, review, and export
- `lib/features/settings/`:
  settings UI and settings-related logic
- `lib/features/home/`:
  main menu and entry screens
- `lib/features/onboarding/`:
  initial onboarding and terms flow
- `lib/features/about/`:
  credits, links, and legal information

### Platform folders

- `android/`
- `ios/`

You usually touch these when changing package identifiers, app names, signing,
permissions, or platform-specific plugins.

### Tests and docs

- `test/`: unit tests
- `integration_test/`:
  integration tests
- `docs/`: user and
  developer documentation

## 4. How the App Starts

The startup path is short and worth understanding early:

1. `lib/main.dart`
2. `lib/app.dart`
3. onboarding gate or home screen

At startup, the app initializes:

- foreground task communication
- survey notifications
- system UI behavior
- `SharedPreferences`
- Riverpod providers

Then `App` builds `MaterialApp`, sets the theme and localization, and chooses
either onboarding or the home screen.

## 5. The Main Architectural Idea

This repository uses **feature-based organization** instead of putting all
screens in one folder and all services in another.

That is useful when adapting the app because you can think in larger chunks:

- remove a feature
- rename a feature
- replace the internals of a feature
- keep shared infrastructure while swapping app-specific workflows

If you want to build a different app from this codebase, this feature-based
layout is one of the biggest strengths of the project.

## 6. What Is Specific to BirdNET Live

If your new app is not about bird audio identification, these parts are the
most BirdNET-specific:

- `assets/models/`
- `assets/species_data/`
- `lib/features/inference/`
- `lib/features/audio/`
- `lib/features/explore/`
- `lib/features/live/`
- `lib/features/point_count/`
- `lib/features/survey/`

If your new app still uses live sensor input, offline inference, or field
sessions, large parts of the architecture may still be reusable.

## 7. What Is Generic and Reusable

These parts are easier to keep even if the app idea changes:

- app bootstrapping with Flutter and Riverpod
- theming and localization setup
- settings persistence through `SharedPreferences`
- feature-based folder structure
- session/history concepts
- export workflows
- documentation setup with MkDocs

## 8. Good First Changes When Turning This Into Another App

If you want to fork this into a different product, the safest first changes are:

1. change branding and app identity
2. decide which existing features stay and which are removed
3. replace the home screen so the app structure reflects the new goal
4. rename or remove BirdNET-specific models and content
5. update settings and localization keys to match the new domain

Important places for app identity changes:

- `pubspec.yaml`
- `android/app/build.gradle`
- `android/app/src/main/AndroidManifest.xml`
- `ios/Runner/Info.plist`

## 9. A Practical Way To Learn This Codebase

If you are new to Flutter, do not try to understand everything at once.

A good order is:

1. read `README.md`
2. read `lib/main.dart`
3. read `lib/app.dart`
4. inspect `lib/features/home/`
5. inspect one feature you care about most
6. only then move into providers, services, and platform folders

That keeps the learning path manageable.

## 10. Summary

BirdNET Live is definitely a Flutter app, and it is organized in a fairly clean
way for reuse:

- Flutter handles the cross-platform UI layer
- Riverpod handles state and dependency wiring
- `lib/features/` holds the business features
- `lib/shared/` and `lib/core/` hold reusable infrastructure
- BirdNET-specific model and domain logic are concentrated in a few areas

So if your goal is to turn this into another app, the repository is a workable
starting point. The easiest strategy is to preserve the app shell and shared
infrastructure while gradually replacing BirdNET-specific features with your own
domain logic.
