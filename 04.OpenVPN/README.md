## OpenVPN on pfSense:

OpenVPN is an open source VPN solution which can provide access to remote access clients and enable site-to-site connectivity. OpenVPN supports clients on a wide range of operating systems including all the BSDs, Linux, Android, macOS, iOS, Solaris and Windows.

Every OpenVPN connection consists of a server and a client, for both remote access and site-to-site deployments. In the case of site-to-site VPNs, one firewall acts as the server and the other as the client. In most cases it does not matter which firewall acts in a particular role.



### Prerequisites:
- pfSense installed.
- A static public IP. 
- At least one user account for VPN access.



## Configure OpenVPN:


### Step-1: Generating the Certificate Authority (CA):

Navigate to `System` > `Cert. Manager` or `Certificates` > `Authorities` Tab > click `Add`:

- Create / Edit CA:
    - Descriptive name: `openVpnCert`
	- Method: **Create an internal Certificate Authority**
	
	- Trust Store: [ ] Add this Certificate Authority to the Operating System Trust Store
	- Randomize Serial: [ ] Use random serial numbers when signing certificates

- Internal Certificate Authority:
	- Key type: `RSA`
		- `2048`					[-> at least 2048, I will be using 4096 for example]
	- Digest Algorithm: `sha256`	[-> at least sha256, I will be using sha512 for example]
	- Lifetime (days): `3650`		[-> by default]
	- Common Name: `internal-ca`	[-> by default]
	
    - `Note`: The following certificate authority subject components are optional and may be left blank.
	- Country Code: `None`
	- State or Province:
	- City:
	- Organization:
	- Organizational Unit:
	
	- click `Save`.





### Step-2: Generating the Server Certificate:

Navigate to `System` > `Cert. Manager` or `Certificates` > `Certificates` Tab > click `Add/Sign`:

- Add/Sign a New Certificate:
	- Method: **Create an internal Certificate**
	- Descriptive name: `openVpnServerCert`

- Internal Certificate:
	- Certificate authority: `openVpnCert`
	
	- Key type: `RSA`				[-> Use the same values set for the Certificate Authority for the Key type and length,and Digest Algorithm]
		- `2048`					[-> at least 2048, I will be using 4096 for example]
	- Digest Algorithm: `sha256`	[-> at least sha256, I will be using sha512 for example]
	- Lifetime (days): `365`		
	- Common Name: `internal-sc` 	
	
    - `Note`: The following certificate subject components are optional and may be left blank.
	- Country Code: `None`
	- State or Province:
	- City:
	- Organization:
	- Organizational Unit:
	

- Certificate Attributes:
	- Attribute Notes
	- Certificate Type: `Server Certificate`
	- Alternative Names: FQDN or Hostname: [technbd.net]
	- click `Save`.





### Step-3: Create OpenVPN user and your user certificate:

Navigate to `System` > `User Manager` > `Users` Tab > click `Add`:

- User Properties:
	- Defined by:	USER
	- Username: `user1`
	- Password: `secure_pass`
	- Full name: `user1` 
	
	- Expiration date:

	- Custom Settings: [ ] Use individual customized GUI options and dashboard layout for this user.

	- Group membership:

	- Certificate: [ ] Click to create a user certificate
	
- Keys:
	- Authorized SSH Keys:
	
	- IPsec Pre-Shared Key: 

- Shell Behavior:
	- Keep Command History: [ ] Keep shell command history between login sessions

- click Save.



`Note`: If you chose to set up your server for certificate-based authentication or for certificate and password-based authentication, click the pencil icon to the right of your new user. You’re taken back to the Edit User window.






### Step-4: Creating the OpenVPN server:

Navigate to `VPN` > `OpenVPN` > `Servers` Tab > click `Add`:

- General Information:
	- Description: OpenVPN server
	- Disabled: 

- Mode Configuration:
	- Server mode: **Remote Access (User Auth)** or [-> Remote Access (SSL/TLS) or Remote Access (SSL/TLS + User Auth)]
	
	- Backend for authentication: `Local Database`
	- Device mode: **tun - Layer 3 Tunnel Mode**

- Endpoint Configuration:
	- Protocol: `UDP on IPv4 only`
	- Interface: `WAN`
	- Local port: `1194`

- Cryptographic Settings: 
	- TLS Configuration: [✔] Use a TLS Key
	    - [✔] Automatically generate a TLS Key.
	
	- Peer Certificate Authority: `openVpnCert`
	
    - Peer Certificate Revocation list: 

	- OCSP Check: [ ] Check client certificates with OCSP
	
	- Server certificate: select `openVpnServerCert` (Server: Yes, CA: openVpnCert)
	- DH Parameter Length: `4096 bit` 
	
	- ECDH Curve: `Use Default` 
	
	- Data Encryption Algorithms:
	
	- Fallback Data Encryption Algorithm: 
	
	- Auth digest algorithm: `SHA512 (512-bit)`
	- Hardware Crypto: N/A
	
	- Certificate Depth: `One (Client+Server)`
	- Client Certificate Key Usage Validation: [✔] Enforce key usage

