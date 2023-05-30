### Elastic File System

It is one of the three primary storage services by AWS. It is a scalable cloud file system for Linux-based applications and workloads that can be combined with AWS cloud services and on-premises resources. Standard access storage is designed for frequently used files, while occasional access is designed to store long-term but less frequently used files at a lower cost.

EFS uses the NFSv4 protocol for its file system structure which mirrors the standard local structure and simplifies the transfer and access of your files. It can be used with Elastic Cloud Compute (EC2) instances or as a standalone file system. EFS requires no storage provisioning and is pay-per-use, allowing you to scale services as needed.

It is designed to provide scalable and elastic storage for applications and workloads that require shared file storage accessible from multiple instances simultaneously.

<p align="center">
  <img src="https://github.com/jijinmichael/EFS/assets/134680540/e2302d19-e20f-49f8-aaee-af012523e2c1"/>
  </p>

EFS offers a simple and scalable file storage solution that can be accessed by multiple instances within the same AWS region. It provides a file system interface, which means you can mount EFS to your instances using standard file system commands. This makes it easy to migrate existing applications that rely on traditional file systems to the cloud without modifying your application code.

<p align="center">
  <img src="https://github.com/jijinmichael/EFS/assets/134680540/b25e76f7-c96a-4b05-9348-74b088b43510"/>
</p>

Let's see how can we creat a simple EFS in AWS.

Go to AWS console >> EFS >> Create File System.

Enter the name and select the VPC. Click on the Customized option and make the changes as below.

![image](https://github.com/jijinmichael/EFS/assets/134680540/05ab38ba-b349-4c3f-949e-da23081ce719)

![image](https://github.com/jijinmichael/EFS/assets/134680540/9e22b0da-67cf-4d59-b76a-f14d1cec61ee)

Please note that on the Network option, select the VPC and make sure the Availability zone's security group must opened the port **2049**.

Then create an Instance and ssh to the same and install a package called efs to mout the EFS. This will be the master template for the instance. 

In this example I'm creating an Instance with AMI Amazon Linux.

Once it is created ssh to the machine.
```
#ssh -i keypair ec2-user@Public IP or Public DNS
[ec2-user@ip-172-31-15-100 ~]$ sudo yum install amazon-efs-utils -y
```
For testing purpose we need to put some file to the doc root of the apache server. So I'm installing apache and php to the master instance.
```
[ec2-user@ip-172-31-15-100 ~]$ sudo yum install httpd php -y
[ec2-user@ip-172-31-15-100 ~]$ sudo systemctl restart httpd php-fpm 
[ec2-user@ip-172-31-15-100 ~]$ sudo systemctl enable httpd php-fpm
```
After this we need to mount the file system to the instance. 
```
[ec2-user@ip-172-31-15-100 ~]$ sudo vim /etc/fstab
```
Then add an entry like below in fstab. Please change the fs-ID according to yours.

![image](https://github.com/jijinmichael/EFS/assets/134680540/d30b83d9-b885-434a-bc6a-45a23e2d628a)

Then follow the below steps to mount and verify.
```
[ec2-user@ip-172-31-15-100 ~]$ sudo mount -a
[ec2-user@ip-172-31-15-100 ~]$ df -Th
Filesystem                                          Type      Size  Used Avail Use% Mounted on
devtmpfs                                            devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs                                               tmpfs     475M     0  475M   0% /dev/shm
tmpfs                                               tmpfs     190M  2.8M  188M   2% /run
/dev/xvda1                                          xfs       8.0G  1.6G  6.4G  20% /
tmpfs                                               tmpfs     475M     0  475M   0% /tmp
tmpfs                                               tmpfs      95M     0   95M   0% /run/user/1000
/dev/xvda128                                        vfat       10M  1.3M  8.7M  13% /boot/efi
fs-0e668d3aef007a870.efs.ap-south-1.amazonaws.com:/ nfs4      8.0E     0  8.0E   0% /var/www/html
```
Now upload or clone some web content to the doc root and change its ownership.
```
[ec2-user@ip-172-31-15-100 ~]$ sudo git clone https://github.com/jijinmichael/AWS-ELB-Site.git  /var/website/ 
[ec2-user@ip-172-31-15-100 ~]$ sudo cp -r /var/website/*  /var/www/html/
[ec2-user@ip-172-31-15-100 ~]$ sudo chown -R apache:apache /var/www/html/*
```

### Mount the file system on the EC2 instance which is created by Auto Scaling Group

Here we are going to see how we can mount the above EFS to an ASG created Instances.

For this we need to create a Launch Configuration.

Since I have taken Amazon Linux in the previous section, under the LC AMI choose the same AMI ID.

On the Advance details, write down the user data as follows. Please chnage the fs-ID according to yours.
```
#!/bin/bash

yum install amazon-efs-utils httpd php  -y
echo "fs-0e668d3aef007a870:/    /var/www/html/  efs  defaults,_netdev  0  0"  >> /etc/fstab
mount -a
systemctl restart httpd php-fpm
systemctl enable httpd php-fpm
```
Then create an Auto Scaling Group with the LC which we created above. Now the newly deployed instance will have the same web doc root and its contents of the master instance.

To check this login to the newly created instance and type the below command.
```
[ec2-user@ip-172-31-39-135 ~]$ df -Th
Filesystem                                          Type      Size  Used Avail Use% Mounted on
devtmpfs                                            devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs                                               tmpfs     475M     0  475M   0% /dev/shm
tmpfs                                               tmpfs     190M  2.9M  188M   2% /run
/dev/xvda1                                          xfs       8.0G  1.6G  6.4G  20% /
tmpfs                                               tmpfs     475M     0  475M   0% /tmp
/dev/xvda128                                        vfat       10M  1.3M  8.7M  13% /boot/efi
fs-0e668d3aef007a870.efs.ap-south-1.amazonaws.com:/ nfs4      8.0E  5.0M  8.0E   1% /var/www/html
tmpfs                                               tmpfs      95M     0   95M   0% /run/user/1000
```






