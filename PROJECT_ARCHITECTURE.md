# Pickleball Watch Scorekeeper + Stacking Coach
## Architecture & Development Guide

---

## 0) One‑liner / Executive summary

A **watch‑first** app that lets players **score matches with two big taps**, gives **haptic cues** for server/side‑out, and acts as a **"stacking coach"** that always shows where each partner should stand next. An **iPhone companion** mirrors the scoreboard (via **Live Activities**) for courtside viewing.

---

## 1) Naming & ASO (ready for App Store Connect)

* **App Name (≤30):** *Pickleball Watch Scorer*
* **Subtitle (≤30):** *Tap to score. Get stacking.*
* **Keywords (≤100):** `pickleball watch,scorekeeper,stacking,side out,rally scoring,Apple Watch,Live Activities`
* **Positioning line (marketing):** *Watch‑first scoring with a smart stacking coach and iPhone scoreboard.*

---

## 2) Platforms, targets & constraints

* **Primary:** watchOS **10+** (SwiftUI)
* **Companion:** iOS **17+** (SwiftUI + **ActivityKit** for Live Activities)
* **Hardware:** Apple Watch Series 6+ recommended (performance/haptics)
* **Accessibility:** VoiceOver/sensory feedback, extra‑large controls

---

## 3) MVP feature set

### Core (watch)

1. **Quick scoring UI** – Two large buttons: **"We won rally"** / **"They won"**. One‑tap **Undo**.
2. **Server/position guidance** – Always show *who* serves from **right/even** or **left/odd**, and *where* the receiving partner belongs. (Even/odd positioning comes from the official rules; the starting server's correct side is **right/even** when the team's score is even.)
3. **Stacking Coach** – Optional overlay that:
   * Displays the **correct server/receiver** and legal partner spots, then shows **post‑serve movement arrows** to your preferred formation. (Stacking is legal; players may stand anywhere on their side as long as the correct server/receiver are in position at serve.)
4. **Scoring modes**
   * **Side‑out (standard)** to 11/15/21, win by 2; three‑number call in doubles (e.g., *5‑3‑2*)
   * **Rally scoring (provisional)** per USA Pickleball 2025: point on each rally, **but the winning point must be on serve** ("freeze at game point"). Configure 11/15/21
   * (Optional toggle) **MLP‑style "freeze at 20"** variant for rec play (not an official tournament rule)
5. **Haptics & cues** – Distinct taps for point‑won, side‑out, time‑out; use WatchKit haptics
6. **Game/match setup** – Singles or doubles, race‑to, win‑by, change‑ends midpoint, team names/colors. (Mid‑match end changes at 6/8/11 per target score.)
7. **History (light)** – Last 10 games with scoreline and duration

### Companion (iPhone)

8. **Live scoreboard** – A **Live Activity** that mirrors points, serving team, and game race (for lock screen/Dynamic Island); tap opens **Spectator Screen** (full‑screen scoreboard, AirPlay‑friendly)
9. **Settings & export** – Configure defaults; export match summaries (CSV)

### Deferred (post‑MVP)

* Stats (serving %, unforced errors), Apple Watch **Complications**, team/club presets, SharePlay/remote spectators

---

## 4) Non‑functional requirements

* **Latency:** Score update → haptic < 100 ms
* **Resilience:** Works fully **offline**; watch is the source of truth
* **Privacy:** No accounts; all data on device(s)
* **Battery:** Avoid continuous timers; prefer time‑based schedule/Live Activity updates

---

## 5) System architecture

```
[watchOS App (SwiftUI, MVVM)]
  ├─ MatchSetupView / MatchPlayView / HistoryView / SettingsView
  ├─ ViewModels
  │   ├─ MatchSetupVM
  │   ├─ MatchPlayVM  <-- talks to ScoringEngine & StackingCoach
  │   └─ SettingsVM
  ├─ Domain
  │   ├─ ScoringEngine (side-out & rally rules)
  │   ├─ StackingCoach (position/rotation solver)
  │   ├─ MatchRepository (persistence)
  │   └─ HapticsService
  └─ Infra
      ├─ Persistence (SwiftData/Core Data on watch)
      ├─ WatchConnectivityClient  <-->  [iOS Companion]
      └─ ActivityUpdatePublisher  ---->  [ActivityKit Live Activity]
```

**iOS Companion (SwiftUI):** Spectator screen, settings, history mirror, **ActivityKit** Live Activity host

---

## 6) Key frameworks & why

* **SwiftUI** (watch+iOS) for single codebase UI
* **ActivityKit** to show a live scoreboard on iPhone lock screen / Dynamic Island
* **WatchConnectivity** for real‑time bridge from watch → iPhone
* **SwiftData/Core Data** for match logs
* **WatchKit haptics** for tactile cues

---

## 7) Data model (SwiftData/Core Data)

### Player

* `id: UUID`
* `displayName: String`
* `hand: enum {left,right,unknown}`
* `isStartingServer: Bool`

### Team

* `id: UUID`
* `name: String`
* `color: ColorHex`
* `players: [Player]`

### MatchSettings

