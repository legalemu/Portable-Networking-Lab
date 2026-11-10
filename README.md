```
 _          _       _   _      _                      _
| |    __ _| |__   | \ | | ___| |___      _____  _ __| | __
| |   / _` | '_ \  |  \| |/ _ \ __\ \ /\ / / _ \| '__| |/ /
| |__| (_| | |_) | | |\  |  __/ |_ \ V  V / (_) | |  |   <
|_____\__,_|_.__/  |_| \_|\___|\__| \_/\_/ \___/|_|  |_|\_\
```

## The layout

```
                  home router (internet)
                          |
                       ge-0/0/0
                  +---------------+
                  |   SRX240      |  192.168.10.1
                  |   lab-srx     |
                  +---------------+
                       ge-0/0/1
                          |  every VLAN
                       port 23
                  +---------------+
                  |   EX3400      |  192.168.10.3
                  |   lab-ex3400  |
                  +---------------+
                  |  |  |  |  |  |
   +--------------+  |  |  |  |  +--------------+
   |                 |  |  |  |                 |
 ports 0-11     ports 12-16 |  |             port 20, 21
 laptops/PCs    lab boxes   |  |             pve1, pve2
 (student)      (no net)    |  |             (all VLANs)
                            |  |
                        port 17, 18, 19
                       console, pi, linksys
```

## Networks

The SRX is the gateway on all of them and always takes `.1`.

```
VLAN  Name      Subnet            Gateway        Used for                           Internet
10    MGMT      192.168.10.0/24   192.168.10.1   switch, pi, proxmox, mgmt laptop   yes
20    STUDENT   192.168.20.0/24   192.168.20.1   laptops, PCs, wifi                 yes
30    LAB       192.168.30.0/24   192.168.30.1   lab machines                       no
40    GUEST     192.168.40.0/24   192.168.40.1   not used yet                       yes
```

## Devices

```
Device              Hostname       IP              Network   Address type
SRX240 router       lab-srx        192.168.10.1    MGMT      static
EX3400 switch       lab-ex3400     192.168.10.3    MGMT      static
Raspberry Pi        pi             192.168.10.10   MGMT      static
Proxmox 1           pve1           192.168.10.11   MGMT      static
Proxmox 2           pve2           192.168.10.12   MGMT      static
Management laptop   mgmt-console   192.168.10.30   MGMT      static
Linksys AP          linksys        192.168.20.13   STUDENT   DHCP reservation
```

The SRX's internet port (`ge-0/0/0`) gets its address from the home router on its own.
`ge-0/0/2` stays at 192.168.1.1 as a backup way in if we ever lock ourselves out.

## DHCP

Every network hands out `.100` to `.250`. Anything with a fixed address lives below `.100`.

```
Network   Range
MGMT      192.168.10.100 to .250
STUDENT   192.168.20.100 to .250
LAB       192.168.30.100 to .250
GUEST     192.168.40.100 to .250
```

## Switch ports

```
Port    Plugged into            Network
0-11    laptops and PCs         STUDENT
12-16   lab machines            LAB
17      management laptop       MGMT
18      Raspberry Pi            MGMT
19      Linksys (bridge mode)   STUDENT
20      Proxmox 1               all VLANs, MGMT untagged
21      Proxmox 2               all VLANs, MGMT untagged
22      spare, turned off       none
23      SRX240 ge-0/0/1         all VLANs
```

## Who can talk to who

- **MGMT** can reach every other network and the internet.
- **STUDENT** can reach the internet and LAB.
- **LAB** has no internet and can't start connections to anything else.
- **GUEST** gets the internet and nothing more.




