# Wallpaper memory patch kit

This branch publishes the KISA wallpaper memory repairs separately from desktop
configuration. It builds on `fix/media-dbus-polling` at
`7f94c7739b83790c384ded3c03074763495eeb3a`; the engine's asynchronous MPRIS code
is already included. The patches below modify mpv, not the engine source tree.

| Component | Repair |
| --- | --- |
| Engine MPRIS/D-Bus | Throttle polling, release error allocations, bound dispatch and pending-request lifetime, recover stalled players and authenticate signal owners |
| `mpv-opengl-fence.patch` | Avoid retaining swapchain fences when libmpv's render API has no swap-buffers callback |
| `mpv-nvdec-lazy-pool.patch` | Prefer NVDEC's supported device-context path so FFmpeg avoids preallocating an unused CUDA frame pool |

## Apply to pinned mpv

Use mpv revision `41f6a645068483470267271e1d09966ca3b9f413` (0.41.0).
From that mpv checkout, with this repository available as `$ENGINE_SOURCE`:

```bash
git apply "$ENGINE_SOURCE/contrib/wallpaper-memory/mpv-opengl-fence.patch"
git apply "$ENGINE_SOURCE/contrib/wallpaper-memory/mpv-nvdec-lazy-pool.patch"
```

The OpenGL fix was also submitted upstream as
[mpv-player/mpv#17303](https://github.com/mpv-player/mpv/pull/17303).
These patches target the pinned revision; inspect newer sources before applying
them. The NVDEC change only affects the hardware-decoding path that supports
`AV_CODEC_HW_CONFIG_METHOD_HW_DEVICE_CTX`.

For the complete pinned private-library build and installation recipe, use
[KISA Stack's wallpaper guide](https://github.com/howdeploy/kisa-stack/blob/main/dotfiles/docs/wallpaper-engine.md).
Its builder includes both mpv patches and the engine repair. The helper loads
the private runtime only for wallpapers, leaving other applications' system mpv
unchanged. No Steam assets or compiled libraries are distributed here.

## Evidence and limits

The October 28 local synchronous-engine/OpenGL repair recorded 180 retained
fences after 180 frames before the fix, and zero after 180 and 1,800 frames with
the fix. A ten-minute observation of two 4K wallpaper processes ended at about
456 and 513 MiB RSS, with anonymous-memory growth of about 1.7 and 2.1 MiB.
Those observations involved multiple fixes and do not isolate the D-Bus change.

The later asynchronous engine revision, the November 2 NVDEC patch and the
combined public build recipe are not validated by those older measurements.
No new build, tests, playback or memory measurements were run for this source
publication. The reported near-exhaustion of system RAM is the author's incident,
not a measured before/after percentage or a guaranteed memory ceiling.

Retain upstream notices: both modified mpv files are LGPL-2.1-or-later; complete
mpv builds may include GPL components. The engine retains its own GPL license.
