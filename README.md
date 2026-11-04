# kmp-ios-on-linux

Agent skills to build, sign, and install Kotlin Multiplatform iOS apps from Linux.

The skills guide a coding agent, such as Claude Code, OpenCode, Codex, or Cursor, through the full Linux-to-iOS path for an existing Kotlin Multiplatform (KMP) project. The path goes from toolchain setup to a signed development IPA on a real device. The agent works from evidence and stops at explicit approval gates. It reports blockers instead of producing placeholder output.

## Skills

| Skill | Purpose |
| --- | --- |
| [`kmp-ios-setup-linux`](skills/kmp-ios-setup-linux/SKILL.md) | Inspects the host and project, then prepares Swift, xtool, the Apple SDK, a signer, and device tools. Ends with a verified setup handoff. |
| [`kmp-ios-build-linux`](skills/kmp-ios-build-linux/SKILL.md) | Builds the real KMP framework, wraps the existing Swift host, packages and signs the IPA, verifies it, and installs and launches it on an authorized device. |

Each skill has a reference file with exact commands, version notes, and sources:
- [`references/debian12.md`](skills/kmp-ios-setup-linux/references/debian12.md)
- [`references/build.md`](skills/kmp-ios-build-linux/references/build.md)

## Requirements

- An existing KMP project with an `iosArm64` framework and a real Swift/iOS host app.
- A Linux host. The recipes are Debian-first. For other distributions, the agent researches official sources and adapts.
- One of these for the Kotlin framework:
  - A qualified, version-matched Kotlin/Native port for Linux. Stock Kotlin/Native on Linux does not support Apple final-binary targets, and these skills do not ship a compiler patch.
  - A framework built on supported macOS.
- An Xcode SDK that you obtain from Apple under its license terms.
- For a signed build: an Apple Development certificate with its private key (`.p12`), and a matching development provisioning profile.
- For device installation: a connected, trusted iOS device with Developer Mode on.

Tested baseline: Debian 12 x86_64 (with Debian 13 notes), Swift 6.4.1, xtool 1.22.2, an Xcode 27 SDK, zsign v1.1.3, and libimobiledevice. The skills treat these versions as a baseline, not as requirements. The agent selects versions by project compatibility.

## Installation

Install with the [`skills`](https://github.com/vercel-labs/skills) CLI:

```bash
npx skills add syamsudotdev/kmp-ios-on-linux
```

Or copy both directories from `skills/` into your agent's skills directory, for example `~/.claude/skills/` or `~/.config/opencode/skills/`. Install both skills together. They link to each other with relative paths.

## Usage

Ask your agent in plain language, for example:

- "Set up this machine to build the iOS app of this KMP project."
- "Build a signed development IPA of this project for my iPhone."
- "Rebuild the IPA from the latest commit and reinstall it on the device."

The agent asks for approval before host changes, before signing, before it handles a private key, and before it installs to a device.

## Security

The skills treat the `.p12`, private keys, passwords, decrypted PEM files, and Apple ID tokens as secrets. They require the agent to:

- Keep secrets in owner-only locations outside shared workspaces, repositories, build outputs, environment variables, and logs.
- Read the `.p12` password only from a file (`-passin file:`). The password never goes through command arguments, environment variables, or the chat.
- Never print, open, or upload a secret file, and check secrets only through results that do not contain them.
- Decrypt the key only in memory-backed temporary storage, inside one guarded command with cleanup.
- Stop and warn the user if a secret leaks, without repeating the value, and recommend rotation.

See [Secret placement and leak handling](skills/kmp-ios-build-linux/references/build.md#secret-placement-and-leak-handling) for the full rules.

## Disclaimer

This project is not affiliated with or endorsed by Apple or JetBrains. xtool and zsign are community tools. You are responsible for your compliance with the Apple SDK and Developer Program license terms.

## Contributing

Issues and pull requests are welcome. Keep the skills project- and host-agnostic: use clearly labeled example values, and state the tested versions as a baseline.

## License

[MIT](LICENSE)
