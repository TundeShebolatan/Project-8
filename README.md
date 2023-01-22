# Project 8: LOAD BALANCER SOLUTION WITH APACHE

In this project a Load Balancer solution that consists of following components was implement:

Infrastructure: AWS
Webservers Linux: Red Hat Enterprise Linux 8
Database Server: Ubuntu 20.04 + MySQL
Storage Server: Red Hat Enterprise Linux 8 + NFS Server
Load Balancer: Ubuntu Server 20.04
Programming Language: PHP
Code Repository: GitHub

Step 0:  Preparing prerequisites

- Sign in to AWS free tier account and create four new EC2 Instances of t2.nano family with RedHat OS, and one new EC2 Instance of Ubuntu 20.04
- Name the new instances thus;
- Server A name - "NFS server" (Red Hat Enterprise Linux 8 + NFS Server)
- Server B name - "webserver 1" (Red Hat Enterprise Linux 8)
- Server C name - "webserver 2" (Red Hat Enterprise Linux 8)
- Server D name - "Apache Load Balancer" (Ubuntu Server 20.04)
- Server C name - "database server" (Ubuntu 20.04 + MySQL)
