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

Run it from the Actions tab ("kvm-boot", Run workflow). It also runs when the workflow file changes. Results are in
the run summary and the `boot-kvm` / `boot-tcg` artifacts.

Run it in a **public** repository: that's the runner class RenkuOS uses, and the free tier for private repos
differs.
