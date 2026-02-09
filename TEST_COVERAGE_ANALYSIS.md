# Test Coverage Analysis: ZEN-EBook

## Executive Summary

The ZEN-EBook project currently has **zero test coverage**. There are no test files, no testing framework installed, no test scripts in `package.json`, and no test step in the CI/CD pipeline. The codebase contains ~2,700 lines of TypeScript/React code across 11 source files with significant testable logic.

This document identifies the highest-priority areas where tests should be added, categorized by risk and impact.

---

## Current State

| Metric | Value |
|---|---|
| Source files | 11 |
| Lines of code | ~2,700 |
| Test files | 0 |
| Test framework | None installed |
| CI test step | None |
| Code coverage | 0% |

---

## Recommended Testing Framework Setup

Before writing any tests, the project needs a testing foundation:

1. **Vitest** - natural choice since the project already uses Vite; zero-config integration
2. **React Testing Library** (`@testing-library/react`) - for component rendering and interaction tests
3. **jsdom** - browser environment simulation for Vitest
4. **@testing-library/user-event** - for realistic user interaction simulation

The Three.js scene components (`QuantumScene.tsx`, `ZenScene.tsx`) would need mocking via a canvas mock library or should be excluded from unit tests and covered by visual/E2E tests instead.

---

## Priority 1: High-Risk Stateful Logic (Critical)

These areas contain complex state machines, timers, and user interaction logic that are most likely to regress.

### 1.1 EbookReader Navigation (`components/EbookReader.tsx`)

**Why it matters:** This is the core navigation engine of the entire application. Bugs here break the whole reading experience.

**What to test:**
- **`goTo()` boundary checking** (lines 123-127): Verify it rejects `target < 0` and `target >= chapters.length`, and correctly sets direction
- **`next()` / `prev()`** (lines 129-130): Verify page increments/decrements correctly and stops at boundaries
- **`handleDragEnd()` swipe logic** (lines 171-177): Test that swipes below `SWIPE_THRESHOLD` (50px) are ignored, and velocity thresholds are respected
- **`handleTap()` tap zones** (lines 180-186): Test the 25%/75% tap zone calculation — left 25% goes back, right 25% goes forward, middle 50% does nothing
- **`slugify()` utility** (line 131-134): Pure function, trivial to unit test; verify it handles special characters, leading/trailing dashes, empty strings
- **`downloadChapter()` blob generation** (lines 136-152): Verify correct filename format (`ebook-01-the-path-to-mindful-living.txt`), content assembly, and blob creation
- **Keyboard navigation** (lines 155-162): Test that ArrowRight/Space advance pages and ArrowLeft goes back
- **Scroll-to-top on page change** (lines 165-169): Verify scroll reset occurs when page state changes

### 1.2 MeditationTimer (`components/ZenTools.tsx:23-126`)

**Why it matters:** Contains interval-based countdown logic with multiple states (running, paused, finished).

**What to test:**
- Timer counts down from `duration` to 0
- Timer stops and sets `finished = true` when reaching 0
- `reset()` restores initial state
- `selectPreset()` changes duration and resets timer
- Progress calculation: `1 - remaining / duration`
- Time display formatting: minutes/seconds with zero-padding

### 1.3 MindfulMinute (`components/ZenTools.tsx:452-543`)

**Why it matters:** Dual-timer component (60s countdown + 12s prompt rotation) with three distinct view states.

**What to test:**
- 60-second countdown behavior
- Prompt cycling every 12 seconds during active state
- Transition from active to completed state when timer hits 0
- `reset()` restores all state to initial values
- View state transitions: idle -> active -> completed -> idle

### 1.4 BreathingPacer (`components/Practices.tsx:11-82`)

**Why it matters:** Runs a continuous animation cycle with phase transitions and scale calculations.

**What to test:**
- Phase cycle order: Inhale -> Hold -> Exhale -> Wait -> Inhale
- Phase transitions occur after timer reaches 3
- `getScale()` returns correct values per phase (1.0-1.8 range for Inhale, fixed 1.8 for Hold, 1.8-1.0 for Exhale, fixed 1.0 for Wait)
- `getMessage()` returns correct message for each phase

---

## Priority 2: User Input and Data Handling (High)

These components accept user input and manage data collections.

### 2.1 MoodTracker (`components/ZenTools.tsx:234-307`)

**What to test:**
- `logMood()` does nothing when no mood is selected (`selected === null`)
- `logMood()` creates a `MoodEntry` with correct mood, note, and timestamp
- Entries list is capped at 5 items (`.slice(0, 5)`)
- Newest entries appear first (prepend behavior)
- State resets after logging (selected -> null, note -> empty)

### 2.2 GratitudeGarden (`components/Practices.tsx:85-136`)

**What to test:**
- `plantSeed()` adds input to the beginning of the seeds array
- Empty/whitespace-only input is rejected
- Seeds list is capped at 5 items
- Input field clears after planting
- Default seeds are present on initial render

### 2.3 IntentionSetter (`components/ZenTools.tsx:697-758`)

**What to test:**
- `setIt()` prevents default form submission
- Empty/whitespace-only intentions are rejected
- `intention.trim()` is stored (whitespace stripped)
- Input clears and saved intention displays after setting
- "Set New Intention" resets back to input view

