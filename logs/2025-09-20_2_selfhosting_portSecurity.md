<h3>Self Hosting Apps</h3>
Okay. So I have the slightest inkling of all of this stuff really. But the small things I've seen make me very excited, and are kind of the entire point of this project.

So I have a q&a with ChatGPT below that is great info.

But first:

I started with Cockpit, which does a bunch of stuff, but it's mainly a test for me.
Install was simple, I just followed the website and installed the right packages.

Then, I had to start it using `sudo systemctl enable --now cockpit.socket`. I'm not sure if this is necessary and the same for every server yet.

Now, this app defaults to using port 9090. So to access it, I need to go to https://192.168.1.50:9090/. But first, my ufw is auto denying all incoming connections. So first, I have to explicitly allow connections to port 9090 in my firewall with 
`sudo ufw allow 9090/tcp`, or even better, `sudo ufw allow from 192.168.1.0/24 to any port 9090 proto tcp`. 
Now, I have an ssh rule on port 22, and a rule on port 9090. Both only allow devices connected to my LAN through the router to see the open ports. Otherwise, if the port was forwarded for some reason, or if I brought my server to a public wifi, port 9090 could be accessed.

Now, here's where we get into network security. There are a lot of potential holes here anyway. For example, if there was another compromised device on the LAN, and a bad actor had root access on it, then they could access my ports that are locked down to the LAN. Network security goes super deep though and I'll spend some time focused on it later.


<h4>GPT discussion</h4>
<details>
    <summary>Question 1 - General info about self hosting</summary>
    <h5>Question:</h5>
So one thing I have a (very slight) glimpse into with these servers and self hosting, is like...
people self host apps or servers that basically give them a cool GUI that i can access over the web for various things?
So like, are there self hosting things I can setup to keep track of my servers stats? CPU temp, hard drive status,
RAID config, etc? Enlighten me here


<h5>Answer</h5>

## 🔹 What you’re talking about: Self-hosted GUIs

Instead of just managing by command line, you can install **web apps** that run locally on your server and expose a nice interface you can open in your browser (`http://192.168.1.50:xxxx`). Some common categories:

### 1. **System Monitoring & Stats**

* **Netdata** → one of the easiest & prettiest. Real-time CPU, RAM, disk, network, temps, services.
* **Glances (with web mode)** → lighter weight, nice overview of CPU, RAM, disk, network, sensors.
* **Prometheus + Grafana** → the “pro” stack. Grafana gives beautiful dashboards, Prometheus collects metrics. More setup, but endless customization.

### 2. **Drive & RAID Monitoring**

* **SMART monitoring** (`smartmontools`) → check HDD/SSD health (reallocated sectors, temps, lifespan).
* **mdadm** → if you build a software RAID on Linux, it has its own monitoring and alerting.
* **OpenMediaVault (OMV)** → Debian-based NAS/server OS with a full GUI for RAID, disks, users, shares, Docker, etc. Many people run OMV on top of Debian or instead of vanilla Debian for home servers.

### 3. **Server Management GUIs**

* **Cockpit** → web-based server manager (comes from Red Hat, works on Debian). Lets you see CPU, memory, update packages, manage services, look at logs, even manage storage/RAID. Install, go to `https://server-ip:9090`, and you’ve got a full dashboard.
* **Webmin** → older but still powerful “control panel” for managing Linux machines.

---

## 🔹 Example Setup for You

If you want to start simple on your Debian server:

1. Install **Cockpit**:

   ```bash
   sudo apt update
   sudo apt install cockpit -y
   sudo systemctl enable --now cockpit
   ```

   → Then go to: `https://192.168.1.50:9090` in your browser.
   You’ll get a full server dashboard: CPU temp, memory, storage, journal logs, network usage, even manage RAID (`mdadm`).

2. Add **Netdata** for eye candy:

   ```bash
   bash <(curl -Ss https://my-netdata.io/kickstart.sh)
   ```

   → Then visit `http://192.168.1.50:19999`.
   You’ll see a live “Mission Control” of your server (CPU spikes, disk I/O, network throughput, etc.).

