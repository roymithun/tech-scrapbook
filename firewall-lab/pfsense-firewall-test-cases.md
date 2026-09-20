# pfSense Firewall Practice — Test Cases

Lab topology: **Kali (attacker, 10.0.1.10)** → **pfSense (WAN 10.0.1.1 / LAN 192.168.1.1)** → **Server (192.168.1.10)**

General workflow for every test: **make the change → Save → Apply Changes → test from Kali → check Status → System Logs → Firewall.**

---

## 0. Server-side setup — installing services to test against

Each exercise below needs something actually listening on the server for the firewall/NAT rule to reach. Install these on the **server VM** (not pfSense) before running the matching exercise. A rule can be correctly configured and still "fail" simply because nothing is listening on the other end — see the troubleshooting note at the end of this section.

**Getting internet access on the server VM to install packages:**
The server's LAN adapter (wired to pfSense's LAN switch) has no route to the real internet, so `apt install` needs a temporary second path out:
1. Shut the VM down completely
2. VirtualBox → Settings → Network → enable a spare, currently-unused adapter slot → Attached to: **NAT**, Adapter Type: **Intel PRO/1000 MT Desktop (82540EM)**
3. Boot the VM (via GNS3 or directly in VirtualBox)
4. Check `ip a` — confirm the new interface got a `10.0.2.x`-range IP. If it shows `DOWN` with no IP, that specific adapter slot may be stuck (a recurring VirtualBox quirk in this lab) — shut down, disable that slot, enable a **different** unused slot instead, and retry
5. Install what you need (see table below), then shut down and set that adapter back to **Not attached** so GNS3 keeps managing the VM cleanly

**Packages for each exercise:**

| Package | Covers | Install command |
|---|---|---|
| `nginx` | HTTP/HTTPS (port 80/443), NAT forwarding | `sudo apt install -y nginx` |
| `openssh-server` | SSH (port 22) on the server itself, separate from pfSense's own SSH | `sudo apt install -y openssh-server` |
| `vsftpd` | FTP — good for practicing multi-port NAT (passive mode needs a port range, not just one port) | `sudo apt install -y vsftpd` |
| `netcat-traditional` | Instant one-off listener on any port, no real service needed: `nc -lvp <port>` | `sudo apt install -y netcat-traditional` |
| `bind9` | DNS — UDP port 53, good for practicing UDP vs TCP rule differences | `sudo apt install -y bind9` |

**After installing, always verify the service is actually running before testing from Kali:**
```bash
sudo systemctl status <service-name>     # e.g. nginx, ssh
sudo ss -tlnp | grep :<port>             # confirms it's listening
curl http://localhost                    # or equivalent local test
```

**Troubleshooting note — `openssh-server` fails to start with "no host keys available":**
Fresh installs sometimes don't auto-generate SSH host keys. Fix:
```bash
sudo ssh-keygen -A
sudo systemctl start ssh
sudo systemctl enable ssh
```
Then verify with `sudo systemctl status ssh` (service is named `ssh`, not `sshd`, on Ubuntu) and test locally with `ssh localhost` before testing through the firewall.

**Troubleshooting note — port 22 conflicts with pfSense's own SSH:**
If NAT-forwarding to the server's SSH, avoid clashing with pfSense's own SSH service (if enabled) by using a different external port, e.g. forward WAN:`2222` → server:`22`, rather than reusing WAN:`22`.

---

## 1. Default-deny behavior

**Goal:** confirm pfSense blocks everything inbound to WAN by default.

```bash
ping <WAN IP>
nc -zv <WAN IP> 22
nmap -p 1-100 <WAN IP>
```
Expected: all fail/filtered with no rules present. This is the baseline every other test builds on.

---

## 2. ICMP allow rule ✅ (done)

**Rule:** Pass, ICMP, Source=`10.0.1.0/24`, Destination=WAN address

```bash
ping -c 4 <WAN IP>
```
Expected: succeeds once rule is applied, fails once removed.

---

## 3. Port-specific allow rule ✅ (done)

**Rule:** Pass, TCP, Source=`10.0.1.0/24`, Destination=WAN address, Port=22

