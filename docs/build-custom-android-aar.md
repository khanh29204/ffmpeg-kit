# Build ffmpeg-kit Android AAR (custom GPL, minimal)

## Lệnh build

```bash
FFMPEG_KIT_TRIM=1 ./android.sh \
  --enable-gpl --enable-x264 --enable-lame --enable-libwebp \
  --enable-android-zlib --disable-arm-v7a --disable-x86
```

- ABI: armeabi-v7a (neon), arm64-v8a, x86_64. `--disable-arm-v7a` = chỉ build bản neon (mọi thiết bị Android 7+ có NEON, neon nhanh hơn, cùng thư mục `jni/armeabi-v7a` trong AAR).
- `--enable-android-zlib`: **bắt buộc** — không có flag này ffmpeg build với `--disable-zlib` → encoder PNG chết.
- `FFMPEG_KIT_TRIM=1`: whitelist ffmpeg core (chỉ giữ h264/aac/mp3/mjpeg/png/webp + mp4/mp3/image2 + filter crop/transpose/hflip/vflip/scale). Không set → build full codec, nặng hơn nhiều. Xem block trong `scripts/android/ffmpeg.sh`.
- License: **GPLv3.0** (do dùng x264 với `--enable-gpl`) — phải note trong app.

## Prerequisites — Linux local (Arch)

```bash
sudo pacman -S --needed autoconf automake libtool pkg-config make cmake ninja \
  gperf texi2html nasm jdk17-openjdk
```

- Android SDK + NDK **r27+** (r27 bắt buộc cho 16KB page-size; repo này dùng `27.1.12297006`).
  NDK được resolve qua env `ANDROID_NDK_ROOT`, mặc định `$ANDROID_SDK_ROOT/ndk/27.1.12297006`.
- JDK 17.

## Prerequisites — GitHub Actions

Workflow: `.github/workflows/custom-android-aar.yml` (chỉ chạy `workflow_dispatch`).
Bước `prerequisites` đã cài: `gperf texi2html nasm` (runner không có sẵn — thiếu cái nào chết đúng bước đó):

| Thiếu | Lỗi lúc build |
|---|---|
| gperf | `gperf: not found` khi reconf libiconv |
| texi2html/groff | lỗi target `man/iconv.1.html` (Makefile.devel) — đã patch no-op trong `scripts/android/libiconv.sh` |
| nasm | `(*) nasm command not found` khi build x264 |

## Sau build

- Output: `prebuilt/android/ffmpeg-kit-custom-gpl-*.aar` (workflow upload artifact `ffmpeg-kit-custom-gpl`).
- Verify 16KB alignment từng `.so`: `objdump -p <file>.so | grep LOAD` → cột align phải ≥ `2**14`. Workflow tự chạy bước này.
- Xác nhận lib enable: grep `--enable-libx264\|--enable-libmp3lame\|--enable-libwebp` trong `build.log` / `src/ffmpeg/ffbuild/config.log`.
