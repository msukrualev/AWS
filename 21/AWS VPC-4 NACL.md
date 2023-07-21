
# Hands-On AWS VPC NACL:
-----------------------------


Architecture Diagram
====================

![Architecture Diagram](./task_id_aws_vpc_nacl.png)

Task Details 
============


7.  Launching an EC2 Instance in the Public Subnet

8.  Launching an EC2 Instance in the Private Subnet

9.  Testing Both EC2 instances.

10. Creating Custom NACL and Associate it to the Subnet

11. Testing the Public and Private Server 

12. Adding Rules to Custom NACL (MyPublicNACL)

13. Testing Both EC2 instances

14. Validation of the lab

Lab Steps
=========


Task 7: Launching an EC2 Instance in the Public Subnet
------------------------------------------------------

1.  Navigate to **EC2** by clicking on the **Services** menu in the top, then click on **ÈC2** in the **Compute**  section.

2.  Navigate to **Instances**from the left side menu and click on **Launch Instances**button.

3.  **Choose an Amazon Machine Image (AMI):** Search for **Amazon Linux 2 AMI** in the search box and click on the **select** button.

4.  Choose an Instance Type: Select **t2.micro** and click on   **Next: Configure Instance Details**

5.  Configure Instance Details:

    -   **Network        :** Choose **MyVPC**

    -   **Subnet            :** Choose **MyPublicSubnet**

    -   **Auto-assign Public IP    :**  **Enable**

    -   Under the **User data:** section, enter the following script to create an HTML page served by Apache:


```bash
#!/bin/bash      
yum update -y                                                       
yum install httpd -y                                                
echo "<html><h1>Welcome to my Website</h1\><html>" >> /var/www/html/index.html                                              
systemctl start httpd                                               
systemctl enable httpd                                             
```
   
   -   Leave all other settings as default. Click on  **Next: Add Storage**

6.  Add Storage: No need to change anything in this step, click on **Add Tag**

7.  **Add Tags:** Click on **Add Tag**

    -   Key    : Enter ***Name***

    -   Value    : Enter ***MyPublicEC2Server***

    -   Click on **Next: Configure Security Group**

8.  Configure Security Group:

    -   Security group name: Enter** ***MyWebserverSG***

    -   Description : Enter ***My EC2 Security Group***

    -   SSH is already available,

        -   **Choose Type:** ***SSH***

        -   **Source :** Select ***Anywhere*** 

    -   For **HTTP**, click on ***Add Rule***,

    -   **ChooseType:** ***HTTP***

    -   **Source :** Enter ***Anywhere*** (From ALL IP addresses accessible).

    -   Click on **Review and Launch**.

        1.  **Review and Launch:** Review all settings and click on **Launch.**

        2.  **Key Pair :** Create a new key Pair, Key pair name : ***Mykey*** and click on****Download Key Pair**after that click on **Launch Instances**.

        3.  **Launch Status:** Your instance is now launching. Select the instance and wait for the instance to change status to **Running**.

        4.  Note the Public IP address of **MyPublicEC2Server** 

Task 8: Launching an EC2 Instance in the Private Subnet
-------------------------------------------------------

1.  Click on **Launch Instances**

2.  ***Search and Choose Amazon Linux 2 AMI:***

3.  Choose an Instance Type: Select **t2.micro**  and click on the **Next: Configure Instance Details**

4.  Configure Instance Details: 

    -   **Network**        : Choose **MyVPC**

    -   **Subnet**           : Choose **MyPrivateSubnet**

    -   **Auto-assign Public IP**    : Use Subnet Setting(Disable) - default

    -   Leave all other settings as default.

    -   Click on **Next: Add Storage**

5.  Add Storage: No need to change anything in this step, click on  **Next: Add Tags**

6.  **Add Tags:** Click on **Add Tag**

    -   Key    : Enter ***Name***

    -   Value    : Enter ***MyPrivateEC2Server***

    -   Click on **Next: Configure Security Group**

7.  Configure Security Group:

    -   Security group name: Enter ***MyServerSG***

    -   Description : Enter ***My EC2 Security Group***

    -   SSH is already available,

        -   **Choose Type:** ***SSH***

        -   **Source**: ***custom : 0.0.0.0/0***

    -   For **ALL ICMP IPv4** , Click on ***All ICMP -IPv4***

        -   **Choose Type:** ***All ICMP -IPv4***

        -   **Source:**   **Anywhere** (From ALL IP addresses accessible).

    -   Click on **Review and Launch**

8.  Review and Launch : Review all settings and click on **Launch**

9.  Key Pair : Select the existing key pair created earlier (**MyKey.pem**).

10. Click on **Launch Instances**.

11. Launch Status: Your instance is now launching, Select the instance and wait for the instance to change status to **Running**.

12. Note the Private IP Address of **MyPrivateEC2Server.**

13. Two servers are launched and ready.

Task 9: Testing Both EC2 instances
-----------------------------------

1.  **Public EC2 instances**: We have installed a web application on  this server.

    -   Select the **MyPublicEC2Server**EC2 instance from the instancelist.

    -   From the Description tab, copy the **IPv4 Public IP**.

    -   Now paste this IP in you Web Browser

    -   You will be able to see a simple web page.
  
2.  Next, we will try to ping the Private EC2 from the Public EC2instance.

    -   SSH into EC2 Instance

-   Copy the Private IP of **MyPrivateEC2Server** from the Description tab.

-   Ping the Private Instance using the Private IPv4
-   Example:

