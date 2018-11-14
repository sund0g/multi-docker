## A Continuous Integration Workflow for Multiple Images

This repo contains coursework from the [docker-and-kubernetes-course](https://github.com/sund0g/docker-tutorials/tree/master/docker-and-kubernetes) starting with section 10 and onwards. The reason for this separate repo is because we are using Travis CI for CI/CD, which requires a single **.travis.yml** file at the root of a repo. As I have no desire to hack Travis such that I can keep this in the **complex** project subbdirectory in the [docker-and-kubernetes-course](https://github.com/sund0g/docker-tutorials/tree/master/docker-and-kubernetes), I am opting for just creating this new repo.

In **Section 10** we learn,

1. How to create production, multi-container builds.
2. Lots of info about nginx.
3. How to configure Travis CI to build our project and push production assets to Docker Hub.

In **Section 11** we learn,

1. How to deploy to [AWS Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/?sc_channel=PS&sc_campaign=acquisition_US&sc_publisher=google&sc_medium=ACQ-P%7CPS-GO%7CBrand%7CDesktop%7CSU%7CMachine%20Learning%7CElastic%20Beanstalk%7CUS%7CEN%7CText&sc_content=elastic_beanstalk_e&sc_detail=aws%20elastic%20beanstalk&sc_category=Machine%20Learning&sc_segment=293647516945&sc_matchtype=e&sc_country=US&s_kwcid=AL!4422!3!293647516945!e!!g!!aws%20elastic%20beanstalk&ef_id=EAIaIQobChMItf_exfzR3gIVA-DICh3ssQN6EAAYASAAEgLbSfD_BwE:G:s) using a special file, [Dockerrun.aws.json](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_v2config.html)
2. The similarities and differences between **Dockerrun.aws.json** and **docker-compose.yml**, (lesson 133).
3.  How to use [AWS ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html) to tell Elastic Beanstalk what to do with the containers using [task definitions](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definitions.html).  Task definitions are essentially what the dockerrun file consists of.
4. Which [Task definition parameters](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html) and [Container definitions](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html#container_definitions) to use when creating **Dockerrun.aws.json** (lessons 134-136).

	> The format described in the Container definitions docuementation is what is used to create this projects' Dockerrun file.
	
5. How to use links for routing with nginx, (lesson 137). *see details in the explanation of the Dockerrun file below*.
6. How to create a production environment using [AWS Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/?sc_channel=PS&sc_campaign=acquisition_US&sc_publisher=google&sc_medium=ACQ-P%7CPS-GO%7CBrand%7CDesktop%7CSU%7CMachine%20Learning%7CElastic%20Beanstalk%7CUS%7CEN%7CText&sc_content=elastic_beanstalk_e&sc_detail=aws%20elastic%20beanstalk&sc_category=Machine%20Learning&sc_segment=293647516945&sc_matchtype=e&sc_country=US&s_kwcid=AL!4422!3!293647516945!e!!g!!aws%20elastic%20beanstalk&ef_id=EAIaIQobChMItf_exfzR3gIVA-DICh3ssQN6EAAYASAAEgLbSfD_BwE:G:s), (lesson 138). To do this execute the following steps,

	**WARNING!!! You will be charged to use any AWS service if your account is 1+ years old. To avoid excess charges, you MUST remember to delete all instances created from the lessons in this course.**
		
	1. Log into the AWS console
	2. Navigate to the Elastic Beanstalk service
	3. Select **Create New Application**
	4.  Enter **Multi-Docker** as the **Application Name** and then select the **Create** button to create the application 
	5. Create a **Web server environment** via the links
	6. In the subsequent environment settings page, select **Multi-container Docker** platform in the **Base configuration** section
	7. Select **Create** atthe bottom of the page, (we don't care about the other environment settings as part of this lesson)

7. Why leveraging [AWS Elasticache](https://aws.amazon.com/elasticache/), specifically for [Redis](https://aws.amazon.com/elasticache/redis/), and [AWS RDS](https://aws.amazon.com/rds/) for [Postgres](https://aws.amazon.com/rds/postgresql/) is preferrable to creating and running databases inside of containers we create. Two major advantages are production-quality **maintenance** and **security**, (lesson 139).  

	> **All that being said... we will NOT be using any external services for the purpose of this course. Why? Because we are here to learn about containers. Plus not all service platforms provide these types of instances, e.g. Digital Ocean.**
	
8. How to create an AWS **Security Group** so that our services/containers can communicate with each other in the **VPC** (lesson 140).

9. Create a RDS (Postgres) instance (lesson 141) as follows,
	1. Navigate to the [AWS RDS homepage](https://us-west-1.console.aws.amazon.com/rds/home?region=us-west-1)
	2. Select **Create database**
	3. Select **Postgres** as the database engine

		> It is recommended for this course to check the box to **Only enable options eligible for RDS Free Usage Tier** 
	4. Select **Next**
	5. In the **Settings** section enter,
		* **multi-docker-postgres** as the **DB instance identifier**
		* **postgres** as the **Master username**
		* **postgrespassword** as the **Master password**

			> **Master username** and **Master password** map to the **PGUSER** and **PGPASSWORD** environment variables set in docker-compose.yml for the **api** service.
		
		* Select **Next**
	6. In **Configure Advanced Settings** enter **database name** as **fibvalues**

		> **database name** maps to **PGDATABASE** in docker-compose.yml.
		* The default values for the other settings are acceptable for this course, so select **Create database** to create the db instance.

10. Create an elasticache (Redis) instance (lesson 142) as follows,
	1. Navigate to the [AWS elasticache homepage](https://us-west-1.console.aws.amazon.com/elasticache/home?region=us-west-1#)
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
		
			> \<Default VPC\> is where the rest of the services/containers are.
		* select all the available **Subnets**
		* The default values for the other settings are acceptable for this course, so select **Create** to create the Redis instance.
	
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