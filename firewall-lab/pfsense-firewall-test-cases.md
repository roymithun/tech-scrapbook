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

# Phase 1 — Baseline Topology (Kali → pfSense → Ubuntu Desktop)

Everything below (Parts 1–6) is tested against the original three-node topology from the design doc — no DMZ yet. Once Phase 2 (DMZ) is built per the design doc, its own test cases get a separate `# Phase 2 — DMZ` section further down, keeping the two topologies' exercises clearly separated.


# Part 1 — Firewall Rules & NAT

## 1. Default-deny behavior

**Goal:** confirm pfSense blocks everything inbound to WAN by default.

```bash
ping <WAN IP>
nc -zv <WAN IP> 22
nmap -p 1-100 <WAN IP>
```
Expected: all fail/filtered with no rules present. This is the baseline every other test builds on.

---

## 2. ICMP allow rule

**Rule:** 

| Field                        | Value                          | Notes / Example                          |
|-----------------------------|--------------------------------|------------------------------------------|
| Action                      | Pass         |       |
| Interface                   | WAN           | Interface where traffic **enters**       |
| Address Family              | IPv4        |                              |
| Protocol                    | TCP      |                                          |
| Source                      |`10.0.1.0/24`      | Who is initiating the connection         |
| Source Port                 | any                            | Almost always leave as any               |
| Destination                 | WAN address      | Where the traffic is going               |
| Destination Port            |   Disabled by default      |     Disabled by default             |
| Description                 | Allows ping traffic    |                   |
| Log                         | Unchecked            |                   |

```bash
ping -c 4 <WAN IP>
```
Expected: succeeds once rule is applied, fails once removed.

---

## 3. Port-specific allow rule

**Rule:** 

| Field                        | Value                          | Notes / Example                          |
|-----------------------------|--------------------------------|------------------------------------------|
| Action                      | Pass         |       |
| Interface                   | WAN           | Interface where traffic **enters**       |
| Address Family              | IPv4        |                              |
| Protocol                    | TCP      |                                          |
| Source                      |`10.0.1.0/24`      | Who is initiating the connection         |
| Source Port                 | any                            | Almost always leave as any               |
| Destination                 | WAN address      | Where the traffic is going               |
| Destination Port            |   From: 22    To: 22      |                |
| Description                 | Allows ssh traffic    |                   |
| Log                         | Unchecked            |                   

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

## 5a. NAT port forwarding

**Goal:** reach an internal LAN service from the WAN/attacker side, through the firewall.

**Setup:**
- **Firewall → NAT → Port Forward → Add**

### Rule – Forward HTTP (Port 80) Traffic to Ubuntu Web Server

| Field                        | Value                          | Notes                              |
|-----------------------------|--------------------------------|------------------------------------|
| Interface                   | WAN                            |                                    |
| Protocol                    | TCP                            |                                    |
| Destination                 | WAN address                    | (default is fine)                  |
| Destination Port Range      | From: 8080 &nbsp;&nbsp; To: 8080   |       |
| Redirect target IP          | Ubuntu_LAN_IP                  | ← Put your **Ubuntu LAN IP** here  |
| Redirect target port        | HTTP                            |                                    |
| Description                 |     Forward HTTP (Port 80) Traffic to Ubuntu Web Server                |                                    |
| Filter rule association     | Rule NAT     |  |


