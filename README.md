# renkuos-msp

**Not a fork, not a distribution – a test bench.**
RenkuOS nightly plus a series of patches, built and used daily on a Lenovo ThinkPad T470s.
Most patches are open pull requests to [RenkuOS/Source](https://github.com/RenkuOS/Source); the
goal is that every one of them goes upstream and this repository becomes empty again.

If you have an open RenkuOS PR that touches hardware a T470s has (Intel 7260 Wi-Fi, Intel HD 620,
xHCI/USB, UVC webcams, HDA audio, ACPI/EC, Synaptics/TrackPoint), send me a note: I am happy to
add it here, test it on real hardware and report back **in your PR**.

## Current series: `build00096-msp4`

Base: nightly build00096, commit `fb702668d7ed759521faa42f06c701fbcd77f6f8` (`hrev60072+1180`).
Patches: [`patches/build00096-msp4/`](patches/build00096-msp4), apply in order with `git am`.
The older series `build00096-msp2` stays for reference.

| # | Patch | Upstream |
|---|---|---|
| 1 | hid: take mouse buttons from all reports of the collection | [#529](https://github.com/RenkuOS/Source/pull/529) |
| 2 | xhci: fix high speed isochronous endpoints with extra transactions | [#534](https://github.com/RenkuOS/Source/pull/534) |
| 3 | usb_raw: let transfers of one device run at the same time | [#535](https://github.com/RenkuOS/Source/pull/535) |
| 4 | usb: end transfers before switching an alternate setting | [#548](https://github.com/RenkuOS/Source/pull/548) |
| 5 | usb_webcam: assemble UVC frames from checked payload headers | [#549](https://github.com/RenkuOS/Source/pull/549) |
| 6–15 | usb_webcam: YUY2, streaming, sizes, UVC controls, MJPEG, unplugging, one node per camera | follow-ups to #549, not yet submitted; cover most of [#170](https://github.com/RenkuOS/Source/issues/170) |
| 16 | acpi: zero the EC cookie allocated from the ECDT (lid and AC events) | [#571](https://github.com/RenkuOS/Source/pull/571) |
| 17 | acpi: copy call arguments and results between kernel and userland | [#585](https://github.com/RenkuOS/Source/pull/585) |
| 18–19 | Network: update the wireless menu in place, one entry per SSID (Dru Satori) | [#474](https://github.com/RenkuOS/Source/pull/474) |
| 20 | wlan, usb_webcam: no debug output in normal use (atomozero) | [#544](https://github.com/RenkuOS/Source/pull/544) |
| 21–25 | usb_audio, hda, multi_audio, media_server, libmedia: audio fixes | hda [#589](https://github.com/RenkuOS/Source/pull/589), usb_audio [#590](https://github.com/RenkuOS/Source/pull/590), multi_audio [#591](https://github.com/RenkuOS/Source/pull/591), media_server [#600](https://github.com/RenkuOS/Source/pull/600); libmedia not yet submitted |
| 26 | boot/efi: give the loader a 16 MB heap (needed to list a second system volume) | [#587](https://github.com/RenkuOS/Source/pull/587) |
| 27 | apps: add [Flick](https://github.com/michaelspanier/flick) (webcam viewer) and Pegel (audio level meter) | own apps, will move to HaikuPorts |
| 28–32 | net80211: roaming with hysteresis, keep the link up while roaming within the ESS | [#597](https://github.com/RenkuOS/Source/pull/597), [#598](https://github.com/RenkuOS/Source/pull/598), [#599](https://github.com/RenkuOS/Source/pull/599) (draft, on top of #597) |
| 33 | net80211: do not scan when the scan results are read | [#584](https://github.com/RenkuOS/Source/pull/584) |
| 34 | boot/loader: prefer a volume marked as the default | [#587](https://github.com/RenkuOS/Source/pull/587) (fixes [#467](https://github.com/RenkuOS/Source/issues/467)) |
| 35 | power_daemon: save power while the lid is closed (atomozero) | [#530](https://github.com/RenkuOS/Source/pull/530) |
| 36 | media_server: a default chosen while running is the saved one (atomozero) | [#551](https://github.com/RenkuOS/Source/pull/551) |
| 37–40 | acpi_battery, PowerStatus, power: ThinkPad charge limit (Full / 80 % / 60 %) | not yet submitted |
| 41 | acpi_ac: read the AC state when the driver starts (booted on AC showed "offline") | not yet submitted |
| 42 | kernel/debug: time since boot in front of syslog lines, to time the boot | test bench aid, not yet submitted |
| 43 | usb_webcam: drive only the first video function of a camera (color + IR cameras) | for [#348](https://github.com/RenkuOS/Source/issues/348); build-tested only, no camera with two functions here |

Patches by other people keep their authorship; all credit for them goes to their authors.

## What works on the T470s with this series

- **Wi-Fi** (Intel Dual Band Wireless-AC 7260): roams through a 5-AP mesh without dropping the
  connection; flat latency even while the network list is polled
- **Webcams**: built-in UVC camera and Logitech C920 (YUY2 and MJPEG)
- **USB**: mice with buttons in several reports, isochronous transfers
- **ACPI**: lid and AC adapter events
- **Firefox**: Google Meet via WebRTC (software rendering, so background effects are too heavy)

## Building

The build uses the same cross-compiler image as the RenkuOS CI:

```sh
git clone https://github.com/RenkuOS/Source.git renkuos && cd renkuos
git checkout -b msp4 fb702668d7ed759521faa42f06c701fbcd77f6f8
git am /path/to/renkuos-msp/patches/build00096-msp4/*.patch
docker run --rm -v "$PWD:/src" -w /src ghcr.io/renkuos/cross-compiler:x86_64-master \
	sh -c 'mkdir -p generated && cd generated && ../configure --cross-tools-source ../buildtools \
		--build-cross-tools x86_64 -j6 && jam -q -j6 @nightly-anyboot'
```

Adjust the configure line to what `.github/workflows/image.yml` of the base commit does.
**Use at your own risk** – this is a test bench, install it next to a working system, never
instead of it.

## License

The patches are under the license of the files they change (MIT for RenkuOS code).
Patches by me are `Copyright 2026, Michael Spanier`, MIT.
