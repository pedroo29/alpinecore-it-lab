# CHG-0001 — Prepare PVE01 Host

## Change status
PREPARED — NOT YET IMPLEMENTED

## Target
PVE01

## Objective
Prepare the old laptop for a future bare-metal installation of Proxmox VE.

## Pre-change state
- Current OS: Linux Mint
- RAM: 16 GB
- CPU: 11th Gen Intel(R) Core(TM) i7-1165G7 @ 2.80 GHz
- Storage detected by Linux:
  - nvme0n1 — 476.9 GB — INTEL HBRPEKNL0202A
  - nvme1n1 — 27.3 GB — INTEL HBRPEKNL0202AO
- CPU virtualization: Intel VT-x
- UEFI virtualization setting: Enabled

## Data protection
No data on the laptop needs to be retained.
The future Proxmox installation is authorized to erase the current disk contents.

## Installation media preparation
- Proxmox VE ISO downloaded
- SHA-256 verified against the official published hash
- Result: MATCH
- Bootable USB created successfully

## Planned change
Install Proxmox VE bare metal on PVE01.

## Main risk
The installation will erase the existing operating system and data on the target disk.

## Validation after implementation
- PVE01 boots into Proxmox VE
- Correct hostname configured
- Management network reachable
- Proxmox Web UI accessible from the administration workstation

## Rollback
No in-place rollback is planned.
A previous operating system would require a fresh reinstall from separate installation media.