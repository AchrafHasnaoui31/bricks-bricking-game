# Brick Breaker Game

Browser-based brick-breaker game built with HTML, CSS, JavaScript, and jQuery. The project is a focused exercise in interactive UI, game-state management, collision feedback, and audio integration.

## Features

- Paddle, ball, and brick gameplay loop.
- Score, lives, win, and game-over states.
- Level and loading visuals.
- Sound effects for collisions, life loss, and victory.
- Toggleable sound assets and themed visual resources.

## Run locally

```bash
cd "brick breaking"
python3 -m http.server 8000
```

Open `http://localhost:8000/`.

## Problems solved

- Translates game rules into browser events and state transitions.
- Uses visual and audio feedback to make state changes understandable.
- Keeps the project dependency-light and easy to run.

## Roadmap

- Add keyboard/touch controls and responsive sizing.
- Separate game state from rendering for easier testing.
- Add deterministic collision tests and configurable level data.
- Add accessibility alternatives for audio-only feedback.

## Status

Small playable frontend project focused on JavaScript fundamentals.