# Prompts Used to Create This Game

A record of all user prompts from the session that designed and built the Rubik's Cube × Tic-Tac-Toe game.

---

## 1. Initial game design prompt

> Plan only, do not execute
>
> Create a playable browser game that combines a 3x3 Rubik's Cube with Tic Tac Toe. The cube should look like a Rubik's Cube with a black body, but all white stickers. The cube must be fully rotatable by dragging so the player can inspect all sides.
>
> Rules:
> Player 1 uses X
> Player 2 uses O
>
> On each turn, the active player first places their mark on any empty sticker on the cube. After placing their mark, that player must make exactly one standard move of the cube.
>
> Players continue alternating turns.
>
> A player wins by making 3 of their marks in a row on a single 3x3 face.
>
> Victory is checked only after the rotation is completed, not immediately after placing the mark.

---

## 2. Clarification answers (during planning)

When asked about scope, the following choices were made:

- **Opponent:** Both — start menu lets you pick 2-player hotseat or vs Computer
- **Move input:** Both — on-screen buttons (U U' D D' R R' L L' F F' B B') and drag-to-twist a layer
- **Delivery:** Single self-contained HTML file using Three.js from a CDN

---

## 3. Build the game

> now build the game as per the proposed plan

---

## 4. Visual cue for rotations

> I cant tell when I pick a rotation, which one and which way it will turn. Can you add a visual cue for me

---

## 5. Rename the file

> Can you rename index.html to 3D-tic-tac-toe-cube-game

---

## 6. Organise into a folder

> Can you move all the details of the session and artifact to 3D_Tic_Tac_Toe_Game folder inside Claude Code Code

---

## 7. Generate CLAUDE.md

> /init

---

## 8. Session documentation

> How should I document what we did in this session so that I can come back to the same point when I am starting from another session

---

## 9. Commit and push to GitHub

> now commit my code in git
>
> git config --global user.name "Vinay Roy"
> git config --global user.email "vroy@berkeley.edu"
