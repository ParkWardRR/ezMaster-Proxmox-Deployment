# **ezMaster-Proxmox-Deployment**

This repository provides step-by-step instructions to convert the EnGenius ezMaster OVA file and deploy it on a Proxmox VE server. Follow these steps to extract, convert, and configure the VM for successful deployment.

---

## **Prerequisites**

1. Proxmox VE installed and configured.
2. SSH access to your Proxmox server.
3. The ezMaster OVA file downloaded (e.g., `ezMaster_Rel_v1.0.62.ova`).

---

## **Steps to Deploy ezMaster on Proxmox**

### **1. Transfer the OVA File to Proxmox**
Copy the OVA file to your Proxmox server using `scp`:
```bash
scp ~/Downloads/ezMaster_Rel_v1.0.62.ova root@<Proxmox_IP>:/var/lib/vz/template/
```

---

### **2. Extract the OVA File**
SSH into your Proxmox server and extract the OVA file:
```bash
ssh root@<Proxmox_IP>
cd /var/lib/vz/template/
tar -xvf ezMaster_Rel_v1.0.62.ova
```
This will extract the following files:
- `ezMaster_Rel_v1.0.62.ovf` (VM configuration)
- `ezMaster_Rel_v1.0.62-disk1.vmdk` (disk image)

---

### **3. Convert the Disk Image to QCOW2**
Convert the VMDK disk image to QCOW2 format using `qemu-img`:
```bash
qemu-img convert -f vmdk -O qcow2 ezMaster_Rel_v1.0.62-disk1.vmdk ezMaster_Rel_v1.0.62.qcow2
```

---

### **4. Create a New VM**
Create a new VM in Proxmox with basic settings:
```bash
qm create 101 --name ezMaster --memory 2048 --net0 virtio,bridge=vmbr0
```
Replace `101` with your desired VM ID.

---

### **5. Import the Converted Disk**
Import the QCOW2 disk into Proxmox storage (e.g., `local-lvm`):
```bash
qm importdisk 101 /var/lib/vz/template/ezMaster_Rel_v1.0.62.qcow2 local-lvm
```

---

### **6. Attach the Disk to the VM**
Attach the imported disk to your VM as a SCSI drive:
```bash
qm set 101 --scsi0 local-lvm:vm-101-disk-0
```

---

### **7. Set Boot Order**
Ensure the VM boots from the SCSI drive by configuring the boot order:
```bash
qm set 101 --boot order=scsi0
```

---

### **8. Start the VM**
Start your VM and verify that it boots correctly:
```bash
qm start 101
```

Access the VM console via the Proxmox web interface.

---

## **Cleanup Steps**
After verifying that your VM is working, clean up unnecessary files to save disk space:
```bash
rm /var/lib/vz/template/ezMaster_Rel_v1.0.62-disk1.vmdk
rm /var/lib/vz/template/ezMaster_Rel_v1.0.62.qcow2
rm /var/lib/vz/template/ezMaster_Rel_v1.0.62.ovf
```

---

## **Default Login Credentials**
The default login credentials for EnGenius ezMaster are:
- **Username:** `admin`
- **Password:** `password`

*Make sure to change these credentials after logging in for security purposes.*

---

## **Troubleshooting**

- If the VM does not boot correctly, ensure that you have set the correct boot order using:
  ```bash
  qm set 101 --boot order=scsi0
  ```
- If you encounter issues with disk conversion, double-check that `qemu-img` is installed on your Proxmox server.
