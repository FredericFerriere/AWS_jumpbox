# AWS_jumpbox
Steps to setup a jumpbox in AWS
We will set up the elements in AWS so that an instance located in a private subnet can still access the internet.
The idea being that http/https traffic initiated by the instance is routed via the subnet's route table
to an instance located in the public subnet.
As a POC, we will ping a website from the instance located in the private subnet.


AWS Architecture:  
NAMES         Types  
JB_VPC        VPC  
JB_IGW        Internet Gateway  
JB_SN_PUBLIC  Subnet  
JB_SN_PRIVATE Subnet  
JB            EC2 Instance (Type Amazon Linux 2 AMI HVM)  
FI            EC2 Instance (Type Amazon Linux 2 AMI HVM)  
NAT           EC2 Instance (Type amzn-ami-upc-nat which you can find in the search textbox in Community AMI section)  
IGW_RT        Route Table  
NAT_RT        Route Table  
JB_SG         Security Group  
FI_SG         Security Group  
NAT_SG        Security Group  


IMPORTANT NOTES:  
* Both subnets need to sit in the same Availability Zone
* the NAT instance has to be of type amzn-ami-upc-nat
* JB & NAT reside in the public SN
* FI resides in the private SN
* IGW_RT is associated to the public SN
* NAT_RT is associated to the private SN, it routes unknown traffic to the NAT instance

Steps
1) Create VPC JB_VPC
2) Create subnets JB_SN_PUBLIC and JB_SN_PRIVATE
3) Create Internet Gateway IGW
4) Create 3 EC2 Instances: JB, FI & NAT
      * use a public IP for JB & NAT
      * for the NAT instance, make sure Source/Dest. check is disabled (Actions/Networking/Change Source/Dest check)
5) Edit Route Tables
6) Edit Security Groups

Security Groups setup  
| JB_SG | In  SSH |
| JB_SG | Out All |
| FI_SG | In  SSH (from JB) |
| FI_SG | Out HTTP/HTTPS/ICMPv4 to 0.0.0.0/0 |
| NAT_SG | In  SSH + HTTP/HTTPS/ICMPv4 from FI |
| NAT_SG| Out Out HTTP/HTTPS/ICMPv4 to 0.0.0.0/0 |

<table>
     <tr>
          <td> JB_SG <\td>
          <td> In <\td>
          <td> SSH <\td>
     <\tr>
     <tr>
          <td> JB_SG <\td>
          <td> Out <\td>
          <td> All <\td>
     <\tr>          
<\table>

----------------------------------------------------  
Testing on Ubuntu  
Assuming jbkp.pem as the keypair file name  
$ chmod 600 jbkp.pem        //to avoid file permissioning issues  
$ eval $(ssh-agent -s)      //running ssh-agent to add keypair and use JB as an agent to ssh into FI without the need for keypair  
$ ssh-add jbkp.pem          
$ ssh -A ec2-user@<JB Public IP>      //ssh into JB acting as agent to log next into FI (private subnet)  
$ ssh ec2-user@<FI Private IP>
$ ping www.aws.com                    //now into FI EC2 Instance, send ping request to check internet access  