3. If you go RAID (via `mdadm`), you can configure it to **email you** if a drive goes bad. Or let Cockpit show RAID health in the GUI.

---

## 🔹 Why people love this

Once you self-host one GUI, you get hooked — then you start adding others:

* **Nextcloud** → your own Google Drive/Dropbox.
* **Plex/Jellyfin** → your own Netflix.
* **Home Assistant** → your own smart home hub.
* **Portainer** → a GUI for managing Docker containers if you start hosting apps.

And then you usually want **one dashboard to see them all** → that’s where tools like **Dashy** or **Heimdall** come in (self-hosted startpages with links to all your apps).

---

✅ So yes: you can absolutely set up GUIs to monitor CPU temp, drives, RAID config, network traffic, etc.
For your immediate needs, I’d suggest starting with **Cockpit** (general system + storage) and **Netdata** (beautiful real-time stats).

---
</details>



<details>

<summary>Question 2 - Technical specifics of how each hosted program is a process bound to a port</summary>

  <h5>Question</h5>
  
  Okay wow cool. So how does hosting work? Like once I start hosting all of these locally, are they just running 24/7 no issues? I could be hosting like 20 things, all on different ports, and they'll run fine? (As long as they aren't using too many resources of course). And how can I see them all on the cli?
  
  <h5>Answer</h5>
  
  Yes! You’ve got it 💯 — that’s basically what “self-hosting” means: your box runs apps 24/7, and each app usually “listens” on a different port or address. Let’s unpack this.
  
  ---
  
  ## 🔹 How hosting works (locally)
  
  * Your server runs **processes** (programs).
  * Each service binds to a **port** on an IP (e.g. `192.168.1.50:9090` for Cockpit, `:19999` for Netdata, `:22` for SSH).
  * Your server’s IP + port is how you reach that service from your LAN.
  
  As long as:
  
  * The service is running,
  * The port isn’t blocked by a firewall,
  * No two apps fight for the same port,
    👉 you can run **20+ services side by side**, all accessible in a browser or client.
  
  ---
  
  ## 🔹 Examples
  
  * `192.168.1.50:22` → SSH
  * `192.168.1.50:9090` → Cockpit
  * `192.168.1.50:19999` → Netdata
  * `192.168.1.50:32400` → Plex (if installed)
  * `192.168.1.50:8989` → Sonarr (TV automation, if installed)
  
  ---
  
  ## 🔹 How to see them in the CLI
  
  Several useful commands:
  
  ### 1. Show **open ports & listening services**
  
  ```bash
  sudo ss -tulpn
  ```
  
  or
  
  ```bash
  sudo netstat -tulpn   # install with: sudo apt install net-tools
  ```
  
  This will show:
  
  * Protocol (tcp/udp)
  * Local address + port
  * PID/program name
  
  Example output:
  
  ```
  tcp   LISTEN  0  4096  0.0.0.0:22     *:*    users:(("sshd",pid=753,fd=3))
  tcp   LISTEN  0  4096  0.0.0.0:9090   *:*    users:(("cockpit-ws",pid=1200,fd=5))
  tcp   LISTEN  0  4096  0.0.0.0:19999  *:*    users:(("netdata",pid=1345,fd=4))
  ```
  
  ### 2. See **running processes**
  
  ```bash
  htop
  ```
  
  (interactive, shows CPU/RAM usage per process; install with `sudo apt install htop`).
  
  ### 3. Check if a service is enabled at boot
  
  ```bash
  systemctl list-unit-files --type=service | grep enabled
  ```
  
  ### 4. Check logs of a service
  
  ```bash
  journalctl -u cockpit
  journalctl -u ssh
  ```
  
  ---
  
  ## 🔹 Managing many services
  
  When you start having 10–20 self-hosted apps:
  
  * **Option A: run them all as system services** (each has a `systemd` unit, like Cockpit).
  * **Option B: use Docker** → most home-labbers eventually containerize apps because it’s easier to run, update, and isolate dozens of services. (Portainer = GUI to manage Docker containers).
  
  ---
  
  ✅ So yes: you *can* host 20 apps side-by-side, each with its own port. They’ll happily run 24/7 as long as your box has enough CPU/RAM/storage.
  
  ---
  
  👉 Want me to show you what a **typical self-hoster’s `ss -tulpn` output** looks like with 10+ services, so you can see what your server might look like once you’re running multiple apps?

