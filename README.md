
# Nokia SRLinux Containerlab Multi-Area OSPF Configuration on Windows 10 WSL2 Ubuntu-22.04

```markdown
## System Information
- Windows version: 10.0.19045.4780
- WSL version: 2.2.4.0
- Ubuntu version: 22.04.4 LTS
- Containerlab version: 0.57.0
- Docker version: 27.2.0, build 3ab4256
- Docker Desktop version: 4.33.1 (161083)
```

### Install WSL2 on Windows
1. **Enable Virtual Machine support in BIOS** (SVM for AMD).
2. **Update Windows** to the latest features and restart.
3. **Enable Windows features** in Control Panel:
   - Windows Subsystem for Linux
   - Virtual Machine Platform
   - Restart your computer.
4. **Run PowerShell as Administrator** and check WSL version:
   ```sh
   wsl -v
   ```
   Ensure it shows WSL logo on the Windows app start menu

5. **Update WSL**:
   ```sh
   wsl --update
   ```
6. **Install Ubuntu 22.04** on PowerShell:
   ```sh
   wsl --list --online
   wsl --install -d Ubuntu-22.04
   ```
   Enter your username and password.
   
8. **Verify WSL app logo** appears in the start menu.
9. **Update and upgrade Ubuntu**:
   ```sh
   sudo apt update && sudo apt full-upgrade -y
   ```
