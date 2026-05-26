# Crochet Audio Guidance App — Product Requirements Document

| Version | Date | Changes |
|---|---|---|
| v1 | 2026-05 | Initial draft |
| v2 | 2026-05-22 | Scope narrowed, risks documented, chunking logic defined |
| v3 | 2026-05-22 | Tech stack decided, service architecture defined, data contract added, Python service scoped |
| v4 | 2026-05-22 | Development methodology updated to TDD; testing approach documented |
| v5 | 2026-05-22 | Chart import scope redefined: XLSX from Stitch Fiddle as V1 primary input; image import moved to V2; colour review screen added (reduction, override, naming); data model updated; user flow revised |
| v6 | 2026-05-22 | Image import restored to V1; sourceColor field added to data contract for accurate colour reduction |
| v7 | 2026-05-22 | Workflow revised: colour naming combined with colour picker; XLSX and image paths unified at review screen; image quantisation runs before user colour choices |
| v8 | 2026-05-26 | Import flow split into Import Chart (XLSX or chart image) and Create Chart (design image); Mean Shift replaces upfront KMeans for initial colour detection; KMeans implemented directly in NumPy (not scikit-learn) for user-driven colour reduction; gauge-based grid for Create Chart path; auto-detect grid for chart image import; units preference (cm default) added to settings |
| v9 | 2026-05-26 | Full user flow and requirements defined: multi-chart projects; project rename and delete; notes at project and chart level; loading states and error handling for import; back navigation from colour review; single-colour warning; completion screen; row number repositioning; row narration on change only; narration format user setting; chunk size setting; pacing profile overridable per project; Settings section added (units, narration format, chunk size, pacing, theme, accessible mode) |

---

## 1. Product Summary

An offline-first crochet companion app focused on reducing screen interaction during tapestry and mosaic crochet work through audio-guided chart playback. The primary workflow centers around:

- importing or generating charts
- converting charts into logical audio instruction chunks
- navigating instructions through lock-screen/media controls
- maintaining progress without constant screen interaction

The app includes lightweight project tracking but is not intended to compete with full-featured craft management platforms.

---

## 2. Core Product Thesis

Users do not primarily need another crochet organizer. The core problem being addressed is:

- losing place in charts
- repeatedly interacting with phones while crafting
- cognitive overload from dense chart layouts

The product should optimize for low-attention crafting workflows.

---

## 3. Primary User

Primary target user:
- intermediate tapestry or mosaic crochet users
- users who craft while watching TV/videos/listening to media
- users who dislike repeatedly unlocking devices
- users who prefer passive or audio-assisted workflows

Secondary audience:
- users with attention regulation difficulties
- users seeking lower cognitive load crafting workflows

---

## 4. Product Goals

Primary goals:
- reduce screen interaction during crafting
- reduce chart-related cognitive load
- allow users to continue consuming media while crafting
- support interruption recovery
- support offline crafting sessions

Non-goals:
- replacing Ravelry
- building a social network
- becoming a marketplace
- becoming a full chart design suite in V1

---

## 5. Core User Flow

### Project creation
1. User creates a new project (name, optional project-level notes).
2. A project holds multiple charts — e.g. front and back panels, or multiple colourway sections of a pattern.

### Adding a chart
3. User chooses an entry point — **Import Chart** or **Create Chart**:
   - **Import Chart → XLSX** — file picker → API call → colour review
   - **Import Chart → Image** — camera/gallery → API call → colour review
   - **Create Chart → Design image** — camera/gallery (image first) → gauge input (stitches × rows per 10cm + target finished size) → API call → colour review
4. During the API call, a loading screen shows a spinner with step-by-step status labels ("Uploading…", "Detecting colours…", "Building chart…"). If the call fails, the app retries once silently, then shows an error message with a Retry button.
5. The **colour review screen** is identical for all three import paths. A back button returns to the import step without saving. If only one colour is detected, a warning is shown but the user can still proceed.
6. User names colours and optionally adjusts the palette, then confirms. The chart is saved to MMKV — fully offline from this point. Per-chart notes can be added here or from the project home screen later.

