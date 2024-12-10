
<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/f/f5/OpenVPN_logo.svg/512px-OpenVPN_logo.svg.png?20210503094426" alt="OpenVPN logo" width="200"/>
</p>

# OpenVPN Setup on Windows 10 Azure VM

This tutorial outlines the prerequisites and installation of OpenVPN on a Windows 10 virtual machine (VM) hosted in Microsoft Azure.

## Environments and Technologies Used

- Microsoft Azure (Virtual Machines/Compute)
- OpenVPN Community Edition
- EasyRSA (for certificate management)

## Operating Systems Used

- Windows 10 Pro (22H2), 2vCPUs, 4GB RAM

## List of Prerequisites

- **Microsoft Azure Account**: Required to create and manage virtual machines.
- **OpenVPN Installer**: Download from the official OpenVPN site.
- **Firewall Configuration**: Open UDP port 1194 on the VM and network.
- **Remote Desktop Access**: To configure the Windows 10 VM.

## Installation Steps

Follow these steps to set up OpenVPN on a Windows 10 Azure VM.

### Step 1: Create a Windows 10 Virtual Machine in Azure

1. Sign in to the [Azure Portal](https://portal.azure.com/).
2. Navigate to **Create a resource** > **Compute** > **Windows 10 Pro**.
3. Configure the VM:
   - **Resource Group**: Select or create a new group.
   - **VM Name**: Assign a unique name.
   - **Region**: Choose a suitable region.
   - **Size**: At least 2vCPUs and 4GB RAM.
   - **Administrator Account**: Set a username and password.
   - **Inbound Port Rules**: Allow RDP (port 3389).
4. Review and create the VM.

### Step 2: Configure Networking for OpenVPN

1. Add an inbound security rule for OpenVPN UDP traffic:
   - Navigate to the **Networking** section of your VM in the Azure portal.
   - Add a rule for **UDP port 1194**:
     - **Source**: Any
     - **Source Port Ranges**: `*`
     - **Destination**: Any
     - **Destination Port Ranges**: `1194`
     - **Protocol**: UDP
     - **Action**: Allow
     - **Name**: `Allow-OpenVPN-UDP-1194`

### Step 3: Install OpenVPN

1. Use Remote Desktop to connect to your VM using its public IP address.
2. Download the OpenVPN installer from the [OpenVPN Community Downloads](https://openvpn.net/community-downloads/) page.
3. Run the installer and select the "EasyRSA" component during installation.
4. Complete the installation process.

### Step 4: Configure OpenVPN Server

1. Open Command Prompt as Administrator and navigate to the EasyRSA directory:
   ```bash
   cd "C:\Program Files\OpenVPN\easy-rsa"
   ```
2. Initialize EasyRSA and generate certificates:
   ```bash
   .\EasyRSA-Start.bat
   .\easyrsa init-pki
   .\easyrsa build-ca nopass
   .\easyrsa gen-dh
   .\easyrsa build-server-full server nopass
   .\easyrsa build-client-full client1 nopass
   .\easyrsa gen-crl
   ```
3. Configure the server:
   - Copy the `server.ovpn` sample file from `C:\Program Files\OpenVPN\sample-config` to `C:\Program Files\OpenVPN\config`.
   - Edit `server.ovpn` to include:
     ```
     port 1194
     proto udp
     dev tun
     ```
     - Add paths to the generated certificates and keys.

### Step 5: Enable IP Forwarding and NAT

1. Open Registry Editor (`regedit`) and navigate to:
   ```
   HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters
   ```
2. Set `IPEnableRouter` to `1`.
3. Open Command Prompt as Administrator and enable NAT:
   ```bash
   netsh interface ipv4 set interface <InterfaceNumber> forwarding=enabled
   netsh interface portproxy add v4tov4 listenport=1194 listenaddress=0.0.0.0 connectport=1194 connectaddress=<VM's Private IP>
   ```

### Step 6: Start OpenVPN Service

1. Open the Services app (`services.msc`).
2. Locate `OpenVPNService`.
3. Set its startup type to `Automatic`.
4. Start the service.

### Step 7: Configure Client Devices

1. Generate a client configuration file:
   - Copy the `client.ovpn` file to the `C:\Program Files\OpenVPN\config` folder.
   - Update the file with your Azure VM's public IP address and the client key paths.
2. Share the `client.ovpn` file with your client devices for connection.

### Step 8: Test the VPN

1. Connect a client device using the generated configuration file.
2. Verify the connection by visiting [ipleak.net](https://ipleak.net) to ensure your public IP matches the Azure VM.