* `format: enum {singles,doubles}`
* `scoring: enum {sideOut, rally(provisional, freezeAt: GamePoint|At20)}`
* `targetPoints: Int` (11/15/21)
* `winBy: Int` (2 default)
* `changeEndsAt: Int` (6/8/11 based on target)
* `stacking: Bool`
* `preferredFormation: enum {keepFHMiddle,keepBHMiddle,custom}`

### Game

* `id: UUID`
* `matchId: UUID`
* `startTime: Date`
* `endTime: Date?`
* `scoreA: Int`
* `scoreB: Int`
* `servingTeam: enum {A,B}`
* `serverNumber: Int` (1 or 2, doubles only)
* `startingServerTeam: enum {A,B}`
* `rallyHistory: [RallyEvent]`

### RallyEvent

* `timestamp: Date`
* `winner: enum {A,B}`
* `sideOut: Bool`
* `notes: String?`

---

## 8) Logic: ScoringEngine (deterministic)

### Side‑out scoring (doubles)

* **Serve order:** Team A starts at **0‑0‑2**; "First Server" rotates after side‑out
* **Even/odd rule:** The team's **starting server** stands **right/even** when the team score is even; **left/odd** when odd
* **Point:** If *serving* team wins rally → increment their point, **alternate service court**; otherwise advance **serverNumber** (1→2) or side‑out to the opponent
* **Score call:** *server score – receiver score – server number*

### Rally scoring (provisional, 2025)

* **Point on each rally**, **except the winning point must be on serve**. Implement as `freezeMode=.gamePointOnly`
* Optional rec mode: `freezeMode=.at20` (MLP style)

### Ends

* Auto‑prompt to switch ends at the **midpoint** (e.g., 6/8/11)

### Pseudocode (sketch)

```swift
func applyRallyResult(winner: TeamID) {
  if settings.scoring.isRally {
    if !isGamePoint(winner) || servingTeam == winner { addPoint(to: winner) }
    else { /* freeze: winner must serve to win */ }
    if winner != servingTeam { sideOutOrRotateForRally() } else { alternateServiceCourt() }
  } else {
    if winner == servingTeam { addPoint(to: winner); alternateServiceCourt() }
    else { advanceServerOrSideOut() }
  }
  if reachedMidpoint() { promptChangeEnds() }
  if winBySatisfied() { endGame() }
}
```

---

## 9) Logic: StackingCoach

**Goal:** Always show the **legal** server/receiver and where partners **should start**, then (if enabled) show **movement arrows** to the desired formation after the serve lands.

* **Legality checks:**
  * Correct **server** must serve from the correct box; correct **receiver** must receive; partners may stand **anywhere** on their side (including off‑court) → stacking is allowed

* **Computation:**
  1. Determine correct server/receiver from **even/odd** & service sequence
  2. Map **preferred formation** (e.g., keep Player X on right) into **starting spots** that satisfy legality
  3. Draw **on‑watch overlay**: small court diagram with **initial dots** + **post‑serve arrows**
  4. Provide text/haptic prompt: *"Alex serves right; Jordan start left; slide after return."*

---

## 10) Watch UX (MVP screens)

### 1. Match Setup
* Mode (Singles/Doubles), Scoring (Side‑out / Rally), target & win‑by, change‑ends, team names/colors, **Stacking ON/OFF**

### 2. Play Screen
* Header: *Team A 8 – 7 Team B*, serving icon (A), server number (2)
* **Two primary buttons** (A wins / B wins). Secondary: **Undo**, **Timeout**
* **Stacking chip**: toggles mini‑diagram overlay
* **Haptic cues:** point‑won, side‑out, server prompt

### 3. History
* List of last matches with scores; tap for details

### 4. Settings
* Defaults, rec‑mode rally freeze, export CSV

---

## 11) iPhone companion UX

* **Live Activity**: Large score, serving indicator, race target; taps open app. Updates pushed from watch via **WatchConnectivity** and posted with **ActivityKit** APIs
* **Spectator Screen**: Full‑screen scoreboard (AirPlay‑friendly)
* **History & Export**: Mirror of watch logs; CSV export

---

## 12) Persistence & sync

* **On watch:** SwiftData/Core Data store (matches, games)
* **Sync:** `WCSession` messages for **near‑real‑time** mirror to iPhone
* **Conflict policy:** Watch is authoritative during an active match; last‑write‑wins for history items

---

## 13) Notifications, haptics, and audio

* **Haptics:** Dedicated patterns for *point won*, *side‑out*, *timeout start/expire*. (Use only documented haptic types.)
* **Audio (later):** Optional spoken score via iPhone (companion) to avoid on‑watch TTS complexity during MVP

---

## 14) Privacy & safety

* No accounts, no location, no contacts
* Local data only; Live Activity carries **score + team names** only
* "Heads‑up" disclaimer: Don't operate the watch while ball is in play

---

## 15) Acceptance criteria (examples)

### Scoring basics
* Given **doubles, side‑out**, at *0‑0‑2*, app shows Team A serving, **server #2**; pressing **A wins** increments to *1‑0‑2* and flips service court
* Given Team A loses a rally on **server #1**, the app advances to **server #2**; losing again triggers **side‑out** to Team B

