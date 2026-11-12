# aur-built-packages

Rolling releases of pacman packages **built here from source**, consumed by
AUR packages maintained in [collegeming/aur-release](https://github.com/collegeming/aur-release)
(private).

Current packages:

| asset | built against | consumer |
|---|---|---|
| `mark-shot-arch-bin-<ver>-1-x86_64.pkg.tar.zst` | Arch Linux x86_64 | [mark-shot-arch-bin (AUR)](https://aur.archlinux.org/packages/mark-shot-arch-bin) |
| `mark-shot-arch-bin-<ver>-1-aarch64.pkg.tar.zst` | Arch Linux ARM aarch64 | same |

## Why this exists

Upstream [jswysnemc/mark-shot](https://github.com/jswysnemc/mark-shot) publishes
an aarch64 Linux build compiled on Ubuntu: it links FFmpeg 6
(`libavcodec.so.60`) and cannot be installed on Arch Linux ARM, where FFmpeg 9
(`libavcodec.so.63`) is current. Upstream's x86_64 build is fine (built on
Arch), but shipping one arch from upstream and one from us is inconsistent, so
both arches are compiled here against their target systems.

See upstream issue [jswysnemc/mark-shot#155](https://github.com/jswysnemc/mark-shot/issues/155).

## How it works

`.github/workflows/build.yaml` runs every 6 hours and on demand:

1. resolves the newest **stable** tag of jswysnemc/mark-shot,
2. builds it natively: x86_64 in an Arch container on an x86_64 runner,
   aarch64 in an Arch Linux ARM chroot on a native arm64 runner,
3. packs the install tree into `.pkg.tar.zst` with soname-pinned FFmpeg
   dependencies (pacman refuses the package on a library-generation mismatch),
4. publishes both assets to the rolling `aur-built-latest` release.

The AUR side (`mark-shot-arch-bin`) tracks these assets; its updater in
`aur-release` only publishes a new AUR version once artifacts for the matching
upstream version exist here.
