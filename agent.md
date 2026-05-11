# Multi-agent coordination protocol

This repository is the shared coordination channel for three agents: **A**, **B**, and **C**.

## Goal

Agents take turns incrementing the integer stored in `number.txt` and committing that change to the repository.

Turn order is strict and repeats forever:

**A -> B -> C -> A -> ...**

## Coordination rules

1. **Use the git repository as the source of truth.**
   - Coordinate only through committed files, commit history, pull/rebase, and push.
   - Do not rely on private memory or out-of-band coordination.

2. **Before starting any agentic work loop, sync first.**
   - Run `git fetch origin`
   - Check out the working branch
   - Rebase or pull the latest changes from the remote branch before making any decision

3. **Determine whose turn it is from repository state.**
   - If there is no prior coordination commit and `number.txt` does not exist yet, it is **Agent A's** turn to start.
   - Otherwise, inspect the latest successful increment commit and follow the rotation:
     - after **A**, it is **B**'s turn
     - after **B**, it is **C**'s turn
     - after **C**, it is **A**'s turn

4. **If it is not your turn, do not modify `number.txt`.**
   - Fetch latest changes again later and wait for your turn.

5. **If it is your turn, make exactly one increment.**
   - Read the current integer in `number.txt`
   - Increment it by exactly `1`
   - Commit only that increment

6. **Use clear commit messages.**
   Recommended format:
   - `agent A: increment number.txt to 1`
   - `agent B: increment number.txt to 2`
   - `agent C: increment number.txt to 3`

7. **Push after committing.**
   - If push is rejected, fetch/rebase, re-check whose turn it is, and only proceed if it is still your turn.

8. **Do not skip turns, batch increments, or overwrite another agent's work.**
   - One turn
   - One increment
   - One commit

## Expected behavior by agent

### Agent A
- Starts the sequence if the repository has no prior increment history
- Then acts after Agent C

### Agent B
- Acts only after Agent A

### Agent C
- Acts only after Agent B

## Safety checks before committing

Before each commit, confirm all of the following:
- local branch is synced with remote
- `number.txt` reflects the latest committed value
- it is your turn in the A/B/C rotation
- your change is exactly `+1`

## Conflict resolution

If multiple agents race:
- fetch latest remote state
- rebase
- inspect the newest increment commit
- if another agent already took the turn, stand down and wait for your next turn

This file defines the required coordination behavior for all agents working in this repository.
