# Containerlab multi-area OSPF configuration Nokia SRLinux in Windows WSL Ubuntu-22.04

>Install WSL feature on Control Panel
	check:	Windows Subsystem Linux
	Restart PC
>Powershell (run as user)
	wsl --update
>Powershell Install Ubuntu 22.04 LTS
	wsl --install Ubuntu-22.04
	set username & password
	sudo apt-get update && upgrade
>Install docker on Ubuntu-22.04 LTS
	https://docs.docker.com/engine/install/ubuntu/?ref=crapts.org	
===============================================================
>cek docker status
	sudo service docker status
>test docker
	sudo docker run hello-world
>Install containerlab
	curl -sL https://containerlab.dev/setup | sudo bash -s "all"
>cek container lab status
	containerlab version
>install nokia srlinux (with sudo) 874.7mb
	docker pull ghcr.io/nokia/srlinux
>show images
	docker images
>check container name (empty)
	docker ps -a

------------------------------------------------------------------------------------
>copy lab folder from /home/username/clab-nokia-lab
	cp -r clab-nokia-lab clab-nokia-lab2
	cd clab-nokia-lab2/
//you can directly create topofile .yml or create new directory for it, i copied existing example lab directory from containerlab and make my topofile file inside

>make topology file
	vi nokia-lab.clab.yml
		name: nokia-lab
		topology:
		nodes:
			srl1:
			kind: srl
			image: ghcr.io/nokia/srlinux
			srl2:
			kind: srl
			image: ghcr.io/nokia/srlinux
			srl3:
			kind: srl
			image: ghcr.io/nokia/srlinux
		links:
			- endpoints: ["srl1:e1-1", "srl2:e1-1"]
			- endpoints: ["srl1:e1-2", "srl3:e1-1"]

>verify topology file
	cat nokia-lab-clab.yml
	
>deploy
	sudo containerlab deploy <topofile>
	//if you need privilage add sudo 
	
>check docker container to verify
	docker ps -a
	
>generate topology
	sudo containerlab graph --topo <topofile>
	http://0.0.0.0:50080
	ifconfig => eth0
	change 0.0.0.0 with eth0 IP

.....................................................................................
>copy lab folder from /home/username/clab-nokia-lab
	cp -r clab-nokia-lab clab-nokia-lab2
	cd clab-nokia-lab2/
//you can directly create topofile .yml or create new directory for it, i copied existing example lab directory from containerlab and make my topofile file inside

>make topology file
	vi nokia-lab.clab.yml
		name: nokia-lab
		topology:
		nodes:
			srl1:
			kind: srl
			image: ghcr.io/nokia/srlinux
			srl2:
			kind: srl
			image: ghcr.io/nokia/srlinux
			srl3:
			kind: srl
			image: ghcr.io/nokia/srlinux
		links:
			- endpoints: ["srl1:e1-1", "srl2:e1-1"]
			- endpoints: ["srl1:e1-2", "srl3:e1-1"]

>verify topology file
	cat nokia-lab-clab.yml
	
>deploy
	sudo containerlab deploy <topofile>
	//if you need privilage add sudo 
	
>check docker container to verify
	docker ps -a
	
>generate topology
	sudo containerlab graph --topo <topofile>
	http://0.0.0.0:50080
	ifconfig => eth0
	change 0.0.0.0 with eth0 IP

.....................................................................................
//Configuration Topology: 3 Routers; 2 area: Normal and Stub; broadcast.
R01 as ABR between R02 in normal area and R03 in stub

>system interface IP address ipv4
	#R01> 1.1.1.1/32
	#R02> 2.2.2.2/32
	#R03> 3.3.3.3/32

>link interface IP addr IPv4
	#R01
		e1-1
			name: to_R02
			addr: 10.4.5.1/30
		e1-2
			name: to_R03
			addr: 10.2.4.1/30
	#R02
		e1-1
			name: to_R01
			addr: 10.4.5.2/30
	#R03
		e1-1
			name: to_R01
			addr: 10.2.4.2/30

>OSPF areaID
	Area 0.0.0.0
		link R01 to R02
		type: normal

	Area 0.0.0.1
		link R01 to R03
		type: stub

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
>login to srl1
	ssh admin@clab-nokia-lab-srl1
	password= NokiaSrl1!
	//or you can login with docker exec command
	docker exec -it clab-nokia-lab-srl1 sr_cli
	
	info flat
	//make sure Port mgmt0, ethernet and system; Admin up and Oper up before configuring IP, and next step after configuring IP succesful by ping to peer link we could continue config IGP routing

