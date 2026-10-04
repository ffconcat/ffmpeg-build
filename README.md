# ffmpeg-build

Automated builds of a **stripped-down FFmpeg for Windows**, released automatically whenever
[upstream FFmpeg](https://ffmpeg.org/download.html) publishes a new version.

The build is not a general-purpose FFmpeg. It is compiled with `--disable-everything` and only
the pieces needed to **concatenate existing media files without re-encoding** are enabled, which
keeps the resulting `ffmpeg.exe` far smaller than a normal full build.

Download the latest build from the
[Releases page](https://github.com/ffconcat/ffmpeg-build/releases/latest).

### Enabled components

| Component         | Enabled                                              |
| ----------------- | ---------------------------------------------------- |
| Demuxers          | `concat`, `mov`                                      |
| Muxer             | `mp4`                                                |
| Protocol          | `file`                                               |
| Bitstream filters | `h264_mp4toannexb`, `hevc_mp4toannexb`               |
| Architecture      | `--enable-small`, `--disable-debug`, `--disable-doc` |

Everything else is compiled out: all decoders, encoders, filters, `avdevice`, `swscale`,
`swresample`, and all network protocols.

## Usage

The intended workflow is a stream copy through the **concat demuxer**, which joins files by
concatenating their streams — no re-encoding, so it is fast and lossless.

Create a list file describing the inputs, in the concat demuxer format:

```text
# list.txt
file 'part1.mp4'
file 'part2.mp4'
```

Then remux them into a single file:

```console
> ffmpeg -f concat -safe 0 -i list.txt -c copy output.mp4
```

Notes:

- `-safe 0` is required if the paths in your list file are absolute or contain characters the
  concat demuxer considers unsafe.
- Paths in the list file use forward or backslashes, but quoting with single quotes is
  recommended when paths contain spaces.
- Only `file` protocol is compiled in. You cannot read from stdin, from a URL, or from a network
  stream — inputs must be local files.

## Limitations

Because the build is intentionally minimal, expect these to be missing or unsupported:

- **No encoding or decoding.** Transcoding, resizing, filtering, and audio processing are not
  possible — the binary can only remux.
- **No filters, no `-vf`/`-af`.** `swscale` and `swresample` are disabled, so there is no scaling
  or sample-format conversion either.
- **No network access.** Only the `file` protocol is enabled.
- **Narrow container support.** Only MP4/MOV input and MP4 output. Other input and output
  containers are unavailable.
- **x86_64 Windows only.** There are no 32-bit or ARM64 builds.

## License

The build scripts and workflow in this repository are released under the [MIT License](LICENSE).

This project only automates the compilation of FFmpeg. FFmpeg itself is licensed separately under
the [LGPL v2.1 or later](https://ffmpeg.org/legal.html); the terms of that license apply to the
binaries produced by this repository.
