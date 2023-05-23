# Setting up a Virtual Private Cloud (VPC)

When setting up the VPC, it is first important to understand how each component of the setup work with on another. The diagram below is a visualisation of the connection between these components. 

The steps to setup the VPC for this tutorial are outlined below:

- Step 1: Create a VPC.
- Step 2: Configure Internet gateway (IGW).
- Step 3: Connect IGW to VPC.
- Step 4: Create Public subnet.
- Step 5: Setup route table.
- Step 6: Connect route table to subnet.
- Step 7: Connect internet gateway to the route table.

![](vpc/vpc.PNG)

1. The initial step is to create the VPC through the AWS VPC service and selecting "Create VPC".

![](vpc/1-create.PNG)

2. Within the VPC settings, only a VPC is being created with the default CIDR block `10.0.0.0/16` then `create VPC`.

![](vpc/2-vpc.PNG)

3. Navigate to the `Internet gateways` tab and `Create Internet gateway`. Provde a suitable tag and `Create Internet gateway`.

![](vpc/3.PNG)

![](vpc/4.PNG)

4. A notification should now appear that informs the user to `Attach to a VPC`, select this recommendation and enter the newly created VPC thenm `Attach internet gateway`.

![](vpc/5.PNG)

![](vpc/6.PNG)

5. Next, `Create subnet` through the `Subnets` tab within the same AWS VPC service.

![](vpc/7.PNG)

6. Configure the subnet settings to suite; in this example, the availability zone is assigned to `eu-west-1a` with an IPv4 CIDR block `10.0.2.0/24`. A `Name` key should be automatically created then `Create subnet`.

![](vpc/8.PNG)

7. `Create route table` within the `Route tables` tab and allocated a suitable name as well as the recently created VPC.

![](vpc/9.PNG)

![](vpc/10.PNG)

8. The route table should now be displayed and after navigating to the `Subnet associations` tab, the new subnet will appear in `Subnets without explicit associations`.

![](vpc/11.PNG)

9. Transfer the VPC to `Explicit subnet associations` through `Edit subnet associations` and highlighting the specific subnet. The subnet should now be displayed within the correct association.

![](vpc/12.PNG)

![](vpc/13.PNG)

10. Staying in the route tables tab, switch to the `Routes` tab and add the connection to the internet gateway by `Edit routes`.

![](vpc/14.PNG)

11. Add a new route with `0.0.0.0/0` as the destination and target as the internet gateway.

![](vpc/15.PNG)

12. The VPC has now been setup; this VPC can now be selected when creating an instance within the `network settings`. `Enable` auto-assign public IP and `Create security group`; follow this step if the VPC rules clash with an existing security group. Finally, configure the required protocols and the instance be initialised.

![](vpc/16.PNG)

![](vpc/17.PNG)

 >In this example, nginx was installed and configured into the new VPC.

![](vpc/18.PNG)