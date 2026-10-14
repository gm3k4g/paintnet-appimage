# Paint.NET 5.2 for Linux — AppImage

A single-file [AppImage](https://appimage.org/) of **Paint.NET 5.2** running
under Wine, with a fix for the black-rendering problem that affects Paint.NET's
Direct2D surfaces on Wine + DXVK. The image bundles everything the app needs —
Wine, DXVK, and a preconfigured Wine prefix — so it runs on a plain x86_64
glibc Linux distribution with no installation and no Nix.

> **Releases contain the AppImage binary. This repository intentionally
> contains no build artifacts, no third-party code, and no application
> binaries — only this document.**

## About Paint.NET

[Paint.NET](https://getpaint.net) is **closed-source freeware** for Windows,
© dotPDN LLC / Rick Brewster. It is *not* open source, and nothing of it is
reproduced in this repository. The application files inside the release image
are the unmodified output of the official distribution; the only change made
to them at packaging time is one entry in a JSON manifest registering a helper
assembly (see *The rendering fix* below).

The Wine packaging this image is based on comes from the community project
[Paint.NET-on-Wine](https://github.com/bluesillybeard/Paint.NETOnWine).

## What is in the image

| Component | Source | License |
|---|---|---|
| Paint.NET 5.2 (β 5.200.9775.2252) | getpaint.net / Paint.NET-on-Wine | freeware, closed source |
| Wine 11.19 (staging) | WineHQ AppImage | LGPL 2.1+ |
| DXVK 3.1.1 | [doitsujin/dxvk](https://github.com/doitsujin/dxvk) | zlib |
| Vulkan loader 1.4.357 | KhronosGroup/Vulkan-Loader | Apache-2.0 / MIT |
| `D2DSwapChainFix` hook | this project (≈5 KB, no third-party code) | MIT |

## The rendering fix

Paint.NET draws its canvas and panels through Direct2D, presenting via
flip-model DXGI swapchains. Under Wine + DXVK those frames are composed but
arrive black. At startup, a tiny .NET *startup hook* sets three internal
flags (`AllowSwapChains`, `AllowFlipSwapChains`, `EnableSwapChainAsyncPresent`)
so every Direct2D control uses the GDI device-context presentation path,
which renders correctly. No Paint.NET file is modified on disk — the hook is
a separate assembly that adjusts runtime state, and Paint.NET's own integrity
check still runs.

Practically: the canvas, tool strip, layers/history/color panels all render
correctly, at the cost of GPU-accelerated presentation (painting is CPU/GDI).
A full write-up of the investigation lives in the (private) `paintnet-build`
repository.

## Install and run

Download `PaintNET.AppImage` from [Releases](../../releases), then:

```bash
chmod +x PaintNET.AppImage
./PaintNET.AppImage
```

If your distribution lacks FUSE2 (e.g. NixOS), run it self-extracting:

```bash
APPIMAGE_EXTRACT_AND_RUN=1 ./PaintNET.AppImage
```

**First run** copies the bundled Wine prefix (~744 MB) to
`~/.local/share/paintnet/prefix`; later runs reuse it. Deleting that directory
resets the application to a fresh state. The app's Windows-side profile
(including any installed plugins, once persistent-mode packaging lands) lives
inside that prefix.

## Requirements

- Linux x86_64, glibc
- Working Vulkan drivers (DXVK needs them)
- ~1.2 GB free in `$HOME` for the copied prefix
- FUSE2, or use `APPIMAGE_EXTRACT_AND_RUN=1`

## Known limitations

- Presentation goes through GDI rather than the GPU, so very large canvases
  are slower than native.
- First launch is slower (prefix copy +, on FUSE-less systems, self-extraction).
- Built for the 5.200.9775.2252 beta; internal flags may change in future
  Paint.NET versions, which would need a hook update.

## License

The content of this repository (documentation) is provided under the
[MIT license](https://opensource.org/licenses/MIT). Paint.NET remains
© dotPDN LLC and is distributed under its own license; Wine, DXVK and the
Vulkan loader remain under their respective licenses. This project is not
affiliated with or endorsed by dotPDN LLC.