</details>


<details>

<summary>Question 3 - How network exposure works, how holes get opened even on seemingly unexposed ports, and how to shore up</summary>

  <h5>Question</h5>
  
  Okay so if I didn't do `sudo ufw allow from 192.168.1.0/24 to any port 9090 proto tcp`, then what could a potential attack look like the way things are setup right now? How could anyone access it anyway when it's only showing up on my ip?

  <h5>Answer</h5>

  Great question — this gets to the heart of how network exposure actually works. Short answer first, then a clear list of possible attack paths and concrete mitigations you can run *right now*.

### Short answer

If your router does **not** forward port `9090` to your server, then *nobody on the public Internet* can connect to `https://YOUR-PUBLIC-IP:9090` — the router will drop it. However there are other ways an attacker could reach that port:

* someone already on your **local network** (Wi-Fi or wired) can connect to `192.168.1.50:9090`
* a malicious device or compromised machine already inside your network can reach it
* if your router has **UPnP** or you enabled port forwarding by accident, the port may be reachable from the Internet
* ISP remote management or Carrier-Grade NAT (rarely) can also expose ports if misconfigured
* any software vulnerability in Cockpit could be abused by anyone who *can* connect

So “it’s only on my IP” means “it’s listening on that local IP,” but *who can route to that IP* is what matters.

---

## Possible attack scenarios (what an attacker could do)

1. **Internet attacker via port-forwarding**

   * Router has a forward for 9090 → attacker scans the Internet, finds your public IP:9090, exploits a vulnerability or brute-forces credentials.

2. **Compromised device on your LAN**

   * A phone/PC on your Wi-Fi gets infected → it can access 192.168.1.50:9090 directly and attempt exploitation or credential stuffing.

3. **Rogue Wi-Fi guest or neighbor**

   * If your Wi-Fi is insecure or guest isolation is off, a guest/neighbor could connect and reach your server.

4. **Router features (UPnP / remote admin)**

   * UPnP can allow apps on the LAN to open ports on the router automatically. Router remote admin (by ISP or vendor cloud) could present risks.

5. **Insider/physical access**

   * Someone with physical access or Ethernet port in your house could plug in and reach services.

6. **Credential reuse / weak admin password**

   * If Cockpit uses a weak/known password, an attacker that reaches the port can log in even without exploiting code vulnerabilities.

---

## What a successful attack *could* look like

* Discover the open web UI → try default/weak credentials → gain admin → install malware, exfiltrate data, pivot to other devices.
* Exploit a software bug in Cockpit to run commands remotely (less common but possible with unpatched versions).
* Reuse credentials found in a breach to log in.

---

## Concrete mitigations — what I recommend you do now

(I’ll include exact commands you can run)

### 1) Make Cockpit LAN-only (tighten your UFW rules)

Remove the existing “allow anywhere” rule for 9090, then allow only your LAN:

```bash
# Remove existing allow-anywhere rule for 9090
sudo ufw delete allow 9090/tcp

# Add rule: only accept connections from 192.168.1.0/24
sudo ufw allow from 192.168.1.0/24 to any port 9090 proto tcp

# Confirm
sudo ufw status numbered
```

This closes the hole from the Internet while keeping your browser access on the LAN.

---

### 2) Confirm router has no accidental forwarding / turn off UPnP

* Log into your router UI and check port forwarding and UPnP settings. Disable UPnP if you don’t need it.
* Disable any “remote management” features on the router unless you explicitly need them.

