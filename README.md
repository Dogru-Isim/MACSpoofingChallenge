# MAC Spoofing Challenge

# Before Starting

## For Linux Users

> **Note:** You can use a Linux virtual machine, but make sure you use **Bridged Mode**.

Run `ip a` to see which interfaces your laptop currently has. For example, a laptop with `lo`, `wlan0`, and `tailscale0`:

```
~/redact1/redact2/week2_q2 > ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 24:ee:9a:99:4a:80 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.16/24 brd 192.168.1.255 scope global dynamic noprefixroute wlan0
       valid_lft 76879sec preferred_lft 76879sec
5: tailscale0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1280 qdisc fq_codel state UNKNOWN group default qlen 500
```

Then connect to the switch.

After connecting to the switch, run `ip a` again. Look at the output and find the name of the new interface (e.g. `eth0` or `enp0s3`) that was added after connecting to the switch. Note down the interface name and the IP address. For example, in the output below the interface is `enp0s3` and the IP is `192.168.123.37`.

```
6: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 34:ce:9a:93:ba:90 brd ff:ff:ff:ff:ff:ff
    inet 192.168.123.37/24 brd 192.168.123.255 scope global dynamic noprefixroute enp0s3   <<<---- SEE IP IN THIS LINE
       valid_lft 76879sec preferred_lft 76879sec
```

> If you can't find your IP address at this stage, you might need to assign a **static IP address**. The static IP must be in the range `192.168.123.0/24`. **Please choose something random to avoid IP collisions.**

## For Windows Users

If you are on Windows, you can do this challenge natively. You do **not** need Linux or Kali for it. Follow these steps.

> **Note:** If you are running Kali/Linux inside WSL, WSL cannot change the MAC address of your physical network card. Use the native Windows steps in this section, or run Kali in a VM using bridged networking (see the VM note at the end).

**Step 1:** Before plugging into the switch, list your adapters:

```powershell
Get-NetAdapter | Format-Table Name, InterfaceDescription, Status, MacAddress
```

Example output (a laptop with Wi-Fi and a Tailscale adapter):

```
Name       InterfaceDescription            Status       MacAddress
----       --------------------            ------       ----------
Wi-Fi      Intel(R) Wi-Fi 6 AX201 160MHz   Up           24-EE-9A-99-4A-80
Tailscale  Tailscale Tunnel                Up
```

**Step 2:** Connect to the switch.

**Step 3:** Run the same command again:

```powershell
Get-NetAdapter | Format-Table Name, InterfaceDescription, Status, MacAddress
```

Find the **new** adapter that appeared (usually named "Ethernet" or "Ethernet 2"). Note down its `Name` (the interface name).

Example output after connecting:

```
Name       InterfaceDescription            Status       MacAddress
----       --------------------            ------       ----------
Wi-Fi      Intel(R) Wi-Fi 6 AX201 160MHz   Up           24-EE-9A-99-4A-80
Tailscale  Tailscale Tunnel                Up
Ethernet   Realtek USB GbE Family Control  Up           34-CE-9A-93-BA-90
```

**Step 4:** Find the IP address of the new adapter. Replace `"Ethernet"` with the name you noted:

```powershell
Get-NetIPAddress -InterfaceAlias "Ethernet" -AddressFamily IPv4
```

> You can also run `ipconfig /all` and look at the "Ethernet adapter" section. The **IPv4 Address** line is your IP, and the **Physical Address** line is your current MAC.

Example: the interface is `"Ethernet"` and the IP is `192.168.123.37`.

> **Note:** If you can't find your IP address at this stage, you might need to assign a **static IP address on Windows**. The static IP must be in the range `192.168.123.0/24`. **Please choose something random to avoid IP collisions.**

---

# The Setup

You see a laptop connected to the switch. There's a service that this laptop is running on TCP port `1337`. If you can connect to that port, the laptop will respond with a message. However, the laptop only allows a specific MAC address to communicate with it.

---

# How to Win

## Linux Users: How to Win

Find the MAC address that the laptop allows (remember the slides — you might need to install Wireshark). Change your MAC address to it, then connect to the service running on TCP port `1337` and receive the message `ACCESS GRANTED`.

```bash
nc 192.168.123.100 1337
```

## Windows Users: How to Win

The goal is the same as Linux users: find the allowed MAC address (remember the slides — you might need to install Wireshark), change your MAC address to it, connect to TCP port `1337` on `192.168.123.100`, and receive the message `ACCESS GRANTED`.

On Windows, you probably don't have `nc` (netcat). For this, you need to install Nmap from [https://nmap.org/download](https://nmap.org/download). To beat the challenge, connect to port `1337` with `ncat` and receive the message `ACCESS GRANTED`.

---

## Troubleshooting

- **Using a VM (VMware, VirtualBox, Hyper-V) on a Windows host:** set the VM network to **Bridged** mode, pick your wired Ethernet adapter, and follow the original Linux instructions inside the VM.