```bash
curl http://<WAN IP>:8080
```
Expected: reaches nginx (or whatever's running) on the internal server, even though the server itself isn't directly reachable from WAN.

**Follow-up check:** go to Firewall → Rules → WAN afterward — pfSense auto-creates an associated "pass" rule for the NAT entry. Note it, and try deleting *just* the firewall rule (leaving the NAT mapping) — does the port forward still work? (It shouldn't — NAT translation and the firewall pass rule are separate; both are needed.)

---

## 5b. Outbound NAT (the reverse direction)

pfSense typically uses **Automatic Outbound NAT** to translate traffic leaving the LAN so that it appears to originate from pfSense's WAN IP address. In some lab topologies, especially when routing between two private networks, traffic may be routed without NAT. The following exercise verifies whether outbound NAT is occurring and demonstrates how to enable it manually if necessary.

### Verify Outbound NAT

1. Navigate to **Firewall → NAT → Outbound**.

2. If **Automatic Outbound NAT** is enabled, switch to **Hybrid** or **Manual** mode and apply changes.

3. Check whether any outbound NAT rules exist:
   - GUI: Review the rule list under **Firewall → NAT → Outbound**
   - CLI (optional):
     ```bash
     pfctl -sn
     ```
     You should see a rule resembling:
     ```text
     nat on <wan-interface> from 192.168.1.0/24 to any -> (<wan-interface>)
     ```

4. On Kali, start a listener:
   ```bash
   nc -lvnp 8081
   ```
5. On Ubuntu Desktop, initiate a connection:
   ```bash
   wget -qO- http://10.0.1.10:8081
   ```
   (or use curl if installed)

6. On Kali, observe the connection source:
   ```
   sudo tcpdump -ni eth0 port 8081
   ```
   Expected: 
   
   **NAT is working**

   Kali should see the source IP as pfSense's WAN IP:
   ```
   connect to 10.0.1.10 from 10.0.1.1 <port>
   ```

   This demonstrates source NAT, where pfSense rewrites:
   ```
   192.168.1.100 → 10.0.1.10
   ```
   to:
   ```
   10.0.1.1 → 10.0.1.10
   ```

   **NAT is NOT occurring**

   Kali may instead see:
   ```
   connect to 10.0.1.10 from 192.168.1.100 <port>
   ```
   This indicates that pfSense is routing traffic between the networks without performing source NAT.

   You can confirm the absence of NAT rules with:
   ```
   pfctl -sn
   ````

   If no nat on ... rules are displayed, outbound NAT is not currently being applied.

   ### Manually Create an Outbound NAT Rule

   If no NAT rules exist, create one:

    - Firewall → NAT → Outbound
    - Select Manual mode
    - Add a rule:

      ```
      Interface: WAN
      Protocol: any
      Source: 192.168.1.0/24
      Destination: any
      Translation / Address: Interface Address
      Description: Lab - Outbound NAT LAN to WAN
      ```

      Apply changes and verify that:

      ```
      pfctl -sn
      ```
      now displays a NAT rule.
      
      Such as:
      ```
      nat on em0 inet from 192.168.1.0/24 to any -> 10.0.1.1 port 1024:65535
      ```

    Repeat the test. Kali should now observe pfSense's WAN address as the source, demonstrating outbound NAT in action and providing the reverse perspective of the inbound port-forward configured in Section 5.


## 5c. 1:1 NAT (optional – needs an extra WAN-side IP)

### Rule – Virtual IP (IP Alias) for Extra WAN Address

| Field       | Value          | Notes                                              |
|-------------|----------------|----------------------------------------------------|
| Type        | IP Alias       |                                                    |
| Interface   | WAN            |                                                    |
| Address(es) | 10.0.1.2/24    | Extra WAN IP (adjust if your extra IP is different)|
| Description | Extra WAN IP for 1:1 NAT | Optional but recommended                     |

Click **Save** → **Apply Changes**.

---

### Rule – 1:1 NAT Mapping

| Field              | Value          | Notes                                              |
|--------------------|----------------|----------------------------------------------------|
| Interface          | WAN            |                                                    |
| External subnet IP | 10.0.1.2       | Same IP you just created as Virtual IP             |
| Internal IP        | 192.168.1.10   | ← Put your **Ubuntu / target LAN IP** here         |
| Destination        | Any            | (default is fine)                                  |
| Description        | 1:1 NAT for web server | Optional but recommended                    |

Click **Save** → **Apply Changes**.

---

### Verification

From Kali (or any external machine on the WAN side):

```bash
curl http://10.0.1.2
```
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

# Part 2 — Traffic control

## 12. Virtual IPs

- **Firewall → Virtual IPs → Add**: type **IP Alias**, interface WAN, address `10.0.1.2/24`
- Lets pfSense answer to more than one address on the same interface — used above in Section 5c
```bash
ping -c 4 10.0.1.2   # from Kali — pfSense should answer on this second address too
```

## 13. Floating rules

- **Firewall → Rules → Floating → Add**: rules that apply across multiple interfaces or a specific packet direction (in/out/either), evaluated *before* interface-specific tabs when "quick" is checked
- Exercise: create a floating rule blocking ICMP in the "out" direction, applied to WAN and LAN both — confirm it overrides what the individual interface tabs allow
- Useful for understanding rule evaluation order across the whole system, not just one interface

## 14. Traffic shaping / Limiters

- **Firewall → Traffic Shaper → Limiters → Add**: create a limiter capping bandwidth (e.g. 1 Mbit/s)
- Apply it to the NAT port-forward rule from Section 5 (Advanced Options → In/Out pipe)
- From Kali, download something through the forwarded port and compare throttled vs. unthrottled speed:
```bash
curl -o /dev/null -w "%{speed_download}\n" http://<WAN IP>:8080/somefile
```
Demonstrates QoS as a distinct concept from allow/deny rules.

---

# Part 3 — Services

## 15. DHCP server

- **Services → DHCP Server → LAN** — enable if not already, range e.g. `192.168.1.100`–`192.168.1.200`
- On Ubuntu Desktop, switch from static to DHCP temporarily and confirm a lease:
```bash
sudo dhclient -r enp0s3   # release
sudo dhclient enp0s3      # renew
ip a                      # confirm new lease
```
- **Status → DHCP Leases** in the GUI — confirm it shows up with the right hostname/MAC

## 16. DNS Resolver / Forwarder

- **Services → DNS Resolver** — pfSense runs Unbound by default, resolving DNS for LAN clients
- From Ubuntu Desktop: `nslookup google.com 192.168.1.1` — confirm pfSense answers
- **DNS-blocking exercise:** Services → DNS Resolver → Host Overrides → add an override for a test domain pointing to `0.0.0.0` — from Ubuntu Desktop, `curl` that domain and watch it fail. This is the mechanism pfBlockerNG (Section 19) automates at scale.

## 17. Dynamic DNS

Needs a real WAN internet route, which this isolated lab topology doesn't have — worth knowing it exists for real deployments, but skip actually testing it here.

---

# Part 4 — VPN

## 18. OpenVPN (remote access style)

- **VPN → OpenVPN → Wizard** — set up a server on pfSense (a self-signed CA is fine — see Section 21)
- Install the **Client Export** package first (System → Package Manager) to generate a ready-to-use client config
- On Kali:
```bash
sudo apt install -y openvpn
sudo openvpn --config kali-client.ovpn
```
- Once connected, confirm Kali gets a VPN-assigned IP and can reach `192.168.1.10` — demonstrates VPN as an alternate path into the LAN, separate from the physical/GNS3-wired topology

---

# Part 5 — Detection & advanced filtering

*(Brief here — see the separate NGFW extensions document for Suricata/Zenarmor in depth.)*

## 19. pfBlockerNG (package)

- System → Package Manager → Available Packages → install `pfBlockerNG-devel`
- Enable a reputation/blocklist feed, apply to WAN
- From Kali, attempt to reach a blocklisted IP — confirm it's dropped and logged distinctly from your manual rules

## 20. Suricata (IDS/IPS package)

- Install Suricata, enable on WAN, enable a signature ruleset (e.g. ET Open)
- From Kali: `nmap -sV -A <WAN IP>` (aggressive scan) — Suricata should flag this in Services → Suricata → Alerts, even for traffic your plain firewall rules would have passed through

---

# Part 6 — System & diagnostics

## 21. Certificate Manager

- **System → Cert Manager → CAs → Add** — create a local CA
- **Certificates → Add** — issue a cert signed by that CA (for the GUI or for OpenVPN, Section 18)
- Replace the GUI's default self-signed cert (System → Advanced → Admin Access → SSL Certificate) — reload the GUI and inspect the cert in-browser to confirm it's now signed by your own CA

## 22. Users & privilege separation

- **System → User Manager → Add** — create a limited-privilege user (not full admin)
- Log out, log back in as that user — confirm Firewall → Rules is inaccessible or read-only, demonstrating role-based access before handing pfSense access to someone else

## 23. Diagnostics tools (GUI equivalents of the shell commands you've been using)

- **Diagnostics → Ping / Traceroute** — GUI-based, useful without shell access
- **Diagnostics → Packet Capture** — capture on WAN while pinging from Kali, download the `.pcap`, open in Wireshark for a proper packet-level view instead of raw `tcpdump` text
- **Diagnostics → States** — live state table; trigger a few connections from Kali and watch entries appear/expire — ties directly back to "stateful firewall" as a concept
- **Diagnostics → ARP Table** — same table you checked manually during earlier connectivity troubleshooting

## 24. Backup & restore

- **Diagnostics → Backup & Restore → Download configuration** — save `config.xml`
- Make a deliberately bad change (e.g. delete the LAN IP), confirm it breaks connectivity, then restore from the saved `config.xml` — practice recovering from your own mistakes, and mirrors real operational practice of backing up config before changes

---

# Phase 2 — DMZ (Kali → pfSense → Ubuntu Desktop, Alpine DMZ Host)

Builds on Phase 1's topology by adding the third pfSense interface and DMZ host, per Section 8 of the design doc. Tests here specifically target the DMZ's containment property — that WAN can reach it, LAN can manage it, but it cannot reach LAN — rather than repeating the basic rule/NAT mechanics already covered in Phase 1.


## Firewall rules — the zone matrix

This is the actual exercise. Three rules define the DMZ's behavior:

| From → To | Rule | Why |
|---|---|---|
| **WAN → DMZ** | Pass, TCP, Source: `10.0.1.0/24`, Destination: DMZ address, Port 80/443 only | Public-facing service should be reachable, but only on the ports it actually serves |
| **DMZ → LAN** | **Block**, all protocols, Source: DMZ net, Destination: LAN net | The critical rule — this is what actually makes it a DMZ rather than just another open segment |
| **LAN → DMZ** | Pass, Source: LAN net, Destination: DMZ net | You manage/administer the DMZ host from the trusted side |

Add these under **Firewall → Rules → DMZ** (for DMZ-outbound rules) and **Firewall → Rules → WAN** (for the WAN→DMZ NAT-associated rule, likely via a port forward similar to Section 5 in the test-cases document, targeting `172.16.1.10` instead of the LAN server).


## 25. WAN → DMZ (reach the DMZ web server from Kali)

This is a NAT port-forward (same pattern as the LAN server exercise), not a plain firewall rule, since Kali needs to reach the DMZ host's private IP through pfSense's public-facing WAN address.

Firewall → NAT → Port Forward → Add

| Field                        | Value                          | Notes                              |
|-----------------------------|--------------------------------|------------------------------------|
| Interface                   | WAN                            |                                    |
| Protocol                    | TCP                            |                                    |
| Destination                 | WAN address                    | (default is fine)                  |
| Destination Port Range      | From: 8081 &nbsp;&nbsp; To: 8081   |       |
| Redirect target IP          | DMZ HOST                  | 172.16.1.10  |
| Redirect target port        | HTTP                            |    80                                |
| Description                 |     WAN to DMZ web                |                                    |
| Filter rule association     | Rule NAT     |  |

**Test from Kali:**
```bash
curl http://10.0.1.1:8081
```

### Troubleshooting Guide — DMZ nginx returning 404 (Alpine)

- **1. Initial test from Kali:**
  ```bash
  curl http://10.0.1.1:8081
  ```
  If this returns HTML content with a 404 Not Found page, it confirms the NAT/firewall chain is working end-to-end (the request reaches nginx and gets a response), but nginx has nothing to serve.

- **2. Check the web root on the DMZ host:**
  ```sh
  ls -la /var/www/html/
  ```
  If no files or directory exist, note that the Debian/Ubuntu-style web root path does not exist by default on an Alpine host.

- **3. Test locally with wget (handling BusyBox limitations):**
  ```sh
  wget -qO- http://localhost
  ```
  If this shows usage/options rather than output, Alpine's trimmed-down BusyBox wget is rejecting the combined GNU short-flag syntax. Use the flags separately:
  ```sh
  wget -O - http://localhost
  ```

- **4. Confirm nginx's actual configured root:**
  ```sh
  cat /etc/nginx/http.d/default.conf
  ```

- **5. Inspect the configured directory content:**
  ```sh
  ls -la /var/lib/nginx/html/
  ```

- **6. Verify root cause — default config returns 404 unconditionally:**

  Inspecting `default.conf` may show that Alpine's nginx package ships a default site config designed to return 404 regardless of files present in the web root. Adding an `index.html` alone will not resolve the issue.

- **7. Back up the original config before editing:**
  ```sh
  sudo cp /etc/nginx/http.d/default.conf /etc/nginx/http.d/default.conf.bak
  ```

- **8. Replace it with a proper static-serving config:**
  ```sh
  sudo vi /etc/nginx/http.d/default.conf
  ```
  ```nginx
  server {
      listen 80;
      server_name _;
      root /var/lib/nginx/html;
      index index.html;

      location / {
          try_files $uri $uri/ =404;
      }
  }
  ```

- **9. Create the index file:**
  ```sh
  sudo mkdir -p /var/lib/nginx/html
  echo "<h1>DMZ web server — reachable!</h1>" | sudo tee /var/lib/nginx/html/index.html
  ```

- **10. Validate config syntax before reloading:**
  ```sh
  sudo nginx -t
  ```

- **11. Reload nginx:**
  ```sh
  sudo rc-service nginx reload
  ```

- **12. Verify resolution locally and from Kali:**
  ```sh
  wget -O - http://localhost
  ```
  ```bash
  curl http://10.0.1.1:8081
  ```
  If successful, these commands will render the HTML content instead of returning a 404 page.

## 26. DMZ → LAN

LAN's existing default "Allow LAN to any" rule already covers it

## 27. LAN → DMZ (administer the DMZ host from your trusted side)

**Firewall → Rules → LAN → Add**

| Field                        | Value                          | Notes / Example                          |
|-----------------------------|--------------------------------|------------------------------------------|
| Action                      | Pass         |       |
| Interface                   | LAN           | Interface where traffic **enters**       |
| Address Family              | IPv4        |                              |
| Protocol                    | Any      |                                          |
| Source                      |LAN subnets      | Who is initiating the connection         |
| Source Port                 | Not Available                            |                |
| Destination                 | DMZ subnets      | Where the traffic is going               |
| Destination Port            |   Not Available      |                |
| Description                 | LAN to DMZ management    |                   |
| Log                         | Unchecked            |   

---

## Summary
| Rule | Needed? | Why |
|---|---|---|
| WAN → DMZ | Yes (NAT port-forward) | Nothing allows it by default — WAN is default-deny too |
| LAN → DMZ | Not needed | LAN's existing default "Allow LAN to any" rule already covers it |
| DMZ → LAN | Not needed — and shouldn't be added | Default-deny on the DMZ interface already blocks it; adding a rule here would only be relevant if you *do* want to permit something specific, which defeats the point |
| DMZ → anywhere else | Not needed | Same default-deny logic |

The DMZ tab staying completely empty is correct and sufficient — that emptiness **is** the containment mechanism, not a placeholder waiting for rules.

# Quick reference — Kali test commands

```bash
nc -zv <WAN IP> <port>              # single port check
nmap -p 1-1000 <WAN IP>             # port range scan
nmap -sS -p <port> <WAN IP>         # SYN scan
nmap -sA -p <port> <WAN IP>         # ACK scan
nmap -f -p <port> <WAN IP>          # fragmented
nmap -sU -p <port> <WAN IP>         # UDP scan
nmap -sV -A <WAN IP>                # aggressive/version scan (for IDS testing)
curl http://<WAN IP>:<port>         # HTTP through NAT
ping -c 4 <target IP>               # reachability
sudo tcpdump -i eth0 -n <filter>    # packet capture
```


