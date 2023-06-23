## Project Explained:
---
 Resources in my project include: 
1. Network Resources
2. LoadBalancer Resources
3. Autoscaling Resources
4. Autoscaling Policy



### 1. Network Resources
> **Two public subnets** and **two private subnets** are created (for LoadBalancer and EC2 instances that will be deployed by AutoScaling Group).
>
> **IGW** and **NAT Gateway** are created. IGW is for **PublicSubnets** and is attached to the **PublicRouteTables** via **PublicRoutes**. NATGateway is for **PrivateSubnets** and is attached to the **PrivateRouteTables** via **PrivateRoutes**.
>
> Then, Subnets and RouteTables are associated.

### 2. LoadBalancer Resources
<div style="text-align: justify">
> *ALBSecurityGroup* allows HTTP(80) and HTTPS(443) from the Internet. Two PublicSubnets (with IGW) are added to ALB. ALB Scheme is set to internet-facing. If my ALB only needs to receive incoming traffic from the internet, I will use IGW. If my ALB needs to initiate outbound connections, I will use both IGW and NAT Gateway.
As I set my ALB to internet-facing, it will have a public IP address or public DNS name that will allow clients to connect directly to it.
> 
> In TargetGroup, HealthCheck is enabled for EC2 instances in AutoScalingGroup. Every 60 seconds, ALB will send HTTP requests to each target's root path (e.g., http://<target-ip>/) to check their health. If the target responds with a successful HTTP status code (e.g., 200 OK), the ALB considers the target healthy. It is unhealthy if the target responds with an error code (e.g., 4xx or 5xx status codes). ALB will wait 10 seconds to get a response from targets. If the target successfully responds 3 times (60 x 3 = 180s), ALB will mark the target as healthy. If the response is unsuccessful 5 times (60 x 5 = 300s), the target is marked as unhealthy. "deregistration_delay.timeout_seconds" attribute is set to 0, that is targets are deregistered immediately upon detection of unhealthy status.
      ALBListener will "listen" incoming HTTP requests on port 80 and "forward" them to the TargetGroup. The TargetGroup is responsible for distributing the requests to the appropriate targets (EC2 instances) based on the configured load balancing algorithm and health checks.
</div>
### 3.Autoscaling Resources
MyAutoScalingGroup is associated with ALBTargetGroup. Two PrivateSubnets are added to MyAutoScalingGroup. Therefore, EC2 instances will be in PrivateSubnet and they will get Load Balancing, Health Checks, and Autoscaling effects. In MyAutoScalingGroup, the minimum number of EC2 instances that will be running is 2 and the maximum number is 4. When the group is launched, it will start with 2 instances as I set the DesiredCapacity to 2. Emails from SNSTopic will be received when EC2 instances are started, stopped, or facing errors when starting and stopping.
      In EC2InstanceSecurityGroup, required protocols are allowed. SSH is also allowed for Testing. As my instances are in PrivateSubnet, public IP will not be available. Then, I will create Bastion Host, first SSH to the host, and remotely manage the EC2s for Stress testing.
      In LaunchConfiguration, Instance type (t2.micro) and the latest AmiId are used. Key pair is also created. UserData is the script I downloaded for custom memory metrics. It will be sent to CloudWatch Alarm.

### 4.AutoScaling Policy 
> Simple Scaling Policy is used to Scale down and Scale-up EC2 instances based on CPUUtilization and Memory Utilization. 

---
# Detail Steps:

## I. Installation Stage:

1. It is the Design of my Lab shown by Designer.

