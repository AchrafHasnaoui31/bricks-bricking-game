# Brick Breaker Game

> Browser-based brick-breaker game built with HTML, CSS, JavaScript, and jQuery as a focused exercise in interactive UI, game-state management, collision feedback, and audio integration.

## Features

- Paddle, ball, and brick gameplay loop.
- Score, lives, win, and game-over states.
- Loading, level, and end-state visuals.
- Sound effects for collisions, life loss, and victory.
- Toggleable sound assets and themed visual resources.
- Static hosting with no build system required.

## Technical model

```text
Browser events -> game state -> collision rules -> rendering and audio feedback
```

The project is intentionally small: it makes the core loop visible and easy to modify. A future refactor should separate game state, collision detection, rendering, and audio so each concern can be tested independently.

## Run locally

```bash
git clone https://github.com/AchrafHasnaoui31/bricks-bricking-game.git
cd bricks-bricking-game/brick\ breaking
python3 -m http.server 8000
```

Open `http://localhost:8000/`.

## Problems solved

- Translates game rules into browser events and state transitions.
- Uses visual and audio feedback to communicate gameplay state.
- Keeps dependencies and setup minimal.

## Roadmap

- Add keyboard and touch controls with responsive sizing.
- Separate state from rendering for deterministic tests.
- Add configurable level data and collision tests.
- Add pause/resume and persisted high scores.
- Add accessibility alternatives for audio-only feedback.

## Status

Small playable frontend project focused on JavaScript fundamentals.

## License

No open-source license is currently specified.