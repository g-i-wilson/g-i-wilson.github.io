# VPN Gateway Router

## Install & configure wireguard

(installation steps here)

Configuration file in `/etc/wireguard`

```
[Interface]
PrivateKey = <LOCAL_PRIVATE_KEY>
Address = <VPN_IP_ADDR>/32
DNS = 84.200.69.80, 84.200.70.40

# Reverse of what is on the cloud server
PostUp = iptables -A FORWARD -i eth0 -j ACCEPT; iptables -A FORWARD -o eth0 -j ACCEPT; $
PostDown = iptables -D FORWARD -i eth0 -j ACCEPT; iptables -D FORWARD -o eth0 -j ACCEPT$

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
Endpoint = <SERVER_WAN_ADDR>:<SERVER_PORT>
AllowedIPs = 0.0.0.0/0, ::/0
PersistentKeepalive = 25

```

WireGuard will tunnel through whichever interface (e.g. `wlan0`) has an internet connection, and `eth0` interface will forward to that tunnel.


## Determine how network interfaces will be configured
- Raspberry Pi
	- Old style: `/etc/network/interfaces`
		- Will need to **disable dhcpcd**: `sudo systemctl disable dhcpcd`
	- Default style: using `dhcpcd.service`
	- Systemd style: using `networkd`
		- *more to be added here*

## Configure network interfaces
### Old style

`/etc/network/interfaces.d/wlan0`


```
allow-hotplug wlan0
iface wlan0 inet dhcp
wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

```

`/etc/network/interfaces.d/eth0`


```
auto eth0
iface eth0 inet static
    address 10.1.1.1
    netmask 255.255.255.0

```

## Install & configure isc-dhcp-server

(installation steps here)

`/etc/default/isc-dhcp-server`

```
INTERFACESv4="eth0"
INTERFACESv6=""

```

`/etc/dhcp/dhcpd.conf `

```
# option definitions common to all supported networks...
option domain-name "<LOCAL_HOSTNAME>.local";
option domain-name-servers 84.200.69.80, 84.200.70.40;

default-lease-time 600;
max-lease-time 7200;

# The ddns-updates-style parameter controls whether or not the server will
# attempt to do a DNS update when a lease is confirmed. We default to the
# behavior of the version 2 packages ('none', since DHCP v2 didn't
# have support for DDNS.)
ddns-update-style none;

# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
authoritative;

# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
#log-facility local7;


subnet 10.1.1.0 netmask 255.255.255.0 {
  range 10.1.1.11 10.1.1.254;
  option routers 10.1.1.1;
  option subnet-mask 255.255.255.0;
}

```