```bash
nc -zv <WAN IP> 22
nmap -p 22 <WAN IP>
```
**Key lesson learned:** a passing firewall rule ≠ a service listening on the other end. If nmap shows `filtered` even with the rule in place, check whether the actual service (e.g. SSH under System → Advanced → Admin Access) is enabled. `filtered` = firewall/no response; `closed` = actively rejected (something's there, just not accepting); `open` = fully working.

Repeat for a few more ports to build the pattern:
- Port 80 (HTTP)
- Port 443 (HTTPS)
- Port 3389 (RDP) — should stay blocked (no rule), useful as a control/comparison

---

## 4. Rule ordering — first-match-wins

**Goal:** understand pfSense evaluates rules top-to-bottom and stops at the first match (not the most specific).

1. Add Rule A (place at top): **Block**, ICMP, Source = Kali's exact IP (`10.0.1.10/32`)
2. Add Rule B (place below A): **Pass**, ICMP, Source = `10.0.1.0/24`

```bash
ping -c 4 <WAN IP>
```
Expected: fails — Rule A matches first even though Rule B would also match.

3. Drag Rule B above Rule A → Apply → retest
Expected: succeeds now — the allow rule is evaluated first.

**Takeaway:** more specific rules don't automatically win — position does. This is one of the most common real-world firewall misconfigurations.

---

## 5. NAT port forwarding

**Goal:** reach an internal LAN service from the WAN/attacker side, through the firewall.

**Setup:**
- **Firewall → NAT → Port Forward → Add**
- Interface: WAN, Protocol: TCP
- Destination port: 8080
- Redirect target IP: `192.168.1.10` (server), Redirect target port: 80

```bash
curl http://<WAN IP>:8080
```
Expected: reaches nginx (or whatever's running) on the internal server, even though the server itself isn't directly reachable from WAN.

**Follow-up check:** go to Firewall → Rules → WAN afterward — pfSense auto-creates an associated "pass" rule for the NAT entry. Note it, and try deleting *just* the firewall rule (leaving the NAT mapping) — does the port forward still work? (It shouldn't — NAT translation and the firewall pass rule are separate; both are needed.)

---

## 6. Watching live traffic in the logs

- **Status → System Logs → Firewall**, leave open in a tab
- From Kali:
  ```bash
  nmap -p 1-1000 <WAN IP>
  ```
- Refresh the log page — observe each blocked attempt logged with source IP, destination port, and action
- Try the same scan against a port you *have* allowed — compare how a pass vs. block entry looks in the log

---

## 7. Scan type comparison / basic evasion awareness

Once you have a block rule active on a port (e.g. 22):

```bash
nmap -sS -p 22 <WAN IP>     # SYN scan (default, "stealthy")
nmap -sA -p 22 <WAN IP>     # ACK scan — probes firewall state tracking
nmap -f -p 22 <WAN IP>      # fragmented packets
nmap -sU -p 53 <WAN IP>     # UDP scan on a different port
```
Expected: pfSense's stateful engine should catch all of these consistently (unlike a simple non-stateful packet filter, which ACK/fragment scans can sometimes slip past). Compare log entries for each scan type — do they look different?

---

## 8. Egress filtering (LAN → WAN direction)

So far all tests were inbound (WAN → LAN). Flip it — block the server from pinging Kali.

- **Firewall → Rules → LAN → Add** (place **above** the default "allow LAN to any" rule)
- Action: Block, Protocol: ICMP, Source: LAN net, Destination: Single host = Kali's IP (`10.0.1.10`)
- Save → Apply Changes → confirm it sits above the default allow rule (first-match-wins)
- From Ubuntu Desktop:
  ```bash
  ping -c 4 10.0.1.10
  ```
Expected: fails (100% loss) — confirm in Status → System Logs → Firewall that it's this rule blocking it. Note the contrast with WAN's default-deny: LAN is permissive by default, so this is the first time you're adding a restriction rather than an exception.

---

## 9. Aliases (rule management at scale)

- **Firewall → Aliases → Add** → create `Kali_Attacker` = `10.0.1.10`
- Edit an existing rule to use the alias as Source instead of typing the IP directly
- Add a second host to the alias later and confirm the existing rule automatically covers it too — no need to edit the rule itself

**Why this matters:** mirrors how real firewall rulesets stay maintainable — you update one alias instead of hunting through dozens of rules.

---

## 10. Rate limiting (advanced, optional)

- Edit a rule → **Advanced Options** → set a **Max. states / Max. new connections per second** limit
- From Kali, hammer the allowed port rapidly:
  ```bash
  for i in {1..50}; do nc -zv <WAN IP> 80; done
  ```
- Check logs/behavior once the limit is exceeded — later connections should be dropped even though the base rule still allows the port

---

## 11. Logging behavior itself

- Edit any rule → check/uncheck the **"Log packets that are handled by this rule"** option
- Confirm: does disabling logging on a pass/block rule change whether it shows up in Status → System Logs → Firewall? (It should stop appearing once logging is off — useful to know for reducing log noise on high-volume allow rules in real deployments.)

---

## Quick reference — Kali test commands

```bash
# Single port check
nc -zv <WAN IP> <port>

# Port range scan
nmap -p 1-1000 <WAN IP>

# Specific scan types
nmap -sS -p <port> <WAN IP>   # SYN
nmap -sA -p <port> <WAN IP>   # ACK
nmap -f -p <port> <WAN IP>    # fragmented
nmap -sU -p <port> <WAN IP>   # UDP

# HTTP through NAT
curl http://<WAN IP>:<forwarded port>

# Reachability
ping -c 4 <target IP>
```

## pfSense-side reference

```
Firewall → Rules → WAN / LAN     — allow/block rules, drag to reorder
Firewall → NAT → Port Forward    — expose internal services through WAN
Firewall → Aliases               — named groups of IPs/ports for reuse in rules
Status → System Logs → Firewall  — live pass/block log
System → Advanced → Admin Access — enable/disable services like SSH
```