### Playback
7. From the project home screen, user starts or resumes any chart in the project.
8. App narrates chunked stitch instructions using yarn colour names in the user's chosen format (number first or colour first). Row number is announced when moving to a new row.
9. User advances manually or via assisted timing. Lock screen media controls mirror the in-app controls.
10. Progress is tracked visually (completed stitches and current chunk highlighted) and audibly.
11. If the user loses their place, they enter a row number to jump playback there.
12. When the final chunk of the last row completes, a **completion screen** is shown with a summary.

---

## 6. Core Features — MVP

### A. Audio Playback Engine
- spoken stitch instructions
- chunk-based narration (chunk size adjustable in Settings)
- narration format: number first ("5 in Toffee Brown") or colour first ("Toffee Brown: 5") — user setting
- row number announced when moving to a new row (not repeated on chunk replay or row replay)
- narration speed control
- repeat previous chunk
- back/forward navigation
- row replay
- full row narration mode
- progress persistence
- **Accessible mode** — narration text scrolls on screen in sync with audio, for users with audio processing difficulties or in noisy environments

### B. Playback Modes
1. **Manual Mode** — user manually advances chunks
2. **Assisted Mode** (default) — app auto-advances after inactivity timer; timer resets on manual interaction; timing influenced by chunk complexity and chunk size settings
3. **Continuous Row Mode** — row plays automatically with pauses between chunks

### C. Lock Screen / Media Controls
- next chunk
- previous chunk
- repeat chunk
- replay row
- pause playback
- resume playback

### D. Visual Progress Tracking
- completed stitches highlighted
- current chunk highlighted
- row number input for manual repositioning — user types or scrolls to a row number to jump playback there
- **completion screen** — shown when the final chunk of the last row completes; displays total rows completed with a button to return to the project home screen

### E. Lightweight Project Management
- create, rename, and delete projects (delete requires confirmation prompt)
- a project holds multiple charts (e.g. multiple pattern sections or panels)
- per-chart progress tracked independently; playback state saved per chart
- project home screen shows: all charts with per-chart progress, overall project progress, project-level notes, and Start/Resume button per chart
- project-level notes (yarn weight, hook size, general notes)
- per-chart notes (colour substitutions, section-specific notes)

### F. Settings

| Setting | Options | Default |
|---|---|---|
| Units | cm / inches | cm |
| Narration format | Number first ("5 in Colour") / Colour first ("Colour: 5") | Number first |
| Chunk size | Adjustable — max colour groups per narration chunk | TBD |
| Pacing profile | Slow / Standard / Fast — global default, overridable per project | Standard |
| Theme | Light / Dark | System default |
| Accessible mode | On / Off — narration text scrolls on screen in sync with audio | Off |

---

## 7. Chart Import

V1 offers two entry points, with three underlying import paths:

- **Import Chart** — the user has an existing chart to import, as either a Stitch Fiddle XLSX file or a chart image (photo or scan of an existing grid-based chart)
- **Create Chart** — the user has a design image (photo, illustration, downloaded image) they want to convert into a tapestry or mosaic crochet chart

All three paths converge at the same colour review screen.

### Import Chart — XLSX

