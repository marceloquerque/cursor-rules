---
name: macos-computer-use
description: 'Control macOS apps by taking real window screenshots and sending mouse, keyboard, and scroll input like a user. Use when a user wants you to click buttons, type text, scroll, drag, or inspect a macOS app window and there is no built-in computer-use mouse tool. This skill is macOS-only and is useful for apps on another monitor or Space, for windows that do not show up in a normal screen capture, and for workflows that need screenshot-guided input.'
---

# macOS Computer Use

Use this skill when you need to control a macOS app by looking at screenshots and sending real input events like a user.

## Before You Start

- If something does not work, ask the user to check the **main app** that is running the agent.
- Do not guess that the main app is a terminal.
- The main app might be Cursor, an IDE, a plugin host, a desktop app, or something else.
- That main app needs **Accessibility** permission for mouse moves, clicks, typing, and scrolling.
- That same app may also need **Screen & System Audio Recording** permission for screenshots.

Simple rule:

- If clicking or typing fails, ask the user to check **Accessibility** for the main app.
- If screenshots fail, ask the user to check **Screen & System Audio Recording** for the main app.

## Main Idea

Do not trust a normal full-screen screenshot first.

Why:

- A full-screen screenshot only shows what is visible right now.
- The app you need might be on another monitor or another Space.
- A real **window screenshot** can still work even when the normal screen screenshot is misleading.

## Fast Workflow

1. Find the right window id.
   - See `references/examples.md`.
2. Bring that window to the front.
   - Do this before any click, typing, or scroll.
3. Take a screenshot of that exact window.
   - See `references/examples.md`.
4. Find the target spot.
   - Use the window screenshot.
   - Think like a user: look at the window, find the thing, click it.
5. Send the input.
   - Click, type, scroll, or drag.
6. Check what changed.
   - Take another window screenshot or read app state again.

## What To Read

- Read `references/basics.md` first for the simple rules.
- Read `references/examples.md` for copyable command examples.
- Read `references/apis.md` if you need the Apple API background.

## Important Rules

- Prefer a real window screenshot over a full-display screenshot.
- Bring the target window to the front before every click, type, or scroll.
- Use small steps when scrolling.
- For text, paste is usually safer than fake letter-by-letter typing.
- Verify each important step with another screenshot.
- Act like a user. Do not inspect app internals unless the user explicitly asks for that.

## Common Problems

- If `screencapture` says it cannot create an image from the display, ask the user to check screenshot permission for the main app.
- If a click or key press does nothing, ask the user to check Accessibility permission for the main app.
- If the screen screenshot shows the wrong thing, switch to a window-specific screenshot.
- If the target window is not frontmost, bring it to the front and then try again.
- If one click is risky, take another screenshot, then click the next thing.
