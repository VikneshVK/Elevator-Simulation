# 2D Elevator Simulation — Unity Engine 6

## Overview

A 3-elevator, 4-floor simulation where players can call elevators from floor wall panels and select destination floors from inside each elevator. The system dispatches elevators intelligently based on their current state and direction of travel.

---

## Scripts

**Elevator.cs** — Handles movement, request queuing, and UI for a single elevator.

**ElevatorManager.cs** — Singleton dispatcher. Scores all elevators and assigns the best one to each floor call. Tracks active assignments to prevent two elevators responding to the same request.

**FloorButton.cs** — Wall panel call button on each floor. Requests an elevator via ElevatorManager.

**ElevatorControlButton.cs** — Interior panel button inside each elevator. Sends floor requests directly to its assigned elevator.

**SceneManager.cs** — Singleton that wraps Unity's scene loading. Persists across scenes via DontDestroyOnLoad.

**ButtonClickUI.cs** — Attached to UI buttons on the title screen. Calls SceneManager to load a named scene on click.

---

## How Elevators Are Dispatched

When a floor button is pressed, every elevator is given a score. The lowest score wins.

- Idle, already at the floor → 0
- Idle, elsewhere in the shaft → distance × 0.25
- Moving toward the request → distance × 0.5
- Moving away from the request → distance + 20
- At the floor but already leaving → 50

---

## Elevator Movement — SCAN/LOOK Algorithm

The elevator serves all floors in its current direction before reversing — the same way real elevators work. It only travels as far as the last request in that direction, then reverses if there are requests behind it, or goes idle if there are none.

---

## Key Design Decisions

- **Single state flag** — `ElevatorState` (Idle/Moving) is the only flag controlling coroutine logic. An earlier `isMoving` boolean caused race conditions when both fell out of sync, so it was removed.
- **Register before request** — Buttons register themselves with the elevator before `AddRequest()` is called, so they are always reset correctly even if the elevator arrives almost immediately.
- **`Start` vs `Awake` for dependencies** — `ElevatorManager.Instance` is checked in `Start()` because all `Awake()` calls run simultaneously and the manager may not have set its Instance yet.

---

## Inspector Setup

- **ElevatorManager** — Add all 3 elevators to the `elevators` list.
- **Elevator** — Assign `floorPoints[]` in order (0 = Ground), set `startFloor`, assign TMP text fields.
- **FloorButton** — Set `floor` index and `direction`, assign `buttonImage`, wire `OnClick` to `OnPress()`.
- **ElevatorControlButton** — Assign `targetElevator` and `targetFloor`, assign `buttonImage`, wire `OnClick` to `OnPress()`.
