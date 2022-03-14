
```
auto lo
iface lo inet loopback

iface lo inet6 loopback

auto enp0s31f6
iface enp0s31f6 inet static
        address X.X.X.254/27
        gateway X.X.X.225
        up route add -net X.X.X.224 netmask 255.255.255.224 gw X.X.X.225 dev enp0s31f6
# route X.X.X.224/27 via X.X.X.225

iface enp0s31f6 inet6 static
        address X:X:X:X::2/64
        gateway fe80::1

auto vmbr0
iface vmbr0 inet static
        address 10.10.10.1/24
        bridge-ports none
        bridge-stp off
        bridge-fd 0

        post-up   echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up   iptables -t nat -A POSTROUTING -s '10.10.10.0/24' -o enp0s31f6 -j MASQUERADE
        post-down iptables -t nat -D POSTROUTING -s '10.10.10.0/24' -o enp0s31f6 -j MASQUERADE

        # SERVIDOR NGINX
        post-up iptables -t nat -A PREROUTING -p tcp -d X.X.X.254 --dport 80 -i enp0s31f6 -j DNAT --to-destination 10.10.10.10:80
        post-up iptables -t nat -A PREROUTING -p tcp -d X.X.X.254 --dport 443 -i enp0s31f6 -j DNAT --to-destination 10.10.10.10:443


        post-up   iptables -t raw -I PREROUTING -i fwbr+ -j CT --zone 1
        post-down iptables -t raw -D PREROUTING -i fwbr+ -j CT --zone 1
```
