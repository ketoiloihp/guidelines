####Install openswan

	yum install strongswan -y
	
add service automatically at boot & start:

	systemctl enable strongswan
	systemctl start strongswan

enable ***firewalld*** for ***openswan***:

	firewall-cmd --add-service ipsec
	firewall-cmd --permanent --add-service ipsec

make sure the udp port is opened:

	nc -vz -u {vpn_public_ip} 4500
	
#
Ref:
- [5 Useful Examples of firewall-cmd command](https://www.thegeekdiary.com/5-useful-examples-of-firewall-cmd-command/) 
- [IPSEC VPN on Centos 7 with StrongSwan](https://raymii.org/s/tutorials/IPSEC_vpn_with_CentOS_7.html)

####Create VPC on AWS

###
Ref:
- [Extending VPN Connectivity to Amazon AWS VPC using AWS VPC VPN Gateway Service](https://docs.openvpn.net/configuration/extending-vpn-connectivity-to-amazon-aws-vpc-using-aws-vpc-vpn-gateway-service/) 

####Configure on-premises VPN server
Edit ***/etc/strongswan/ipsec.conf***: ( [Ref.](https://gist.github.com/heri16/2f59d22d1d5980796bfb#file-ipsec-conf-L60) )

	vi /etc/strongswan/ipsec.conf
###
	# ipsec.conf - strongSwan IPsec configuration file
	
	# basic configuration
	
	config setup
	        charondebug="cfg 2, ike 3"
	conn %default
	        # Authentication Method    : Pre-Shared Key
	        authby=psk
	        leftauth=psk
	        rightauth=psk
	        # Encryption Algorithm     : aes-128-cbc
	        # Authentication Algorithm : sha1
	        # Perfect Forward Secrecy  : Diffie-Hellman Group 2
	        ike=aes256-sha256-modp2048s256,aes128-sha1-modp1024!
	        # Lifetime                 : 28800 seconds
	        ikelifetime=28800s
	        # Phase 1 Negotiation Mode : main
	        aggressive=no
	        # Protocol                 : esp
	        # Encryption Algorithm     : aes-128-cbc
	        # Authentication Algorithm : hmac-sha1-96
	        # Perfect Forward Secrecy  : Diffie-Hellman Group 2
	        esp=aes128-sha256-modp2048s256,aes128-sha1-modp1024!
	        # Lifetime                 : 3600 seconds
	        lifetime=3600s
	        # Mode                     : tunnel
	        type=tunnel
	        # DPD Interval             : 10
	        dpddelay=10s
	        # DPD Retries              : 3
	        dpdtimeout=30s
	        # Tuning Parameters for AWS Virtual Private Gateway:
	        keyexchange=ikev1
	        #keyingtries=%forever
	        rekey=yes
	        reauth=no
	        dpdaction=restart
	        closeaction=restart
	        left=%defaultroute
	        #leftsubnet=0.0.0.0/0,::/0
	        #rightsubnet=0.0.0.0/0,::/0
	        #leftupdown=/etc/strongswan/ipsec-vti.sh
	        installpolicy=yes
	        compress=no
	        leftsubnet= {vpn_subnetmask} #10.0.0.0/16
	        rightsubnet={vpc_subnetmask} #10.7.0.0/16
	        
	conn VPC-GW1
		right={vpc_vpn_public_ip1}
		auto=start

	conn VPC-GW2
		right={vpc_vpn_public_ip2}
		auto=start

We secure the connection between both machines with PSK. Therefor we have to set the PSK for machine.

	vi /etc/strongswan/ipsec.secrets
###
	# ipsec.secrets - strongSwan IPsec secrets file
	{ip_vpn_public} {vpc_vpn_public_ip1} : PSK "..."
	{ip_vpn_public} {vpc_vpn_public_ip2} : PSK ".."

Finally we have to configure port forwarding on vpn. Add the following line to ***/etc/sysctl.conf***.

	net.ipv4.ip_forward=1
run command

	sysctl -p
	
restart *openswan*

	strongswan restart
	
check status:

	strongswan status
	
view logs ( openswan save the logs on ***/var/log/message*** ):

	tail /var/log/message

###
Ref: 
- [Connect two AWS VPCs with StrongSwan](https://www.peternijssen.nl/connect-multiple-aws-regions-with-strongswan) 
- [How to Build a Secure Tunnel from Your On-Premises Data Center to Amazon Cloud](https://www.stratoscale.com/blog/cloud/build-secure-tunnel-on-prem-data-center-amazon-cloud/) 
- [Connecting Multiple VPCs with EC2 Instances (IPSec)](https://aws.amazon.com/articles/connecting-multiple-vpcs-with-ec2-instances-ipsec/) 
