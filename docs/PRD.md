# Crochet Audio Guidance App — Product Requirements Document

| Version | Date | Changes |
|---|---|---|
| v1 | 2026-05 | Initial draft |
| v2 | 2026-05-22 | Scope narrowed, risks documented, chunking logic defined |
| v3 | 2026-05-22 | Tech stack decided, service architecture defined, data contract added, Python service scoped |
| v4 | 2026-05-22 | Development methodology updated to TDD; testing approach documented |

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

1. User imports a chart image via the app.
2. App sends the image to the chart conversion API (Python service).
3. API returns structured chart data in the standard data contract format.
4. App stores the converted chart locally (offline from this point).
5. User starts a playback session.
6. App narrates chunked stitch instructions.
7. User advances manually or through assisted timing.
8. Progress is tracked visually and audibly.
9. User can recover position after interruptions.

---

## 6. Core Features — MVP

### A. Audio Playback Engine
- spoken stitch instructions
- chunk-based narration
- narration speed control
- repeat previous chunk
- back/forward navigation
- row replay
- full row narration mode
- progress persistence

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
- visual recovery support
- manual repositioning support

### E. Lightweight Project Management
- save projects
- associate charts with projects
- save playback state
- project notes
- yarn/material notes

---

## 7. Chart Import and Generation

Supported V1 inputs:
- clean chart images
- photos of physical charts

User-configurable generation settings:
- stitch width
- stitch height
- number of colors
- color simplification level

Important limitation: V1 should prioritize controlled/clean imports rather than arbitrary image parsing. Manual cleanup/editing tools may be required after generation and are deferred to V2.

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

1. **Image-to-chart conversion complexity** — scoped to KMeans color quantisation in v3. Approach: Pillow + NumPy for preprocessing, scikit-learn KMeans for color reduction, run-length encoding to produce chart rows. Risk remains for low-quality inputs; manual correction UI deferred to V2.
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
| Pillow | Image loading and preprocessing |
| NumPy | Pixel array operations |
| scikit-learn (KMeans) | Color quantisation / palette reduction |
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
[Chart image]
     |
     | POST /convert (multipart upload)
     v
[Python FastAPI API]
  - Pillow: load + preprocess image
  - NumPy: pixel array manipulation
  - KMeans: color quantisation
  - Run-length encode rows → ChartResponse JSON
     |
     | ChartResponse JSON
     v
[Expo mobile app]
  - Stores ChartResponse in MMKV (offline from here)
  - All playback, chunking, narration, state from local data
```

The API is called once per chart import. Everything after that is offline.

---

## 20. Data Contract

The canonical JSON structure shared between the Python API and the TypeScript mobile app.

```json
{
  "rows": [
    {
      "rowNumber": 1,
      "chunks": [
        { "color": "#3d2b1f", "count": 5 },
        { "color": "#ffffff", "count": 3 },
        { "color": "#3d2b1f", "count": 2 }
      ]
    },
    {
      "rowNumber": 2,
      "chunks": [
        { "color": "#ffffff", "count": 4 },
        { "color": "#3d2b1f", "count": 6 }
      ]
    }
  ]
}
```

### TypeScript type (mobile)
```ts
export interface ChunkItem { color: string; count: number }
export interface Row { rowNumber: number; chunks: ChunkItem[] }
export interface Chart { rows: Row[] }
```

### Pydantic model (API)
```python
class ChunkItem(BaseModel):
    color: str
    count: int

class Row(BaseModel):
    rowNumber: int
    chunks: list[ChunkItem]

class ChartResponse(BaseModel):
    rows: list[Row]
```

Colors are stored as hex strings. The API returns the closest palette color; the mobile app is responsible for display labeling (e.g. mapping `#3d2b1f` → "brown" based on user-defined palette names, V2 feature).
