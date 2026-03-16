# Examples

These are example commands and short code blocks.

Use them as patterns. Adjust the app name, window id, text, and coordinates.

## 1. Find A Window

List visible windows:

```bash
swift -e '
import CoreGraphics
if let rows = CGWindowListCopyWindowInfo([.optionOnScreenOnly, .excludeDesktopElements], kCGNullWindowID) as? [[String: Any]] {
  for row in rows {
    let owner = row[kCGWindowOwnerName as String] as? String ?? ""
    let id = row[kCGWindowNumber as String] as? Int ?? -1
    let name = row[kCGWindowName as String] as? String ?? ""
    print("id=\(id) owner=\(owner) name=\(name)")
  }
}
'
```

List all windows, even if they are not visible right now:

```bash
swift -e '
import CoreGraphics
if let rows = CGWindowListCopyWindowInfo([.optionAll], kCGNullWindowID) as? [[String: Any]] {
  for row in rows {
    let owner = row[kCGWindowOwnerName as String] as? String ?? ""
    let id = row[kCGWindowNumber as String] as? Int ?? -1
    let name = row[kCGWindowName as String] as? String ?? ""
    let on = row[kCGWindowIsOnscreen as String] as? Int ?? 0
    print("id=\(id) onscreen=\(on) owner=\(owner) name=\(name)")
  }
}
'
```

## 2. Capture One Window

```bash
screencapture -x -l 6106 /tmp/window.png
```

Use the window id you found in step 1.

## 3. Bring The App To The Front

```bash
osascript -e 'tell application "Slack" to activate'
```

Do this before any click, typing, or scroll.

## 4. Click A Point

```bash
swift -e '
import ApplicationServices
let p = CGPoint(x: 2464.5, y: 449)
let move = CGEvent(mouseEventSource: nil, mouseType: .mouseMoved, mouseCursorPosition: p, mouseButton: .left)!
let down = CGEvent(mouseEventSource: nil, mouseType: .leftMouseDown, mouseCursorPosition: p, mouseButton: .left)!
let up = CGEvent(mouseEventSource: nil, mouseType: .leftMouseUp, mouseCursorPosition: p, mouseButton: .left)!
move.post(tap: .cghidEventTap)
usleep(120000)
down.post(tap: .cghidEventTap)
usleep(50000)
up.post(tap: .cghidEventTap)
'
```

## 5. Type Text

Paste is usually safer:

```bash
osascript -e 'set the clipboard to "hello from codex"'
swift -e '
import ApplicationServices
let down = CGEvent(keyboardEventSource: nil, virtualKey: 9, keyDown: true)!
let up = CGEvent(keyboardEventSource: nil, virtualKey: 9, keyDown: false)!
down.flags = .maskCommand
up.flags = .maskCommand
down.post(tap: .cghidEventTap)
usleep(40000)
up.post(tap: .cghidEventTap)
'
```

Unicode text typing example:

```bash
swift -e '
import ApplicationServices
for unit in "hello".utf16 {
  var value = unit
  let down = CGEvent(keyboardEventSource: nil, virtualKey: 0, keyDown: true)!
  let up = CGEvent(keyboardEventSource: nil, virtualKey: 0, keyDown: false)!
  down.keyboardSetUnicodeString(stringLength: 1, unicodeString: &value)
  up.keyboardSetUnicodeString(stringLength: 1, unicodeString: &value)
  down.post(tap: .cghidEventTap)
  usleep(15000)
  up.post(tap: .cghidEventTap)
  usleep(15000)
}
'
```

## 6. Scroll

```bash
swift -e '
import ApplicationServices
let event = CGEvent(scrollWheelEvent2Source: nil, units: .line, wheelCount: 2, wheel1: -4, wheel2: 0, wheel3: 0)!
event.post(tap: .cghidEventTap)
'
```

Use small numbers like `-4`, `-3`, `4`, or `5`.

## 7. Drag

```bash
swift -e '
import ApplicationServices
let start = CGPoint(x: 900, y: 600)
let end = CGPoint(x: 1200, y: 600)
let down = CGEvent(mouseEventSource: nil, mouseType: .leftMouseDown, mouseCursorPosition: start, mouseButton: .left)!
down.post(tap: .cghidEventTap)
for step in 1...20 {
  let t = Double(step) / 20.0
  let p = CGPoint(x: start.x + (end.x - start.x) * t, y: start.y + (end.y - start.y) * t)
  let drag = CGEvent(mouseEventSource: nil, mouseType: .leftMouseDragged, mouseCursorPosition: p, mouseButton: .left)!
  drag.post(tap: .cghidEventTap)
  usleep(12000)
}
let up = CGEvent(mouseEventSource: nil, mouseType: .leftMouseUp, mouseCursorPosition: end, mouseButton: .left)!
up.post(tap: .cghidEventTap)
'
```

## 8. Verify After Each Step

Good pattern:

1. take a screenshot of the target window
2. make sure that window is frontmost
3. click one thing
4. take another screenshot
5. make sure the app changed the way you expected
6. only then do the next step

For example in Slack:

1. capture the Slack window
2. bring Slack to the front
3. click the channel
4. capture the window again
5. make sure the channel header changed
6. click the message box
7. paste the message
8. press Enter
9. capture the window again
10. make sure the new message is visible
