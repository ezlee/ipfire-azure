Prepping a VHD for IPFire on Azure
====

Prepping a disk for IPFire on Azure is pretty straight forward, and follows the typical procedure to prep any other operating system to be used on Azure using Hyper-V. You'll need to download the [i586 ISO for IPFire](http://download.ipfire.org/release/ipfire-2.19-core109). The 64 bit ISO will not work.

1. Launch Hyper-V Manager. From the **Action** menu, select **New** -> **Virtual Machine**

1. In the Wizard, click **Next** on the **Before You Begin** screen.

1. For **Name and Location**, **name** the VM something and take the defaults on the **location**.

1. For **Generation**, select **Generation 1**.

1. For **Assign Memory**, Assign **1024** MB.

1. For **Configure Networking**, select **nat**.

1. For 	**Connect Virtual Hard Drive**, set the size to **8** GB

1. For **Installation Options**, Select **Install Operating System from a bootable CD/DVD Drive**, then select **Image file (.iso)**, then browse to the downloaded ISO for IPFire.

1. Click **Finish** and let the Wizard create the VM.

1. Right click the newly created VM, and select **Settings**.

1. Select **Add Hardware**, select **Network Adapter**, then click **Add**.

1. Select the newly added **Network Adapter**, and set the Virtual Switch to whatever virtual Switch is bound to the NIC in your computer.

1. Once this is set, close the settings and start the VM and install IPFire as normal.

1. On the reboot when you run the wizard for IPFire, bind the RED interface to the first Network Adapter in the list and then bind the GREEN interface to the second adapter in the list. Set the **admin** and **root** passwords.

1. For the address settings, set the RED interface to use DHCP and set the GREEN interface's IP to 10.0.1.5. Do not enable the DHCP server on the GREEN interface. [Add an additional IP address of 10.0.1.10 to the Virtual Switch](https://www.oclc.org/support/services/ezproxy/documentation/technote/2w.en.html) on the Network Settings in Windows 

1. Open a web browser and point the URL to https://10.0.1.5:444 and login with **admin** and the password you set. This should launch the IPFire web interface

1. On the On the **System** menu, select **SSH Access**. Enable all the following: 
	* **SSH Access**, 
	* **Allow TCP Forwarding**, 
	* **Allow password based authentication**
	* **SSH port set to 22 (default is 222)**. 

	Then click **Save**.

1. On the **Firewall Menu**, select **Firewall Rules**, then click **New Rule**. On the new rule:

	* Under **Source**, select **Standard Network** and pick **Any** from the list.
	* For **Destination**, select **Firewall** and pick **RED** from the list.
	* Under Protocol, select TCP, and set the **Destination port** to **22**.
	* Then select **Accept** as the policy.

	Lastly, click **Add**

1. Click **Apply Rules** to apply the new rule.

1. SSH into the VM with an SSH client. Simply use the IP (10.0.1.5) as the host name and **root** as the logon name.

1. In the console, edit **/var/sysconfig/firewall.local** and add the following code in the Start block.

	````
	if [ -e "/sys/class/net/eth0/address" ]; then
			ethad=$(cat /sys/class/net/eth0/address)
			sed "s/RED_MACADDR=.*/RED_MACADDR=$ethad/g" /var/ipfire/ethernet/settings
			ifconfig eth0 down
			ip link set eth0 name red0
			ifconfig red0 up
	fi

	if [ -e "/sys/class/net/eth1/address" ]; then
			ethad=$(cat /sys/class/net/eth1/address)
			sed "s/GREEN_MACADDR=.*/GREEN_MACADDR=$ethad/g" /var/ipfire/ethernet/settings
			ifconfig eth1 down
			ip link set eth1 name green0
			ifconfig green0 up
	fi

	````

	Save the file.

1. Shut down the VM.

1. In Hyper-V after the VM shuts down, Right click the VM and click **Settings**. Select the **ipfire.vhdx** and click **Edit**. Click **Choose Action**, then click **Convert**. On the next screen, select **VHD**, then click next.  First **Disk Type**, select **Dynamically expanding**. Note the location of the output, and name the disk image ipfire.vhd. then click **Finish**.

The Image is prepped and ready for use on Azure.
	


