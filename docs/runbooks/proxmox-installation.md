# AlpineCore AG — PVE01 Proxmox VE installation runbook

**Project:** PRJ-F0-001 — AlpineCore IT Lab Foundation  
**Phase:** F0 · Week 1 · Day 4  
**Session:** SESSION-0005 (31 October 2026; installation and validation reported by operator)  
**System:** PVE01 — personal, portable homelab; NOT a production server  
**Execution:** GUIDED; post-install results reported by operator.  
**Status:** INSTALLED / POST-INSTALL CHECKS PASSED; portfolio evidence review and documentation commit pending.

## 1. Purpose and scope

Replace the previous Linux Mint installation with a bare-metal Proxmox VE host, reachable from the Windows administration workstation through the GL-MT3000 private LAN, whether Internet connectivity is available or not.

**Safety:** This procedure ERASED the previous contents of the chosen installation disk. No credentials, administrator email address, password, recovery keys, serial numbers or real adapter MAC addresses should be added to this public-facing runbook.

## 2. Observed hardware and software

| Component | Observed configuration |
|---|---|
| Host | PVE01 (former Linux Mint laptop) |
| CPU | Intel Core i7-1165G7 |
| RAM | 16 GB |
| Virtualization | Intel VT-x enabled in UEFI |
| Installed Proxmox | Proxmox VE 9.2; `pve-manager/9.2.24` (reported by `pveversion`) |
| Running kernel after update/reboot | `7.0.14-26-pve` (verified using `uname -r` and `pveversion`) |
| Previous running kernel | `7.0.2-6-pve` |
| Installation target | `/dev/nvme0n1` — approximately 476.9 GiB |
| Another exposed NVMe namespace | `/dev/nvme1n1` — approximately 27.3 GiB; **not** selected as target |
| Root filesystem choice | `ext4` (installer default layout with LVM/LVM-Thin) |
| Physical management connection | USB-to-Ethernet adapter connected to GL-MT3000 LAN |
| Proxmox bridge | `vmbr0` with physical bridge port `nic0` |

**Disk identity evidence before installation:** Linux Mint reported `/` mounted from `/dev/nvme0n1p2` with `findmnt -no SOURCE /`. Device models matched the observed target sizes. The 27.3-GiB namespace may be part of a hybrid storage module; its physical relationship was not conclusively established and it must not be treated as an independent backup.

## 3. AlpineCore management network

| Item | Value |
|---|---|
| Private LAN | `10.77.77.0/24` |
| Router / default gateway | `10.77.77.1` (GL-MT3000) |
| Router DHCP pool | `10.77.77.100–10.77.77.199` |
| PVE01 management address | `10.77.77.10/24` (static, on `vmbr0`) |
| Host FQDN | `pve01.alpinecore.test` |
| PVE01 DNS setting | `10.77.77.1` |
| Proxmox Web UI | `https://10.77.77.10:8006` (LAN-only) |
| Workstation | DHCP via private router Wi-Fi; `10.77.77.194` observed during installation |
| Wired interface | `nic0`, identified against the previously observed USB-Ethernet adapter |

The uplink (WAN/Repeater) can change between authorized locations while this private management LAN remains the same. Do not configure WAN port forwarding to the Proxmox management port or SSH.

## 4. Installation sequence (performed)

1. Verified Proxmox ISO SHA-256 against the publisher's checksum; prepared bootable USB.
2. Enabled/verified VT-x in UEFI, verified USB-Ethernet and private LAN connectivity.
3. Inspected the intended target with `lsblk` and `findmnt`; selected **only** `/dev/nvme0n1` (~476.9 GiB).
4. Booted the USB installer using UEFI and selected **Install Proxmox VE (Graphical)**.
5. Chose `ext4`, Switzerland, `Europe/Zurich`, keyboard layout `es`; configured administrator password and notification email privately (not recorded here).
6. Configured `nic0`, `pve01.alpinecore.test`, `10.77.77.10/24`, gateway `10.77.77.1`, DNS `10.77.77.1`.
7. Reviewed installer summary and installed Proxmox, erasing the former Linux Mint system on the selected disk.
8. Booted the installed system and accessed the Web UI from Windows using `root` and the Linux PAM realm.
9. Disabled unused Enterprise and Ceph Enterprise sources; enabled Proxmox No-Subscription; kept Debian repositories enabled.
10. Ran APT package-list refresh; the reported output ended in `TASK OK`.
11. Ran the package upgrade from Proxmox UI. **Process deviation:** the operator accepted the APT confirmation without first reviewing all proposed changes; record as a change-safety error, not a successful independent pre-change review.
12. Completed upgrade, rebooted to activate the newer kernel, and regained remote management access.

