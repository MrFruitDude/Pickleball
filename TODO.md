# Pickleball Watch Scorer - Development TODO

**Last Updated:** November 1, 2025
**Project:** Pickleball Watch Scorer + Stacking Coach
**Status:** Planning & Architecture Complete

---

## Priority 0: Project Setup

- [ ] **Create Xcode project structure**
  - [ ] Create new watchOS App with iOS companion
  - [ ] Configure deployment targets (watchOS 10+, iOS 17+)
  - [ ] Set up proper bundle identifiers
  - [ ] Enable Swift 6 strict concurrency checking
  - [ ] Configure build schemes (Debug/Release)

- [ ] **Set up project architecture**
  - [x] Create folder structure: Domain / ViewModels / Views / Infrastructure
  - [x] Add SwiftData framework
  - [ ] Add ActivityKit framework (iOS)
  - [ ] Add WatchConnectivity framework
  - [x] Set up unit testing target with Swift Testing framework

- [ ] **Configure version control**
  - [ ] Initialize git repository
  - [ ] Create .gitignore for Xcode projects
  - [ ] Create initial commit with project structure

---

## EPIC A: Domain & Storage

### A1: Define Swift Models

- [x] **Create Player model**
  - [x] Define Player struct/class with SwiftData macros
  - [x] Add properties: id, displayName, hand (enum), isStartingServer
  - [x] Implement Codable, Hashable conformance
  - [x] Add validation logic for displayName

- [x] **Create Team model**
  - [x] Define Team struct/class
  - [x] Add properties: id, name, color (ColorHex), players array
  - [x] Implement relationship to Player
  - [x] Add validation (2 players for doubles, 1 for singles)

- [x] **Create MatchSettings model**
  - [x] Define MatchSettings struct
  - [x] Create Format enum (singles/doubles)
  - [x] Create ScoringMode enum with associated values
    - [x] sideOut(target: Int, winBy: Int)
    - [x] rally(target: Int, winBy: Int, freeze: Freeze)
  - [x] Create Freeze enum (gamePointOnly, at20)
  - [x] Create PreferredFormation enum
  - [x] Add computed property for changeEndsAt based on target

- [x] **Create Game model**
  - [x] Define Game class with SwiftData
  - [x] Add all required properties (id, matchId, scores, serving state)
  - [x] Implement relationship to RallyEvent
  - [x] Add computed properties (isComplete, duration, winner)

- [x] **Create RallyEvent model**
  - [x] Define RallyEvent struct
  - [x] Add properties: timestamp, winner, sideOut, notes
  - [x] Implement Codable for persistence

### A2: Implement ScoringEngine

- [x] **Create ScoringEngine class**
  - [x] Define GameState struct to hold current state
  - [x] Create TeamID enum (A, B)
  - [x] Implement initialization with MatchSettings

- [x] **Implement side-out scoring logic**
  - [x] `pointWon(by: TeamID)` method
  - [x] `advanceServerOrSideOut()` helper
  - [x] `alternateServiceCourt()` helper
  - [x] Even/odd server positioning logic
  - [x] Server number tracking (1→2→side-out)
  - [x] Implement 0-0-2 starting rule

- [x] **Implement rally scoring logic**
  - [x] Point-on-every-rally logic
  - [x] Game-point freeze logic (must be on serve)
  - [x] Optional freeze-at-20 variant
  - [x] `isGamePoint(for: TeamID)` helper

- [x] **Implement shared game logic**
  - [x] `undo()` functionality (revert last rally)
  - [x] `reachedMidpoint()` checker
  - [x] `winBySatisfied()` checker
  - [x] End-change prompt triggering
  - [x] Game completion detection

- [ ] **Write unit tests for ScoringEngine**
  - [x] Test complete 0-0-2 → side-out cycle
  - [ ] Test even/odd positioning for all scores (0-21)
  - [x] Test server number advancement
  - [x] Test rally scoring game-point freeze
  - [x] Test freeze-at-20 variant
  - [ ] Test undo for various game states
  - [x] Test win-by-2 logic
  - [x] Test midpoint detection (6/8/11)
  - [ ] Edge cases (11-11, 15-15, 21-21 with win-by-2)

### A3: Implement StackingCoach

- [ ] **Create StackingCoach class**
  - [ ] Define Position struct (court coordinates)
  - [ ] Define PositionRecommendation struct
  - [ ] Initialize with preferred formation settings

