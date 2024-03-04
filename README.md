Open5GS with Docker
Ubuntu package YouTube Video Likes

This repository contains the following parts:

Run Open5gs with user plane, control plane, combining eNB, UE with srsRAN
Overview of the Location Based Services system, using GMLC, E-SMLC
Notes on Java, Spring Framework
Notes on installing and running JDK, maven, JDiameter
Implementation video

Contributors:
Ma Viet Duc
Pham Thanh Hai
Table of contents

Create a communication gateway

Pull images from Docker Hub

Docker Run the received images

Run the Containers and configure them

4.1. User Plane Part
4.2. Control Plane Part
4.3. eNB, UE Part

Work performed

Create a communication gateway
Add a gateway with IP is 20.0.0.1

docker network create --gateway 20.0.0.1 --subnet 20.0.0.0/24 4g
Configuration diagram:

+-----------+ +---------------------+ +------------------+ if: ogs-internet 60.17.0.23/16
| eNodeB | | EPC Control Plane | | EPC User Plane |------------------------------------- INTERNET
+-----------+ +---------------------+ +------------------+
| | |
| 20.0.0.20 | 20.0.0.2,3,4 | 20.0.0.5,6 if: 20.0.0.0/24
--+-------------------------+----------------------------+--------------------------------------------------
Docker information table:

Docker # Component IP Address OS
EPC Control Plane MME
SGW-C
SMF 20.0.0.2/24
20.0.0.3/24
20.0.0.4/24 Ubuntu 20.04
EPC User Plane SGW-U
UPF 20.0.0.5/24
20.0.0.6/24 Ubuntu 20.04
srsRAN eNodeB, UE 20.0.0.20/24 Ubuntu 20.04
Check Docker network with command

docker network ls

Pull images from Docker Hub
Docker for the User Plane:
docker pull maduc238/open5gs:user-plane
Docker for the Control Plane:

docker pull maduc238/open5gs:control-plane
Docker for srsRAN

docker pull aothatday/open5gs:srsenb

Docker Run the received images
Note: Two Dockers run on two different terminals
User Plane: requires connection to the network, therefore need to create a virtual interface with tun mode

docker run --name open5gs-u -d -t --cap-add=NET_ADMIN --cap-add=NET_RAW --net 4g --ip 20.0.0.5 --device /dev/net/tun maduc238/open5gs:user-plane
Can be run on the host's network: --network host

Control Plane:

docker run --name open5gs-c -d -t --cap-add=NET_ADMIN --cap-add=NET_RAW --net 4g --ip 20.0.0.2 maduc238/open5gs:control-plane
Add host machine connection port, for example: -p 36412:36412/sctp

srsRAN:

docker run --name srsenb -d -t --privileged -v /dev/bus/usb:/dev/bus/usb --net 4g --ip 20.0.0.20 aothatday/open5gs:srsenb

Run the Containers and configure them
4.1. User Plane Part
Configure network and run container:

Note: Need to adjust IP of S1-U interface (gtpu) for SGW-U: vim install/etc/open5gs/sgwu.yaml

docker exec -it open5gs-u bash
ip addr add 20.0.0.6/24 dev eth0
ip tuntap add name ogs-internet mode tun
ip addr add 60.17.0.23/16 dev ogs-internet
ip link set ogs-internet up
iptables -t nat -A POSTROUTING -s 60.17.0.23 ! -o ogs-internet -j MASQUERADE
Note: Change IP if there were modifications before running in sgw-u

cd home/open5gs
./run.sh

4.2. Control Plane Part
Configure network and run container:

docker exec -it open5gs-c bash
ip addr add 20.0.0.3/24 dev eth0
ip addr add 20.0.0.4/24 dev eth0
cd open5gs
./run4g_cp.sh
Access web UI: Go to the address 20.0.0.2:3000 Username: admin Password: 1423

4.3. eNB, UE Part
Note: Run eNB and UE on two different terminals

On eNB:

docker exec -it srsenb bash
cd srsRAN/srsenb
../build/srsenb/src/srsenb ./enb.conf
For real eNB, modify the file /root/.config/srsran/enb.conf and run srsenb

On UE:

docker exec -it srsenb bash
cd srsRAN/srsue
../build/srsue/src/srsue ./ue.conf

Work performed
Get the id of the 4g subnet created earlier
docker network ls | grep 4g
Docker host: capture packets passing through the newly created interface of docker in the form of br-id

sudo wireshark
Container machines: use tcpdump to capture packets in the loopback interface

tcpdump -i lo -s 65535 -w loopback.pcap
