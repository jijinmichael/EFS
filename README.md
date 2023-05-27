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




