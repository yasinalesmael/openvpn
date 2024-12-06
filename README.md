
<p align="center">
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/f/f5/OpenVPN_logo.svg/512px-OpenVPN_logo.svg.png?20210503094426" alt="OpenVPN logo" width="200"/>
</p>

<h1>OpenVPN Setup on Windows 10 Azure VM</h1>
This tutorial outlines the prerequisites and installation of OpenVPN on a Windows 10 virtual machine (VM) hosted in Microsoft Azure.<br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)<br>
- OpenVPN Community Edition<br>
- EasyRSA (for certificate management)<br>

<h2>Operating Systems Used</h2>

- Windows 10 Pro (22H2), 2vCPUs, 4GB RAM<br>

<h2>List of Prerequisites</h2>

- <b>Microsoft Azure Account</b> - Required to create and manage virtual machines.<br>
- <b>OpenVPN Installer</b> - Download from the official OpenVPN site.<br>
- <b>Firewall Configuration</b> - Open UDP port 1194 on the VM and network.<br>
- <b>Remote Desktop Access</b> - To configure the Windows 10 VM.<br>

<h2>Installation Steps</h2>
<p>
Follow these steps to set up OpenVPN on a Windows 10 Azure VM.<br />
</p>

<h3>Step 1: Create a Windows 10 Virtual Machine in Azure</h3>
- Sign in to the <a href="https://portal.azure.com/">Azure Portal</a>.<br>
- Navigate to "Create a resource" > "Compute" > "Windows 10 Pro".<br>
- Configure the VM:<br>
  - **Resource Group**: Select or create a new group.<br>
  - **VM Name**: Assign a unique name.<br>
  - **Region**: Choose a suitable region.<br>
  - **Size**: At least 2vCPUs and 4GB RAM.<br>
  - **Administrator Account**: Set a username and password.<br>
  - **Inbound Port Rules**: Allow RDP (port 3389).<br>
- Review and create the VM.<br>

<h3>Step 2: Configure Networking for OpenVPN</h3>
- Add an inbound security rule for OpenVPN UDP traffic:<br>
  - Navigate to the "Networking" section of your VM in the Azure portal.<br>
  - Add a rule for **UDP port 1194**:<br>
    - **Source**: Any<br>
    - **Source Port Ranges**: *<br>
    - **Destination**: Any<br>
    - **Destination Port Ranges**: 1194<br>
    - **Protocol**: UDP<br>
    - **Action**: Allow<br>
    - **Name**: "Allow-OpenVPN-UDP-1194".<br>

<h3>Step 3: Install OpenVPN</h3>
- Use Remote Desktop to connect to your VM using its public IP address.<br>
- Download the OpenVPN installer from the <a href="https://openvpn.net/community-downloads/">OpenVPN Community Downloads</a> page.<br>
- Run the installer and select the "EasyRSA" component during installation.<br>
- Complete the installation process.<br>

<h3>Step 4: Configure OpenVPN Server</h3>
- Open Command Prompt as Administrator and navigate to the EasyRSA directory:<br>
  ```bash
  cd "C:\Program Files\OpenVPN\easy-rsa"
  ```<br>
- Initialize EasyRSA and generate certificates:<br>
  ```bash
  .\EasyRSA-Start.bat
  .\easyrsa init-pki
  .\easyrsa build-ca nopass
  .\easyrsa gen-dh
  .\easyrsa build-server-full server nopass
  .\easyrsa build-client-full client1 nopass
  .\easyrsa gen-crl
  ```<br>
- Configure the server:<br>
  - Copy the `server.ovpn` sample file from `C:\Program Files\OpenVPN\sample-config` to `C:\Program Files\OpenVPN\config`.<br>
  - Edit `server.ovpn` to include:<br>
    - `port 1194`<br>
    - `proto udp`<br>
    - `dev tun`<br>
    - Paths to the generated certificates and keys.<br>

<h3>Step 5: Enable IP Forwarding and NAT</h3>
- Open Registry Editor (`regedit`) and navigate to:<br>
  ```
  HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters
  ```<br>
- Set `IPEnableRouter` to `1`.<br>
- Open Command Prompt as Administrator and enable NAT:<br>
  ```bash
  netsh interface ipv4 set interface <InterfaceNumber> forwarding=enabled
  netsh interface portproxy add v4tov4 listenport=1194 listenaddress=0.0.0.0 connectport=1194 connectaddress=<VM's Private IP>
  ```<br>

<h3>Step 6: Start OpenVPN Service</h3>
- Open the Services app (`services.msc`).<br>
- Locate "OpenVPNService".<br>
- Set its startup type to "Automatic".<br>
- Start the service.<br>

<h3>Step 7: Configure Client Devices</h3>
- Generate a client configuration file:<br>
  - Copy the `client.ovpn` file to the `C:\Program Files\OpenVPN\config` folder.<br>
  - Update the file with your Azure VM's public IP address and the client key paths.<br>
- Share the `client.ovpn` file with your client devices for connection.<br>

<h3>Step 8: Test the VPN</h3>
- Connect a client device using the generated configuration file.<br>
- Verify the connection by visiting <a href="https://ipleak.net">ipleak.net</a> to ensure your public IP matches the Azure VM.<br>