- [ ] **Implement legality checker**
  - [ ] `correctServer(for: GameState)` → Player
  - [ ] `correctReceiver(for: GameState)` → Player
  - [ ] `legalServerPosition(for: GameState)` → Position
  - [ ] `legalReceiverPosition(for: GameState)` → Position

- [ ] **Implement formation solver**
  - [ ] `calculateStartingPositions(for: GameState)` → [Player: Position]
  - [ ] `calculatePostServeMovements(for: GameState)` → [Player: Position]
  - [ ] Handle keepFHMiddle formation
  - [ ] Handle keepBHMiddle formation
  - [ ] Handle custom formations

- [ ] **Create diagram coordinates**
  - [ ] Define court coordinate system for watch display
  - [ ] `generateCourtDiagram()` → overlay data structure
  - [ ] Arrow path calculations for movements

- [ ] **Write unit tests for StackingCoach**
  - [ ] Test correct server identification for all scores
  - [ ] Test legal positioning never violated
  - [ ] Test formation preferences honored
  - [ ] Test movement arrows calculated correctly

### A4: SwiftData Schema & Migrations

- [ ] **Set up SwiftData container**
  - [ ] Create ModelContainer configuration
  - [ ] Define schema version
  - [ ] Configure for watch app

- [ ] **Implement persistence layer**
  - [ ] Create MatchRepository protocol
  - [ ] Implement SwiftDataMatchRepository
  - [ ] CRUD operations for Game entities
  - [ ] Query methods (recent games, game by ID)

- [ ] **Plan migration strategy**
  - [ ] Document schema version 1
  - [ ] Create migration plan for future versions
  - [ ] Test data persistence/retrieval

---

## EPIC B: Watch App

### B1: Match Setup Flow

- [ ] **Create MatchSetupView**
  - [ ] Design UI with Digital Crown support
  - [ ] Format picker (Singles/Doubles)
  - [ ] Scoring mode picker (Side-out/Rally)
  - [ ] Target points picker (11/15/21)
  - [ ] Win-by picker (default 2)
  - [ ] Team name inputs
  - [ ] Team color pickers
  - [ ] Stacking toggle
  - [ ] Preferred formation picker (if stacking enabled)

- [ ] **Create MatchSetupViewModel**
  - [ ] Implement @Observable macro
  - [ ] Validation logic for all inputs
  - [ ] State management for form
  - [ ] Save/load preset functionality
  - [ ] Navigation to play screen

- [ ] **Create player selection UI**
  - [ ] Player name inputs
  - [ ] Hand preference picker (L/R/Unknown)
  - [ ] Starting server designation for doubles

- [ ] **Implement form validation**
  - [ ] Require team names
  - [ ] Require 2 players for doubles
  - [ ] Validate target points (11/15/21)
  - [ ] Show error states

### B2: Play Screen

- [ ] **Create MatchPlayView**
  - [ ] Large header with scores (Team A X - Y Team B)
  - [ ] Serving indicator (A or B)
  - [ ] Server number badge (doubles only)
  - [ ] Two primary action buttons (oversized)
    - [ ] "We won rally" button
    - [ ] "They won rally" button
  - [ ] Secondary action toolbar
    - [ ] Undo button
    - [ ] Timeout button
    - [ ] End match button
  - [ ] Stacking overlay toggle chip

- [ ] **Create MatchPlayViewModel**
  - [ ] Integrate ScoringEngine
  - [ ] Handle rally win/loss actions
  - [ ] Implement undo functionality
  - [ ] Manage timeout state
  - [ ] Trigger end-change prompts
  - [ ] Detect game completion
  - [ ] Coordinate with HapticsService
  - [ ] Publish updates to WatchConnectivity

- [ ] **Implement timeout functionality**
  - [ ] Timeout counter view
  - [ ] Resume play action
  - [ ] Track timeout history (optional)

- [ ] **Implement end-change prompt**
  - [ ] Alert/sheet at midpoint (6/8/11)
  - [ ] "Ready to continue" confirmation
  - [ ] Haptic notification

- [ ] **Implement game-over flow**
  - [ ] Winner announcement screen
  - [ ] Final score display
  - [ ] Options: New game / View history / Done

### B3: Positioning Banner & Stacking Overlay

