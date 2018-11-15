## Overview

This repo contains coursework from the [docker-and-kubernetes-course](https://github.com/sund0g/docker-tutorials/tree/master/docker-and-kubernetes) starting with section 10 and onwards. The reason for this separate repo is because we are using Travis CI for CI/CD, which requires a single **.travis.yml** file at the root of a repo. As I have no desire to hack Travis such that I can keep this in the **complex** project subbdirectory in the [docker-and-kubernetes-course](https://github.com/sund0g/docker-tutorials/tree/master/docker-and-kubernetes), I am opting for just creating this new repo.

### Section 10: A Continuous Integration Workflow for Multiple Images

In this section, we learn,

1. How to create production, multi-container builds.
2. Lots of info about nginx.
3. How to configure Travis CI to build our project and push production assets to Docker Hub.

### Section 11: Multi-Container Deployments to AWS 

#### Lesson 133

* Learn how to deploy to [AWS Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/?sc_channel=PS&sc_campaign=acquisition_US&sc_publisher=google&sc_medium=ACQ-P%7CPS-GO%7CBrand%7CDesktop%7CSU%7CMachine%20Learning%7CElastic%20Beanstalk%7CUS%7CEN%7CText&sc_content=elastic_beanstalk_e&sc_detail=aws%20elastic%20beanstalk&sc_category=Machine%20Learning&sc_segment=293647516945&sc_matchtype=e&sc_country=US&s_kwcid=AL!4422!3!293647516945!e!!g!!aws%20elastic%20beanstalk&ef_id=EAIaIQobChMItf_exfzR3gIVA-DICh3ssQN6EAAYASAAEgLbSfD_BwE:G:s) using a special file, [Dockerrun.aws.json](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_v2config.html)

* Get an overview of the similarities and differences between **Dockerrun.aws.json** and **docker-compose.yml**.

#### Lessons 134-136

* Use [AWS ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html) to tell Elastic Beanstalk what to do with the containers using [task definitions](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definitions.html).  Task definitions are essentially what the dockerrun file consists of.

* Learn which [Task definition parameters](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html) and [Container definitions](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html#container_definitions) to use when creating **Dockerrun.aws.json**.

	> The format described in the Container definitions docuementation is what is used to create this projects' Dockerrun file.

#### Lesson 137
	
* Learn how to use links for routing with nginx. *see details in the explanation of the Dockerrun file below*.

#### Lesson 138

> **WARNING!!! You will be charged to use any AWS service if your account is 1+ years old. To avoid excess charges, you MUST remember to delete all instances created from the lessons in this course.**

* Create a production environment using [AWS Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/?sc_channel=PS&sc_campaign=acquisition_US&sc_publisher=google&sc_medium=ACQ-P%7CPS-GO%7CBrand%7CDesktop%7CSU%7CMachine%20Learning%7CElastic%20Beanstalk%7CUS%7CEN%7CText&sc_content=elastic_beanstalk_e&sc_detail=aws%20elastic%20beanstalk&sc_category=Machine%20Learning&sc_segment=293647516945&sc_matchtype=e&sc_country=US&s_kwcid=AL!4422!3!293647516945!e!!g!!aws%20elastic%20beanstalk&ef_id=EAIaIQobChMItf_exfzR3gIVA-DICh3ssQN6EAAYASAAEgLbSfD_BwE:G:s) executing the following steps,
		
	1. Log into the AWS console
	2. Navigate to the Elastic Beanstalk service
	3. Select **Create New Application**
	4.  Enter **Multi-Docker** as the **Application Name** and then select the **Create** button to create the application 
	5. Create a **Web server environment** via the links
	6. In the subsequent environment settings page, select **Multi-container Docker** platform in the **Base configuration** section
	7. Select **Create** at the bottom of the page, (we don't care about the other environment settings as part of this lesson)

#### Lesson 139

* Learn why leveraging [AWS ElastiCache](https://aws.amazon.com/elasticache/), specifically for [Redis](https://aws.amazon.com/elasticache/redis/), and [AWS RDS](https://aws.amazon.com/rds/) for [Postgres](https://aws.amazon.com/rds/postgresql/) is preferrable to creating and running databases inside of containers we create. Two major advantages are production-quality **maintenance** and **security**.  

	> **All that being said... we will NOT be using any external services for the purpose of this course. Why? Because we are here to learn about containers. Plus not all service platforms provide these types of instances, e.g. Digital Ocean.**

#### Lesson 140

* Create an AWS **Security Group** so that our services/containers can communicate with each other in the **VPC**. *Instructions for this are in Lesson 143*.

#### Lesson 141

* Create a RDS (Postgres) instance as follows,

	1. Navigate to the [AWS RDS homepage](https://us-west-1.console.aws.amazon.com/rds/home?region=us-west-1)
	2. Select **Create database**
	3. Select **Postgres** as the database engine

		> Course recommendation: check the box to **Only enable options eligible for RDS Free Usage Tier** 
	4. Select **Next**
	5. In the **Settings** section enter,
		* **multi-docker-postgres** as the **DB instance identifier**
		* **postgres** as the **Master username**
		* **postgrespassword** as the **Master password**

			> **Master username** and **Master password** map to the **PGUSER** and **PGPASSWORD** environment variables set in docker-compose.yml for the **api** service.
		
		* Select **Next**
	6. In **Configure Advanced Settings** enter **database name** as **fibvalues**

		> **database name** maps to **PGDATABASE** in docker-compose.yml.
		
	7. The default values for the other settings are acceptable for this course, so select **Create database** to create the db instance.

#### Lesson 142

* Create an ElastiCache (Redis) instance as follows,
	1. Navigate to the [AWS ElastiCache homepage](https://us-west-1.console.aws.amazon.com/elasticache/home?region=us-west-1#)
	2. Select **Redis** from the dashboard.
	3. Select **Create**
	4. In **Create your Amazon ElastiCache cluster | Redis** enter,
		* **multi-docker-redis** as the **Name**
		* **cache.t2.micro** as the **Node type**
		
			> This is **NOT** the default which is much more expensive, so make sure this is set to micro for this course.
		* **None** as the **Number of replicas**, (again, not needed for this course).
	5. In **Advanced Redis settings**,
		* enter **redis-group** as the **Name**
		* select **\<Default VPC\>** as the **VPC ID**
		
			> **\<Default VPC\>** is where the rest of the services/containers are.
		* select all the available **Subnets**
	6. The default values for the other settings are acceptable for this course, so select **Create** to create the Redis instance.

#### Lesson 143

* Create an AWS **custom security group** to connect the services, as follows,

	1. Navigate to the **VPC** dashboard.
	2. Select **Security Groups** card
	3. Click the **Create Security Group** button
	4. Enter **multi-docker** as the **Name tag**
	5. Enter **Traffic for services in multi-docker app** as the **Description**
	6. Select **\<Default VPC\>** as the **VPC**
	7. Click the **Yes, Create** button
	8. Select the newly created **multi-docker** security group
	9. Select the **Inbound Rules** tab
	10. Click **Edit**
	11. Set the **Port Range** as **5432-6379**
	
		> The port ranges are the same as the **PGPORT (5432)** and **REDIS_PORT (6379)** environment variables in **docker-compose**.
	12. Select the newly created **multi-docker** security group from the **Source** listbox. 
	
		> This allows traffic from any other service in the same security group.
	13. Click the **Save** button

#### Lesson 144

* Assign the EB, RDS, and Redis services to the **multi-docker** security group, as follows,
	* **ElastiCache/Redis**
		1. Navigate to the dashboard
		2. Select the box for **multi-docker-redis**
		3. Click the **Modify** button
		4. Edit the **VPC Security Groups**
		5. Add the **multi-docker** security group
		6. Click **Save**
		
			> We have a maintenance window option available for ElastiCache so we could minimize downtime for users should we need to.
		7. Click **Modify**
	* **RDS/Postgres**
		1. Navigate to the dashboard
		2. Select **Instances**
		3. Select the **multi-docker-postgres** instance
		4. Scroll down to the **Details** section and click the **Modify** button.
		5. Scroll down to the **Network & Security** section
		6. Select the **Security Group** listbox
		7. Add the **multi-docker** security group
		8. Scroll to the bottom and click the **Continue** button
			
			> For the purpose of this course you may want to **unselect** the **Enable deletion protection option before continuing. This makes it easier when you're cleaning up the instances at the end when you're done with them.
		9. Select **Apply immediately** (See maintenance comment in the RDS section above for details)
		10. Click the **Modify DB Instance** button
	
	* **Elastic Beanstalk**
		1. Navigate to the dashboard
		2. Select the **MultiDocker-env** application
		3. Select **Configuration**
		4. Click **Modify** at the bottom of the **Instances** card
		5. Scroll down to **EC2 Security Groups**
		6. Check the **multi-docker** security group
		7. Click **Apply**
		
			> The warning notifies that all the EC2 instances that are part of the application will be restarted so the update will take effect.
		8. Click the **Confirm** button

#### Lesson 145

* Set environment variables such that the containers in the Elastic Beanstalk instance know how to communicate with the RDS and ElastiCache services, (the same as was done in **docker-compose**)
	1. Navigate to the **Elastic Beanstalk** dashboard (probably still there from step 12)
	2. Select **Configuration**
	3. Click **Modify** at the bottom of the **Software** card
	4. Scroll to the **Environment Properties** section
	5. Add the environment variables from **docker-compose.yml**,

		Name       | Value
		---------- | ----------
		REDIS_HOST | \<ElastiCache Primary Endpoint\>
		REDIS_PORT | 6379
		PGUSER     | postgres
		PGPASSWORD | postgrespassword
		PGHOST     | \<RDS Endpoint\>
		PGDATABASE | fibvalues
		PGPORT     | 5432
		
		> Get the **\<ElastiCache Primary Endpoint\>** from the Redis instance of the ElastiCache dashboard. Do Not include the port.
		
		> Get the **\<RDS Endpoint\>** from the **multi-docker-postgres** instance on the RDS dashboard, in the **Connect** section.
		
	6. Click the **Apply** button

#### Lesson 146

* Create **IAM Keys for deployment,
	1. Navigate to the IAM dashboard
	2. Select **Users**
	3. Click the **Add Users** button
	4. Enter **multi-docker-deployer** as the user name
	5. Select **Programmatic access** as **Access type**
	6. Click the **Next:Permissions** button
	7. Click the **Attach existing policies directly** card
	8. Search for **beanstalk** and add all returned services.
	9. Click the **Next:Review** button
	10. Review and click the **Create User** button

* Add the keys to Travis CI

	1. Navigate to the Travis CI dashboard
	2. Navigate to the **multi-docker** project
	3. Select **Options | Settings**
	4. Scroll to the **Environment Variables** section
	5. Add the following environment vairables,
	
		Name           | Value
		-------------- | ----------
		AWS\_ACCESS_KEY | \<Access key ID\>
		AWS\_SECRET_KEY | \<Secret access key\>
		
		> Get the \<Access key ID\> and \<Secret access key\> from the IAM console

#### Lesson 147

* Add the following variables, (as a single object) to a **deploy** section in **.travis.yml**,

	Name               | Value
	------------------ | ----------
	provider           | elasticbeanstalk
	region             | us-west-1
	app                | multi-docker
	env                | MultiDocker-env
	bucket_name        | \<bucket name\>
	bucket_path        | docker-multi
	access\_key_id     | $AWS\_ACCESS_KEY
	secret\_access_key | secure: $AWS\_SECRET_KEY
		
	> **\<bucket name\>** comes from the **S3** dashboard
	
* Check everything in and create a Pull Request to merge the branch back to master. This will initiate a Travis buid, which, if there are no syntax errors will result in a deployment to AWS.
	
#### Lesson 148

* Fix an error in **Dockerrun.aws.json**

	* After succesfully deploying to AWS in Lesson 147, navigate to the Elastic Beanstalk dashboard. There should be an error something like,

	*Service:AmazonECS, Code:ClientException, Message:Invalid setting for container 'client'. At least one of 'memory' or 'memoryReservation' must be specified., Class:com.amazonaws.services.ecs.model.ClientException*
	
	> This error is telling us that the **memory** option must be specified in the **Dockerrun.aws.json** file. This option tells the service provider how much memory to allocate to the service.
	
	To fix the error,

	1. add the following to all of the **containerDefinitions** in **Dockerrun.aws.json**,

		**`"memory": 128`**
		
		> 128 (Mb) is what we're using for all our services in this course. It is not a general guideline. you have to research how much memory is required for your specific service(s).
	
	2. Check **Dockerrun.aws.json** into GitHub to kick off a Travis build/re-deploy to AWS.


---
		
### Explanation of Dockerrun.aws.json

Because json doesn't support comments, its contents are described here. For full details on the parameters, please reference the [AWS documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html#container_definitions),

* **AWSEBDockerrunVersion** specified as 2 because we're in a multi-container environment.
* **containerDefinitions** an array of the container definitions.
* **name** Name of the container
* **image** ECS knows to pull the image from Docker.
* **hostname** While docker-compose uses the service "name" as a catch-all reference, including hostname, Dockerrun does not, which is why we have to specifically define it.

	> **hostname** is optional, it is only required if the service has to be directly accessed, e.g. **client** and **server** are directly accessed while **worker** and **nginx** are not. It can be added for completeness if the service is not directly accessed.
	
* **essential** when **false** means that the container is not critical and other containers will not stop if ot does. If **true** all associated containers will stop if the container stops.

	> At least one service in the containerDefinitions section must have essential set to true.

* **portMappings** (used for the nginx service.) Contains an array of ports that will be used. In the multi-container project a single mapping is defined as,
	* **hostPort** port on the machine hosting the containers
	* **containerPort** port inside of the container.

		> 80 is the default port nginx uses inside containers.
	
	> **portMappings** does the same thing that **ports** does in **docker-compose**.
	
* **links** (used for the nginx service.) Creates uni-directional links between services/containers

	> The link name maps to the **name** property in the containerDefinitions. Calling out here because the **server** service is renamed as **api** for the hostname.

 #### Best practice: When working with json files a great way to save time debugging is to verify json code with a json validator. [JSONLint](https://jsonlint.com/) is what I use to validate json code.