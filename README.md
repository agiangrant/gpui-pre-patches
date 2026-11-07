# gpui-pre-patches

Dacto's patches to the `gpui-pre` crates, pinned by revision (Dacto's ADR 0006, decision 17).

This repository holds the published sources of two crates from crates.io, unpacked from their crate tarballs so that
nothing else of Zed's tree is fetched:

| Directory         | Crate             | Version | Tarball sha256                                                     |
|-------------------|-------------------|---------|--------------------------------------------------------------------|
| `gpui-pre/`       | `gpui-pre`        | 0.3.10  | `df4b5f8d0c278308df2f11e0bfe0212dd77d7848ce1bdc4b9d513334414396f6` |
| `gpui-pre-macos/` | `gpui-pre-macos`  | 0.3.10  | `7e07a5e5c84f0b2160c194a1c852cc89d1942db66867938ceabcd7f763cb4386` |

The first commit is those sources unchanged (the checksums above are the ones crates.io and Cargo.lock record), so the
second commit's diff is the patch alone.

## The patch: text dragged in from other apps

`gpui-pre-macos` 0.3.10 registers its windows for `NSFilenamesPboardType` only, so text dragged from another
application (a selection in Notes, Safari or Mail) is refused at the window. The patch registers
`NSPasteboardTypeString` beside it and reads the string when the pasteboard holds no file names; `gpui-pre` delivers
it as `FileDropEvent::TextEntered` with an `ExternalText` as the drag's value, which elements receive with
`on_drop::<ExternalText>`. File drops are unchanged: a pasteboard with file names is still a file drop carrying
`ExternalPaths`, and every existing `on_drop::<ExternalPaths>` handler fires as before. Windows and Linux are not
patched (their platforms register text drops their own way).

## How Dacto pins it

Dacto's `client/Cargo.toml` names this repository by URL and full revision in `[patch.crates-io]` (never a path), and
`client/deny.toml` allows exactly this URL as a git source. The pinned revision is the patch commit; the patch is
dropped when a `gpui-pre` release carries the change.

## Upstream

The same change is proposed to `zed-industries/zed`: `upstream/0001-text-drops.patch` (against these published
sources) and `upstream/PR.md` (the pull request's title and body).

## License

The sources are Zed Industries' under the Apache License 2.0 (`LICENSE`, copied from the crates' `LICENSE-APACHE`);
the patch is offered under the same license.