### Even/odd positioning
* When Team A's score is even, starting server is on **right/even**; odd → **left/odd**. The app's "You are here" bubble matches this

### Rally scoring (provisional)
* With rally scoring enabled, each rally adds a point to its winner, **except game‑winning point requires serve**; app prevents non‑serving game‑point

### Stacking legality
* Stacking overlay never suggests an illegal start; the correct server/receiver are always in legal boxes; partners may be anywhere else on their side

### Ends
* App prompts to change ends exactly at 6/8/11 midpoint for 11/15/21 games

### Live Activity
* Starting a match on watch starts/updates a Live Activity on iPhone within 1s; lock screen shows score & serving team

---

## 16) Engineering tasks (Jira‑ready EPICs)

### EPIC A – Domain & Storage
* **A1:** Define Swift models (Player, Team, MatchSettings, Game, RallyEvent)
* **A2:** Implement **ScoringEngine** (side‑out + rally variants) with unit tests
* **A3:** Implement **StackingCoach** (solver + diagram coordinates)
* **A4:** SwiftData/Core Data schema & migrations

### EPIC B – Watch App
* **B1:** Match setup flow (forms + presets; validation)
* **B2:** Play screen (two‑button scoring, Undo, Timeout)
* **B3:** Positioning banner & Stacking overlay
* **B4:** Haptics service (map events → haptics)
* **B5:** History list & details

### EPIC C – iPhone Companion
* **C1:** WCSession bridge (watch → phone)
* **C2:** Live Activity lifecycle (start/update/end)
* **C3:** Spectator screen UI
* **C4:** CSV export

### EPIC D – QA & Accessibility
* **D1:** Unit tests (all scoring paths; undo; freeze logic)
* **D2:** UI tests (happy path, undo, side‑out)
* **D3:** VoiceOver labels, Dynamic Type
* **D4:** Battery profiling (watch)

---

## 17) Test plan

### Unit (ScoringEngine)
* Side‑out: full server #1 → #2 → side‑out cycle; even/odd server positions for every score
* Rally (provisional): verify **game‑point must be on serve**; variant freeze‑at‑20

### UI
* Tap flows (win/lose/undo)
* Stacking overlay never suggests illegal start position
* End‑change prompt timing per target score

### Manual
* Real matches (singles/doubles); Live Activity visibility on iPhone

---

## 18) Monetization & packaging

* **Free:** One saved matchup, side‑out scoring, Live Activity
* **Pro (one‑time unlock or small sub):** Unlimited history, rally variants, CSV export, team presets, color themes, advanced haptics pack

---

## 19) Sample Swift types (sketch)

```swift
enum ScoringMode {
  case sideOut(target: Int, winBy: Int)
  case rally(target: Int, winBy: Int, freeze: Freeze)
  enum Freeze { case gamePointOnly, at20 }
}

struct MatchSettings: Codable, Hashable {
  var format: Format // .singles/.doubles
  var mode: ScoringMode
  var changeEndsAt: Int // 6/8/11
  var stacking: Bool
}

final class ScoringEngine {
  private(set) var state: GameState
  func pointWon(by team: TeamID) { /* see pseudocode in §8 */ }
  func undo() { /* revert last RallyEvent */ }
  // helpers: isGamePoint(), alternateServiceCourt(), advanceServerOrSideOut(), etc.
}
```

---

## 20) Copy deck (MVP, 5 screenshot captions)

1. **Tap to score** – Two big buttons; instant haptics
2. **Never lose track** – The watch tells you who serves next
3. **Stacking made simple** – Start legal, slide into your preferred positions
4. **Side‑out or rally** – Pick your format; win by two
5. **Live scoreboard** – Lock‑screen updates for your court

---

## 21) Risks & mitigations

* **Rule nuance/changes:** Engine references 2025 USA Pickleball rules (partner positions, serve sequence, rally scoring option). Keep a small **rules constants file** for quick updates
* **On‑court glare & micro‑taps:** Oversized touch targets; strong haptics
* **Sync fragility:** Watch is authoritative; queue updates to iPhone with retries

---

## Sources & References

* **USA Pickleball 2025 Official Rulebook** (serve sequence, even/odd positioning, partner positions; rally scoring provisional rules; mid‑game end change; doubles score call)
* **Apple ActivityKit docs** (Live Activities) - https://developer.apple.com/documentation/ActivityKit
* **Watch haptics API** (`WKInterfaceDevice.play`) - https://developer.apple.com/documentation/watchkit/wkinterfacedevice/play(_:)
* **Rally scoring "freeze at 20"** (rec/MLP variant explanation) - https://www.pickleballmax.com/2019/02/pickleball-rally-scoring/

---

## Next Steps

If helpful, we can create:
* A **Jira‑formatted backlog** (EPIC → stories with acceptance criteria)
* A **wireframe PDF** for the 4 watch screens + iPhone spectator view
* A **domain test suite** scaffold (XCTest) for ScoringEngine & StackingCoach
