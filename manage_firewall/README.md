# Manage Firewall

Wrapper script around UFW (Uncomplicated FireWall; Linux) for fast, easy and simple firewall management via CLI.

## Supported Options (CLI)

`-l|--lan`
* enables firewall rules - "LAN" profile (user-specified)
* allows specified inbound and outbound LAN traffic

`-v|--vpn`
* enables firewall rules - "VPN" profile (user-specified)
* allows only specified outbound VPN traffic

`-b|--block`
* enables firewall rules - "block" profile
* denies ALL inbound and outbound traffic

`-d|--dis`
* disables firewall
* allows ALL inbound and outbound traffic

`-m|--msg`
* displays firewall log messages
* used for troubleshooting and tuning firewall rules

`-s|--stat`
* displays current firewall status and currently enforced firewall rules

## Setup

1. download script to target OS

    ```
    sudo su
    wget https://raw.githubusercontent.com/lighthouseitsecurity/linux/main/manage_firewall/manage_firewall -O /tmp/manage_firewall && chmod 700 /tmp/manage_firewall && chown root:root /tmp/manage_firewall && mv /tmp/manage_firewall /usr/local/sbin/manage_firewall
    ```

2. confirm following paths match the ones of binaries on target OS:

    `/bin/bash`

    `/bin/echo`

    `/usr/sbin/ufw`

    `/bin/rm`

    `/bin/sed`

    `/bin/journalctl`

    `/bin/grep`

    * **NOTE**: the paths listed above should match the paths of preinstalled binaries on most Linux distributions
      * in case any does not, update the script accordingly

3. configure user-specified firewall rule profiles - LAN; VPN

    *(modify script - specify local network access (LAN) firewall rules under function `setup_fw_rules_lan`)*

    *(modify script - specify remote access (VPN) firewall rules under function `setup_fw_rules_vpn`)*

    * **NOTE**: if required, add and configure additional firewall rule profiles
      * requires further script modifications (process not documented; out of scope)

## Firewall Rule Creation Tips

* LAN profile - specify all legitimate traffic for communication with/via the local network, depending on the purpose of the target OS
  * allow all desired outgoing traffic outbound (i.e. `out`) on the local network interface (e.g. `enp1s0`)
    * typical examples include:
      * DNS traffic
      * NTP traffic
      * OS update traffic
      * git (application update traffic)
      * web traffic
      * email traffic
      * printer traffic
  * in case any incoming traffic is required, allow it inbound (i.e. `in`) on the local network interface (e.g. `enp1s0`)
    * any listening network service (TCP/UDP) on the target OS, required to be accessible from other systems, within the local (or another) network
* VPN profile - specify all legitimate traffic for communication via the remote network (i.e. requiring anonymity), depending on the purpose of the target OS
  * allow all desired outgoing traffic outbound (i.e. `out`) on the VPN tunnel network interface (e.g. `tun0`)
    * typical examples include:
      * web traffic
      * VoIP traffic
      * chat/messenger traffic
      * SSH traffic (remote access)
  * allow all VPN traffic outbound (i.e. `out`) on the local network interface (e.g. `enp1s0`)
    * specify the whole VPN subnet (i.e. `X.X.X.X./YY`; VPN provider dependent) as the destination
* having the same rules specified on both profiles may or may not make sense, depending on the requirements/scenario
  * e.g., in case you want to access web pages/surf via both profiles, it makes sense to put them in both profiles (i.e. LAN and VPN)
    * it may make sense to further refine the rules and allow access to only specific resources for each profile (**NOTE**: creates additional workload and rule management overhead)
  * e.g., in case you want to access some service anonymously (e.g. VoIP), it does not make sense to put it on both profiles (i.e. use only VPN)
* it makes sense to use encrypted DNS (e.g. DNS over TLS) for traffic over VPN (to preserve anonymity)
  * route and allow all DNS over TLS traffic (i.e. TCP/853) over the VPN tunnel network interface (e.g. `tun0`)