- [ ] **Create PositioningBannerView**
  - [ ] Compact header showing current server
  - [ ] Text: "Alex serves from Right"
  - [ ] Icon indicator for server position

- [ ] **Create StackingOverlayView**
  - [ ] Mini court diagram (simplified)
  - [ ] Player position dots
  - [ ] Legal server/receiver highlighting
  - [ ] Movement arrows (post-serve)
  - [ ] Toggle expand/collapse
  - [ ] Text prompt: "Start here → Move here after serve"

- [ ] **Integrate StackingCoach**
  - [ ] Wire up StackingCoach to ViewModel
  - [ ] Update overlay on each point
  - [ ] Animate position changes
  - [ ] Handle stacking disabled state

### B4: Haptics Service

- [ ] **Create HapticsService**
  - [ ] Define HapticEvent enum
    - [ ] pointWon
    - [ ] sideOut
    - [ ] serverAdvance
    - [ ] timeout
    - [ ] endChange
    - [ ] gameOver
  - [ ] Map events to WKHapticType
  - [ ] Implement `play(_ event: HapticEvent)`
  - [ ] Ensure <100ms latency from score tap

- [ ] **Test haptic patterns**
  - [ ] Point won: .success or .click
  - [ ] Side-out: .notification or custom
  - [ ] Timeout: .directionUp
  - [ ] End game: .notification

### B5: History List & Details

- [ ] **Create HistoryView**
  - [ ] List of recent games (last 10 for free tier)
  - [ ] Show: Date, Team A vs Team B, Final score, Duration
  - [ ] Pull-to-refresh (optional)
  - [ ] Tap to view details

- [ ] **Create GameDetailView**
  - [ ] Full scoreline
  - [ ] Rally-by-rally history (optional)
  - [ ] Match duration
  - [ ] Serving stats (optional for MVP)
  - [ ] Delete game option

- [ ] **Create HistoryViewModel**
  - [ ] Fetch games from MatchRepository
  - [ ] Sort by date (most recent first)
  - [ ] Delete game functionality
  - [ ] Export game (CSV) - Pro feature

- [ ] **Create SettingsView**
  - [ ] Default match settings
  - [ ] Rally freeze mode toggle
  - [ ] Haptic intensity (if customizable)
  - [ ] About / version info
  - [ ] Export data (CSV) - Pro feature

---

## EPIC C: iPhone Companion

### C1: WCSession Bridge

- [ ] **Create WatchConnectivityClient (watchOS)**
  - [ ] Singleton or injected service
  - [ ] Configure WCSession
  - [ ] Implement delegate methods
  - [ ] Send messages: match start, score update, match end
  - [ ] Handle reachability state

- [ ] **Create WatchConnectivityClient (iOS)**
  - [ ] Configure WCSession
  - [ ] Receive messages from watch
  - [ ] Decode match state updates
  - [ ] Handle errors gracefully

- [ ] **Define message protocol**
  - [ ] MatchStarted: teams, settings
  - [ ] ScoreUpdated: scoreA, scoreB, serving, serverNum
  - [ ] MatchEnded: final score, duration
  - [ ] Use Codable structs for message payloads

- [ ] **Test connectivity**
  - [ ] Verify message delivery watch → phone
  - [ ] Test when phone unreachable
  - [ ] Test reconnection after loss

### C2: Live Activity Lifecycle

- [ ] **Create ActivityAttributes model**
  - [ ] Define attributes (static data: team names, colors, target)
  - [ ] Define ContentState (dynamic data: scores, serving, serverNum)
  - [ ] Conform to ActivityAttributes protocol

- [ ] **Create ActivityManager**
  - [ ] Start Live Activity on match start
  - [ ] Update activity on score change
  - [ ] End activity on match completion
  - [ ] Handle errors (activity not available, etc.)

- [ ] **Design Live Activity UI**
  - [ ] Compact view (Dynamic Island/Lock Screen)
    - [ ] Team A score – Team B score
    - [ ] Serving indicator (icon or color)
  - [ ] Expanded view (Lock Screen)
    - [ ] Larger scores
    - [ ] Server number (doubles)
    - [ ] Progress indicator (X of 11/15/21)
  - [ ] Use Live Activity best practices (frequent updates OK)