Users export their chart from [Stitch Fiddle](https://www.stitchfiddle.com) as an XLSX file. The Python API parses the Excel grid: each cell represents one stitch, and the cell background colour (as a hex value) is the stitch colour. The grid structure (rows × columns) is taken directly from the spreadsheet. This approach is highly accurate because both the colour data and the grid structure are already defined — no image processing or approximation is required.

The Python library used for XLSX parsing is **openpyxl**, which provides direct access to cell fill colours.

### Import Chart — Image

For photos or scans of existing grid-based charts (hand-drawn, printed, or photographed). The Python API:
- Uses Pillow to load and preprocess the image
- Uses NumPy for pixel array operations
- Auto-detects the stitch grid from the image (identifies the regular grid structure and stitch boundaries)
- Samples the dominant colour of each detected stitch cell
- Runs **Mean Shift** colour clustering (scikit-learn) on the sampled stitch colours to identify the natural colour palette — no K (number of colours) needs to be specified upfront; Mean Shift finds natural colour groupings in colour space without being told how many to look for
- Stores the original per-stitch source colour alongside the detected assignment (see `sourceColor` in Section 20)

### Create Chart — Design Image

For photos, illustrations, or any image the user wants to convert into a new crochet chart. The user picks the image first (camera or gallery), then provides:
- **Gauge** — stitches × rows per 10cm (cm is the default unit, changeable in Settings)
- **Target finished size** — desired width × height in the user's chosen unit

The app calculates and shows back to the user:
- Grid dimensions (stitches × rows)
- Expected finished size

The user confirms, then the image is uploaded. The image is divided into the calculated grid; each cell's dominant colour is sampled. **Mean Shift** colour clustering then identifies the natural colour palette from the sampled cells.

### Two-stage colour pipeline (both image paths)

Both image import paths use a two-stage colour pipeline:

**Stage 1 — Mean Shift (API, automatic)**

Detects the colours genuinely present in the image without requiring a number of colours to be specified upfront. Each cluster returned by Mean Shift becomes a palette colour; cluster size gives the stitch count for that colour. This is the starting palette shown on the colour review screen.

**Stage 2 — KMeans or nearest-match (colour review screen, user-driven)**

When the user adjusts the palette on the review screen, two tools handle different cases:
- **"Reduce to N colours"** (user specifies a count, not which colours): KMeans runs with K=N on the `sourceColor` values to find the optimal N colours to represent the chart. KMeans is implemented directly in NumPy rather than using scikit-learn — the algorithm is simple enough (assign each point to its nearest centroid, recompute centroids, repeat until stable) to implement cleanly in about 20 lines of NumPy, and keeps the scikit-learn dependency scoped to Mean Shift only.
- **Specific colour changes** (user modifies or removes particular colours): nearest-match from `sourceColor` using RGB Euclidean distance reassigns each stitch to its closest remaining palette colour

### Colour review screen (V1, all import paths)

After any import, all three paths arrive at an identical review screen:
- **Back button** — returns to the import step without saving; allows the user to pick a different file
- **Single-colour warning** — if only one colour is detected, a warning is shown ("Only 1 colour detected — your image may not contain a chart"); the user can still proceed or go back
- Chart preview showing current colour assignments
- List of distinct colours — swatch, stitch count, and colour name
- Tapping a swatch opens a **combined colour picker + name input** — the user changes the colour and names it in one interaction. Colour names are stored per colour hex and used for narration.
- **Colour count reduction** — slider/stepper; if a count is specified without choosing colours, KMeans finds the optimal palette; if colours are modified directly, nearest-match from `sourceColor` reassigns stitches. LAB ΔE distance (more perceptually accurate than RGB Euclidean) may replace nearest-match in V2.
- **XLSX only**: reset option to restore original spreadsheet colours
- All colour operations happen client-side. No further API calls after initial import.

---

## 8. Audio and Media Behavior

The app should coexist with external media playback. Desired behavior:
- narration temporarily lowers external media volume
- narration overlays on top of video/music/podcasts
- media volume restores after narration

Audio behavior requirements:
- concise narration
- low interruption frequency
- predictable cadence
- minimal conversational language

---

## 9. Chunking Logic

Instruction chunking is a core differentiator. Rules:
- stop at logical color boundaries
- avoid splitting groups unnaturally
- prioritize readability over exact chunk limits

Example (max chunk size = 10, pattern: 3 brown, 3 green, 3 red, 1 brown):
- Preferred: `"3 brown, 3 green, 3 red"` then `"1 brown"`
- Not: `"3 brown, 3 green, 3 red, 1 brown"` as one chunk

Future considerations:
- repeat detection
- compressed narration
- adaptive pacing

---

## 10. Playback Timing System

Auto-advance timing should not rely solely on stitch count. Timing factors may include:
- stitch count
- number of color changes
- narration length
- user pacing profile

Initial implementation: simple heuristics rather than complex adaptive systems.

Pacing profiles: slow, standard, fast.

---

## 11. Offline Requirements

Must work offline:
- chart access
- project access
- playback
- narration
- progress persistence
- lock-screen/media controls

Chart conversion (image → structured data) requires a network connection. All subsequent use of a converted chart is fully offline.

---

## 12. Technical Architecture

The playback system operates on structured chart data rather than raw image data. This abstraction is critical for narration, playback timing, chunking, visual highlighting, and future repeat compression.

### iOS and Android Audio

Background audio and lock-screen media controls are handled natively by `react-native-track-player`, which uses AVAudioSession on iOS and ExoPlayer on Android. This mitigates the iOS background audio risk identified in v2.

---

## 13. Major Risks and Open Questions

1. **Image-to-chart conversion complexity** — updated in v8. Approach: Pillow + NumPy for preprocessing; Mean Shift (scikit-learn) for initial colour detection without requiring a colour count; auto-detect grid for chart images; gauge-based grid calculation for design images; KMeans for user-driven colour reduction; run-length encoding to produce chart rows. Risk remains for low-quality photo inputs where auto-detect grid may struggle; manual correction UI deferred to V2.
2. **Synchronization risk** — auto-advance may desync from actual crafting pace; users may stop mid-chunk without interaction.
3. **Scope creep risk** — chart editing tools can become a separate product category; project management features may dilute differentiation.
4. **Playback trust** — incorrect pacing or progression reduces trust rapidly.
5. ~~**iOS background behavior restrictions**~~ — mitigated by `react-native-track-player` (see Section 12).

---

## 14. Out of Scope — V1

- social feed
- marketplaces
- creator economy features
- AI-generated patterns
- wearable-native apps
- advanced chart editor
- collaborative editing
- beginner tutorials
- knitting support
- manual chart correction UI (deferred to V2)

---

## 15. Monetization Considerations

Tentative model:
- free tier with limited projects
- premium unlock for expanded usage/features

Open question: project-count limitations may not align well with how users evaluate craft apps.

Alternative monetization (future consideration):
- advanced chart conversion
- cloud sync
- export tools
- advanced playback customization

---

## 16. Success Criteria

The product succeeds if users:
- complete projects primarily using playback mode
- reduce screen interaction during crafting
- maintain crafting flow more consistently
- continue using playback across multiple projects

Key qualitative outcome: users feel less overwhelmed by charts.

---

## 17. Immediate Product Questions

Questions still requiring validation:
- Do users actually prefer assisted playback over manual?
- How often will full-row narration be used?
- How accurate does image conversion need to be for MVP usefulness?
- What level of manual cleanup is acceptable after chart generation?
- Is lightweight chart editing required in MVP?
- What pacing defaults work for most users?
- How tolerant are users of narration interruptions while watching media?

---

## 18. Technical Stack

Decided in v3.

### Mobile App
| Tool | Purpose |
|---|---|
| Expo (React Native) | Cross-platform iOS + Android app framework |
| TypeScript | Language throughout |
| Zustand | Client-side state management |
| MMKV | Fast offline-first local storage |
| react-native-track-player | Background audio + lock screen media controls |
| NativeWind | Tailwind CSS utility styling for React Native |
| Jest + React Testing Library | Unit and component testing |

### Chart Conversion API
| Tool | Purpose |
|---|---|
| Python 3 | Language |
| FastAPI | REST API framework |
| openpyxl | XLSX parsing — reads Stitch Fiddle cell background colours directly |
| Pillow | Image loading and preprocessing — image and design import paths |
| NumPy | Pixel array operations — image and design import paths |
| scikit-learn (Mean Shift) | Initial colour detection — finds natural colour groups without requiring K to be specified; used for both image import paths |
| NumPy (custom KMeans) | User-driven colour reduction — KMeans implemented directly in NumPy rather than scikit-learn; algorithm is simple enough to write cleanly without a library, keeping the scikit-learn dependency scoped to Mean Shift only |
| Pydantic | Data validation and typed models |
| pytest | API and service testing |

**Development methodology:** Test-Driven Development (TDD) throughout. Tests are written before implementation on both the Python API and the mobile app, following the Red → Green → Refactor cycle. TDD was chosen over BDD (Behaviour-Driven Development) because this is a solo project with no non-technical stakeholders — BDD's plain English Gherkin syntax adds overhead without benefit here. BDD may be introduced as a learning exercise at the integration test level in a later version.

### Infrastructure
| Concern | Decision |
|---|---|
| Platform | iOS + Android cross-platform from day one |
| API hosting | Render (free tier) |
| Mobile dev | Expo Go (development), EAS Build (production) |
| Version control | GitHub — github.com/GretaIvanovaite/crochet-audio-app |

---

## 19. Service Architecture

The app is two components sharing one data contract:

```
[Import Chart — XLSX]     [Import Chart — Image]     [Create Chart — Design Image]
        |                          |                            |
  openpyxl parses            auto-detect grid           user provides gauge
  cell colours + grid        Mean Shift colours          + target size
                                   |                    calculate grid
                                   |                    Mean Shift colours
                                   |                            |
                     +-------------+----------------------------+
                     |
                     | POST /convert (multipart upload)
                     v
            [Python FastAPI API]
          - openpyxl / Pillow / NumPy
          - Mean Shift: initial colour detection
          - Run-length encode rows → ChartResponse JSON
                     |
                     | ChartResponse JSON
                     v
            [Expo mobile app]
          - Colour review screen
            (KMeans or nearest-match, client-side)
          - Stores confirmed chart in MMKV (offline from here)
          - All playback, chunking, narration, state from local data
```

The API is called once per chart import. Everything after that is offline.

---

## 20. Data Contract

The canonical JSON structure shared between the Python API and the TypeScript mobile app.

`ChunkItem` includes a `name` field (populated by the user during the colour review screen) and a `sourceColor` field (the original colour from the source file, preserved for accurate colour reduction).

### Why `sourceColor` matters

When a user reduces the number of distinct colours, each stitch must independently find its closest match from the remaining palette — not inherit the result of a palette-level merge. To do this accurately, the app compares each stitch's **original** source colour against the remaining options, not its currently assigned colour. Without `sourceColor`, repeated reductions would compound errors. For XLSX import, `sourceColor` is the exact cell hex from the spreadsheet. For image import (both Import Chart — Image and Create Chart paths), it is the dominant colour of the stitch cell sampled from the source image before any user-driven reduction.

```json
{
  "rows": [
    {
      "rowNumber": 1,
      "chunks": [
        { "sourceColor": "#3d2b1f", "color": "#3d2b1f", "name": "Toffee Brown", "count": 5 },
        { "sourceColor": "#f8f4e8", "color": "#ffffff", "name": "Cream", "count": 3 },
        { "sourceColor": "#3d2b1f", "color": "#3d2b1f", "name": "Toffee Brown", "count": 2 }
      ]
    }
  ]
}
```

`sourceColor` is set by the API and never changed. `color` is the currently assigned palette colour and may be updated by the user during the review step. `name` is always user-supplied.

### TypeScript type (mobile)
```ts
export interface ChunkItem {
  sourceColor: string  // original colour from source — never mutated
  color: string        // currently assigned palette colour
  name: string         // yarn name for narration — user supplied
  count: number
}
export interface Row { rowNumber: number; chunks: ChunkItem[] }
export interface Chart { rows: Row[] }
```

### Pydantic model (API)
```python
class ChunkItem(BaseModel):
    sourceColor: str       # original colour from source file — never changed
    color: str             # assigned palette colour — same as sourceColor on first return
    name: str = ""         # empty on API response; populated by user during colour review
    count: int

class Row(BaseModel):
    rowNumber: int
    chunks: list[ChunkItem]

class ChartResponse(BaseModel):
    rows: list[Row]
```

Colours are stored as hex strings. The `name` field is what gets narrated during playback — the mobile app must not narrate the hex value directly. If a name is empty at playback time, the app should prompt the user to complete the colour review step.
