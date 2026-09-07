# Help the next wheel

Thanks for helping make scrolling a less ambitious activity.

## Report what you observed

Use the [compatibility report](https://github.com/atk0309/McScrolly/issues/new?template=compatibility-report.yml) for either a working setup or a failure. Include your mouse model and connection mode, input name/IDs, distro and desktop session, input-tool versions, native GeForce NOW version, and the actual in-game result.

Distinguish a rule that parses, one that matches a device, a fresh libinput test, and successful scrolling inside GeForce NOW. Each establishes a different part of the result. Report sleep/wake or reconnect behaviour only if you tested it.

After reviewing a successful streamed-game report, add its mouse, connection, tested desktop and result to the README's [confirmed working hardware table](README.md#confirmed-working-hardware), with a link to the issue as evidence. A parser check or desktop-only test does not qualify. Keep unknowns and any failed or untested checks explicit in the linked report, and do not generalise a result to other connection modes or versions.

Device serials, credentials, private paths and full input captures are unnecessary. See the [read-only identification check](docs/troubleshooting.md#identify-the-input-device) for the small set of useful device details.

## Suggest a change

Keep each change focused. Documentation corrections are welcome. For another device, provide its input-device identity and test evidence before describing it as supported. A dongle's `lsusb` ID may differ from its paired mouse's input ID.

Preserve existing libinput sections in any installation instructions, keep device matching specific, and retain an explicit undo path. This project currently ships a manual configuration recipe rather than an installer. Discuss a proposed installer or broad device support before expanding the scope.

## Check your contribution

- For a quirk change, validate it using the intended libinput version and confirm which device it matches.
- For a compatibility claim, record the actual GeForce NOW test separately from lower-level input checks.
- For documentation, check links, copyable commands and GitHub's rendered Markdown. Keep the README's configuration block aligned with the file under `quirks/`.
- For artwork, preserve readable text, alternative text and a self-contained SVG without external scripts or assets.

There is no automated device test suite. Describe checks you actually performed and leave untested behaviour clearly marked.

Keep humour in the welcome mat and precision in the instructions. The scroll wheel has already suffered enough ambiguity.

Contributions are covered by the repository's [MIT licence](LICENSE).