10. **Install Docker in WSL** following [Docker's official guide](https://docs.docker.com/engine/install/ubuntu/).
11. **Check Docker service status**:
    ```sh
    sudo service docker status
    ```
    Test Docker
    ```sh
    sudo docker run hello-world
    ```

    Ensure it is 'ON'.
12. **Install Docker Desktop on Windows** and integrate with WSL2 in settings > resources. Ensure Ubuntu-22.04 is checked.

13. **Verify Docker integration** on PowerShell:
    ```sh
    wsl -l
    ```
    Ensure WSL, Ubuntu, and Docker Desktop are listed.

14. **Optionally switch iptables to legacy version**:
    ```sh
    sudo update-alternatives --config iptables
    ```
    Choose option 1 for legacy.

15. **Install Containerlab**:
    ```sh
    curl -sL https://containerlab.dev/setup | sudo bash -s "all"
    ```
![containerlab](https://github.com/user-attachments/assets/34d43b1e-ce6b-43f6-adc3-a5fdac5d1821)

### Install Nokia SRLinux
1. **Pull the Nokia SRLinux Containerlab images**:
   ```sh
   sudo docker pull ghcr.io/nokia/srlinux
   ```
2. **Verify router images installed in Docker**:
   ```sh
   docker images
   ```

## Topology Plan
```
=====================================================
|            > System Interface IP Address IPv4     |
|             #srl1 > 1.1.1.1/32                    |
|             #srl2 > 2.2.2.2/32                    |
|             #srl3 > 3.3.3.3/32                    |
|             #srl4 > 4.4.4.4/32                    |
|---------------------------------------------------|
|            > Link Interface IP Address IPv4       |
|             #srl1     RID 1.1.1.1                 |
|                 e1-1                              |
|                     area: 0.0.0.0                 |
|                     addr: 10.4.5.1/30             |
|                 e1-2                              |
|                     addr: 10.2.4.1/30             |
|             #srl2     RID 2.2.2.2                 |
|                 e1-1                              |
|                     area: 0.0.0.0                 |
|                     addr: 10.4.5.2/30             |
|             #srl3     RID 3.3.3.3                 |
|                 e1-1                              |
|                     area: 0.0.0.0                 |
|                     addr: 10.2.4.2/30             |
|                 e1-2                              |
|                     area: 0.0.0.1 stub            |
|                     addr: 10.6.5.1/30             |
|             #srl4     RID 4.4.4.4                 |
|                 e1-1                              |
|                     area: 0.0.0.1 stub            |
|                     addr: 10.6.5.2/30             |
=====================================================
```

## Create Topology File
1. **Create a directory**:
   ```sh
   mkdir -p /home/user/srl02-lab
   ```
2. **Create the topology file**:
   ```sh
   vi /home/user/srl02-lab/srl02-lab.yml
   ```
   ```yaml
   name: srl02-lab

   topology:
     nodes:
       srl1:
         kind: nokia_srlinux
         image: ghcr.io/nokia/srlinux
         startup-config: srl1.cfg
       srl2:
         kind: nokia_srlinux
         image: ghcr.io/nokia/srlinux
         startup-config: srl2.cfg
       srl3:
         kind: nokia_srlinux
         image: ghcr.io/nokia/srlinux
         startup-config: srl3.cfg
       srl4:
         kind: nokia_srlinux
         image: ghcr.io/nokia/srlinux
         startup-config: srl4.cfg
     links:
       - endpoints: ["srl1:e1-1", "srl2:e1-1"]
       - endpoints: ["srl1:e1-2", "srl3:e1-1"]
       - endpoints: ["srl3:e1-2", "srl4:e1-1"]
   ```


## Create interface Configuration Files for Router Links

### srl1.cfg
```sh
vi /home/user/srl02-lab/srl1.cfg
```
```sh
set / interface ethernet-1/1 admin-state enable
set / interface ethernet-1/1 subinterface 0
set / interface ethernet-1/1 subinterface 0 ipv4
set / interface ethernet-1/1 subinterface 0 ipv4 admin-state enable
set / interface ethernet-1/1 subinterface 0 ipv4 address 10.4.5.1/30

set / network-instance default
set / network-instance default interface ethernet-1/1.0

set / interface ethernet-1/2 admin-state enable
set / interface ethernet-1/2 subinterface 0
set / interface ethernet-1/2 subinterface 0 ipv4
set / interface ethernet-1/2 subinterface 0 ipv4 admin-state enable
set / interface ethernet-1/2 subinterface 0 ipv4 address 10.2.4.1/30

set / network-instance default
set / network-instance default interface ethernet-1/2.0
```

### srl2.cfg
```sh
vi /home/user/srl02-lab/srl2.cfg
```
```sh
set / interface ethernet-1/1 admin-state enable
set / interface ethernet-1/1 subinterface 0
set / interface ethernet-1/1 subinterface 0 ipv4
set / interface ethernet-1/1 subinterface 0 ipv4 admin-state enable
set / interface ethernet-1/1 subinterface 0 ipv4 address 10.4.5.2/30

set / network-instance default
set / network-instance default interface ethernet-1/1.0
```

### srl3.cfg
```sh
vi /home/user/srl02-lab/srl3.cfg
```
```sh
set / interface ethernet-1/1 admin-state enable
set / interface ethernet-1/1 subinterface 0
set / interface ethernet-1/1 subinterface 0 ipv4
set / interface ethernet-1/1 subinterface 0 ipv4 admin-state enable
set / interface ethernet-1/1 subinterface 0 ipv4 address 10.2.4.2/30

set / network-instance default
set / network-instance default interface ethernet-1/1.0

set / interface ethernet-1/2 admin-state enable
set / interface ethernet-1/2 subinterface 0
set / interface ethernet-1/2 subinterface 0 ipv4
set / interface ethernet-1/2 subinterface 0 ipv4 admin-state enable
set / interface ethernet-1/2 subinterface 0 ipv4 address 10.6.5.1/30

set / network-instance default
set / network-instance default interface ethernet-1/2.0
```

### srl4.cfg
```sh
vi /home/user/srl02-lab/srl4.cfg
```
```sh
set / interface ethernet-1/1 admin-state enable
set / interface ethernet-1/1 subinterface 0
set / interface ethernet-1/1 subinterface 0 ipv4
set / interface ethernet-1/1 subinterface 0 ipv4 admin-state enable
set / interface ethernet-1/1 subinterface 0 ipv4 address 10.6.5.2/30

set / network-instance default
set / network-instance default interface ethernet-1/1.0
```

## Deploy the Topology
```sh
sudo containerlab deploy -t /home/user/srl02-lab/srl02-lab.yml
```
output shown as below:
```
+---+---------------------+--------------+-----------------------+---------------+---------+----------------+----------------------+
| # |        Name         | Container ID |         Image         |     Kind      |  State  |  IPv4 Address  |     IPv6 Address     |
+---+---------------------+--------------+-----------------------+---------------+---------+----------------+----------------------+
| 1 | clab-srl02-lab-srl1 | 483460df06ff | ghcr.io/nokia/srlinux | nokia_srlinux | running | 172.20.20.5/24 | 2001:172:20:20::5/64 |
| 2 | clab-srl02-lab-srl2 | 2db6a02b4ea6 | ghcr.io/nokia/srlinux | nokia_srlinux | running | 172.20.20.3/24 | 2001:172:20:20::3/64 |
| 3 | clab-srl02-lab-srl3 | c1925aaf2aaa | ghcr.io/nokia/srlinux | nokia_srlinux | running | 172.20.20.2/24 | 2001:172:20:20::2/64 |
| 4 | clab-srl02-lab-srl4 | 92c176ef43ec | ghcr.io/nokia/srlinux | nokia_srlinux | running | 172.20.20.4/24 | 2001:172:20:20::4/64 |
+---+---------------------+--------------+-----------------------+---------------+---------+----------------+----------------------+
```
![graph topo](https://github.com/user-attachments/assets/0d743c0d-c30e-4efa-b700-9990a18f6496)

## Check SRLinux Router Images Status on Docker
```sh
docker ps -a
```

## Generate Topology Graph
```sh
sudo containerlab graph --topo /home/user/srl02-lab/srl02-lab.yml
```
Access the graph in your browser at:
```
http://0.0.0.0:50080
```
The address of containerlab is `eth0` on Ubuntu. To find it:
```sh
ifconfig
```
Look for `eth0`.

## SSH Remote Access to the Router
```sh
ssh admin@172.20.20.0
```
or
```sh
containerlab -t exec clab-srl02-lab-srl1 sr_cli
```
The default password is `NokiaSrl1!`.


## Configure SRLinux Routers

### srl1 Configuration

#### Show Nokia SRLinux router Version
```sh
A:srl1# show version
```
```
----------------------------------------------------------------------------------------------
Hostname             : srl1
Chassis Type         : 7220 IXR-D2L
Part Number          : Sim Part No.
Serial Number        : Sim Serial No.
System HW MAC Address: 1A:94:00:FF:00:00
OS                   : SR Linux
Software Version     : v24.7.1
Build Number         : 330-g38f237abfe
Architecture         : x86_64
Last Booted          : 2024-08-30T06:16:14.005Z
Total Memory         : 7906339 kB
Free Memory          : 495281 kB
-----------------------------------------------------------------------------------------------
```

#### Assign Interface Ethernet and System to Network-Instance
```sh
A:srl1# enter candidate
A:srl1# set interface ethernet-1/1 admin-state enable subinterface 0 admin-state enable ipv4 address 10.4.5.1/30
A:srl1# set interface ethernet-1/2 admin-state enable subinterface 0 admin-state enable ipv4 address 10.2.4.1/30
A:srl1# set admin-state enable interface system0.0
A:srl1# info
```
```
admin-state enable
interface ethernet-1/1.0 {
}
interface ethernet-1/2.0 {
}
interface system0.0 {
}
```

#### Check Interface Operational Status
```sh
A:srl1# show interface ethernet-1/1 all
A:srl1# show interface ethernet-1/1 brief
A:srl1# show interface ethernet-1/1 detail
```
#### if the interfaces operational is up you could test ping to each peer. and then you could continue configuring OSPF
#### if you found lower-layer down interface status, check up on your yaml file configuration. (optionally you could destroy the lab and re-deploy)


#### Configure OSPF

```sh
A:srl1# set / network-instance default protocols ospf instance default version ospf-v2 instance-id 0 admin-state enable router-id 1.1.1.1 area 0.0.0.0 advertise-router-capability true interface ethernet-1/1.0 admin-state enable interface-type point-to-point
A:srl1# set / network-instance default protocols ospf instance default area 0.0.0.0 advertise-router-capability true interface ethernet-1/2.0 admin-state enable interface-type point-to-point
A:srl1# info
```
```
interface ethernet-1/1.0 {
}
interface ethernet-1/2.0 {
}
protocols {
    ospf {
        instance default {
            admin-state enable
            version ospf-v2
            address-family ipv4-unicast
            instance-id 0
            router-id 1.1.1.1
            area 0.0.0.0 {
                advertise-router-capability true
                interface ethernet-1/1.0 {
                    admin-state enable
                }
                interface ethernet-1/2.0 {
                    admin-state enable
                }
            }
        }
    }
}
```

### srl2 Configuration

#### Same configuration format as srl1. Ensure Configuration Output
```sh
A:srl2# info
```
```
admin-state enable
version ospf-v2
instance-id 0
router-id 2.2.2.2
area 0.0.0.0 {
    advertise-router-capability true
    interface ethernet-1/1.0 {
        admin-state enable
        interface-type point-to-point
    }
}
```

### srl3 Configuration

### Creating Configuration for srl3 with Area 0 on eth1 and Area 1 Stub on eth2

```bash
set network-instance default protocols ospf instance default admin-state enable version ospf-v2 instance-id 0 area 0.0.0.0 router-id 3.3.3.3 advertise-router-capability true interface ethernet-1/1.0 admin-state enable interface-type point-to-point

set network-instance default protocols ospf instance default area 0.0.0.1 advertise-router-capability true interface ethernet-1/2.0 admin-state enable interface-type point-to-point

A:srl3# instance default area 0.0.0.1
A:srl3# set stub
```

### Verification

```bash
--{ running }--[ network-instance default protocols ]--
A:srl3# info
    ospf {
        instance default {
            admin-state enable
            version ospf-v2
            instance-id 0
            router-id 3.3.3.3
            area 0.0.0.0 {
                advertise-router-capability true
                interface ethernet-1/1.0 {
                    admin-state enable
                    interface-type point-to-point
                }
            }
            area 0.0.0.1 {
                advertise-router-capability true
                stub {
                }
                interface ethernet-1/2.0 {
                    admin-state enable
                    interface-type point-to-point
                }
            }
        }
    }
```

## srl4 Configuration

### Setting srl4 with Stub area

```bash
A:srl4# set protocols ospf instance default admin-state enable version ospf-v2 instance-id 0 router-id 4.4.4.4 area 0.0.0.1 advertise-router-capability true interface ethernet-1/1.0 admin-state enable interface-type point-to-point

A:srl4# protocols ospf instance default area 0.0.0.1
A:srl4# set stub
```

### Verification

```bash
--{ + candidate shared default }--[ network-instance default protocols ]--
A:srl4# info
    ospf {
        instance default {
            admin-state enable
            version ospf-v2
            instance-id 0
            router-id 4.4.4.4
            area 0.0.0.1 {
                advertise-router-capability true
                stub {
                }
                interface ethernet-1/1.0 {
                    admin-state enable
                    interface-type point-to-point
                }
            }
        }
    }
```

## Test Commands

```bash
ping
show network-instance default protocols ospf status
show network-instance default protocols ospf database
show network-instance default protocols ospf neighbor detail
show network-instance default route-table all
show network-instance default protocols ospf area 0.0.0.0 detail
show network-instance default protocols ospf interface detail
```
#### ping srl1 to srl4 result
![ping srl1 to srl4](https://github.com/user-attachments/assets/475a01fc-fbef-48bd-b1e2-4bff92d6f629)

#### ospf database srl1
![ospf database srl1](https://github.com/user-attachments/assets/dccaf430-aa46-49a4-a743-a7f006048960)

#### srl4 stub lsa database
![srl4 stub lsa database](https://github.com/user-attachments/assets/1fa9184d-73dd-4df2-968c-635d0b98747f)

#### srl1 route-table
![srl1 route-table](https://github.com/user-attachments/assets/0ef55a7a-2e39-4c9c-ab42-ed6ac2078170)

## Saving and Manage the Lab

```bash
containerlab save -t srl02-lab.yml

# Close the lab with 'CTRL + D'

# re-Start the lab by starting Docker container
docker ps -a
docker container start CONTAINER_ID

# If you find trouble with container status, destroy
containerlab destroy -t srl02-lab.yml

# You can start over by re-deploying. The config is saved inside the file in the directory
```
