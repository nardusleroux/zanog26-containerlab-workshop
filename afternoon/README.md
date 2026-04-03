## Teacher folder

> [!WARNING]
>
> This lab requires a containerlab build that includes [[this
> commit](https://github.com/srl-labs/containerlab/commit/d5c6bbaafbc96659b91981632e70e51e1c3609eb)].
> The servers used at ZANOG'26 will have a containerlab/clab binary that contains this change, and
> will be using the `git.ipng.ch/ipng/vpp-containerlab:stable` image.


This is for the afternoon session at ZANOG. In the morning, students will familiarize themselves
with containerlab itself: installing, building a topology, a and two first tests (a hello-world; and
a srlinux+multitool playground).

In the afternoon, students will continue with a VPP Containerlab exercise, which is modeled in this
directory.

### Anatomy of this lab

Running on a shared server, there will be 'participant' and 'server' labs, each lab will have one
network interface in a shared Linux bridge, so they can communicate amongst themselves.

#### Server Lab

The server lab runs two VPP routers and one Linux node: `vpp1 <-> vpp2 <-> web`. The webserver image
runs `git.ipng.ch/ipng/clab-webserver:latest`, which is configured to listen on port 80, and will
answer HTTP requests from participant labs, with individualized responses based on their IPv4 or
IPv6 addresses.

*   [[client-map.yaml](server/config/client-map.yaml)]
*   [[docroot](server/docroot/)]
*   Server: `192.0.2.254/24` and `2001:db8:8298::fe/64` and ASN `65000`

#### Participant Lab

The participant lab configs are auto-generated based on a participant ID which starts at 1 and can
run up to 100. Each participant gets assigned a unique IPv4 and IPv6 prefix and AS number. They use
this information to configure their topology: `client <-> vpp2 <-> vpp1` where vpp1 shares one
interface with all other participants and the Server Lab.

Participants will receive:
*   A participant ID (between 1 and 100 inclusive)
*   A username + password to log in to the shared server.
*   Shared LAN: `192.0.2.ID/24` and `2001:db8:8298::ID_HEX/64` and ASN `65000+ID`
*   IPv4: `10.0.ID.0/24` and IPv6: `2001:db8:ID_HEX::/48`

Participants will build up their LAB one step at a time:
1.  Starting the participant's containerlab
1.  `vpp2 <-> client` for IPv4 and IPv6
1.  `vpp2 <-> vpp1` transit network for IPv4 and IPv6
1.  `vpp2` OSPF and OSPFv3
1.  `vpp1` OSPF and OSPFv3
1.  `vpp1 <-> Shared LAN`
1.  `vpp1 <-> server:vpp1` eBGP
1.   End to end test from `client` all the way to `server:web` using `curl(1)` for IPv6 and IPv4

### Building

The server needs to have a pre-configured bridge called `vpplab` with an MTU of 9216 and no
link-local address. The server can run as-is, with no additional configuration:

```
sudo ip link add vpplab type bridge
sudo ip link set vpplab mtu 9216 up
sudo sysctl -w net.ipv6.conf.vpplab.disable_ipv6=1
cd afternoon/server/
clab deploy
```

The participants each get their own lab config (and tailored HOWTO) based on their unique ID. You
can generate these as follows:

```
sudo apt install libpango-1.0-0 libpangoft2-1.0-0 libpangocairo-1.0-0 \
                 libgdk-pixbuf-2.0-0 libcairo2 libffi-dev
pip install --break-system-packages markdown jinja2 weasyprint

cd afternoon/participant/
for i in `seq 1 64`; do \
  ./generate -id $i -indir files-start -outdir p$i/; \
done
```

You can now rsync these to the participants' homedir:
```
cd afternoon/participant/
for i in `seq 1 64`; do 
  if ! id "zanog$i" &>/dev/null; then
    sudo useradd -m -s /bin/bash "zanog$i"
  fi
  if ! id "zanog$i" | grep -q 'clab_admins'; then
    sudo usermod -aG clab_admins "zanog$i"
  fi
  if ! id "zanog$i" | grep -q 'docker'; then
    sudo usermod -aG docker "zanog$i"
  fi
  PASSWORD=$(grep 'Password' p$i/HOWTO.md | cut -f2 -d'`')
  echo "zanog$i:$PASSWORD" | sudo chpasswd
  sudo rsync -avg p$i/ /home/zanog$i/containerlab-p$i/
  sudo chown -R zanog$i:zanog$i /home/zanog$i/
  sudo chmod -R o-rwx /home/zanog$i/
done
```

Print out or distribute `containerlab-p$i/HOWTO-p$i.pdf` to each participant. Those PDFs are
tailored to contain the correct instructions, IP addresses, ASNs, etc for each individual lab.
