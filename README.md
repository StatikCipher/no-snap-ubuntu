# Ubuntu Snap Removal

A Bash script that removes Snap from **Ubuntu 26.04 LTS**, cleans up leftover Snap data,
and keeps APT from selecting `snapd` for reinstallation.

The script started as a manually tested removal process. It was then revised so that it no
longer hard-codes snap names and so that it checks the final state of the system before
reporting success.

---

## What It Does

The script:

- confirms the system is running **Ubuntu 26.04**
- must be run as a normal user, not as root
- finds the snaps that are actually installed
- removes each one with `--purge`
- repeats the removal in passes, so dependent snaps can be cleared without a fixed
  package list
- removes the `snapd` snap last, if it is present
- purges the Debian `snapd` package
- deletes remaining Snap data for the current user and root
- creates an APT preference that blocks all versions of `snapd`
- verifies that:
  - the `snap` command is gone
  - the Debian `snapd` package is not installed
  - APT reports no installation candidate for `snapd`

---

## Important

This script deliberately removes Snap and the per-user Snap data that goes with it.

If they exist, these directories are deleted:

```text
~/snap
~/.snap
/root/snap
/root/.snap
```

If any Snap application holds data you want to keep, back it up before running the script.

This project currently supports **Ubuntu 26.04 LTS Classic installations only**.
Do not use it on an Ubuntu 26.04 install with TPM-backed full-disk encryption, because
those systems depend on Snap-delivered kernel and boot components.

---

## Testing

The script has been run successfully on a fresh Ubuntu 26.04 LTS virtual machine after the
initial system updates.

Test environment:

| Component | Test Environment |
|---|---|
| Host OS | Windows 11 Pro |
| Hypervisor | VMware Workstation Pro |
| Guest OS | Ubuntu 26.04 LTS |
| Test state | Fresh installation with initial updates applied |

A successful run finished with verification output like this:

```text
PASS: snap command is absent.
PASS: snapd Debian package is not installed.
PASS: APT has no snapd installation candidate.

==> Snap removal and APT blocking completed successfully.
```

Reports from other Ubuntu 26.04 installations are welcome.

---

## Usage

If `git` is not installed yet (a fresh Ubuntu installation does not always include it),
install it first:

```bash
sudo apt install git
```

Download the repository:

```bash
git clone https://github.com//StatikCipher/no-snap-ubuntu.git
cd no-snap-ubuntu
```

Make the script executable:

```bash
chmod +x remove-snap-ubuntu-26.04.sh
```

Run it as your **normal user**:

```bash
./remove-snap-ubuntu-26.04.sh
```

Do **not** run the whole script with `sudo`.

The script asks for `sudo` itself, only for the steps that need administrative access.

---

## Keeping `snapd` From Coming Back

Once Snap is removed, the script creates this file:

```text
/etc/apt/preferences.d/no-snap.pref
```

with this content:

```text
Package: snapd
Pin: version *
Pin-Priority: -10
```

A negative APT priority stops any available version of `snapd` from being chosen as an
installation candidate.

At the end of its run, the script checks this with:

```bash
apt-cache policy snapd
```

A good result shows:

```text
Installed: (none)
Candidate: (none)
```

with the available `snapd` versions listed at priority `-10`.

---

## Why the Script Looks for Installed Snaps

Fresh Ubuntu installations do not always ship with exactly the same snaps.

Rather than keeping a fixed list such as:

```text
firefox
snap-store
core24
...
```

the script runs `snap list` to find the packages that are really installed on the machine.

It then works through them in repeated passes. A snap that cannot be removed yet because
another installed snap depends on it is put off until a later pass.

If a whole pass finishes without removing anything, the script stops instead of forcing
removal or looping forever.

---

## Final Verification

The script does not take a successful command as proof that the job is finished.

Before it reports success, it checks all three of these conditions:

1. `snap` is no longer available
2. the Debian `snapd` package is not installed
3. APT has no installation candidate for `snapd`

If any check fails, the script exits with an error and says which one failed.

---

## Repository Contents

```text
YOUR-REPO-NAME/
├── LICENSE
├── README.md
└── remove-snap-ubuntu-26.04.sh
```

---

## Current Scope

For now the script supports **Ubuntu 26.04 LTS** only.

Possible future work:

- testing additional Ubuntu releases
- adding release-specific handling where it turns out to be needed
- testing more fresh-install configurations

Support for another release will be added only after its behavior has been verified.

---

## License

Released under the [MIT License](LICENSE).
