# ONTAP Simulator on Proxmox

## Index:
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Instruction](#instruction)
- [Very Important Notes](#very-important-notes)

## Overview:
This is my personal ontap simulator lab setup on Proxmox. I have cover all the details herein. I will try to update the instructions as needed.

The steps mentioned below are generalized steps, assuming that you already know how to install and configure the Proxmox environment also you are knowing ontap simulator installation.

## Prerequisites:
- **ONTAP Simulator version:** 9.10.1
- **Proxmox version:** Virtual Environment 8.3.3
- **Network:** Network interfaces of the ONTAP are bridged to the interface of the Proxmox server.

## Instructions:
### 1. Download the ONTAP Simulator
Download the ONTAP simulator `.ova` file from NetApp's official website:
[NetApp ONTAP Simulator Download](https://mysupport.netapp.com/site/tools/tool-eula/simulate-ontap) 

Ensure you also download the `.txt` file with license keys.

### 2. Extract the `.ova` File
```bash
sudo tar -xvf vsim-netapp-DOT9.10.1-cm_nodar.ova
```

After extracting, you should see the following files:
```bash
ls -l
vsim-NetAppDOT-simulate-disk1.vmdk
vsim-NetAppDOT-simulate-disk2.vmdk
vsim-NetAppDOT-simulate-disk3.vmdk
vsim-NetAppDOT-simulate-disk4.vmdk
vsim-NetAppDOT-simulate.mf
vsim-NetAppDOT-simulate.ovf
```

### 3. Upload `.vmdk` Files to Proxmox
Use `scp` to transfer the `.vmdk` files to your Proxmox server:
```bash
sudo scp vsim-NetAppDOT-simulate-disk1.vmdk root@192.168.1.100:/root
sudo scp vsim-NetAppDOT-simulate-disk2.vmdk root@192.168.1.100:/root
sudo scp vsim-NetAppDOT-simulate-disk3.vmdk root@192.168.1.100:/root
sudo scp vsim-NetAppDOT-simulate-disk4.vmdk root@192.168.1.100:/root
```

### 4. Create a New VM in Proxmox
1. Go to **Proxmox GUI** and create a new VM with the following configuration:
    - **Name:** ONTAP
    - **VM ID:** 500
2. **OS Tab:** Select *Do not use any media*.
3. **System Tab:** Keep default settings.
4. **Disks Tab:** Delete the default disk (system-generated).
5. **CPU Tab:** Assign **at least 2 cores**, set type to *Host*.
6. **Memory:** Allocate **8 GB** (minimum requirement for ONTAP simulator).
7. **Network Tab:** Use default bridge (`vmbr0`) and set model to *Intel E1000*.
8. **Review settings and finish the setup.**

### 5. Add an Additional Network Interface
- Go to the **Hardware** section of the newly created VM.
- Add one more **network interface** with:
  - **Bridge:** `vmbr0`
  - **Model:** *Intel E1000*

### 6. Convert and Import `.vmdk` Files
Run these commands in the **Proxmox console**:
```bash
qm disk import 500 vsim-NetAppDOT-simulate-disk1.vmdk local-lvm -format qcow2
qm disk import 500 vsim-NetAppDOT-simulate-disk2.vmdk local-lvm -format qcow2 
qm disk import 500 vsim-NetAppDOT-simulate-disk3.vmdk local-lvm -format qcow2
qm disk import 500 vsim-NetAppDOT-simulate-disk4.vmdk local-lvm -format qcow2
```

After importing, go to **Hardware** > Edit each imported disk and:
- Change **Bus/Device** to **IDE**.
- Repeat for all three disks, maintaining the original order.

### 7. Configure Boot Order
- Go to **Options** > **Boot Order**
- Set `ide0` as the first boot device.

### 8. Start the VM
Power on the VM and follow the ONTAP simulator setup instructions.

At this point, you should have a running ONTAP simulator on Proxmox.

## Very Important Notes:
- Ensure you have **at least 8GB RAM** allocated for smooth operation.
- The **network interfaces must be bridged** to allow ONTAP to obtain an IP.
- Follow the correct **disk order** while importing and assigning disks.
- Use **Intel E1000** as the network model to avoid compatibility issues.

Happy Lab Building! 🚀