-   Press [Ctrl] + C to stop instead of pause.

-   **Note: You were able to do these tasks because the Default NACL that was created during VPC creation allows both INBOUND and  OUTBOUND by Default.**


Task 9.5: Modifying default NACL and Testing
----------------------------------------------

- Go to VPC and find the default NACL
- Select Network ACL and click Inbound Rules
- Click Edit Inbound Rules to edit
- What you see should look like this

100	All traffic	All	All	0.0.0.0/0	Allow
*	All traffic	All	All	0.0.0.0/0	Deny

- Now add a line after 100 corresponding your IP to deny reach on port 80
100	All traffic	All	All	0.0.0.0/0	Allow
101 HTTP(80) TCP 80    78.45.67.190/32 Deny
*	All traffic	All	All	0.0.0.0/0	Deny

- Save changes
- Now check the public IP of the web instance to see if you can still reach. Can you?
- Now go to default NACL again and this time modify the rule number so that it looks like:
100	All traffic	All	All	0.0.0.0/0	Allow
99 HTTP(80) TCP 80    78.45.67.190/32 Deny
*	All traffic	All	All	0.0.0.0/0	Deny

- Save changes and then check the web instance again. Can you see the page?
- NACL rule numbers are put in ascending order.

Task 10: Creating Custom NACL and Associate it to the Subnet
------------------------------------------------------------

**Note:** By default, all subnets will be associated with the Default NACL of **MyVPC.** Once you create a custom NACL, attach all public and private subnets.

1.  Navigate to **VPC** under the Services menu. Click on ***Network ACL***  under **Security**

2.  Click on **Create Network ACL**

3.  Create Network ACL:

    -   Name tag: Enter ***MyPublicNACL***

    -   VPC: Select **MyVPC****from the dropdown list.

    -   Click on **Create.**

4.  Associating **MyPublicNACL** to the Public Subnet

    -   Select the Action tab and click on **Edit subnet associations**

    -   Select both the **Public and Private** subnets from the table.

    -   Click on **Save changes**

5.  Renaming the Main NACL

    -   Select the **Default NACL** of the VPC MyVPC\

    -   Enter the name ***MyPrivateNACL*** and click on **Save**

Task 11: Testing the Public and Private Server 
-----------------------------------------------

1.  Public EC2 Instance:

    -   Navigate to the **EC2 Instance Dashboard.** Click on   **Instances** from the left side menu.

    -   Select the **MyPublicEC2Server**EC2 instance from the instance list.

    -   From the Description tab, copy the **IPv4 Public IP**.

    -   Now paste this IP into your web browser and click [Enter]

    -   You will see the following page:\

       **Note: This is because the Custom NACL which is attached to your Public subnet restricts both INBOUND and OUTBOUND traffic.**

2.  Private EC2 Instance:

    -   Since the Public NACL restricts all traffic, you won't be able to SSH into the public EC2 Instance to ping the Private Instance.

    -   Next, we are going to solve this.

Task 12: Adding Rules to Custom NACL (MyPublicNACL)
---------------------------------------------------

1.  Navigate to **VPC** under the **Services** menu. Click on **Network ACLs**    under **Security.**

2.  Select **MyPublicNACL** from the list.

3.  In the Inbound rules, click **Edit inbound rules**

4.  Add the following rules:

    -   **HTTP** click on **Add rules,**

        -   Rule# : Enter ***100***

        -   Type: Choose **HTTP (80)**

        -   Source: Enter ***0.0.0.0/0***

        -   Allow / Deny: Select Allow

    -   For **ALL ICMP- IPv4**, click on **Add rules**,

        -   **Rule#** : Enter ***150***

        -   Type: Choose**ALL ICMP - IPv4 **

        -   Source: Enter ***0.0.0.0/0***

        -   Allow / Deny: Select Allow

    -   For **SSH**, click on **Add rules**,

        -   **Rule# : Enter** ***200***

        -   Type: Choose **SSH (22) **

        -   Source: Enter ***0.0.0.0/0***

        -   Allow / Deny: Select Allow\

        -   Click on **Save changes**

5.  In the **Outbound rules** Tab, Click Edit outbound rules

6.  Add the following rules:

    -   **Custom Port** is already available,

        -   Rule# : Enter ***100***

        -   Type: Choose **Custom TCP Rule**

        -   Port Range: Enter ***1024 - 65535***

        -   Source: Enter ***0.0.0.0/0***

        -   Allow / Deny: Select *Allow*

    -   For **ALL ICMP- IPv4**, click on **Add rules**,

        -   Rule\# : Enter ***150***

        -   Type: Choose **ALL ICMP - IPv4 **

        -   Source: Enter ***0.0.0.0/0***

        -   Allow / Deny: Select Allow

    -   For **SSH**, click on **Add rules** ,

        -   Rule# : Enter ***200***

        -   Type: Choose **SSH (22) **

        -   Source: Enter******0.0.0.0/0**

        -   Allow / Deny: Select Allow
           
        -   Click on **Save**

Task 14: Testing Both EC2 instances
-----------------------------------

1.  We will try to ping the Private EC2 from the Public EC2 instance.

    -   SSH into EC2 Instance

    -   Copy the Private IP of **MyPrivateEC2Server** from the Description tab.![]

    -   Ping to the Private Instance using the Private IPv4


-   Press [Ctrl] + C again to cancel the process instead of pausing it.

-   Note: You were able to do these tasks because we added NACL Rules.


End Lab 
