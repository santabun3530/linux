# DNS Resolution Process on Linux: From `amazon.com` to Server Connection

## 🔹 Step 1: Application Request (Browser, curl, ping)

When you type `amazon.com` in your browser or use a command-line tool like `ping` or `curl`, the application calls the **glibc resolver library** to resolve the hostname.

---

## 🔹 Step 2: nsswitch.conf → Resolution Order

The glibc resolver checks `/etc/nsswitch.conf` for the `hosts:` line:

```
hosts: files mdns4_minimal [NOTFOUND=return] dns mymachines
```

This means hostname resolution occurs in this order:

1. `files` → `/etc/hosts`
2. `mdns4_minimal` (mainly for .local addresses)
3. `dns` → `/etc/resolv.conf`
4. `mymachines` → local VM/container resolution

---

## 🔹 Step 3: `/etc/hosts` File

The system first checks the `/etc/hosts` file:

```bash
cat /etc/hosts
```

👉 If there's an entry for `amazon.com`, it uses that IP directly. **No DNS server is needed.**

---

## 🔹 Step 4: mDNS (Multicast DNS)

If the hostname ends with `.local` (e.g., `printer.local`), Avahi/Bonjour mDNS protocol is used.  
But `amazon.com` ends with `.com`, so mDNS is skipped.

---

## 🔹 Step 5: DNS Query via `/etc/resolv.conf`

The system reads `/etc/resolv.conf` to get the list of nameservers:

```bash
cat /etc/resolv.conf
```

Example output:
```
nameserver 8.8.8.8
nameserver 1.1.1.1
```

👉 The resolver library now sends a DNS query (UDP/TCP port 53) to the specified DNS server (e.g., Google or Cloudflare): *"Give me the A/AAAA record for amazon.com."*

---

## 🔹 Step 6: Nameserver → Recursive Query Process

For example, if using 8.8.8.8, it performs:

1. Asks the Root DNS servers → "Who manages `.com`?"
2. The `.com` TLD servers respond → "Here are the authoritative servers for `amazon.com`"
3. Amazon's authoritative DNS servers reply → "The IP for `amazon.com` is 205.251.242.103" (example)
4. The nameserver returns this IP to your machine

---

## 🔹 Step 7: Caching

- The DNS response is cached in the **OS cache** (e.g., `systemd-resolved`) for a period based on TTL.
- Subsequent requests for `amazon.com` may not require querying external nameservers again.

---

## 🔹 Step 8: Browser → IP Connection

Your browser now establishes a **TCP connection (port 443)** to the obtained IP (e.g., `205.251.242.103`) → performs TLS handshake → loads Amazon's website via HTTPS.

---

## 🔹 Summary: File/Directory Flow

1. **Application → glibc resolver library**
2. **/etc/nsswitch.conf** → determines resolution order
3. **/etc/hosts** → local static mapping
4. **mDNS (Avahi)** → for `.local` addresses
5. **/etc/resolv.conf** → list of DNS servers
6. **DNS server (8.8.8.8, etc.)** → recursive resolution
7. **systemd-resolved cache** (if enabled)
8. **Browser → connects to Amazon's server**

---

## 🔹 DNS Client Resolver vs. DNS Server

### DNS Client Resolver
- This is your **Linux OS's internal library/service** (glibc resolver, systemd-resolved, nscd).
- Its job: When a program (browser, curl, ping) needs to resolve a hostname (`amazon.com`), the resolver:
  1. Checks `/etc/nsswitch.conf` for resolution order
  2. Checks `/etc/hosts` for local mappings
  3. Uses `/etc/resolv.conf` to query external DNS servers

👉 Simply put: **The DNS client resolver is your system's "local agent" that fetches name-to-IP mappings from DNS servers.**

### DNS Server
- DNS server is separate server software (e.g., **BIND9, Unbound, dnsmasq, PowerDNS**).
- Its job: Maps hostnames to IPs and answers client resolver queries.
- Examples:
  - Your **ISP's DNS**
  - **Google DNS (8.8.8.8)**
  - **Cloudflare DNS (1.1.1.1)**
  - Or you can install `bind9` on your Linux server to create your own DNS server.

👉 Simply put: **A DNS server is the "big database + service" that holds the actual name-to-IP mappings and serves clients.**

---

## 🔹 Comparison with Nginx

- **Nginx (Web Server)**
  - Client (browser) requests: *"Give me /index.html"*
  - Nginx serves content from its config or disk.

- **DNS Server**
  - Client resolver queries: *"Give me the IP for amazon.com"*
  - DNS server responds from its cache/authoritative zone.

- **DNS Client Resolver (on Linux)**
  - Browser says: *"Resolve amazon.com for me"*
  - Resolver checks local config (`/etc/hosts`, `/etc/resolv.conf`) → queries DNS server → returns answer to browser.

---

## 🔹 Why Run Your Own DNS Server (BIND9, etc.)?

While your machine's resolver can query external DNS servers, there are reasons to run your own DNS server:

### (a) Control for Your Network
- Example: You're a sysadmin at a company.
- You want `internal.company.local` names to resolve only within your office.
- Google DNS doesn't know these mappings → you need a **local DNS server**.

### (b) Caching / Performance
- If many users query DNS simultaneously, querying external servers repeatedly is slow.
- Local BIND9 caches responses → serves them faster.

### (c) Security / Filtering
- Use a local DNS server to blacklist domains (e.g., ads.example.com).
- Enforce DNSSEC.

### (d) Authoritative Role
- If you manage a public domain (e.g., `mycompany.com`), you need an **authoritative DNS server**.
- This tells everyone on the Internet: *"The real IP for mycompany.com is this."*
- This cannot be done via `/etc/resolv.conf`; it requires proper DNS server software (BIND9).

---

## 🔹 Flow with Local BIND9 Server

If you run a local BIND9 server, your `/etc/resolv.conf` might look like:

```
nameserver 127.0.0.1
```

With `/etc/nsswitch.conf`:
```
hosts: files dns
```

1. Browser/ping/curl → asks glibc resolver: *"Resolve amazon.com"*
2. Resolver first checks `/etc/hosts` → if not found
3. Then queries the nameserver (127.0.0.1 → local BIND9)
4. BIND9 server responds recursively or authoritatively
5. Resolver returns the IP to the application (browser)

---

## 🔹 ASCII Flow Diagram

```
+---------+    +----------------+    +------------+    +-------------+
| Browser | -> | glibc resolver | -> | /etc/hosts | -> | (if found)  |
+---------+    +----------------+    +------------+    +-------------+
                                     |
                                     | (not found)
                                     v
                             +-----------------+
                             | /etc/nsswitch.conf |
                             +-----------------+
                                     |
                                     v
                             +-----------------+
                             | /etc/resolv.conf |
                             +-----------------+
                                     |
                                     v
                             +-----------------+
                             |  DNS Server     |
                             | (8.8.8.8 or     |
                             |  local BIND9)   |
                             +-----------------+
                                     |
                                     v
                             +-----------------+
                             | Amazon Server   |
                             | (205.251.242.103)|
                             +-----------------+
```

---

This document explains the complete DNS resolution process on Linux, the role of key files and directories, and the difference between DNS client resolvers and DNS servers.
