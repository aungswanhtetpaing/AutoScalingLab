# Project Explained:
 Resources in my project include: 
1. Network Resources
2. LoadBalancer Resources
3. Autoscaling Resources
4. Autoscaling Policy



### 1. Network Resources
Two public subnets and two private subnets are created (for LoadBalancer and EC2 instances that will be deployed by AutoScaling Group). IGW and NAT Gateway are created. IGW is for PublicSubnets and is attached to the PublicRouteTables via PublicRoutes. NATGateway is for PrivateSubnets and is attached to the PrivateRouteTables via PrivateRoutes. Then, Subnets and RouteTables are associated.

### 2. LoadBalancer Resources
ALBSecurityGroup allows HTTP(80) and HTTPS(443) from the Internet. Two PublicSubnets (with IGW) are added to ALB. ALB Scheme is set to internet-facing. If my ALB only needs to receive incoming traffic from the internet, I will use IGW. If my ALB needs to initiate outbound connections, I will use both IGW and NAT Gateway.
As I set my ALB to internet-facing, it will have a public IP address or public DNS name that will allow clients to connect directly to it.
      In TargetGroup, HealthCheck is enabled for EC2 instances in AutoScalingGroup. Every 60 seconds, ALB will send HTTP requests to each target's root path (e.g., http://<target-ip>/) to check their health. If the target responds with a successful HTTP status code (e.g., 200 OK), the ALB considers the target healthy. It is unhealthy if the target responds with an error code (e.g., 4xx or 5xx status codes). ALB will wait 10 seconds to get response from targets. If target successfully respond 3 times (60 x 3 = 180s), ALB will mark the target as healthy. If respond is unsuccessful 5 times (60 x 5 = 300s), the target is marked as unhealthy. "deregistration_delay.timeout_seconds" attribute is set to 0, that is targets are deregisterd immediately upon detection of unhealthy status.
      ALBListener will "listen" incoming HTTP requests on port 80 and "forward" them to the TargetGroup. The TargetGroup is responsible for distributing the requests to the appropriate targets (EC2 instances) based on the configured load balancing algorithm and health checks.

### 3.Autoscaling Resources
      MyAutoScalingGroup is associated with ALBTargetGroup. Two PrivateSubnets are added to MyAutoScalingGroup. Therefore, EC2 instances will be in PrivateSubnet and they will get Load Balancing, Health Checks and Autoscaling affects. In MyAutoScalingGroup, minimum number of EC2 instances that will be running is 2 and maximum number is 4. When the group is launched, it will start with 2 instances as I set the DesiredCapacity to 2. Emails from SNSTopic will be reveived when EC2 instances are started, stopped, or facing errors when starting and stopping.
      In EC2InstanceSecurityGroup, required protocols are allowed. SSH is also allowed for Testing. As my instances are in PrivateSubnet, public ip will not be available. Then, I will create Bastion Host, and first SSH to the host, and remotely manage the EC2s for Stress testing.
      In LaunchConfiguration, Instance type (t2.micro) and latest AmiId is used. Key pair is also created. UserData is the script I downloaded for custom memory metrics. It will be sent to CloudWatch Alarm.

### 4.AutoScaling Policy 
      Simple Scaling Policy is used to Scale down and Scale up EC2 instances based on CPUUtilization and Memory Utilization. 


# Detail Steps:

## I. Installation Stage:

1. It is the Design of my Lab shown by Designer.



2. I will use CloudFormation to upload file and create stack.


3. SNS email is sent when AutoscalingGroup launch Instances.


4. Stack Creation is completed.


5. Bastion Host is manually created to ssh to EC2 instances and test load.


6. SSH into the Host with public ip address and keypair.


7. CloudWatch Alarm section has 4 alarms. As there is no load on EC2, CPUUtilizationLow is in alarm. But the instances are at minimum capacity and there will be no changes occurs.


II. Scale Out Testing Stage:

1.There is Low CPUUtilization shown in CloudWatch.


2. Then, in the EC2 console, 'htop' command is used to monitor the cpu and memory utilizations.

2.2-

 
3. After installing stress script, the cpu utilization is 'High' as shown below.


4. Let's go back to CloudWatch and check the alarm. After a few minutes, 'the alarm' stage is triggered.


5. And then, SNS email is received that Instance is Launched.


6. Check the EC2 instances to verify that EC2 instance is added.


III. Scale In Testing Stage


1. In the ssh console, the stress task is killed with process id.


2. Check the CloudWatch alarm and wait for 'in alarm'stage.


3. And then, SNS email is received that EC2 instance is 'Terminated'.


4. Go to the EC2 instances and make sure that EC2 instance is terminated.


IV. Deleting Resources

Finally, CloudFormation Template is Deleted. 
