# Ball Sort 3D

Single-file browser puzzle that renders a 3D ball-sorting board with Three.js and generates proof-backed solvable levels at runtime.

> **Project identity:** this repository implements **Ball Sort 3D**. It is distinct from the separate **Sorting Balls 3D** game/listing.

## Links

- Play: https://joenasr.itch.io/ball-sort-3d
- GitHub Pages: https://joenasriani.github.io/ball-sorting-puzzle/

## Repository scope

The entire game is implemented in `index.html`. There is no package manager, build pipeline, framework source tree, or local asset bundle.

```text
.
├── index.html   # game UI, renderer, gameplay, level generation, audio
└── README.md
```

Runtime dependencies are loaded from public CDNs.

## Runtime stack

| Area | Implementation |
| --- | --- |
| 3D rendering | Three.js `0.160.0` |
| Camera controls | Three.js `OrbitControls` |
| UI styling | Tailwind CSS CDN runtime |
| Icons | Lucide CDN runtime |
| Audio | Web Audio API synthesis |
| Input | Pointer events for mouse/touch interaction |
| Deployment model | Static HTML |

## Gameplay rules

Each tube stores an ordered array of color indices.

A move is valid only when:

1. the source and destination tubes are different;
2. the source contains at least one ball;
3. the destination is below its configured capacity; and
4. the destination is empty or its top ball matches the moving ball's color.

A level is complete when every non-empty tube:

- contains exactly the configured number of balls; and
- contains only one color.

Empty tubes are permitted in the solved state.

## Level generation

The game defines **10 base difficulty profiles**. Each profile specifies:

- tube count;
- balls per tube;
- color count;
- target scramble depth; and
- minimum number of mixed tubes.

The base configurations progress from:

```text
Level profile 1:  3 tubes, 3 balls/tube, 2 colors
...
Level profile 10: 11 tubes, 5 balls/tube, 9 colors
```

After the first 10 profiles, level configuration cycles while scramble depth increases by 10 moves per completed 10-level cycle.

### Solvability strategy

Boards are not produced by an unconstrained random shuffle.

`generateLevelData()` starts from a solved board and calls `buildGuaranteedSolvableLevel()` to construct a scrambled state through reversible moves:

1. create a solved tube configuration;
2. choose reverse-scramble moves from physically valid candidate states;
3. record the inverse of each scramble move as a solution path;
4. validate the generated state and replay the stored solution with normal player-move rules;
5. reject states that are already solved or fail validation;
6. prefer candidates that meet the configured mixed-tube threshold.

The generator makes up to 180 attempts. If no candidate reaches the preferred mixing threshold, it returns the strongest verified candidate. If no verified candidate survives validation, it falls back to a solved valid board rather than emitting an unverified puzzle.

## State validation

`validatePhysicalState()` checks:

- expected tube count;
- tube capacity;
- ball-stack height against tube height;
- valid color indices;
- exact per-color ball counts.

`validateSolutionPath()` then replays the stored inverse path and requires the final state to satisfy the solved-state predicate.

This provides a constructive solvability guarantee for generated boards under the implemented move rules.

## Interaction model

### Selecting and moving balls

- Click or tap a tube to select its top ball.
- Click or tap a valid destination tube to move the selected ball.
- Selecting the same tube again cancels the selection.
- Clicking outside the tube hit areas clears the current selection.
- Invalid moves are rejected without changing game state.

Ball transitions are animated in three stages:

1. lift above the source tube;
2. translate horizontally to the destination;
3. descend into the target stack.

Gameplay input is blocked while an animation is in progress.

## Undo and restart

Before each valid move, the complete tube state is cloned and pushed onto `undoStack`.

`Undo` restores the most recent snapshot and re-renders the level.

`Restart` regenerates the current level using the same difficulty profile but a newly generated puzzle state.

## Progression and scoring

- Completing a level adds `100` points.
- `Next` increments the level index and generates the next board.
- The 10 configuration profiles cycle indefinitely.
- Later cycles increase scramble depth, so progression can continue beyond the first 10 displayed levels.

## Rendering

The scene uses:

- a perspective camera;
- `OrbitControls` with damping;
- ambient and directional lighting;
- shadow-enabled spheres for balls;
- vertical cylinder markers and bases for tube positions;
- invisible cylinder hit volumes for tube selection;
- a resizable floor platform.

Tube spacing is reduced as tube count increases. Camera distance is recalculated from board width and viewport aspect ratio.

Responsive behavior includes:

- desktop field of view: `50°`;
- narrow-screen field of view: `60°`;
- device-pixel-ratio cap: `2`;
- adaptive camera distance;
- narrower tube spacing at 8+ and 10+ tubes.

## Audio

All game audio is synthesized at runtime with the Web Audio API; no prerecorded sound files are required.

`AudioEngine` creates:

- oscillator-based select, drop, invalid-move, undo, and completion sounds;
- a master gain stage;
- dynamics compression;
- a short delay send;
- low-pass filtering per sound event.

Audio initialization is deferred until user interaction to satisfy browser autoplay restrictions.

The HUD includes a sound toggle that enables or suppresses synthesized effects.

## Running locally

No build step is required.

Serve the repository through a local HTTP server:

```bash
python3 -m http.server 8080
```

Then open:

```text
http://localhost:8080/
```

Opening the file directly with `file://` is not recommended because the game uses ES modules and remote module imports.

## External dependencies

`index.html` currently loads:

```text
https://cdn.tailwindcss.com
https://unpkg.com/lucide@latest
https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js
https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/
```

A network connection is therefore required unless those dependencies are localized.

## Technical limitations

- The project is intentionally compact and implemented as one HTML file.
- Runtime dependencies are CDN-hosted and are not pinned locally.
- There is no service worker or offline application shell.
- Level generation uses `Math.random()`, so generated boards are not reproducible from a stored seed.
- Restarting a level creates a new board instead of restoring an identical initial board.
- Undo history is in-memory only and is cleared when a new level is generated.
- Progress and score are not persisted across page reloads.
- Automated test files are not included in this repository.

## Verification boundary

The implementation contains internal checks for board validity and constructive solvability. Those checks do not by themselves establish cross-browser compatibility, accessibility conformance, performance targets, or device-specific release readiness.
