---
title: Jet Fighter Simulator
description: Writing a more complex game with Zed and Mistral
pubDatetime: 2026-11-25T11:16:04.931Z
preview: ../../assets/images/jet-fighter-simulator-1.png
draft: false
tags:
  - AI
  - Mistral
  - Zed
categories: []
author: Code by Kolev
featured: false
ogImage: ../../assets/images/jet-fighter-simulator-1.png
---

## Table of contents

## The Story

After vibe-coding a small game in the [previous post](/posts/2026-10-31-jewel-shuffle), it was time to try something more ambitious — a flight simulator. We're still following the book ["Process Over Magic: Beyond Vibe Coding"](https://pragprog.com/titles/ubaidev/process-over-magic-beyond-vibe-coding/).

## Vibe Coding a Flight Simulator Game

### The First Prompt

> Please write the code for a flight simulator of jet fighter on a mountain ground. The goal of the game is to reach the goal avoiding the radar staying as low as possible. Use pygame and any library you think can help. Use the folder 'jet-fighter-simulator'

The agent got to work and came back with a plan: a 3D-ish jet over procedurally generated mountain ridges, radar stations with rotating sweeps that terrain can block, missiles when detection maxes out, fuel and afterburner, a HUD with a radar scope and minimap — plus procedurally synthesized sound effects on top. Quite a bit more than the prompt asked for.

That first burst of writing, testing and fixing on repeat lasted about 20 minutes. Guiding it back towards 2D, dropping the audio, and pointing it at where things were actually broken took another 2-3 hours on top.

> [!NOTE] Time Estimate: 2-3 hours

### Back and Forth

The initial game was overly ambitious, and things didn't work. The game crashed on launch — missing imports, broken files and so on — even though the tests were passing. Only after a few rounds of pointing at the concrete failures did it finally run. The tests told a rosier story than reality: about 80% of them passed, and the gameplay tests flew until some distance before stalling, never finishing the map. But when I actually tried to play it, the jet didn't always obey the direction of the keypresses, and it didn't work nearly as well as the tests suggested.

The final build ended up as a proper little package: terrain generation, flight physics, terrain-masked radar with missiles, HUD, and menu/win/game-over screens, plus a headless test suite that mostly passed. Bot pilots across 8 seeds flew 24-43% of the map before dying to missiles or terrain. On paper the mechanics all worked — in practice the flight model still felt off, and since the environment is headless, neither the visual feel nor the audio could be verified on a real display. That's worth a `python3 main.py` on your own machine.

### Some Screenshots

![jet-fighter-simulator-1](@/assets/images/jet-fighter-simulator-1.png)
![jet-fighter-simulator-2](@/assets/images/jet-fighter-simulator-2.png)
![jet-fighter-simulator-3](@/assets/images/jet-fighter-simulator-3.png)

### What Went Wrong

Complexity is where vibe coding starts to crack. What actually happened here:

- **Hallucinations** — missing imports and confidently marching forward. The smoke tests passed while the game itself crashed on launch.
- **The drunken intern** — cheerful and eager to please, but assumes completed code is working code. "Everything is in place and validated" was not, in fact, validated on a real display.
- **The loop of death** — fix one thing, break another. Each "just fix it" round shuffled the breakage around until I pointed at the actual symptom instead.
- **Losing context** — things already written can fall outside the limited context window and get forgotten.
- **Random results** — the same prompt produced different games on different runs.
- **Low-quality code underneath** — tight coupling and duplicated logic made every fix riskier than the last.
- **Hidden costs** — every detour burned tokens and context, and with them, money.

> [!TIP] When the loop of death kicks in, stop asking "fix it" and point the agent at the concrete symptom — like the Actions log in the [previous post](/posts/2026-10-31-jewel-shuffle). Once we could see where things stalled, it fixed the issue in no time.

## Spec-Driven Development

To address challenges like these, spec-driven development starts from a written spec — goals, tech stack, coding practices — that the agent must follow, instead of letting the agent improvise. That approach deserves its own post, so I'll dig into it properly soon.

## Conclusion

- Vibe coding delivers impressive results fast — a whole flight simulator from a single prompt.
- But on complex tasks it fails in interesting ways, and keeps adding features even when the game isn't working.
- Without a process, trusting the AI to lead leads to unpredictable code and token burn.

The code is available at [https://github.com/codebykolev/jet-fighter-simulator](https://github.com/codebykolev/jet-fighter-simulator)
