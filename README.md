# Brick Breaker — Canvas Arcade Game

> A lightweight browser-based Breakout game built with HTML5 Canvas, vanilla JavaScript, jQuery, CSS, image assets, and browser audio APIs.

[![JavaScript](https://img.shields.io/badge/JavaScript-game%20logic-F7DF1E?style=flat-square&logo=javascript&logoColor=111827)](#technical-architecture)
[![HTML5 Canvas](https://img.shields.io/badge/HTML5-Canvas-E34F26?style=flat-square&logo=html5&logoColor=white)](#rendering-model)
[![jQuery](https://img.shields.io/badge/jQuery-3.6.0-0769AD?style=flat-square&logo=jquery&logoColor=white)](#dependencies)
[![Status](https://img.shields.io/badge/status-playable%20prototype-2563EB?style=flat-square)](#project-status)
[![License](https://img.shields.io/badge/license-not%20specified-6B7280?style=flat-square)](#license)

## Table of contents

- [Project status](#project-status)
- [Overview](#overview)
- [Gameplay](#gameplay)
- [Problems solved](#problems-solved)
- [Features](#features)
- [Technical architecture](#technical-architecture)
- [Rendering model](#rendering-model)
- [Game state](#game-state)
- [Collision and progression rules](#collision-and-progression-rules)
- [Repository structure](#repository-structure)
- [Dependencies](#dependencies)
- [Run locally](#run-locally)
- [Controls](#controls)
- [Audio and visual feedback](#audio-and-visual-feedback)
- [Implementation notes](#implementation-notes)
- [Known limitations](#known-limitations)
- [Testing checklist](#testing-checklist)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)

## Project status

**Playable frontend prototype.**

The repository contains a complete browser game loop rather than a static mockup. The current implementation includes a 400 × 500 Canvas, keyboard-controlled paddle movement, a bouncing ball, brick collision, score tracking, lives, progressive levels, victory/game-over overlays, and sound effects.

It is intentionally small and dependency-light. There is no backend, database, build pipeline, account system, or server-side game state. The browser is the runtime and the source files can be served directly as static assets.

## Overview

Brick Breaker is a focused implementation of the classic paddle-and-ball arcade loop:

1. The player moves a paddle horizontally.
2. The ball bounces against walls and the paddle.
3. Bricks disappear when hit.
4. Each destroyed brick increases the score.
5. Missing the ball consumes a life.
6. Clearing a level creates a larger brick layout and increases ball speed.
7. Clearing the fifth level displays the victory screen.

The project is valuable as a compact study of browser game fundamentals: real-time input, deterministic frame updates, collision checks, state transitions, asset loading, and feedback design.

## Gameplay

### Initial state

- Canvas size: `400 × 500` pixels.
- Starting lives: `3`.
- Starting score: `0`.
- Starting level: `1`.
- Maximum level: `5`.
- Initial brick layout: `1` row × `7` columns.
- Score per brick: `10` points.

### Player objective

Clear every brick while preserving enough lives to reach the final level. The paddle is constrained to the horizontal boundaries of the Canvas, and the ball angle changes according to the point where it hits the paddle.

### Level progression

When all active bricks are destroyed:

- A victory sound is played.
- The level counter increases.
- The brick row count increases.
- The ball speed increases by `0.5`.
- The ball is reset above the paddle.

At level `5`, clearing the final brick stops the game loop and shows the victory overlay.

### End states

The game has two terminal states:

- **Victory:** all bricks on the fifth level are cleared.
- **Game over:** the life counter reaches zero.

The **Play Again!** action reloads the page, resetting the complete JavaScript state.

## Problems solved

### Translating arcade rules into browser state

The game converts player input, object positions, collisions, score, lives, and progression into a small set of mutable state variables and update functions. This keeps the core loop understandable while demonstrating the relationship between simulation and rendering.

### Providing immediate feedback

Gameplay events produce both visual and audio feedback:

- Wall collisions play a wall sound.
- Paddle collisions play a paddle sound.
- Brick collisions play a brick sound and remove the brick.
- Lost lives play a life-loss sound.
- Completed levels play a victory sound.
- Score, lives, and level are drawn directly on the Canvas.

### Increasing difficulty without a level editor

The current implementation generates the next layout by increasing the number of brick rows. This provides a compact progression mechanism without requiring separate scene files or hand-authored level data.

### Keeping the delivery simple

The game has no build step and no custom server. A static HTTP server is enough to run it locally, which makes the project easy to inspect, demonstrate, and deploy to static hosting.

## Features

- HTML5 Canvas rendering.
- Keyboard controls using the left and right arrow keys.
- Paddle boundary constraints.
- Ball movement with horizontal and vertical velocity.
- Wall, paddle, and brick collision detection.
- Dynamic brick grid generation.
- Five-level progression.
- Score and lives display.
- Level display.
- Victory and game-over overlays.
- Restart behavior.
- Background music and event-specific sound effects.
- Sound mute/unmute control.
- Themed image assets and custom typography.
- No backend or installation-time dependency beyond a browser and a static server.

## Technical architecture

```text
HTML document
    │
    ├── Canvas and overlay elements
    ├── jQuery event bindings
    └── JavaScript assets
          │
          ├── loading.js
          │     └── images and Audio objects
          │
          └── settings.js
                ├── game state
                ├── input handling
                ├── movement
                ├── collision rules
                ├── rendering
                ├── level transitions
                └── audio controls
```

### Runtime responsibilities

| Area | Current implementation |
|---|---|
| Document structure | `index.html` |
| Styling and layout | `gamestyle.css` |
| Asset initialization | `loading.js` |
| Game state and rules | `settings.js` |
| Rendering surface | `#breakout` Canvas |
| Input | jQuery `keydown` and `keyup` handlers |
| Animation | `requestAnimationFrame` |
| Audio | Browser `Audio` objects |
| Persistence | None; a restart reloads the page |

## Rendering model

The game uses a single Canvas rendering surface. Each animation frame follows this sequence:

```text
Clear/draw background
        │
        ▼
Draw paddle, ball, bricks, and statistics
        │
        ▼
Move paddle and ball
        │
        ▼
Resolve wall, paddle, and brick collisions
        │
        ▼
Evaluate game-over and level-up conditions
        │
        ▼
Request the next animation frame
```

The main loop is implemented in `looping()` and stops requesting frames once `GAME_OVER` becomes `true`.

### Canvas coordinates

- Origin: top-left corner.
- Positive `x`: right.
- Positive `y`: down.
- Paddle position is calculated relative to the bottom margin.
- Bricks are placed from the configured top and left offsets.

## Game state

The central state currently includes:

| Variable | Meaning | Initial value |
|---|---|---:|
| `LIFE` | Remaining attempts | `3` |
| `SCORE` | Current score | `0` |
| `LEVEL` | Current level | `1` |
| `MAX_LEVEL` | Final level | `5` |
| `GAME_OVER` | Stops the animation loop | `false` |
| `leftArrow` | Left input flag | `false` |
| `rightArrow` | Right input flag | `false` |
| `bricks` | Active brick grid | Generated at startup |

The paddle and ball are represented by objects containing position and size/velocity data. Each brick stores its position and a `status` flag indicating whether it is still active.

## Collision and progression rules

### Wall collision

When the ball reaches the left or right Canvas boundary, its horizontal velocity is inverted. When it reaches the top boundary, its vertical velocity is inverted.

If the ball passes below the Canvas, the player loses one life and the ball is reset above the paddle.

### Paddle collision

The paddle collision calculates the horizontal impact point relative to the paddle center. That normalized impact point determines the ball angle:

- Center hits produce a more vertical trajectory.
- Edge hits produce a wider horizontal trajectory.

This makes paddle positioning affect the next movement direction rather than producing the same bounce every time.

### Brick collision

Each active brick is checked against the ball's bounding area. On collision:

1. The brick is marked inactive.
2. The ball's vertical direction is inverted.
3. The score increases by `10`.
4. The brick-hit sound is played.

### Level completion

The game scans the brick grid to determine whether every brick has become inactive. The next level increases the row count and ball speed, then regenerates the grid.

## Repository structure

```text
bricks-bricking-game/
├── brick breaking/
│   ├── index.html              # Page shell, Canvas, overlays, scripts
│   ├── settings.js             # Game state, loop, rules, input, rendering
│   ├── loading.js              # Image and audio asset initialization
│   ├── gamestyle.css           # Layout, typography, overlays, controls
│   ├── effect/
│   │   ├── bgmusic.mp3
│   │   ├── brick_hit.mp3
│   │   ├── life_lost.mp3
│   │   ├── paddle_hit.mp3
│   │   ├── wall.mp3
│   │   └── win.mp3
│   ├── fonts/
│   │   └── Eater-Regular.ttf
│   └── img/
│       ├── SOUND_ON.png
│       ├── SOUND_OFF.png
│       ├── bg.jpg
│       ├── bol.png
│       ├── gameoverone.png
│       ├── heart.jpeg
│       ├── level.png
│       ├── life.png
│       ├── score.png
│       ├── youwon.png
│       └── ...
└── README.md
```

The space in `brick breaking` is part of the current directory name. Quote the path or escape the space in shell commands.

## Dependencies

### Runtime dependencies

- A modern browser with HTML5 Canvas support.
- Browser support for `requestAnimationFrame`.
- Browser audio support for MP3 assets.
- jQuery `3.6.0`, loaded from the jQuery CDN.
- Google Fonts `Germania One`, loaded from the Google Fonts stylesheet.

### Development dependencies

There is currently no `package.json`, bundler, framework, or package installation step. The game is served as static files.

For a more reproducible production build, future work should pin or locally package external dependencies instead of relying exclusively on CDN availability.

## Run locally

A local HTTP server is recommended because browsers may restrict asset loading when an HTML file is opened directly with `file://`.

### Python

```bash
git clone https://github.com/AchrafHasnaoui31/bricks-bricking-game.git
cd bricks-bricking-game/brick\ breaking
python3 -m http.server 8000
```

Open:

```text
http://localhost:8000/
```

### Node.js

If a static server is already available:

```bash
npx serve .
```

Run the command from inside `brick breaking/`.

### Stop the server

Press `Ctrl+C` in the terminal running the local server.

## Controls

| Action | Input |
|---|---|
| Move paddle left | Left Arrow |
| Move paddle right | Right Arrow |
| Toggle audio | Click the sound icon |
| Restart after win/loss | Click `Play Again!` |

The current implementation does not define touch, gamepad, WASD, pause, or configurable key controls.

## Audio and visual feedback

`loading.js` initializes the image and audio resources before the game loop starts.

### Audio events

| Asset | Event |
|---|---|
| `wall.mp3` | Ball hits a wall |
| `paddle_hit.mp3` | Ball hits the paddle |
| `brick_hit.mp3` | Ball destroys a brick |
| `life_lost.mp3` | Ball leaves the bottom of the Canvas |
| `win.mp3` | Level is cleared |
| `bgmusic.mp3` | Background music |

The sound icon toggles the `muted` state of every audio object and swaps between the on/off images.

### Accessibility considerations

The current UI uses image-based feedback and has limited alternative text. Audio is optional but the game does not yet expose a complete text or visual equivalent for every sound event. Improving labels, focus behavior, contrast, keyboard semantics, and non-audio feedback is part of the roadmap.

## Implementation notes

### Why `requestAnimationFrame`

`requestAnimationFrame` synchronizes rendering with the browser's refresh cycle and avoids a fixed timer for animation. It also naturally pauses requesting new frames after a terminal state.

### Why the brick grid is generated at runtime

Runtime generation makes level scaling compact: increasing `brick.row` creates a larger board without duplicating markup or creating multiple HTML pages.

### Why assets are initialized separately

Keeping image and audio construction in `loading.js` gives the game-rule file named resources such as `BALL_IMG`, `BRICK_HIT`, and `WIN` without mixing every asset path into collision logic.

### Current code-quality improvement opportunities

The current prototype keeps most behavior in one `settings.js` file. A production-quality refactor should separate:

```text
state/
  gameState.js
entities/
  paddle.js
  ball.js
  bricks.js
systems/
  collisionSystem.js
  levelSystem.js
rendering/
  canvasRenderer.js
audio/
  audioManager.js
input/
  keyboardController.js
```

This would make rules easier to test independently and reduce coupling between rendering, audio, and simulation.

## Known limitations

- Fixed Canvas dimensions limit responsive behavior on small screens.
- Controls currently support only the left and right arrow keys.
- No touch, mobile, gamepad, or remappable controls.
- No pause/resume state.
- No persistent high score or local storage.
- No level editor or external level configuration.
- Collision resolution is simple and can be improved for high-speed edge cases.
- Several responsibilities are concentrated in `settings.js`.
- The browser may block autoplay audio until user interaction.
- External jQuery and Google Fonts dependencies are CDN-based.
- No automated unit, integration, or browser tests are included.
- Image `alt` text and semantic control labeling need improvement.
- The game does not currently expose a configurable difficulty mode.
- No deployment workflow or hosted demo URL is defined in the repository.

These are documented constraints of the current prototype, not features that are being presented as complete.

## Testing checklist

There is no automated test suite yet. The following manual smoke test covers the main behavior:

### Startup

- [ ] The page loads without JavaScript errors.
- [ ] The Canvas and background render.
- [ ] The paddle, ball, bricks, score, level, and life indicators are visible.
- [ ] Audio assets load without broken paths.

### Controls

- [ ] Holding Left Arrow moves the paddle left.
- [ ] Holding Right Arrow moves the paddle right.
- [ ] The paddle cannot leave the Canvas.
- [ ] Clicking the sound icon mutes and unmutes all audio.

### Gameplay

- [ ] The ball bounces from the side and top walls.
- [ ] Paddle impact changes the ball direction.
- [ ] A brick disappears after collision.
- [ ] Score increases by `10` per destroyed brick.
- [ ] Missing the ball decreases the life counter.
- [ ] The ball resets after a lost life.

### Progression

- [ ] Clearing all bricks increases the level.
- [ ] Each new level has one additional brick row.
- [ ] Ball speed increases after level completion.
- [ ] Clearing level `5` shows the victory overlay.
- [ ] Reaching zero lives shows the game-over overlay.
- [ ] `Play Again!` resets the game.

### Browser checks

- [ ] Test in a current Chromium-based browser.
- [ ] Test in Firefox or another supported browser.
- [ ] Test at a narrow viewport and record the current fixed-size behavior.
- [ ] Check the browser console for asset, autoplay, or CDN errors.

## Roadmap

### Gameplay

- Add pause and resume.
- Add configurable difficulty.
- Add multiple brick types, health, and power-ups.
- Add more deliberate level layouts.
- Improve ball speed handling and collision response.
- Add a start screen and a clear instructions panel.

### Controls and responsive UX

- Add touch and pointer controls.
- Add WASD and configurable key bindings.
- Scale the Canvas while preserving its aspect ratio.
- Support mobile viewport layouts.
- Add accessible focus states and descriptive labels.

### Architecture and maintainability

- Split simulation, rendering, input, audio, and progression into modules.
- Replace magic numbers with a central configuration object.
- Use explicit game states such as `ready`, `playing`, `paused`, `won`, and `lost`.
- Move level definitions into data files.
- Add a deterministic seeded mode for reproducible debugging.

### Quality and delivery

- Add unit tests for collision and level rules.
- Add browser-based smoke tests.
- Add linting and formatting checks.
- Add a static deployment workflow.
- Bundle or pin runtime dependencies.
- Add performance checks for animation stability.

### Player experience

- Add persistent high scores with an opt-in local storage setting.
- Add reduced-motion and audio preference support.
- Add visual alternatives for audio-only events.
- Add multilingual instructions.
- Add a hosted demo and screenshots to the project documentation.

## Contributing

1. Create a focused branch for the change.
2. Preserve the existing static-file structure unless the change intentionally introduces a build system.
3. Test the game manually across startup, controls, collisions, progression, and terminal states.
4. Keep asset paths working from the `brick breaking/` directory.
5. Avoid adding large binary assets without documenting their source and license.
6. Update this README when gameplay rules, controls, dependencies, or deployment behavior change.
7. Do not commit credentials, private data, or unlicensed media.

## License

No open-source license is currently specified. Until a license file is added, the original source code, artwork, audio, and documentation should be treated as all rights reserved by the author. Third-party assets and dependencies remain subject to their own licenses and terms.