---

### 3) Harden Cockpit / SSH / Admin accounts

* Ensure SSH keys are used and `PasswordAuthentication no`.
* Use strong, unique passwords for any accounts that can log into Cockpit.
* Limit login accounts (use `AllowUsers` in `sshd_config` or Cockpit’s own policies).

---

### 4) Keep software updated

```bash
sudo apt update && sudo apt upgrade
# Consider enabling unattended-upgrades for security updates
sudo apt install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

---

### 5) Use fail2ban or rate-limit access

fail2ban can ban repeated login attempts even on web UIs that log to `auth.log`. Install and enable it (you already have the commands earlier) and ensure it monitors Cockpit logs if applicable.

---

### 6) Use a VPN for remote access (recommended)

Instead of exposing Cockpit to the Internet, use Tailscale or a self-hosted WireGuard server. Then you never open ports publicly — you connect to your home network securely and reach Cockpit as if you’re on the LAN.

---

### 7) Monitor access

* Check logs: `sudo journalctl -u cockpit -f` and `sudo tail -f /var/log/auth.log`
* Consider Netdata/Cockpit alerts to notify you of unusual activity.

---

## How to verify whether the port is reachable from the Internet (test)

From a device *outside your LAN* (phone on cellular or a friend’s computer), try:

```bash
# replace x.x.x.x with your public IP
curl -vk https://x.x.x.x:9090/
```

If it times out → likely not forwarded. If it connects (even with TLS error) → it’s reachable from the Internet and you should lock it down immediately.

You can also use external scanners like `canyouseeme.org` or `shields` to test a specific port.

---

## TL;DR / Final checklist

* If router is **not** forwarding 9090, external attackers can’t reach Cockpit — but local attackers can.
* Best posture: keep Cockpit LAN-only (`ufw allow from 192.168.1.0/24 to any port 9090`), disable UPnP, keep software updated, use strong unique passwords, use SSH keys, and prefer VPN for remote access.
* If you want, I can run you through the exact `ufw` commands (already above) and show how to check router UPnP and port-forward rules — or help you set up Tailscale/WireGuard so you can access Cockpit from anywhere safely. Which would you like to do next?
</details>


<h3>VPN - Wireguard</h3>

Okay this isn't working currently. I spent a long time trying to set this up so that I can vpn into my server from anywhere, but it's not working currently.
I followed this guide https://wiki.debian.org/WireGuard, but when my Window client VPN is active, I can't even see the server. I'll come back to this.


<h3>Setting up Docker and some containers</h3>

So Docker was an easy install, just followed https://docs.docker.com/engine/install/debian/.
Now, containers are easy to setup. I create a new folder for the application I want to make a Docker container of at ~/folder. Then, within that folder, I make a docker-compose.yml file with the right yaml code.

So I decided to use Homepage as a homepage for other containers and whatever else.

docker-compose.yml:
```yaml
services:
  homepage:
    image: ghcr.io/gethomepage/homepage:latest
    container_name: homepage
    environment:
      HOMEPAGE_ALLOWED_HOSTS: gethomepage.dev # required, may need port. See gethomepage.dev/installation/#homepage_allowed_hosts
      PUID: 1000 # optional, your user id
      PGID: 1000 # optional, your group id
    ports:
      - 3000:3000
    volumes:
      - /path/to/config:/app/config # Make sure your local config directory exists
      - /var/run/docker.sock:/var/run/docker.sock:ro # optional, for docker integrations
    restart: unless-stopped
```

Then, each container has its own structure. Homepage uses a folder called config, ~/homepage/config, and has yaml files and other things inside for you to customize the homepage and link to other containers using the urls.

Then, to start the container, I run `docker compose up`, with an optional `-d` flag. Without `-d`, the terminal gets locked up as it shows logs from the container. To end the service, it's a simple `docker compose down`.

There're also docker run commands, which I think are just temporary containers? So like you don't use a docker compose file, which means you can't make a persistent container. It's more for testing I think.