>config on R01
#R01> enter configuration candidate mode
	enter candidate

#R01> set hostnname R01
	set system name host-name R01
	commit stay
	save

#R01> set system interface
	set / interface system0
		/ interface system0 description "System Loopback"
							admin-state enable
							subinterface 0 
							ipv4 address 1.1.1.1/32
	info

	> verification
	/
	show interface system0
	info network-instance default interface system0.0
	
	//SR Linux does not support assigning an IPv4 address directly to a physical Ethernet interface  like ethernet-1/1. the ipv4 will be assigned as subinterface: ethernet-1/1.1
	
#R01> set interface 1/1 for R02
	/ interface ethernet-1/1
							 description "to_R02"
							 admin-state enable
							 mtu 1500
							 subinterface 1
								admin-state enable
								ipv4 address 10.4.5.1/30
	info

#R01> set interface 1/2 for R03
	/ interface ethernet-1/2
							 description "to_R03"
							 admin-state enable
							 mtu 1500
							 subinterface 1
								admin-state enable
								ipv4 address 10.2.4.1/30
	info
	/
	show interface all
	
	show interface brief	
	show interface ethernet-1/1 detail
	????????????????????? down 

#R01> set routing protocol OSPF on interface 1/1
	set / network-instance default protocols ospf instance link_to_R02
		/ network-instance default protocols ospf admin-state enable
												  router-id 1.1.1.1
												  area 0.0.0.0
												  interface ethernet-1/1
												  interface-type broadcast
												  admin-state enable
	
#R01> set routing protocol OSPF on interface 1/1
	set / network-instance default protocols ospf instance link_to_R03
		/ network-instance default protocols ospf admin-state enable
												  router-id 1.1.1.1
												  area 0.0.0.1
												  interface ethernet-1/2
												  interface-type broadcast
												  admin-state enable
	
#R01> config verification
	validate
	commit stay
	save
------------------------------------------------
#R02> enter configuration candidate
	enter candidate

#R02> set hostname R02
	set system name host-name R02

#R02> set system interface
	set / interface system0
		/ interface system0 description "System Loopback"
							admin-state enable
							subinterface 0 
							ipv4 address 2.2.2.2/32
	info

#R02> set interface 1/1 for R01
	/ interface ethernet-1/1
							 description "to_R01"
							 admin-state enable
							 mtu 1500
							 subinterface 1
								admin-state enable
								ipv4 address 10.4.5.2/30
	show interface system0.0
	show interface ethernet-1/1.1

#R02> set routing protocol OSPF on interface 1/1
	set / network-instance default protocols ospf instance link_to_R01
		/ network-instance default protocols ospf admin-state enable
												  router-id 2.2.2.2
												  area 0.0.0.0
												  interface ethernet-1/1
												  interface-type broadcast
												  admin-state enable

#R02> config verification
	validate
	commit stay
	save
------------------------------------------------
#R03> enter configuration candidate
	enter candidate

#R03> set hostname R03
	set system name host-name R03

#R03> set system interface
	set / interface system0
		/ interface system0 description "System Loopback"
							admin-state enable
							subinterface 0 
							ipv4 address 3.3.3.3/32
	info

#R03> set interface 1/1 for R01
	/ interface ethernet-1/1
							 description "to_R01"
							 admin-state enable
							 mtu 1500
							 subinterface 1
								admin-state enable
								ipv4 address 10.2.4.2/30
	show interface all
	
#R03> set routing protocol OSPF on interface 1/1
set routing protocol OSPF on interface 1/1
	set / network-instance default protocols ospf instance link_to_R01
		/ network-instance default protocols ospf admin-state enable
												  router-id 3.3.3.3
												  area 0.0.0.1
												  interface ethernet-1/1
												  interface-type broadcast
												  admin-state enable
												  
#R03> config verification
	validate
	commit stay
	save
================================================
>verification
	show network-instance default/protocols/ospf
	ping 
	info ospf
	show network-instance default protocols ospf interface
	show network-instance default protocols ospf neighbor
	show network-instance default routes