- Tunnel Settings
	- IPv4 Tunnel Network: `192.168.2.0/24`
	- Redirect IPv4 Gateway: [✔] Force all client-generated IPv4 traffic through the tunnel.
		[-> in order to route all IPv4 traffic over the VPN tunnel]
	
	- Concurrent connections: 
	- Allow Compression: `Refuse any non-stub compression (Most secure)`
	
	- Push Compression: [ ] Push the selected Compression setting to connecting clients.
	- Type-of-Service: [ ] Set the TOS IP header value of tunnel packets to match the encapsulated packet value.
	
	- Inter-client communication: [ ] Allow communication between clients connected to this server
	- Duplicate Connection: [ ] Allow multiple concurrent connections from the same user
	
	
- Client Settings
	- Dynamic IP: [ ] Allow connected clients to retain their connections if their IP address changes.
	- Topology: Subnet - One IP address per client in a common subnet 
	
- Ping settings
	- Inactive: 300 
	- Ping method: keepalive - Use keepalive helper to define ping configuration
	- Interval: 10
	- Timeout: 60

- Advanced Client Settings:
	- DNS Default Domain: [ ] Provide a default domain name to clients
	- DNS Server enable: [ ] Provide a DNS server list to clients. Addresses may be IPv4 or IPv6.
	
	- Block Outside DNS: [ ] Make Windows 10 Clients Block access to DNS servers except across OpenVPN while connected, forcing clients to use only VPN DNS servers.
	- Force DNS cache update: [ ] Run "net stop dnscache", "net start dnscache", "ipconfig /flushdns" and "ipconfig /registerdns" on connection initiation.

	- NTP Server enable: [ ] Provide an NTP server list to clients
	- NetBIOS enable: [ ] Enable NetBIOS over TCP/IP

- Advanced Configuration:
	- Custom options: 
	- Username as Common Name: [ ] Use the authenticated client username instead of the certificate common name (CN). 
	
	- UDP Fast I/O: [✔] `Use fast I/O operations with UDP writes to tun/tap. Experimental.`
	
	- Exit Notify: `Disable`
	- Send/Receive Buffer: `Default`
	
	- Gateway creation: [✔] `IPv4 only `
	- Verbosity level: `default`
	- click `Save`



#### Verifying:

Navigate to `Status` > `System Logs` > `OpenVPN` Tab: 

Output: Initialization Sequence Completed




### Step-5: Create firewall OpenVPN rule:

Navigate to `Firewall` > `Rules` > `OpenVPN` Tab > Click `Add`:

- Edit Firewall Rule:
	- Action: Pass 
	
	- Interface: `OpenVPN`
	- Address Family: `IPv4`
	- Protocol: `Any` 		[-> to set "Any"]

- Source:
	- [ ] Invert match	>	 Network	> Source Address: 192.168.2.0 > /24 


- Destination:
	- [ ] Invert match	> any	> Destination Address: 

- Extra Options:
	- Log: [ ] Log packets that are handled by this rule
	- Description: openvpn-incoming-rule
	- Advanced Options: 
	
- click `Save`





### Step-6: Create firewall WAN rule: 

Navigate to `Firewall` > `Rules` > `WAN` Tab > Click `Add`:

- Edit Firewall Rule:
	- Action: `Pass` 
	
	- Interface: `WAN` 
	- Address Family: `IPv4`
	- Protocol: `UDP`  


- Source:	[-> **Make sure Source is set to Any**]
	- [ ] Invert match	> any	> Source Address: 

- Destination:
	- [ ] Invert match	>	WAN address	>	Destination Address:
	- Destination Port Range: (other)   1194  (other) [-> Custom port]

- Extra Options:
	- Log: [] Log packets that are handled by this rule
	- Description: Allow-openvpn
	- Advanced Options: 
	
- click `Save`





### Step-7: Install the OpenVPN Client Export Utility:


_Update Repo:_

```
pkg bootstrap -f
pkg-static install -fy pkg pfSense-repo pfSense-upgrade
```


Navigate to `System` > `Package Manager` > `Available Packages` Tab: 

- Search term: openvpn

- Packages Name: `openvpn-client-export v1.9.2`
- click `Install` and `Confirm`




### Step-8: Export the OpenVPN client configuration: 

Scroll down to the bottom of the page, and you’ll find generated configurations for various systems and apps. Click on the appropriate configuration for your device(s) to download it to your computer.

Navigate to `VPN` > `OpenVPN` > `Cient Export` Tab:

- OpenVPN Server:
	- Remote Access Server: `OpenVPN server UDP4:1194`

- Client Connection Behavior:
	- Host Name Resolution: `Interface IP Address`
	- Verify Server CN:
	- Block Outside DNS:
	- Legacy Client:
	- Silent Installer:
	- Bind Mode:
- Certificate Export Options:
	- PKCS#11 Certificate Storage: 
	- Microsoft Certificate Storage: 
	- Password Protect Certificate:
	- PKCS#12 Encryption:
- Proxy Options:
	- Use A Proxy: 
- Advanced:
	- Additional configuration options:

- Search: 
- OpenVPN Clients: 
	- Inline Configurations: (Download the Clients configure)
		- Most Clients (`pfSense-UDP4-1194-config.ovpn`)
	- Current Windows Installer (2.6.7-Ix001) (Download the Installer)
		- 64-bit 





### Step-9: Verify Client Connect:
- Open `OpenVPN Connect`
- Import to click Plus (+) icon > Upload File > select `pfSense-UDP4-1194-config.ovpn`
	- Username: user1
	- Password: 













