# VPN Access

MICC is an expermient to setup a platform to easily share resources and
build together. Today there are three primary devices (although various 
Pi’s and smaller hardware boards will be added  of interest.

## Hosts
- **Lightbike** - `192.168.33.2`
- **Arcade** - `192.168.33.3`
- **Jetson** - `192.168.33.4`
- **PiHole** - `192.168.40.38` - you can use this for DNS on the VPN if you want 
see later details

## Rules of the road

I've set this up to share with resources with others. If you see something
that is an issue or run into problems let me know, or take a crack at fixing it
and then let me know in case something pops up or I would need to replicate for
new additions.

Additionally you are most likely part of the sudoers/wheel group. Things may get
broken, that's ok but take a crack at fixing it and if you need help feel free to
reach out.

## Joining the network

Right now I haven’t set up any type of domain controller or LDAP for adding
users. Because of this if you want VPN access you need to provide me the
following information:

- Real Name (As you want it in the system)
- User Name (As you want it in the system)
- Email (for me to send notifications for instance if I’m taking a server down
for something)
- IP count (assuming 1 by default if you need more let me know since this 
IP is peered with the routing box)

Along with that I need an ssh public key that I will load into your user 
authorized keys file for ssh access (access via ssh is certificate only). 
You can get this to me via [Keybase](https://keybase.io/alexanderh) or
physical media.

Once I have the above information I’ll add you to the VPN and servers and 
provide back your login and IP(s).

## Wireguard

### Peering

The VPN is setup via Wireguard peering. The 192.168.33.1 box acts as a
router in this scenario (although wireguard supports peer to peer) allowing
connection to the Wireguard peer network I’m running to the outside world at
`vpn.unexpectedeof.site`.

[Installing Wireguard](https://www.wireguard.com/install/)

For configuration (assuming Linux or Mac) you will need to configure a
wireguard interface. If using the Mac App Store the app manage tunnel
interface handles most of this for you. If using Ubuntu or a debian based
system give [this](https://www.erianna.com/wireguard-ubiquity-edgeos/#Setting up the Client)a read. 

The main thing is getting `wg0` setup right. 

An example client wg0 config file looks like:

```bash
# These settings represent your local interface
[Interface]
Address = 192.168.33.x/32 # The IP provided to you for the VPN
PrivateKey = # The private key you generated with `wg genkey`
DNS = # You will want to specify DNS for routing to remote site if you route all your traffic through
# the VPN. The PiHole above can be used on the VPN if routing through local, it uses DNS.WATCH and
# Quad9. 1.1.1.1 is Cloudflare if you want.

# These settings configure communicating with the wireguard VPN endpoint
[Peer] 
PublicKey = # Will be provided
AllowedIPs = 0.0.0.0/0, ::0 #This will route all traffic over the VPN, scope as you want
Endpoint = vpn.unexpectedeof.site:51820
```

As part of your configuration you will need to provide your Wireguard public
key to me so I can peer it on the routing interface. See ssh public key 
sharing above on how to get the key to me.

Once you I’ve setup the peer on the routing interface you should be able 
to `sudo wg-quick up wg0` and connect to the VPN. Once on the VPN you will
be able to ssh to the systems above using your public key.

## SSH

Above I highlighted that ssh auth is done via certificates. Assuming you 
generated a certificate(s) specifically for these connections you can setup
a config file in ~/.ssh to make authentication fast and easy. Examples from
mine:

```sh
cat ~/.ssh/config

# ML server
Host lightbike
  Hostname 192.168.33.2
  User n0mn0m
  IdentityFile /Users/n0mn0m/.ssh/lightbike

# Storage
Host arcade
  Hostname 192.168.33.3
  User n0mn0m
  IdentityFile /Users/n0mn0m/.ssh/arcade

# Jetson
Host jetson
  Hostname 192.168.33.4
  User n0mn0m
  IdentityFile /Users/n0mn0m/.ssh/jetson
```

Then you can use the hostname with ssh

`ssh lightbike`

## Support

If you run into trouble give me a shout (text, alexander@unexpectedeof.net,
keybase: [alexanderh](https://keybase.io/alexanderh) or something else) with a screenshot, logs etc and we
will get it figured out.

## Network Overview
!["MICC network diagram"](micc.jpg"Network overview")

## Containers

`docker` is installed on lightbike. If we want to setup something like `mini k8s`
too we can.

`bhyve` is installed on arcade if you want to make use of BSD Jails

## ML

`lightbike` and `arcade` have the nvidia drivers and Cuda pre installed.


## Status

If you're not able to reach a box check [here](https://status.unexpectedeof.casa/) first.