### 2.4 MantraCounter (`components/ZenTools.tsx:319-386`)

**What to test:**
- `increment()` increases count by 1
- Count cannot exceed target
- Target cycling logic: 108 -> 54 -> 27 -> 9 (decrease) and 9 -> 27 -> 54 -> 108 (increase)
- Count resets when target changes
- Mantra index cycles through `MANTRAS` array (modulo arithmetic)
- Completion detection: `count >= target`

---

## Priority 3: Step-Based Navigation Components (Medium)

### 3.1 BodyScanGuide (`components/ZenTools.tsx:140-215`)

**What to test:**
- Step navigation: prev/next within bounds (0 to 7)
- Auto-play advances step every 8 seconds
- Auto-play stops at last step
- Clicking a step indicator jumps to that step and disables auto-play
- Manual navigation disables auto-play

### 3.2 SleepWindDown (`components/ZenTools.tsx:555-628`)

**What to test:**
- `toggleCheck()` adds/removes step indices from checked set
- "Done - Next Step" button checks current step and auto-advances (with 300ms delay)
- All-complete detection: `checked.size === WIND_DOWN_STEPS.length`
- Step indicators reflect checked/current/other states correctly

### 3.3 AffirmationCards (`components/ZenTools.tsx:403-440`)

**What to test:**
- `next()` / `prev()` cycle through affirmations with correct modulo wrapping
- Direction state sets correctly for animation purposes
- Index stays within bounds of `AFFIRMATIONS` array

---

## Priority 4: Interactive Diagrams (Medium)

### 4.1 SurfaceCodeDiagram (`components/Diagrams.tsx:12-107`)

**What to test:**
- `toggleError()` adds/removes qubit IDs from error array
- Active stabilizer calculation based on parity (odd error count = active)
- Adjacency mapping is correct (qubit 4 affects all stabilizers)
- Status message: "System is stable" with no errors vs. "Detected N parity violations"

### 4.2 PerformanceMetricDiagram (`components/Diagrams.tsx:183-273`)

**What to test:**
- Distance selector updates data correctly for distances 3, 5, and 11
- `formatValue()` formatting: values < 0.01 show 4 decimal places, otherwise 2
- Bar height normalization against `maxVal` (125% of largest value)

### 4.3 TransformerDecoderDiagram (`components/Diagrams.tsx:110-180`)

**What to test:**
- Step auto-cycles through 0-3 every 2 seconds
- Step resets to 0 after reaching 3 (modulo 4)

---

## Priority 5: Rendering and Integration (Lower)

### 5.1 ZenToolsGrid (`components/ZenTools.tsx:774-849`)

**What to test:**
- Clicking a tool button sets it as active
- Clicking the active tool deactivates it (toggle behavior)
- Close button resets active tool to null
- Correct component renders for each tool ID
- All 9 tools are present in the grid

### 5.2 TOCDrawer (`components/EbookReader.tsx:25-96`)

**What to test:**
- Drawer renders when `open` is true
- `onSelect` callback fires with correct index when chapter is clicked
- `onClose` fires when backdrop or X button is clicked
- Current chapter is visually distinguished
- Read chapters (index < current) show completion indicator

### 5.3 Chapter Content (`components/Chapters.tsx`)

**What to test:**
- `CHAPTERS` array has exactly 10 entries
- Each chapter has required fields: `id`, `number`, `title`, `subtitle`, `content`
- Chapter numbers are sequential 1-10
- Chapter IDs are unique
- Each chapter renders without errors (smoke tests)

---

## Priority 6: 3D Scene Components (Low - Requires Special Handling)

### QuantumScene.tsx and ZenScene.tsx

These components use `@react-three/fiber` and `@react-three/drei` which require WebGL context. They are difficult to unit test.

**Recommended approach:**
- Mock the `Canvas` component and verify child component structure
- Use snapshot tests to catch unintended structural changes
- Defer visual correctness testing to Playwright or Cypress E2E tests with screenshot comparison
- Verify that exported components (`HeroScene`, `QuantumComputerScene`, `SerenityPond`) render without throwing

---

## CI/CD Recommendations

The current GitHub Actions workflow (`.github/workflows/static.yml`) only builds and deploys. It should be extended to:

1. **Add a test step** before the build step:
   ```yaml
   - name: Run tests
     run: npm test
   ```
2. **Add a coverage threshold** so the pipeline fails if coverage drops below a minimum (e.g., 60% to start, increasing over time)
3. **Consider adding a lint step** since no linter is currently configured either

---

## Suggested Implementation Order

1. **Install testing dependencies** (Vitest, React Testing Library, jsdom)
2. **Configure Vitest** in `vite.config.ts` or a separate `vitest.config.ts`
3. **Add `"test"` script** to `package.json`
4. **Write tests for Priority 1** (EbookReader, MeditationTimer, MindfulMinute, BreathingPacer) - these cover the core user experience
5. **Write tests for Priority 2** (MoodTracker, GratitudeGarden, IntentionSetter, MantraCounter) - these cover data integrity
6. **Write tests for Priority 3-4** (step-based components, diagrams) - these catch interaction regressions
7. **Add smoke tests for Priority 5** (grid, drawer, chapters)
8. **Add CI test step** to the GitHub Actions workflow
9. **Set initial coverage threshold** and increase it incrementally
