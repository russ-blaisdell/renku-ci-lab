# renku-ci-lab

Experiments to back a CI proposal for RenkuOS with measurements instead of guesses.

## kvm-boot

`.github/workflows/kvm-boot.yml` checks whether `/dev/kvm` works on GitHub's free hosted Linux runners. RenkuOS's
CI currently assumes it doesn't and boots in software emulation (TCG). The workflow boots the published RenkuOS
nightly (x86_64) twice, once with KVM and once with TCG, and records:

- the runner's CPU, memory and whether `/dev/kvm` exists;
- seconds until the kernel banner (`Haiku revision:`);
- the serial log, plus a screenshot every 15 seconds (to see when the desktop appears).

Run it from the Actions tab ("kvm-boot", Run workflow). It also runs when the workflow file changes. Results are in
the run summary and the `boot-kvm` / `boot-tcg` artifacts.

Run it in a **public** repository: that's the runner class RenkuOS uses, and the free tier for private repos
differs.
