# AWS-Route-53-DNS-Routing-and-Failover
**Project Overview**
This project demonstrates an AWS cloud architecture focused on the Mumbai region, showing how Route 53 manages DNS routing and failover using Elastic Load Balancer (ELB), EC2 instances, and an optional S3 static hosting bucket for maintenance mode.
**Architecture Components&Component	Description:**
1.Route 53	DNS service routing traffic for sai.com using a Failover Routing Policy.
2.Elastic Load Balancer (ELB)	Distributes incoming traffic across multiple EC2 instances in Mumbai.
3.EC2 Instances	Host the web application and ensure high availability.
4.S3 Bucket	Serves maintenance.html during downtime or failover events.
5.Health Checks	Monitor ELB and EC2 instance health to trigger failover if needed.
6.VPC & Security Groups	Provide secure networking and controlled access.
**Working Flow:**
1.User requests sai.com.
2.Route 53 checks the health of the Mumbai ELB.
3.If healthy → traffic routes to ELB → EC2 instances.
4.If unhealthy → traffic redirects to S3 static site (maintenance.html).
**How to Deploy**
1.Create a hosted zone in Route 53 for your domain (sai.com).
2.Configure ELB and attach EC2 instances in Mumbai region.
3.Set up health checks for ELB.
4.Create an S3 bucket for static hosting and upload maintenance.html.
5.Configure Route 53 failover routing policy:
6.Primary → ELB DNS (Mumbai)
7.Secondary → S3 bucket endpoint