* overall, keeping everything as simple as possible is usually the best approach (i.e. do not (over)complicate)
  * having a large number of firewall rules makes them harder to understand and maintain, and also has a negative impact on the firewall's performance (i.e. slower traffic)
    * where possible, strive for a smaller number of more general firewall rules
  * be as specific as possible, but do not exaggerate
    * for single targets, specify the target (i.e. `X.X.X.X/32`) as the destination
    * for a larger number of single neighboring targets, use a subnet (e.g. `X.X.X.X/24`) as the destination
    * where possible, consolidate multiple neighboring subnets into one larger subnet (e.g. `X.X.X.X/20`)
  * in case it is not possible to be specific, allow traffic to any destination (i.e. `0.0.0.0/0`)
  * if/where required, further build up from the initial rules and increase their complexity (i.e. further refine them) incrementally

## Firewall Rule Examples

* LAN profile

    * allow all outgoing web traffic (web surfing) - TCP

        ```
        /usr/sbin/ufw allow out on enp1s0 to 0.0.0.0/0 port 443 proto tcp > /dev/null
        ```

    * allow all outgoing web traffic (web surfing) - QUIC

        ```
        /usr/sbin/ufw allow out on enp1s0 to 0.0.0.0/0 port 443 proto udp > /dev/null
        ```

    * allow all outgoing DNS traffic (unencrypted DNS)

        ```
        /usr/sbin/ufw allow out on enp1s0 to 0.0.0.0/0 port 53 proto udp > /dev/null
        ```

    * allow all outgoing DNS over TLS traffic (encrypted DNS)

        ```
        /usr/sbin/ufw allow out on enp1s0 to 0.0.0.0/0 port 853 proto tcp > /dev/null
        ```

    * allow all outgoing git traffic (software update)

        ```
        /usr/sbin/ufw allow out on enp1s0 to 0.0.0.0/0 port 9418 proto tcp > /dev/null
        ```

* VPN profile

    * allow all outgoing OpenVPN traffic (secure point-to-point communication) - UDP (**NOTE**: VPN provider dependent)

        ```
        /usr/sbin/ufw allow out on enp1s0 to XXX.YYY.ZZZ.0/24 proto udp > /dev/null
        ```

    * allow all outgoing web traffic (web surfing via VPN) - TCP

        ```
        /usr/sbin/ufw allow out on tun0 to 0.0.0.0/0 port 443 proto tcp > /dev/null
        ```

    * allow all outgoing web traffic (web surfing via VPN) - QUIC

        ```
        /usr/sbin/ufw allow out on tun0 to 0.0.0.0/0 port 443 proto udp > /dev/null
        ```

    * allow all outgoing DNS traffic (unencrypted DNS via VPN)

        ```
        /usr/sbin/ufw allow out on tun0 to 0.0.0.0/0 port 53 proto udp > /dev/null
        ```

    * allow all outgoing DNS over TLS traffic (encrypted DNS via VPN)

        ```
        /usr/sbin/ufw allow out on tun0 to 0.0.0.0/0 port 853 proto tcp > /dev/null
        ```

    * allow all outgoing TCP traffic (any TCP communication via VPN)

        ```
        /usr/sbin/ufw allow out on tun0 to 0.0.0.0/0 proto tcp > /dev/null
        ```

## Donations

<p align="justify">The content available on this page is free of charge. In case you found any information documented here helpful and/or learned something new and/or it saved time or you just want to support this initiative, please consider donating.</p>

Currently supported donation options:

* Monero [XMR] - `89UmCAajMN9ECm9g9rz3j92xK97qFKDwtPA6SSVuScToNwVSKPP5bjXXNnXqEk4KdjH4aSVHzXLjbNnh45dR9dP1V7wVDGg`

<p align="justify">Received donations will also result with more frequent content updates.</p>

Thanks!
