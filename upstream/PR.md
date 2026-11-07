# gpui: accept text dragged in from other applications on macOS

## Title

gpui: Accept text dragged in from other applications on macOS

## Body

On macOS, GPUI windows register for `NSFilenamesPboardType` only, so a selection dragged in from another
application (a paragraph from Notes, a link from Safari, a line from Mail) is refused at the window: the cursor shows
no drop and no element can receive it. Applications built on GPUI that embed a text editor cannot accept the drop a
native text view accepts.

This change:

- registers `NSPasteboardTypeString` beside `NSFilenamesPboardType` on every window (`dragged_types` in
  `gpui_macos::window`);
- when a drag enters, reads its pasteboard as file names first, so a pasteboard holding both (Finder's, and GPUI's own
  file copy) is still a file drop carrying `ExternalPaths`, else its non-empty plain text;
- delivers the text as a new `FileDropEvent::TextEntered { position, text }`, whose drag value is a new
  `ExternalText(String)`; elements receive it with `on_drop::<ExternalText>` exactly as files are received with
  `on_drop::<ExternalPaths>`, and the rest of the drag is the existing `Pending`, `Submit`, `Exited` and `Ended`
  events.

Why a type of its own rather than an enum around the paths: the drag model dispatches `on_drop` by the dragged value's
type, so wrapping `ExternalPaths` in an enum would stop every existing `on_drop::<ExternalPaths>` handler.
`FileDropEvent::Entered` keeps its fields, so the Linux and Windows platforms compile unchanged and keep their file
drops; their text drops follow their own registration and are left for later.

Release Notes:

- Text dragged from other applications can now be dropped into GPUI windows on macOS.

### Tests

`gpui_macos` window tests: the registered types are file names and text; a text-only pasteboard reads as a text drop;
file names beside text read as the files; a missing pasteboard, an empty one and empty text read as no drop. A
downstream application (Dacto's desktop client) tests the delivery through `Window::dispatch_event` with
`on_drop::<ExternalText>` and `on_drop::<ExternalPaths>` side by side.

### Notes for the reviewer

The diff in `0001-text-drops.patch` is against the published `gpui-pre` and `gpui-pre-macos` 0.3.10 crates
(`gpui-pre/` is `crates/gpui`, `gpui-pre-macos/` is `crates/gpui_macos`).
