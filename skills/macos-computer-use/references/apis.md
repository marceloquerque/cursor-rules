# APIs

This file explains the main macOS APIs behind this skill.

For simple copyable patterns, read `examples.md`.

## Window Discovery

- `CGWindowListCopyWindowInfo`
  - Apple docs: https://developer.apple.com/documentation/coregraphics/cgwindowlistcopywindowinfo(_:_:)
  - Use to enumerate window ids, titles, owners, and bounds in the current GUI session.
  - If called outside a GUI security session or without a window server, it can return `NULL`.

## Window Capture

- `screencapture -l <window_id>`
  - Use the `CGWindowID` returned by `CGWindowListCopyWindowInfo`.
  - This captures a specific macOS window instead of the visible desktop.
  - Practical lesson: when a full-screen screenshot shows the wrong Space or app, window capture can still work.

## Synthetic Mouse And Keyboard Events

- `CGEvent`
  - Apple docs: https://developer.apple.com/documentation/coregraphics/cgevent
  - Use to create mouse, keyboard, and scroll events and post them into the event stream.

- `CGEvent.post(tap:)`
  - Apple docs: https://developer.apple.com/documentation/coregraphics/cgevent/post(tap:)
  - Post synthetic input events at `.cghidEventTap` for global input injection.

- `CGEvent.init(mouseEventSource:mouseType:mouseCursorPosition:mouseButton:)`
  - Available from the `CGEvent` docs above.
  - Use for pointer movement, clicks, releases, and drags.

- `CGEvent.init(keyboardEventSource:virtualKey:keyDown:)`
  - Apple docs: https://developer.apple.com/documentation/coregraphics/cgevent/init(keyboardeventsource:virtualkey:keydown:)
  - Use for shortcuts and key presses defined by virtual key code.

- `CGEvent.keyboardSetUnicodeString`
  - Apple docs: https://developer.apple.com/documentation/coregraphics/cgevent/keyboardsetunicodestring(stringlength:unicodestring:)
  - Apple notes that frameworks may ignore the attached Unicode string and translate from keycode/state instead.
  - Practical lesson: for normal text, clipboard plus `Command-V` is usually more reliable than Unicode typing.

## Synthetic Scroll

- `CGEventCreateScrollWheelEvent`
  - Apple docs: https://developer.apple.com/documentation/coregraphics/cgeventcreatescrollwheelevent
  - Apple notes that scrolling movement is generally represented by small signed integer values, typically around `-10` to `10`.
  - Practical lesson: prefer repeated small scroll steps over large jumps.

## Practical Rules From Testing

- Resolve a real `CGWindowID` before taking a screenshot of one window.
- Use `screencapture -l` instead of a full-screen capture when the app may be on another Space or monitor.
- Use screenshots to decide where to click next.
- Use `CGEvent` pointer clicks, typing, and scroll events to act like a user.
- Use paste mode for text first. Use Unicode typing as a fallback.
