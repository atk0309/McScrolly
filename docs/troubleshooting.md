# When the wheel still refuses to wheel

[Back to the README](../README.md)

Start with the device identity and the layer you are checking. The same physical mouse can expose a different identity through another connection mode.

## Identify the input device

This read-only command lists vertical-wheel-capable input devices without serial numbers, key events or pointer coordinates:

```bash
python3 - <<'PY'
from pathlib import Path

for device in sorted(Path('/sys/class/input').glob('event*/device')):
    relative = device / 'capabilities/rel'
    if not relative.exists():
        continue
    if not int(relative.read_text().strip(), 16) & (1 << 8):
        continue
    print('/dev/input/' + device.parent.name)
    print('  name:', (device / 'name').read_text().strip())
    for field in ('bustype', 'vendor', 'product'):
        print(' ', field + ':', (device / 'id' / field).read_text().strip())
PY
```

For the tested receiver connection, the expected result includes:

```text
name: Logitech G903 LS
bustype: 0003
vendor: 046d
product: 4087
```

Bus `0003` is USB. These are the mouse's Linux input-device IDs. The physical receiver appeared as `046D:C539` in the original setup. Its dongle ID is not a replacement for the mouse's `MatchProduct=0x4087`.

The `/dev/input/eventN` number can change after reconnects or a reboot. Use the current node reported above in the following read-only check, replacing `eventN`:

```bash
libinput quirks list /dev/input/eventN
```

For a matching device with the override loaded, the result should include:

```text
AttrEventCode=-REL_WHEEL_HI_RES;-REL_HWHEEL_HI_RES;
```

If it does not, check the device name, IDs, bus and file syntax. Do not broaden the rule to every mouse as a shortcut. For another model or connection, [report its identity and actual result](https://github.com/atk0309/McScrolly/issues/new?template=compatibility-report.yml).

## Wheel settings do not match

Check the hardware settings through Solaar:

```bash
solaar config 'G903 LS' hires-smooth-resolution
solaar config 'G903 LS' hires-scroll-mode
```

The expected values are resolution `True` and diversion `False`.

If resolution is false, set it on using Solaar or:

```bash
solaar config 'G903 LS' hires-smooth-resolution true
```

If diversion is true, inspect any Solaar wheel rules you deliberately use before changing it. This workaround expects normal wheel output rather than diversion to HID++ notifications. On a setup without an intentional diversion rule, turn “Scroll Wheel Diversion” off in Solaar and check again.

If Solaar cannot find the mouse, wake it, check its connection, and make sure its GUI can see the device. If a setting command fails or the saved preference does not agree with a fresh read, resolve that before declaring the workaround ready. Avoid directly editing Solaar's configuration while its GUI is running, because the running application can overwrite the edit.

## The configuration fails validation

```bash
libinput quirks validate
```

Inspect the reported file and line. Compare the new section with [the shipped quirk](../quirks/logitech-g903-ls.quirks), preserving existing sections. Check the section header, spelling, minus signs and semicolons.

A parsing error can prevent libinput from loading the quirks database. Correct or remove your added section before logging out to activate it. Editing `/usr/share/libinput` is unnecessary; package updates can replace files there.

If the parser rejects a property on a different libinput version, consult that version's documentation. The included syntax was validated on 1.25.0, and quirks are an internal interface rather than a stable configuration API. [Upstream documentation](https://wayland.freedesktop.org/libinput/doc/1.25.0/device-quirks.html)

## The rule matches, but GeForce NOW still does not scroll

`libinput quirks list` and a newly started libinput diagnostic use fresh configuration. They do not establish what the existing desktop session has loaded.

Save your work, log out of the desktop and log back in. Restarting the game alone or reconnecting the receiver has not been established as sufficient to reload the compositor's quirks. Then test both wheel directions inside a streamed GeForce NOW game. Verify the native app is the one being tested; browser streaming and other session types have not been validated by this repository.

If the game still fails, report the specific action that does not respond, your input-stack versions and whether ordinary desktop scrolling works. Do not assume every GeForce NOW wheel problem has this cause.

## Raw events still say HI_RES

That is expected. The hardware remains in high-resolution mode and the driver can emit both kinds of wheel event. The quirk changes libinput's view of the device. It does not remove kernel event capabilities.

The physical verification for this workaround compared ordinary kernel steps with a fresh libinput context. It did not use the presence of raw high-resolution events as a failure condition.

## Scrolling is too slow or too fast

First confirm resolution is true and diversion is false. Turning only hardware resolution off can make the mouse's output disagree with the driver's multiplier.

If those values match and scrolling works but feels wrong, record that separately from the GeForce NOW compatibility result. This repository does not adjust scroll speed. Sensitivity preferences and free-spinning versus ratcheted wheel feel were not calibrated during the test.

## Undoing the workaround

1. Open `sudoedit /etc/libinput/local-overrides.quirks`.
2. Remove only the `[Logitech G903 LS discrete wheel for GeForce NOW]` section. Preserve all other device sections. If no content remains, an empty file is fine.
3. Run `libinput quirks validate` and resolve any error.
4. Save work, log out and log back in to clear the desktop's cached configuration.

Leave Solaar's resolution true by default so the hardware and driver remain consistent. Removing the quirk does not restore the original saved resolution preference: that preference was false while the hardware already reported true. Setting it false through the CLI changes both the saved preference and hardware, and may bring back weak scrolling.

## Collect a useful report

[Open a compatibility report](https://github.com/atk0309/McScrolly/issues/new?template=compatibility-report.yml) with the mouse model, connection mode, input name and vendor/product IDs, distro, desktop/session type, kernel, libinput, Solaar and native GeForce NOW versions. Include the actual in-game result and any sleep/wake or reconnect observations you made.

You can read several relevant versions with:

```bash
uname -r
libinput --version
solaar --version
printf '%s\n' "$XDG_SESSION_TYPE"
```

Use the NVIDIA app's About information for its version. Share the small facts needed to reproduce the result. Full `solaar show` output can contain unique device identifiers; raw input recordings can contain more than wheel events. Please omit those, private paths and unrelated logs.