## 5. Post-install verification evidence

| Test | Observed outcome | Result |
|---|---|---|
| Windows browser → `https://10.77.77.10:8006` | Web UI accessible, including after reboot | PASS |
| `vmbr0` configuration | `10.77.77.10/24`, gateway `10.77.77.1`, bridge port `nic0` | PASS |
| `local` storage | Directory; ~100 GB total / ~6 GB used; enabled | PASS (configuration/capacity) |
| `local-lvm` storage | LVM-Thin; ~373 GB total / 0 GB used; enabled | PASS (configuration/capacity) |
| Repeater disconnected: Windows → router | ICMP replies | PASS |
| Repeater disconnected: Windows → PVE01 | ICMP replies | PASS |
| Repeater disconnected: Proxmox Web UI | Accessible from Windows | PASS |
| Repeater restored: PVE01 → `1.1.1.1` | 4/4 ICMP replies; 0% loss | PASS |
| Repeater restored: `getent hosts deb.debian.org` | `199.232.190.132` returned during the test | PASS |
| `pveversion` | `pve-manager/9.2.24/... (running kernel: 7.0.14-26-pve)` | PASS |
| Upgrade reboot recovery | New kernel running, remote UI accessible | PASS |

**Limitations:** The reported storage tests validate definitions and capacity, not successful VM disk creation or restore. The offline test establishes local-management independence in the tested setup, not compatibility with every future upstream network. No restore/rollback rehearsal is claimed.

## 6. Basic diagnostic / recovery guide (future use)

If the web interface cannot be reached:

1. Check PVE01 power, GL-MT3000 power, USB-Ethernet cable and Wi-Fi connection to the **private** SSID.
2. From Windows, test `ping 10.77.77.1` and `ping 10.77.77.10` separately.
3. If available, use PVE01's **physical** console to inspect the current configuration with `ip -br -4 addr`, `ip -4 route` and `cat /etc/network/interfaces` (read-only commands).
4. Compare observed IP, bridge membership, gateway and link against section 3 before changing any configuration.
5. Do not factory-reset the router or modify the bridge blindly; preserve console access and plan the reversal first.

**Backup limitation:** Both `local` and `local-lvm` use the internal host storage. A backup saved only on `local` does not protect against failure of that physical storage. Add external/independent backup and perform a restore test in the appropriate future lab.

## 7. Professional change-management note

**Deviation observed:** During package upgrade, the APT `Y` confirmation was accepted before reviewing the full install/remove summary.

**Risk:** A proposed package removal or major dependency change could have gone unnoticed. In this instance, the upgrade ultimately completed and post-change validation passed, but the decision process itself was not safe.

**Preventive control for next change:** Read the complete summary; identify packages to be removed and major updates; determine impact and recovery path; only then explicitly approve. Retest this skill on a subsequent real change. Do not mark this change as an independent change-management success.

## 8. Outstanding documentation / validation

- [ ] Add this runbook at `docs/runbooks/proxmox-installation.md` in the canonical `alpinecore-it-lab` repository.
- [ ] Update `docs/network/network-plan.md` to reflect the **observed** LAN settings (not merely the original proposal).
- [ ] Update the physical architecture diagram and hardware inventory, with sensitive device identifiers sanitized before publication.
- [ ] Verify final Git tree, commit and remote sync; record the actual commit ID only after observing it.
- [ ] Review evidence quality and schedule the appropriate retained-mastery checks.

**Source of values:** Instructor-guided session and operator-reported command outputs/screenshots. This file documents the observed lab, not commercial production experience.
