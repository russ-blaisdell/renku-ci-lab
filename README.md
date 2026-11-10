# renku-ci-lab

Experiments to back a CI proposal for RenkuOS with measurements instead of guesses.

## kvm-boot

`.github/workflows/kvm-boot.yml` checks whether `/dev/kvm` works on GitHub's free hosted Linux runners. RenkuOS's
CI currently assumes it doesn't and boots in software emulation (TCG). The workflow boots the published RenkuOS
nightly (x86_64) twice, once with KVM and once with TCG, and records:

- the runner's CPU, memory and whether `/dev/kvm` exists;
- seconds until the kernel banner (`Haiku revision:`);
- seconds until the screen stops changing, i.e. the desktop (Welcome window) is up, from screenshots every 2 seconds;
- the serial log, plus a screenshot every 10 seconds.

## Results

- 2026-11-10, run 42864677268 (nightly build00078, 4-vCPU AMD EPYC runner): `/dev/kvm` exists and works after the
  udev rule (the guest sees `x2apic`, which QEMU only offers under KVM). Desktop up within 15 s with KVM, between 15
  and 30 s with TCG.
- 2026-11-10, run 42865906655 (same build, 2 s resolution): desktop up at **10 s with KVM, 14 s with TCG**. Kernel
  banner within 1 s both ways. Boot is only ~30% faster; the KVM gain should be larger for CPU-heavy work (launching
  every app, unit tests, stress runs), which is the next thing to measure.

## workload

`.github/workflows/workload.yml` builds Haiku's `bfs_shell` host tool in RenkuOS's toolchain container, uses it to put
`workload/UserBootscript` into the nightly image, boots it with KVM and with TCG, presses Enter on the first-boot
prompt, and records what the script reports on the second serial port: CPU benchmarks, then the time from launching
each app and preference panel until it has a window (asked with `hey`).

### Results

2026-11-10, run 42890352048 (nightly build00078, same 4-vCPU runner class). Guest timings match the host's clock to
about 1%.

| Work inside the guest | KVM | TCG | KVM speedup |
|---|---|---|---|
| SHA-256 of 200 MB | 2.1 s | 6.6 s | 3.2× |
| `xz -1` of 50 MB | 9.5 s | 65.8 s | 6.9× |
| 200,000-iteration shell loop | 3.0 s | 52.9 s | 17.8× |
| Launch 49 apps to first window (sum) | 10.0 s | 52.5 s | 5.2× |
| Median app launch | 122 ms | 803 ms | 6.6× |
| Whole job | 6 min | 9.5 min | |

The same 12 entries didn't report a window under both: 5 exit straight away (AutoRaise, RemoteDesktop, Screenshot,
Deskbar, Tracker, which is already running) and 7 time out (HaikuDepot, LaunchBox, PoorMan, PowerStatus,
ProcessController, SoundRecorder, Bluetooth), which looks like a limit of asking via `hey`, not a crash. A real smoke
test needs a per-app check for those.

The first run (42885663432) misparsed `hey`'s reply, so its app timings are all timeouts; its CPU numbers agree with
the table above.

Run it from the Actions tab ("kvm-boot", Run workflow). It also runs when the workflow file changes. Results are in
the run summary and the `boot-kvm` / `boot-tcg` artifacts.

Run it in a **public** repository: that's the runner class RenkuOS uses, and the free tier for private repos
differs.
