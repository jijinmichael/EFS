### Elastic File System

It is one of the three primary storage services by AWS. It is a scalable cloud file system for Linux-based applications and workloads that can be combined with AWS cloud services and on-premises resources. Standard access storage is designed for frequently used files, while occasional access is designed to store long-term but less frequently used files at a lower cost.

EFS uses the NFSv4 protocol for its file system structure which mirrors the standard local structure and simplifies the transfer and access of your files. It can be used with Elastic Cloud Compute (EC2) instances or as a standalone file system. EFS requires no storage provisioning and is pay-per-use, allowing you to scale services as needed.

It is designed to provide scalable and elastic storage for applications and workloads that require shared file storage accessible from multiple instances simultaneously.

                   ┌──────────────────┐
                   │   Application    │
                   └──────────┬───────┘
                              │
                   ┌──────────▼───────┐
                   │    EFS File      │
                   │      System      │
                   └──────────┬───────┘
                              │
                ┌─────┬──────▼───────┬─────┐
                │ EC2 │   EC2│   EC2 │ EC2 │
                │  A  │    B │   C   │  D  │
                └─────┴──────┴───────┴─────┘


EFS offers a simple and scalable file storage solution that can be accessed by multiple instances within the same AWS region. It provides a file system interface, which means you can mount EFS to your instances using standard file system commands. This makes it easy to migrate existing applications that rely on traditional file systems to the cloud without modifying your application code.

              ┌───────────────┐               ┌───────────────┐
              │ Availability  │               │ Availability  │
              │    Zone A     │               │    Zone B     │
              └──────┬────────┘               └──────┬────────┘
                     │                               │
              ┌──────▼────────┐               ┌──────▼────────┐
              │   EFS File    │               │   EFS File    │
              │    System     │               │    System     │
              └──────┬────────┘               └──────┬────────┘
                     │                               │
              ┌──────▼────────┐               ┌──────▼────────┐
              │  EC2 Instance │               │  EC2 Instance │
              │      A        │               │      B        │
              └───────────────┘               └───────────────┘

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
[ec2-user@ip-172-31-15-100 ~]$ sudo git clone https://github.com/Fujikomalan/aws-elb-site.git  /var/website/ 
[ec2-user@ip-172-31-15-100 ~]$ sudo cp -r /var/website/*  /var/www/html/
[ec2-user@ip-172-31-15-100 ~]$ sudo chown -R apache:apache /var/www/html/*
```






