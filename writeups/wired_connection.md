
## **Problem**

On your Ubuntu system, the wired Ethernet interface (`eno1`) was **not connecting to the internet**, while Wi-Fi worked fine.

Symptoms observed:

* `ip a` showed `eno1` with an IP in the **169.254.x.x** range → this is an **automatic link-local address**, meaning DHCP failed.
* `nmcli device show eno1` confirmed **no gateway** and `ipv4.method: link-local`.
* Physical link was fine (`ethtool` showed `Link detected: yes`), so cables and NIC were working.

**Root cause:**

* The wired connection was configured to **use link-local addressing only**, not DHCP.
* As a result, Ubuntu never requested an IP from the router, so internet access failed.

---

## **Resolution**

1. **Turn off Wi-Fi** (to test wired independently):

```bash
nmcli radio wifi off
```

2. **Change the wired connection to use DHCP**:

```bash
sudo nmcli connection modify netplan-eno1 ipv4.method auto
sudo nmcli connection modify netplan-eno1 ipv4.never-default no
sudo nmcli connection modify netplan-eno1 autoconnect yes
```

3. **Restart the connection**:

```bash
sudo nmcli connection down netplan-eno1
sudo nmcli connection up netplan-eno1
```

4. **Verify the IP**:

```bash
ip a | grep eno1 -A3
```

Expected result: `eno1` now has a proper DHCP IP in the router’s range (e.g., 192.168.1.x).

5. **Optional: Make all future wired connections DHCP by default**:

```bash
sudo nmcli connection add type ethernet ifname "*" con-name "auto-eth" ipv4.method auto autoconnect yes
```

6. **Test connectivity**:

```bash
ping -c 3 8.8.8.8   # checks internet via IP
ping -c 3 google.com # checks DNS resolution
```

---

✅ **Outcome:**

* Wired interface now automatically requests DHCP and has full internet access.
* Future wired interfaces will default to DHCP, preventing 169.254 link-local issues.

---

If you want, I can also **draw a simple diagram showing why link-local vs DHCP caused the problem**, which makes it really easy to explain. Do you want me to do that?

