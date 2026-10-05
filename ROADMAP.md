# RGB bridge roadmap

This fork adds an OpenRGB bridge to the existing FanControl plugin. It does not
replace, refactor, or extend the fan-control and telemetry behavior that the
upstream plugin already provides.

## Baseline

- `main` is the merge-safe baseline and `dev` is the RGB development branch.
- The standard, ARGB, and Lighting variants pass `./build.ps1`.
- OpenRGB is installed as the local SDK client used for manual acceptance.
- `E:\code\lian-li-linux` is a protocol and behavior reference only. It is not a
  dependency, submodule, or source of copied implementation.

## MVP scope

The first usable bridge supports only the wireless hardware currently under
test:

- three UNI FAN SL-INF Wireless groups;
- two STRIMER Wireless devices;
- OpenRGB device discovery, `Direct`, and one-colour static updates.

All USB ownership, device discovery, reconnect handling, and RGB writes remain
inside this plugin. The bridge accepts a local OpenRGB SDK request and hands the
latest desired RGB state to the existing wireless controller and worker; an SDK
thread never writes USB directly.

## Delivery sequence

1. **Bridge contract** — expose the current wireless devices through a
   loopback-only (`127.0.0.1`) OpenRGB SDK endpoint. Offline, malformed, and
   unsupported requests must fail without affecting FanControl or fan control.
2. **RGB state handoff** — accept discovery and Direct/static colour requests,
   coalesce each device to its newest update, and apply a bounded send rate so
   client traffic cannot starve the wireless cycle.
3. **Wireless encoding** — map the accepted state to the existing device MAC,
   receiver slot, LED topology, and wireless protocol path. Add deterministic,
   byte-level tests for every new encoder decision.
4. **Hardware acceptance** — with L-Connect stopped and OpenRGB's native Lian Li
   detection disabled, verify enumeration and independent colour changes for all
   five target devices. Recheck FanControl RPM and curve operation during colour
   changes, OpenRGB reconnects, and plugin refresh/reconnect.

## Explicitly outside this MVP

- dynamic effects, profile persistence, a settings UI, or a new configuration
  format;
- wired controllers, AIOs, and other lighting families;
- any change to fan speed, RPM, temperature, discovery, or reconnect behavior;
- copying `lian-li-linux` code or adding it as a dependency.

## Completion gate

Each completed behavior is one reviewable commit with its focused tests. Run the
full `./build.ps1` gate before opening a pull request or merging `dev` into
`main`. Do not merge incomplete experiments, unrelated cleanup, or refactors.
