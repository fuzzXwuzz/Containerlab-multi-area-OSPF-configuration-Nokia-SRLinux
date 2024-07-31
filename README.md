# Containerlab Multi-Area OSPF Configuration on Nokia SR Linux in Windows WSL Ubuntu 22.04

step-by-step instructions for setting up and configuring a multi-area OSPF network using Nokia SR Linux on Containerlab in a Windows WSL Ubuntu 22.04 environment.

## Prerequisites

### Install WSL Feature on Windows

1. Open Control Panel.
2. Enable the **Windows Subsystem for Linux** feature.
3. Restart your PC.

### Install Ubuntu 22.04 LTS

1. Open PowerShell (run as user) and update WSL:
    ```powershell
    wsl --update
    ```

2. Install Ubuntu 22.04 LTS:
    ```powershell
    wsl --install Ubuntu-22.04
    ```
3. Set your username and password.
4. Update and upgrade the system:
    ```bash
    sudo apt-get update && sudo apt-get upgrade
    ```

### Install Docker on Ubuntu 22.04 LTS

Follow the [Docker installation guide for Ubuntu](https://docs.docker.com/engine/install/ubuntu/?ref=crapts.org).

### Check Docker Status

1. Check Docker status:
    ```bash
    sudo service docker status
    ```
2. Test Docker installation:
    ```bash
    sudo docker run hello-world
    ```

### Install Containerlab

1. Install Containerlab:
    ```bash
    curl -sL https://containerlab.dev/setup | sudo bash -s "all"
    ```
2. Check Containerlab status:
    ```bash
    containerlab version
    ```

### Install Nokia SR Linux

1. Pull the Nokia SR Linux image:
    ```bash
    sudo docker pull ghcr.io/nokia/srlinux
    ```
2. Verify the Docker image:
    ```bash
    docker images
    ```

## Setting Up the Lab

### Prepare the Lab Directory

1. Copy the example lab directory:
    ```bash
    cp -r clab-nokia-lab clab-nokia-lab2
    cd clab-nokia-lab2
    ```

### Create the Topology File

1. Create the topology file `nokia-lab.clab.yml`:
    ```bash
    vi nokia-lab.clab.yml
    ```

2. Add the following content:
    ```yaml
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
    ```

3. Verify the topology file:
    ```bash
    cat nokia-lab.clab.yml
    ```

### Deploy the Topology

1. Deploy the topology:
    ```bash
    sudo containerlab deploy --topo nokia-lab.clab.yml
    ```

2. Check Docker containers:
    ```bash
    docker ps -a
    ```

3. Generate the topology graph:
    ```bash
    sudo containerlab graph --topo nokia-lab.clab.yml
    ```

4. Access the topology graph:
    Open a web browser and go to `http://<eth0-IP>:50080` (replace `<eth0-IP>` with the IP address of `eth0`).

## Configuration Topology: 3 Routers; 2 Areas: Normal and Stub; Broadcast

- **R01** as ABR between **R02** in normal area and **R03** in stub.

### System Interface IP Address (IPv4)

- R01: 1.1.1.1/32
- R02: 2.2.2.2/32
- R03: 3.3.3.3/32

### Link Interface IP Address (IPv4)

- **R01**
  - e1-1: to_R02 (10.4.5.1/30)
  - e1-2: to_R03 (10.2.4.1/30)

- **R02**
  - e1-1: to_R01 (10.4.5.2/30)

- **R03**
  - e1-1: to_R01 (10.2.4.2/30)

### OSPF Area ID

- **Area 0.0.0.0**
  - Link R01 to R02
  - Type: Normal

- **Area 0.0.0.1**
  - Link R01 to R03
  - Type: Stub

## Configuration Steps

### Login to SRL1

1. SSH into SRL1:
    ```bash
    ssh admin@clab-nokia-lab-srl1
    ```
   Password: `NokiaSrl1!`

2. Alternatively, login using Docker:
    ```bash
    docker exec -it clab-nokia-lab-srl1 sr_cli
    ```

3. Ensure ports `mgmt0`, `ethernet`, and `system` are Admin UP and Oper UP.

### Configure R01

1. Enter configuration candidate mode:
    ```plaintext
    enter candidate
    ```

2. Set hostname:
    ```plaintext
    set system name host-name R01
    commit stay
    save
    ```

3. Configure system interface:
    ```plaintext
    set / interface system0
        / interface system0 description "System Loopback"
                            admin-state enable
                            subinterface 0 
                            ipv4 address 1.1.1.1/32
    info
    ```

4. Configure interface 1/1 for R02:
    ```plaintext
    / interface ethernet-1/1
                            description "to_R02"
                            admin-state enable
                            mtu 1500
                            subinterface 1
                            admin-state enable
                            ipv4 address 10.4.5.1/30
    info
    ```

5. Configure interface 1/2 for R03:
    ```plaintext
    / interface ethernet-1/2
                            description "to_R03"
                            admin-state enable
                            mtu 1500
                            subinterface 1
                            admin-state enable
                            ipv4 address 10.2.4.1/30
    info
    ```

6. Verify interfaces:
    ```plaintext
    show interface all
    show interface brief
    show interface ethernet-1/1 detail
    ```

7. Set OSPF on interface 1/1:
    ```plaintext
    set / network-instance default protocols ospf instance link_to_R02
        / network-instance default protocols ospf admin-state enable
                                                  router-id 1.1.1.1
                                                  area 0.0.0.0
                                                  interface ethernet-1/1
                                                  interface-type broadcast
                                                  admin-state enable
    ```

8. Set OSPF on interface 1/2:
    ```plaintext
    set / network-instance default protocols ospf instance link_to_R03
        / network-instance default protocols ospf admin-state enable
                                                  router-id 1.1.1.1
                                                  area 0.0.0.1
                                                  interface ethernet-1/2
                                                  interface-type broadcast
                                                  admin-state enable
    ```

9. Verify configuration:
    ```plaintext
    validate
    commit stay
    save
    ```

### Configure R02

1. Enter configuration candidate mode:
    ```plaintext
    enter candidate
    ```

2. Set hostname:
    ```plaintext
    set system name host-name R02
    ```

3. Configure system interface:
    ```plaintext
    set / interface system0
        / interface system0 description "System Loopback"
                            admin-state enable
                            subinterface 0 
                            ipv4 address 2.2.2.2/32
    info
    ```

4. Configure interface 1/1 for R01:
    ```plaintext
    / interface ethernet-1/1
                            description "to_R01"
                            admin-state enable
                            mtu 1500
                            subinterface 1
                            admin-state enable
                            ipv4 address 10.4.5.2/30
    show interface system0.0
    show interface ethernet-1/1.1
    ```

5. Set OSPF on interface 1/1:
    ```plaintext
    set / network-instance default protocols ospf instance link_to_R01
        / network-instance default protocols ospf admin-state enable
                                                  router-id 2.2.2.2
                                                  area 0.0.0.0
                                                  interface ethernet-1/1
                                                  interface-type broadcast
                                                  admin-state enable
    ```

6. Verify configuration:
    ```plaintext
    validate
    commit stay
    save
    ```

### Configure R03

1. Enter configuration candidate mode:
    ```plaintext
    enter candidate
    ```

2. Set hostname:
    ```plaintext
    set system name host-name R03
    ```

3. Configure system interface:
    ```plaintext
    set / interface system0
        / interface system0 description "System Loopback"
                            admin-state enable
                            subinterface 0 
                            ipv4 address 3.3.3.3/32
    info
    ```

4. Configure interface 1/1 for R01:
    ```plaintext
    / interface ethernet-1/1
                            description "to_R01"
                            admin-state enable
                            mtu 1500
                            subinterface 1
                            admin-state enable
                            ipv4 address 10.2.4.2/30
    show interface all
    ```

5. Set OSPF on interface 1/1:
    ```plaintext
    set / network-instance default protocols ospf instance link_to_R01
        / network-instance default protocols ospf admin-state enable
                                                  router-id 3.3.3.3
                                                  area 0.0.0.1
                                                  interface ethernet-1/1
                                                  interface-type broadcast
                                                  admin-state enable
    ```

6. Verify configuration:
    ```plaintext
    validate
    commit stay
    save
    ```

## Verification

1. Show OSPF configuration:
    ```plaintext
    show network-instance default protocols ospf
    ```

2. Verify connectivity:
    ```plaintext
    ping <destination-IP>
    ```

3. Show OSPF interfaces:
    ```plaintext
    show network-instance default protocols ospf interface
    ```

4. Show OSPF neighbors:
    ```plaintext
    show network-instance default protocols ospf neighbor
    ```

5. Show routes:
    ```plaintext
    show network-instance default routes
    ```
