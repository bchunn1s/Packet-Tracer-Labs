# DHCP Lab — Full Walkthrough

Here's the full breakdown of how I built this lab and why each step matters, not just the commands.

---

## Step 1: Topology

**Devices:** 1 Router, 1 Switch, 4 PCs, all connected with Copper Straight-Through cables.

I used the router itself as the DHCP server instead of a separate server device — Cisco IOS has that built in, and it's a really common setup for small networks/branch offices. The switch is there so all four PCs can share the same subnet, which DHCP needs to work.

---

## Step 2: Configure the Router's LAN Interface

Before the router can hand out IPs, it needs one of its own — this becomes the default gateway for every PC, and it's also what tells DHCP which subnet it's serving.

```
enable
configure terminal
interface gigabitethernet0/0
ip address 192.168.1.1 255.255.255.0
no shutdown
```

- `enable` / `configure terminal` — gets you into the modes you need to actually make changes
- `interface gigabitethernet0/0` — selects the port to configure (note: some routers use `fastethernet0/0` instead — always check with `show ip interface brief` rather than assuming)
- `ip address ...` — sets the gateway IP and subnet size
- `no shutdown` — turns the interface on. Cisco interfaces are off by default out of the box, which trips people up constantly

**Check it worked:**
```
show ip interface brief
```
Looking for the interface to show `up / up`.

---

## Step 3: Build the DHCP Pool

This defines the range of addresses the router can give out, plus the extra info (gateway, DNS) that comes with it.

```
configure terminal
ip dhcp pool LAN-POOL
network 192.168.1.0 255.255.255.0
default-router 192.168.1.1
dns-server 8.8.8.8
```

- `ip dhcp pool LAN-POOL` — creates the pool (name doesn't matter, just needs to be consistent)
- `network ...` — the subnet this pool covers
- `default-router` — the gateway address handed to clients
- `dns-server` — the DNS server handed to clients

**Check it:**
```
show ip dhcp pool
```

---

## Step 4: Exclude the Router's Own Address

By default, the whole subnet range is up for grabs — including .1, which I already gave to the router. Without excluding it, DHCP could try to hand that same address to a PC and cause a conflict.

```
configure terminal
ip dhcp excluded-address 192.168.1.1 192.168.1.9
```

I excluded a small block (.1–.9) instead of just .1, so there's room for anything else that might need a static IP later.

**Check it:**
```
show run | include excluded-address
```

**Worth noting:** `show ip dhcp pool` showed "Excluded addresses: 1" instead of 9 even after this was set correctly — turns out that's just a display quirk in Packet Tracer. The `show run` output had the actual correct config. Good reminder that running-config is the real source of truth, and summary commands can occasionally be a little off, especially in simulators.

---

## Step 5: Set the PCs to DHCP

On each PC: **Desktop → IP Configuration → DHCP**.

Results:

| PC | IP Address | Gateway | DNS |
|---|---|---|---|
| PC0 | 192.168.1.10 | 192.168.1.1 | 8.8.8.8 |
| PC1 | 192.168.1.11 | 192.168.1.1 | 8.8.8.8 |
| PC2 | 192.168.1.12 | 192.168.1.1 | 8.8.8.8 |
| PC3 | 192.168.1.13 | 192.168.1.1 | 8.8.8.8 |

Addresses go out in order starting from the lowest one available — which is why it started at .10 instead of .1, thanks to the exclusion.

---

## Step 6: Verify Everything

```
show ip dhcp binding
```

This shows every lease the router has handed out, matched to the device's MAC address. It's a solid go-to command for troubleshooting too — it tells you right away whether a device even made it far enough to talk to DHCP, which helps narrow down where a connection problem actually is.

```
IP address        Hardware address    Type
192.168.1.10      00D0.5840.C279      Automatic
192.168.1.11      0001.9671.D65C      Automatic
192.168.1.12      0001.6468.8509      Automatic
192.168.1.13      00E0.A3A8.5187      Automatic
```

Four different MAC addresses, sequential IPs, all "Automatic" — confirming the whole thing worked the way it was supposed to.

---

## What this lab covers
Cisco IOS CLI navigation, gateway/interface config, DHCP pool setup and exclusions, client-side DHCP behavior, and basic verification/troubleshooting habits.
