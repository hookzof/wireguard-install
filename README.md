# WireGuard installer

**This project is a bash script that aims to setup a [WireGuard](https://www.wireguard.com/) VPN on a Linux server, as easily as possible!**

WireGuard is a point-to-point VPN that can be used in different ways. Here, we mean a VPN as in: the client will forward all its traffic trough an encrypted tunnel to the server.
The server will apply NAT to the client's traffic so it will appear as if the client is browsing the web with the server's IP.

The script supports both IPv4 and IPv6. Please check the [issues](https://github.com/angristan/wireguard-install/issues) for ongoing development, bugs and planned features!

WireGuard does not fit your environment? Check out [openvpn-install](https://github.com/angristan/openvpn-install).

## Requirements

Supported distributions:

- Ubuntu
- Debian
- Fedora
- CentOS
- Arch Linux

I recommend these cheap cloud providers for your VPN server:

- [Vultr](https://goo.gl/Xyd1Sc): Worldwide locations, IPv6 support, starting at $3.50/month
- [PulseHeberg](https://goo.gl/76yqW5): France, unlimited bandwidth, starting at €3/month
- [Digital Ocean](https://goo.gl/qXrNLK): Worldwide locations, IPv6 support, starting at $5/month

## Usage

Download and execute the script. Answer the questions asked by the script and it will take care of the rest.

```bash
curl -O https://raw.githubusercontent.com/angristan/wireguard-install/master/wireguard-install.sh
chmod +x wireguard-install.sh
./wireguard-install.sh
```

It will install WireGuard (kernel module and tools) on the server, configure it, create a systemd service and a client configuration file.

To generate more client files, run the following:

```sh
./wireguard-install.sh add-client
```

Make sure you choose different IPs for you clients.

Contributions are welcome!

## Install the Cloudflared DoH Server

1. [Download the Cloudflared service](https://developers.cloudflare.com/argo-tunnel/downloads/) for your Linux platform. For Ubuntu/Debian download the .deb package:

```bash
wget https://bin.equinox.io/c/VdrWdbjqyF/cloudflared-stable-linux-amd64.deb
```

2. Install the package:

```bash
dpkg -i cloudflared-stable-linux-amd64.deb
```

3. Confirm that it installed correctly:

```bash
cloudflared --version
cloudflared version 2020.7.1 (built 2020-07-06-1751 UTC)
```

4. Configure the service to use Cloudflare’s 1.1.1.1 and 1.0.0.1 resolvers:

```bash
mkdir -p /etc/cloudflared
cat << EOF > /etc/cloudflared/config.yml
proxy-dns: true
proxy-dns-upstream:
    - https://1.1.1.1/dns-query
    - https://1.0.0.1/dns-query
    - https://2606:4700:4700::1111/dns-query
    - https://2606:4700:4700::1001/dns-query
EOF
```

5. Install the service:

```bash
sudo cloudflared service install --legacy
```

6. The service should now be running on localhost. Test it by querying for a DNS record:

```bash
dig +short @127.0.0.1 tau.gr AAAA
2606:4700:30::681b:9ecf
2606:4700:30::681b:9fcf
```

## Configure /etc/sysctl.conf

```bash
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
net.ipv4.icmp_echo_ignore_all = 1
net.ipv4.conf.all.route_localnet = 1
```

## Configure Wireguard Server

Edit your Wireguard config /etc/wireguard/wg0.conf and append the following to the PostUp and PostDown commands:

```bash
PostUp = <other PostUp commands>; iptables -A PREROUTING -t nat -i %i -p udp --dport 53 -j DNAT --to-destination 127.0.0.1:53
PostDown = <other PostDown commands>; iptables -D PREROUTING -t nat -i %i -p udp --dport 53 -j DNAT --to-destination 127.0.0.1:53
```

Save the config file and restart Wireguard for the new changes to take effect:

```bash
wg-quick down wg0
wg-quick up wg0
```

## Configure Wireguard Clients

```bash
DNS = 10.0.0.1
```

[Thank you for DOH + WireGuard setup guide!](https://tau.gr/posts/2019-03-03-set-up-cloudflared-ubuntu-wireguard/)