- [ ] **Test Live Activity**
  - [ ] Verify appears on lock screen within 1s of match start
  - [ ] Verify updates on each score change
  - [ ] Verify dismisses on match end
  - [ ] Test on physical device (required for Live Activities)

### C3: Spectator Screen UI

- [ ] **Create SpectatorView**
  - [ ] Full-screen scoreboard
  - [ ] Large team names + scores
  - [ ] Serving indicator
  - [ ] Server number (doubles)
  - [ ] Game progress (X/11, X/15, X/21)
  - [ ] Optimized for landscape orientation
  - [ ] High contrast for outdoor visibility
  - [ ] Support AirPlay/external display

- [ ] **Create SpectatorViewModel**
  - [ ] Subscribe to WatchConnectivityClient
  - [ ] Update UI in real-time
  - [ ] Handle connection loss gracefully
  - [ ] Show "Waiting for watch..." state

- [ ] **Implement tap-to-open from Live Activity**
  - [ ] Handle deeplink/URL from Live Activity tap
  - [ ] Navigate to SpectatorView

### C4: CSV Export

- [ ] **Create CSVExporter service**
  - [ ] Export single game to CSV
  - [ ] Export all games to CSV
  - [ ] Format: Date, TeamA, TeamB, ScoreA, ScoreB, Duration, Format, Scoring
  - [ ] Share sheet integration

- [ ] **Add export UI**
  - [ ] Export button in HistoryView (iOS)
  - [ ] Export all / export selected
  - [ ] Share to Files, AirDrop, etc.

- [ ] **Lock behind Pro tier** (monetization)
  - [ ] Check Pro status before export
  - [ ] Show paywall if free tier

---

## EPIC D: QA & Accessibility

### D1: Unit Tests

- [ ] **ScoringEngine tests** (expand from A2)
  - [ ] All scoring paths (side-out, rally, freeze variants)
  - [ ] Undo at every game state
  - [ ] Edge cases: 11-11, 21-21, overtime
  - [ ] Server rotation: full cycle (A1→A2→B1→B2→A1)
  - [ ] Even/odd positioning accuracy

- [ ] **StackingCoach tests**
  - [ ] Legality always maintained
  - [ ] Formation preferences honored
  - [ ] Movement arrows calculated correctly

- [ ] **MatchRepository tests**
  - [ ] CRUD operations
  - [ ] Query recent games
  - [ ] Delete game

- [ ] **WatchConnectivityClient tests**
  - [ ] Message encoding/decoding
  - [ ] Mock WCSession for testing

### D2: UI Tests

- [ ] **Match setup flow**
  - [ ] Complete form and start match
  - [ ] Validation errors shown

- [ ] **Play screen**
  - [ ] Tap "We won" increments correct score
  - [ ] Undo reverts last point
  - [ ] Side-out triggers correctly
  - [ ] End-change prompt at midpoint
  - [ ] Game-over flow

- [ ] **Stacking overlay**
  - [ ] Toggle shows/hides overlay
  - [ ] Positions update on point change
  - [ ] Legal positions always shown

- [ ] **History**
  - [ ] Games appear in list
  - [ ] Tap opens detail
  - [ ] Delete removes game

### D3: VoiceOver & Accessibility

- [ ] **Add accessibility labels**
  - [ ] All buttons with clear labels
  - [ ] Score announcements (VoiceOver)
  - [ ] Serving/server number announced

- [ ] **Support Dynamic Type**
  - [ ] Test all screens at largest text size
  - [ ] Ensure buttons remain tappable

- [ ] **High contrast support**
  - [ ] Test with Increase Contrast enabled
  - [ ] Ensure colors meet WCAG contrast ratios

- [ ] **VoiceOver testing**
  - [ ] Navigate entire app with VoiceOver
  - [ ] Ensure logical reading order
  - [ ] Test scoring flow blind

### D4: Battery Profiling

- [ ] **Profile watch app energy usage**
  - [ ] Use Xcode Instruments (Energy Log)
  - [ ] Test during active match (30 min)
  - [ ] Identify CPU/haptic/screen hotspots

- [ ] **Optimize battery usage**
  - [ ] Avoid continuous timers (use scheduled updates)
  - [ ] Minimize haptic feedback intensity if needed
  - [ ] Reduce screen wake frequency
  - [ ] Test on physical watch

---

## Post-MVP Features (Deferred)

