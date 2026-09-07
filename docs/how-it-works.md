# How the wheel gets its job back

[Back to the README](../README.md)

The workaround changes which wheel events libinput consumes. The mouse can continue using high-resolution mode, and the kernel can keep using the multiplier it expects.

## The input path

```text
Logitech G903 wheel
        │  High-resolution movement; reported multiplier: 8
        ▼
Linux Logitech HID++ driver
        ├── REL_WHEEL_HI_RES: fractional wheel movement
        └── REL_WHEEL: ordinary wheel steps
                    │
                    ▼
libinput + the G903-specific quirk
        │  Uses ordinary wheel steps; ignores high-resolution codes
        ▼
Wayland desktop and application input delivery
        │
        ▼
Native NVIDIA GeForce NOW client
```

This diagram describes the layers relevant to the workaround. The client's internal handling of the original input problem was not traced.

## Why leave high-resolution mode on?

In the observed setup, Solaar had `hires-smooth-resolution` saved as false while a fresh read from the G903 returned true. The mouse reported a multiplier of 8.

The kernel's Logitech HID++ driver converts wheel movement into fractions of 120 using its stored multiplier. It also accumulates movement to generate ordinary `REL_WHEEL` events for compatibility. If the mouse is switched to lower-resolution output while the driver continues using the high-resolution multiplier, the resulting scaling can make scrolling abnormally weak. This is a plausible explanation for the weak scrolling reported when changing only the Solaar switch. [Linux driver source](https://github.com/torvalds/linux/blob/v6.17/drivers/hid/hid-logitech-hidpp.c)

We observed the saved/live disagreement directly. We did not trace the exact process or event that restored high-resolution mode. Related interactions have also been discussed in the [Solaar project](https://github.com/pwr-Solaar/Solaar/issues/2404).

The working configuration makes the saved setting and live hardware agree at true. Filtering the high-resolution event codes within libinput then selects the kernel's ordinary wheel events without changing the hardware's units.

## What the quirk does

```ini
AttrEventCode=-REL_WHEEL_HI_RES;-REL_HWHEEL_HI_RES;
```

The minus signs disable those event codes within libinput for a matching input device. The rest of the section restricts the match to the named G903 mouse, bus and input vendor/product IDs. Ordinary `REL_WHEEL` and `REL_HWHEEL` events remain available. [libinput 1.25 documentation](https://wayland.freedesktop.org/libinput/doc/1.25.0/device-quirks.html)

The kernel can still emit high-resolution events. Seeing those events in a raw kernel capture is expected: the filter sits downstream inside libinput. It also does not promise to alter an application that reads the raw device through a different input path.

The workaround applies to this G903 across applications using the desktop input path. It does not modify the NVIDIA client, change the mouse's DPI, or introduce a scroll-speed multiplier.

## Resolution and diversion are different switches

| Solaar setting | Meaning | Verified value |
| --- | --- | --- |
| `hires-smooth-resolution` | Hardware wheel resolution | `True` |
| `hires-scroll-mode` | Divert wheel output to HID++ notifications for Solaar rules | `False` |

Similar names, different jobs. Enabling diversion is not part of this fix.

## What we verified

On the [documented setup](../README.md#tested-with-receipts):

- The staged rule matched the G903 and left the other 23 current input nodes unchanged.
- The installed quirks database passed the installed libinput 1.25.0 parser.
- Fresh Solaar reads confirmed resolution true and diversion false. Only the saved resolution setting changed.
- A physical test generated 153 ordinary kernel wheel events: 83 in one direction and 70 in the other.
- A fresh libinput context received 153 events with corresponding direction counts. Every value was exactly `+120` or `-120` in v120 units, with no fractional values.
- The owner subsequently confirmed that scrolling works in GeForce NOW.

One hundred fifty-three events is the observation count, not a promised success rate for other systems. A fresh libinput context verifies that context; it cannot prove what an already-running compositor has cached.

The exact game, the user's desktop reload sequence, the NVIDIA-side defect and the original reset trigger remain unrecorded or unidentified. Other mice, the wired G903 connection, sleep/wake and receiver reconnect behaviour remain unverified. Scroll-speed tuning was deliberately deferred.

If a future update fixes the original compatibility problem, the override may no longer be useful. Local quirks are an internal libinput interface, so revalidate after an input-stack upgrade and use the [targeted undo instructions](troubleshooting.md#undoing-the-workaround) when appropriate.
