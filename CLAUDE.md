@AGENTS.md

# FanControl.LianLi orientation

`AGENTS.md` is authoritative for contribution rules. This short companion is a
map for a first pass through the repository; it does not replace the four
mandatory `.claude/rules/` documents.

## Fork scope and branch policy

- This fork changes RGB functionality only. The existing fan-control and telemetry
  implementation is trusted scope; do not refactor it while delivering lighting work.
- Work locally on `dev`. Keep `main` as the clean, safe merge baseline.
- Make the smallest change that proves the requested behavior. Do not add a layer,
  abstraction, configuration surface, or future roadmap unless the current feature
  needs it and an existing pattern cannot serve it.
- Commit only a completed, reviewable behavior: its focused tests must pass, and the
  commit must not mix unrelated cleanup, refactoring, or experiments. Run the full
  `./build.ps1` gate before merging to `main` or opening a pull request.
- Push only commits that are ready for review; keep incomplete work local on `dev`.

## Runtime and builds

- This is one `netstandard2.0` FanControl plugin DLL. Keep that target: the same
  binary must load in FanControl's .NET Framework and modern .NET hosts.
- The only public type is `Plugin/LianLiPlugin.cs` (`IPlugin3`). Everything else
  stays `internal`; tests use `InternalsVisibleTo`.
- The shipped variants are mutually exclusive:
  - standard: fan control and telemetry only;
  - `-p:EnableArgb=true`: hands compatible lighting to the motherboard header;
  - `-p:EnableLighting=true`: reads L-Connect's saved files and replays them.
- Run `./build.ps1` for the complete gate. It restores, checks format, and
  builds/tests all three variants with 100% line, branch, and method coverage.

## Read these in this order

1. `README.md` for supported hardware and deployment choices.
2. `docs/architecture.md` and `docs/tfm.md` for host/layering constraints.
3. `docs/wireless.md` for the L-Wireless transmitter/receiver protocol.
4. `docs/lighting.md` before touching saved L-Connect lighting.
5. The closest source and matching `*Tests.cs` in the same layer.

## Code map

- `Plugin/`: composition root, host lifecycle, controller discovery and refresh.
- `Worker/`: per-controller loops and keepalive scheduling.
- `Devices/`: controller state, L-Connect readers, reconnect/replay policy.
- `Transport/`: the sole native HID/WinUSB boundary.
- `Protocol/`: pure byte encoders with full-buffer tests.
- `Logging/`: non-throwing sinks.

`LianLiPlugin.Initialize` is the practical starting point. It builds wired
controllers, paired wireless dongles, and—in the Lighting build—loads L-Connect
configuration. `WirelessController` owns the wireless cycle; its effect replay
uses `WirelessSavedEffect` and `WirelessProtocol` rather than a separate RGB
transport.

## OpenRGB bridge direction (not implemented)

FanControl's supplied plugin contract exposes only fan/control/temperature
sensors; it has no RGB device API. A future bridge therefore must expose its
own loopback-only OpenRGB SDK endpoint rather than try to add RGB devices to
FanControl. It must remain a client-facing adapter: all device ownership and
USB writes stay in the existing Worker → Devices → Transport path. Do not add
HidSharp, let an SDK thread write USB directly, or bind a SYSTEM-hosted listener
to non-loopback interfaces by default. `E:\code\lian-li-linux` is a protocol
and server-design reference, not code to copy into this C# plugin.