- [ ] **Advanced Stats**
  - [ ] Serving percentage
  - [ ] Unforced errors tracking
  - [ ] Rally duration tracking

- [ ] **Apple Watch Complications**
  - [ ] Live score complication (circular, rectangular)
  - [ ] Tap to open current match

- [ ] **Team/Club Presets**
  - [ ] Save favorite teams
  - [ ] Quick-start from preset
  - [ ] Club mode (multiple teams)

- [ ] **SharePlay / Remote Spectators**
  - [ ] Share Live Activity to friends
  - [ ] Remote scoreboard viewing

- [ ] **Themes & Customization**
  - [ ] Color themes (Pro)
  - [ ] Custom haptic packs (Pro)

---

## Monetization Implementation

- [ ] **Define Free vs Pro tiers**
  - [ ] Free: 1 saved matchup, side-out, Live Activity, last 10 games
  - [ ] Pro: Unlimited history, rally scoring, CSV export, presets, themes

- [ ] **Integrate StoreKit 2**
  - [ ] One-time unlock or subscription
  - [ ] Product IDs
  - [ ] Purchase flow UI
  - [ ] Restore purchases
  - [ ] Receipt validation

- [ ] **Add paywalls**
  - [ ] CSV export → paywall
  - [ ] Rally scoring → paywall
  - [ ] History >10 games → paywall
  - [ ] Team presets → paywall

---

## Release Checklist

- [ ] **App Store Assets**
  - [ ] 5 watch screenshots (see Copy Deck section in architecture)
  - [ ] 5 iPhone screenshots (Spectator screen, Live Activity)
  - [ ] App icon (watch + iOS)
  - [ ] App Store description
  - [ ] Keywords for ASO
  - [ ] Privacy policy (even if no data collection)

- [ ] **TestFlight Beta**
  - [ ] Upload build to App Store Connect
  - [ ] Invite beta testers (friends who play pickleball)
  - [ ] Collect feedback on usability, battery, haptics
  - [ ] Iterate on UX

- [ ] **Final QA**
  - [ ] All acceptance criteria met (see architecture doc §15)
  - [ ] No crashes in Crashlytics/logs
  - [ ] VoiceOver fully functional
  - [ ] Battery usage acceptable (<10% per 30-min match)

- [ ] **Submit to App Review**
  - [ ] Complete App Store Connect metadata
  - [ ] Submit for review
  - [ ] Respond to any feedback

---

## Development Workflow Recommendations

### Phase 1: Foundation (Week 1-2)
1. Complete Priority 0 (project setup)
2. Complete EPIC A (Domain & Storage)
3. Write comprehensive unit tests for ScoringEngine
4. Milestone: ScoringEngine fully tested and working

### Phase 2: Watch App Core (Week 3-4)
1. Complete B1 (Match setup)
2. Complete B2 (Play screen)
3. Complete B4 (Haptics)
4. Milestone: Can score a basic match on watch with haptics

### Phase 3: Advanced Watch Features (Week 5)
1. Complete B3 (Stacking overlay)
2. Complete B5 (History & settings)
3. Milestone: Full watch app feature-complete

### Phase 4: iPhone Companion (Week 6-7)
1. Complete C1 (WatchConnectivity)
2. Complete C2 (Live Activity)
3. Complete C3 (Spectator screen)
4. Complete C4 (CSV export)
5. Milestone: Full iPhone companion working

### Phase 5: QA & Polish (Week 8)
1. Complete EPIC D (all QA & accessibility)
2. Fix bugs from testing
3. Battery optimization
4. Milestone: Ready for TestFlight

### Phase 6: Beta & Release (Week 9-10)
1. TestFlight beta testing
2. Iterate based on feedback
3. Prepare App Store assets
4. Submit for review

---

## Notes & Decisions

**Decision Log:**
- Using SwiftData over Core Data (simpler, modern)
- ActivityKit for Live Activities (requires iOS 17+)
- Watch is source of truth (offline-first)
- Free tier limited to 10 games history
- Rally scoring locked behind Pro (niche feature)

**Open Questions:**
- Should we support singles? (Yes, per architecture)
- Do we need timeout tracking beyond simple pause? (No for MVP)
- Should spectator screen show rally history? (Deferred to post-MVP)

---

**Last Updated:** November 1, 2025
**Next Review:** After Phase 1 completion