![1 TemplateDesign](https://github.com/aungswanhtetpaing/AutoScalingLab/assets/135700688/9a6de146-25ee-40f1-9cbd-711bd73c5596)

---
2. I will use CloudFormation to upload the configuration file and create a stack.

![2](https://github.com/aungswanhtetpaing/AutoScalingLab/assets/135700688/26565072-128e-498b-a004-2ba353364e06)

---
3. SNS email is sent when AutoscalingGroup launches Instances.

![3](https://github.com/aungswanhtetpaing/AutoScalingLab/assets/135700688/ac63733f-a377-4647-adc4-6f61e5ff5445)

---
4. Stack Creation is completed.

![4](https://github.com/aungswanhtetpaing/AutoScalingLab/assets/135700688/4d258f41-b62f-439f-94dc-b6c9d776d094)

---
5. Bastion Host is manually created and it is used to ssh into EC2 instances and test load.

![5](https://github.com/aungswanhtetpaing/AutoScalingLab/assets/135700688/7b927a13-f10c-48ce-9398-6235f72118c5)

---
6. SSH into the Host with public IP address and key pair.

![6](https://github.com/aungswanhtetpaing/AutoScalingLab/assets/135700688/76490d5a-2283-432b-a1ce-072bf4c30b5c)

---
7. CloudWatch Alarm section has 4 alarms. As there is no load on EC2, CPUUtilizationLow is in alarm. But the instances are at minimum capacity and there will be no changes occurring.

![7](https://github.com/aungswanhtetpaing/AutoScalingLab/assets/135700688/49ea79b9-486f-48ab-a92e-89d54d7b0064)

---

## II. Scale-Out Testing Stage:

1. There is Low CPU utilization shown in CloudWatch.

![8 1](https://github.com/aungswanhtetpaing/AutoScalingLab/assets/135700688/d6ac8c8d-14b4-4458-a1f1-1fec27cd6e4a)

---
2. Then, in the EC2 console, 'htop' command is used to monitor the CPU and memory utilization.

![8 2](https://github.com/aungswanhtetpaing/AutoScalingLab/assets/135700688/ac2bff2f-6482-45fe-87fd-0e206aae25a7)

![8 2 2](https://github.com/aungswanhtetpaing/AutoScalingLab/assets/135700688/33e9b720-8317-463a-91bf-e17a8469404f) 

---
3. After installing the stress script, the CPU utilization is 'High' as shown below.

![8 3](https://github.com/aungswanhtetpaing/AutoScalingLab/assets/135700688/3fff8597-3d18-44b8-a569-ac7187520d58)
a8469404f)

---
4. Let's go back to CloudWatch and check the alarm. After a few minutes, the 'in alarm' stage is triggered.

![8 4](https://github.com/aungswanhtetpaing/AutoScalingLab/assets/135700688/b8be29aa-8c0a-4b12-b1f3-2ed0f029482e)

---
5. And then, an SNS email is received that the EC2 Instance is Launched.

![8 5](https://github.com/aungswanhtetpaing/AutoScalingLab/assets/135700688/8c626745-bd52-4612-a0aa-43a5db93c8b2)

---
6. Check the EC2 instances to verify that the EC2 instance is added.

![8 6](https://github.com/aungswanhtetpaing/AutoScalingLab/assets/135700688/52d9e2b1-f55d-426d-b37b-38b2268a18bf)

---

## III. Scale In Testing Stage


1. The stress task is killed with the process id in the ssh console.

![9 1](https://github.com/aungswanhtetpaing/AutoScalingLab/assets/135700688/f292efa1-3f75-4f03-8622-cdae5ba6db5d)
b2268a18bf)

---
2. Check the CloudWatch alarm and wait for the 'in alarm' stage.

![9 2](https://github.com/aungswanhtetpaing/AutoScalingLab/assets/135700688/5eaeb3c9-01e6-4392-9d82-a46e179ea12b)

---
3. And then, an SNS email is received that the EC2 instance is 'Terminated'.

![9 3](https://github.com/aungswanhtetpaing/AutoScalingLab/assets/135700688/8e38ff37-a263-452a-99b0-4db20cdb8c30)

---
4. Go to the EC2 instances and make sure that the EC2 instance is terminated.

![9 4](https://github.com/aungswanhtetpaing/AutoScalingLab/assets/135700688/24625aa3-6253-4525-b6df-0dd18b6323c9)

---
## IV. Deleting Resources

Finally, CloudFormation Template is Deleted. 

![10](https://github.com/aungswanhtetpaing/AutoScalingLab/assets/135700688/4220e33e-c582-441b-9516-ab4da9cbd567)
