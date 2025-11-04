# Repository Guidelines

## Project Structure & Module Organization
- Root planning docs: `PROJECT_ARCHITECTURE.md`, `TODO.md`.
- watchOS + iOS sources live in `app/Pickleball Watch Scorer/Pickleball Watch Scorer`. Key folders: `Domain` (SwiftData models, scoring logic), `ViewModels` (SwiftUI `@Observable` state), `Views` (watch-first SwiftUI screens), `Assets.xcassets` (icons, colors).
- Tests: `Pickleball Watch ScorerTests` for unit coverage, `Pickleball Watch ScorerUITests` for interface flows. Add new suites alongside the existing targets.

## Build, Test, and Development Commands
- `xcodebuild -scheme "Pickleball Watch Scorer" -destination 'generic/platform=watchOS' build` — compile the watch app locally.
- `xcodebuild test -scheme "Pickleball Watch Scorer" -destination 'platform=watchOS Simulator,name=Apple Watch Series 9 (45mm)'` — run unit and UI tests.
- `xed app/Pickleball\ Watch\ Scorer/Pickleball\ Watch\ Scorer.xcodeproj` — open the project in Xcode.

## Coding Style & Naming Conventions
- Swift 6, SwiftUI-first: use `@Observable`, `@Environment`, `Observation` over legacy `ObservableObject`.
- Four-space indentation, trailing commas where SwiftFormat would insert them; keep files under 300 lines when practical.
- Types use UpperCamelCase, properties and functions lowerCamelCase, enums with singular cases. Match existing naming (`MatchPlayViewModel`, `ScoringEngine.GameState`).
- Co-locate preview providers with their view in the same file; prefer small composable extensions in `Domain/Support`.

## Testing Guidelines
- Write tests with XCTest in the `Pickleball Watch ScorerTests` target; name files `<Feature>Tests.swift` and methods `test_<behavior>()`.
- Cover scoring edge cases (freeze logic, server rotation) and watch-specific regressions. Add baseline snapshots to UI tests when layout changes.
- Ensure deterministic tests by seeding model data; avoid real dates unless injecting via `@MainActor` test harness helpers.

## Commit & Pull Request Guidelines
- Commit messages follow `<scope>: <concise summary>` (e.g., `scoring: add freeze-at-20 branch`). Group related changes to keep diffs reviewable.
- PRs must include: summary of changes, testing notes (`xcodebuild test …` output), affected tickets, and watch screenshots if UI shifts.
- Request reviews from domain owners (scoring, stacking, activity) and run swift-format/swiftlint locally before marking ready.

## Architecture & Platform Notes
- Aim for a watch-first flow; watch app is the source of truth, with iOS acting as a companion via ActivityKit and WatchConnectivity.
- Persist through SwiftData models annotated with `@Model`; avoid Core Data APIs unless bridging legacy data.
- Keep new libraries Swift Package Manager-compatible; prefer dependency injection via environment values.
