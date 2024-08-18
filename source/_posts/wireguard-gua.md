---
title: Assigning dynamic IPv6 GUA to WireGuard interface
categories: 笔记
tags: Networking
date: 2024-05-24 20:59:00
updated: 2024-08-18 13:13:00
---

**What you need:**

- A Linux server connected to a network on which the router advertises a dynamic `/64` IPv6 GUA prefix.
- A Windows client.
- A working WireGuard tunnel between the server and the client.

**What you get:** A routable IPv6 GUA assigned to the client's WireGuard interface, having the same prefix as advertised on the server's network, updated each time the prefix is updated.

**What this is for:** I use this because my ISP blocks incoming IPv6 traffic which is useful for seeding torrents. You may also use this to build a 6in4 tunnel.

<!-- more -->

## Set up the server

- Set up dynamic DNS (DDNS) for IPv6. Optionally set up DDNS for IPv4 if you need a 6in4 tunnel.

    For [Aliyun DNS], you can use my project [aliddns] to set up DDNS.

- Add the following lines to `/etc/sysctl.conf`:

    ```text /etc/sysctl.conf
    net.ipv6.neigh.default.proxy_delay=0
    net.ipv6.conf.all.proxy_ndp=1
    net.ipv6.conf.all.accept_ra=2
    net.ipv6.conf.all.forwarding=1
    ```

    Beware of the order of `accept_ra` and `forwarding`: reversing them may ruin your IPv6 connection. Run `sysctl -p /etc/sysctl.conf` to apply the changes.

- Install [`wg-ndproxy.sh`] to `/usr/local/sbin/` with `755` permissions. Modify the `eth_if` variable in it when using a WAN interface other than `eth0`.
- Install [`wg-ndproxy@.service`] to `/etc/systemd/system/`. Run `systemctl daemon-reload`.
- Modify your WireGuard configuration (say, `/etc/wireguard/wg0.conf`) as follows:
  - Add the following lines to the `[Interface]` section:

    ```text /etc/wireguard/wg0.conf
    Table = off
    PostUp = systemctl start wg-ndproxy@%i
    PreDown = systemctl stop wg-ndproxy@%i
    ```

  - Connect your favorite suffix with the prefix `2001:db8::/64` to form a full address. Append the address to the `AllowedIPs` field of a `[Peer]` section, for example:

    ```text /etc/wireguard/wg0.conf
    [Peer]
    PublicKey = T5Dkqdp0Ibb0o9HKyGLvvgdbTY3v4LI+up0P4YrEPFo=
    AllowedIPs = 192.168.1.2, fd5d:4bfe:356a::2, 2001:db8::dead:beef
    ```

    Repeat this step for as many peers as you need.

- Run `systemctl restart wg-quick@wg0` to reconfigure the WireGuard interface.
- Run `systemctl status wg-ndproxy@wg0` (optionally `ip -6 route`, `ip -6 neigh show proxy`, and `wg`) to see if the NDP proxy is working normally.
- The server should now be fully set up.

[Aliyun DNS]: https://www.aliyun.com/product/dns
[aliddns]: https://github.com/yescallop/aliddns
[`wg-ndproxy.sh`]: https://gist.github.com/yescallop/556ed8abb3042ae535f338581b471c4b#file-wg-ndproxy-sh
[`wg-ndproxy@.service`]: https://gist.github.com/yescallop/556ed8abb3042ae535f338581b471c4b#file-wg-ndproxy-service

## Set up the client

- Run the following command as administrator to enable script execution for WireGuard:

    ```cmd Command
    reg add HKLM\Software\WireGuard /v DangerousScriptExecution /t REG_DWORD /d 1 /f
    ```

- Install [`wg-dynconf.ps1`] to a path where only administrators have write access, say, `C:\Program Files\Utils\`.
- Prepare the arguments for the script:
  - If you need to periodically re-resolve the server's domain name and update the endpoint:
    - `Peer`: The server's public key.
    - `DnsName`: The server's DNS name to re-resolve as endpoint.
    - `DnsType`: DNS query type used for `DnsName`. Supported values: `A_AAAA` (default), `A`, `AAAA`.
    - `Port`: Port of endpoint.
  - If you need to assign an IPv6 GUA to the interface:
    - `DnsNameV6`: The server's DNS name to extract the IPv6 prefix from. Defaults to `DnsName`.
    - `PrefixLen`: IPv6 prefix length. Defaults to `64`.
    - `Suffix`: IPv6 suffix. Should be a full address.
    - `AddDefaultRouteV6`: Whether to add a `::/0` route with metric `1024` to the interface.
- Modify your WireGuard configuration. For example, add the following lines to the `[Interface]` section:

    ```text WireGuard configuration
    Table = off
    PostUp = start powershell -ExecutionPolicy Bypass -File "C:\Program Files\Utils\wg-dynconf.ps1" -Peer rTJmT+GCUbgfWbbcmfvCgbKxZZqLwQaWvIEX8g4+Iwc= -DnsName example.com -DnsType AAAA -Port 23333 -Suffix ::dead:beef -AddDefaultRouteV6
    PreDown = powershell -ExecutionPolicy Bypass -File "C:\Program Files\Utils\wg-dynconf.ps1" -Stop
    ```

- Start the WireGuard tunnel. Check the UI to see if the endpoint is set. Run `ipconfig` to see if an IPv6 GUA is assigned to the interface. Test the Internet connection by running `ping dns.google -S <assigned GUA>`, for example.

[`wg-dynconf.ps1`]: https://gist.github.com/yescallop/556ed8abb3042ae535f338581b471c4b#file-wg-dynconf-ps1

## Possible questions

- How do I set up a WireGuard tunnel? I might write a tutorial later.
- What about a client script for Linux? You can write one yourself.
- Do you see incoming connections when seeding? Yes, though it may depend on trackers and your client (mine is qBittorrent).
- Why not seed on the server instead? Because it has limited storage.
