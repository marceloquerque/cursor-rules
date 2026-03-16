# Basics

This file explains the skill in very simple words.

## What This Skill Does

It helps you control a macOS app by:

- finding the app window
- taking a picture of that exact window
- finding the spot you want
- sending a click, key press, type action, or scroll

## Permissions

If something breaks, do not guess which app needs permission.

Ask the user to check the **main app** that is running the agent.

Examples:

- Cursor
- an IDE
- a plugin host
- a desktop app
- a terminal app

That app needs:

- **Accessibility** for clicks, typing, key presses, and scrolling
- **Screen & System Audio Recording** for screenshots

## Best Order

1. Find the window id.
2. Bring that window to the front.
3. Take a screenshot of that exact window.
4. Find the target point.
5. Send the input.
6. Check the result.

## Why Window Screenshots Matter

A normal screenshot can lie to you.

The app window may be:

- on another monitor
- on another Space
- behind another app

But a window screenshot can still work if you have the right window id.

## Simple Tips

- Use window screenshots first.
- Bring the target window to the front before each click, type, or scroll.
- Use small scroll steps.
- Use paste for text when you can.
- Verify with another screenshot after each important step.
- Act like a user: look, click, type, scroll, check.
