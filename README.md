# MonolithicNodejsApp
Breaking a Monolithic Node.js Application into Microservices

The following folders:

1-no-container – Contains the files that are related to the monolithic implementation of the application. This implementation is intended to run directly on a Node.js server.
2-containerized-monolith – Contains the files that are related to the monolithic implementation of the application. This implementation is intended to run in a containerized Docker environment orchestrated by Amazon ECS.
3-containerized-microservices – Contains the files that are related to the microservices implementation of the application. This implementation is intended to run in a containerized Docker environment orchestrated by Amazon ECS.

Note: The monolithic implementation of the application uses the Node.js cluster functionality to spawn one worker process per CPU core. The processes share a single port, and are invoked in a round-robin way by the load balancer that is built into Node.js. This feature increases scalability on servers that have multiple CPU cores.

In this task, we will:

Install the Node.js modules required by the application
Review the application design and code
Run the application

by running the codes below

cd ~/environment/1-no-container
npm install koa
npm install koa-router

The modules are downloaded and installed in the 1-no-container/node_modules folder of the AWS Cloud9 ~/environment folder. You can ignore the notice, warnings, and update messages in the output.

Task 2.2: Reviewing the application design and code

The components that implement the monolithic message board application are in the 1-no-container folder. Review them to gain an understanding of the application design and code.

In the Environment window on the left, expand the 1-no-container folder. The components of the application include:

node_modules folder – This folder was created when you installed the required JavaScript modules in the previous subtask. It contains their source code.
db.json – A JavaScript Object Notation (JSON) object that simulates the message board database. It contains attributes that represent users, threads, and posts, with corresponding sample values.
index.js – JavaScript program that is the application's entry point.
package.json – A JSON object that describes the application, its entry point, and its dependencies.
package-lock.json – A JSON object that was automatically generated when you installed the required JavaScript modules in the node_modules folder. It is used by the installation utility, npm, to track the modifications that are made to the folder.
server.js – JavaScript program that defines the application's RESTful API methods, and implements their respective handlers.
Examine the package.json object. In the Environment window, open package.json an editor tab by double-clicking it. Notice the following attributes of the JSON object:

Lines 2 through 5 – The dependencies attribute defines the JavaScript module dependencies for the application. Note that the koa and koa-router modules that you installed in the previous subtask are listed here.
Lines 6 through 8 – The scripts attribute declares the index.js program as the entry point to the application.
Examine the db.json object. In the Environment window, open db.json in an editor tab by double-clicking it. Notice the following attributes of the JSON object:

Lines 2 through 27 – These lines define a users attribute that represents the registered users of the message board. The attribute value is a list of four sample users with the following names: Marcerline Singer, Finn Alberts, Paul Barium, and Jake Storm.

Lines 29 through 45 – These lines define a threads attribute that represents the current active threads on the message board. The attribute value is a list of three sample threads with the following titles:

Did you see the Brazil game?
New French bakery opening in the neighborhood tomorrow
In search of a new guitar
Lines 47 through 78 – These lines define a posts attribute that represents the posted messages on the active threads. The attribute value is a list of six sample message posts.
Review the code for index.js. In the Environment window, open index.js in an editor tab by double-clicking it. Notice the following information:

Lines 1 through 3 – These lines import the JavaScript modules that the program requires, specifically: cluster, http, and os.
Line 3 – This line uses the os module to ask about the number of CPU cores that are available on the server.
Lines 5 through 15 – These lines are run the first time the program is invoked (when the application is started). They create a Leader thread for the cluster and one worker thread for each CPU core that is available on the server.
Lines 16 through 19 – These lines handle each request to the application by invoking the server.js
program in the current worker thread.
Lastly, review the code for server.js. In the Environment window, open server.js in an editor tab by double-clicking it. Use the comments that are provided in the code to facilitate your understanding of the logic. In particular, notice the following information:

Line 3 – This line imports db.json, the JSON object that simulates the database.

Lines 6 through 11 – These lines define a generator function that runs for every request. Its purpose is to print a line that contains the HTTP method, resource path URL, and elapsed time for each request that is processed.
Lines 13 through 47 – These lines define the application's RESTful API methods and their implementation. Specifically, the application can respond to the following RESTful calls.

GET /api/users: Returns the collection of users in the database
GET /api/users/:userId: Returns the information for the user that is identified by :userId
GET /api/threads: Returns the collection of threads in the database
GET /api/threads/:threadId: Returns the information for the thread that is identified by :threadId
GET /api/posts/in-thread/:threadId: Returns the collection of post messages for the thread that is identified by :threadId
GET /api/posts/by-user/:userId: Returns the collection of post messages for the user that is identified by :userId
GET /api/: Returns the message API ready to receive requests
GET /: Returns the message Ready to receive requests
Line 52 – This line defines the port number where the application listens for requests


Task 2.3: Running the application

In this subtask, you will start the Node.js server and run the message board application. Then, you will test some of its RESTful API methods.

In the terminal tab, start Node.js and the application by entering the following command: 
npm start

The server is started, and the application's entry point, index.js, is run. The first time that index.js is invoked, it creates two cluster threads—Leader and Worker—to process requests.

Next, you will leave the current terminal session active and open a second terminal tab to test the application's RESTful API.

In the bottom pane, open a new terminal tab by choosing (+) and selecting New Terminal. You now have two terminals where you can enter commands.

In the right terminal tab, retrieve the /api/users resource by entering the following command:
curl localhost:3000/api/users

The RESTful invocation returns a JSON object that contains the list of users in the message board database.

 

Select the left terminal tab. You see an output message from server.js that it processed a GET method request on the resource, which is identified by the path /api/users. The request took 3 milliseconds to process.

Retrieve the information for only the fourth user in database. In the right terminal tab, enter the following command:

curl localhost:3000/api/users/4

The information for Jake Storm, the fourth user in the database, is returned:


Next, retrieve all the threads that are currently in the database. In the right terminal tab, enter the following command:

curl localhost:3000/api/threads

A JSON object that contains all the threads in the database is returned

Lastly, retrieve all the posts for the first thread in the database. In the right terminal tab, enter the following command:

curl localhost:3000/api/posts/in-thread/1

A JSON object that contains two message posts is returned.

Stop the Node.js server. In the left terminal tab, press CTRL+C to terminate the server process.

You have validated that the application responds properly to GET requests. In the next task, you will containerize the application.

Task 3: Containerizing the monolith for Amazon ECS

Containers wrap application code in a unit of deployment, which captures a snapshot of the code and its dependencies. They can help ensure that applications deploy quickly, reliably, and consistently, regardless of the deployment environment.

In this task, you will build a container image for the monolithic message board application and push it to Amazon Elastic Container Registry (Amazon ECR). This step prepares the application for deployment to Amazon ECS.

Specifically, you will perform the following steps:

Prepare the application for Docker containerization
Provision a repository
Build and push the Docker image to the repository

Task 3.1: Preparing the application for Docker containerization

To put the message board application into a Docker container, the following changes must be made to the application:

Remove the use of the Node.js cluster feature and convert the application to a single-process design. With Docker containers, the goal is to run a single process per container, instead of a cluster of processes.
Create a Dockerfile for the application. This file is basically a build script that contains instructions about how to build a container image for the application.
A container-ready version of the application is provided to you in the 2-containerized-monolith folder of your AWS Cloud9 environment. Take a few minutes to review the files and understand the changes that were made to prepare the application for containerization.

In the Environment window on the left, expand the 2-containerized-monolith folder, and open the package.json in an editor tab by double-clicking it.

In Line 7, notice that the entry point into the application was changed from index.js to server.js. The index.js file is no longer in the application folder. The index.js file contained the initialization logic for the Node.js cluster feature, and you will no longer use that feature.

In the Environment window, expand the 2-containerized-monolith folder, and open the server.js file in an editor tab by double-clicking it.

The only difference from the non-containerized version is the addition of Line 54, which prints the message Worker started when the application is first started.

In the Environment window, expand the 2-containerized-monolith folder, and open the Dockerfile in an editor tab by double-clicking it.

This file contains the instructions about how to build the container image for the application.

Notice the following information:

Line 1 – The base image where the container image will be built. Here, it is alpine-node, which is a Node.js image.
Line 3 – This line sets the working directory of the file system on the image to /srv.
Line 4 – This line adds the contents of the 2-containerized-monolith folder (the application folder) to the current working directory of the image's file system (which is set in the previous line).
Line 5 – This line invokes the npm install command to install all the application's library dependencies that were declared in the package.json file.
Line 7 – This line informs Docker that the container listens on port 3000 when it runs.
Line 8 – This line asks Docker to run the command node server.js, which starts the application when the image is started.
Now that you understand how the container image for the application will be built, you will next examine where to put the image after it is built.


Task 3.2: Provisioning a repository

Docker container images are intended to be stored in a repository for sharing, version control, and easier management purposes. Amazon ECR makes it easy for developers to store, manage, and deploy Docker container images. In addition, Amazon ECR is integrated with Amazon ECS, which enables Amazon ECS to pull container images directly for production deployments.

In this subtask, you will create a repository in Amazon ECR to house the Docker container image for the message board application.

In the Your environments browser tab, choose Services, and then select Container > Elastic Container Registry.

The Amazon ECR console opens.

In Create a repository, choose Get Started.

In the Repository name box, enter mb-repo.

Choose Create repository.

A message at the top of the page indicates that the repository was successfully created.

Note: Do not close the window that shows the message. You will use it in the next subtask.


Task 3.3: Building and pushing the Docker image

You are now ready to build the container image for the application and push it to the Amazon ECR repository that you created.

One useful feature of the Amazon ECR console is that it provides ready-to-use command templates for building and pushing an image to the new repository. You use these provided AWS CLI commands in the next steps.

Before you can successfully run the next steps, you must upgrade the AWS CLI. To do this, go to the AWS Cloud9 IDE browser tab, and in the left terminal tab, enter the following commands:


pip3 install awscli --upgrade --user
export PATH=$HOME/.local/bin:$PATH


Go back to the Amazon ECR console browser tab, and in the message window at the top of the page, choose View push commands.

The Push commands for mb-repo pop-up window opens. This window lists four AWS CLI commands that are customized for the mb-repo, and they are purposely built to:

Authenticate your Docker client to your Amazon ECR registry
Build your Docker image
Tag your Docker image
Push your Docker image to the repository
The pop-up window offers two versions of the commands: one for macOS/linux, and one for Microsoft Windows.

Make sure that the macOS/Linux tab is selected, because you will run these commands in your AWS Cloud9 environment.

First, you will copy and run the command to log in your Docker client to your registry.

In the pop-up window, locate the first command and then copy the command to the clipboard by choosing the Copy icon.

The command looks like the following example:

Switch to the AWS Cloud9 IDE browser tab.

In the left terminal tab, paste the copied command and run it by pressing ENTER


If the command runs successfully, it returns the message Login Succeeded. You can ignore the displayed warnings.

Next, you will build the Docker image for your application.

Note: When a specific terminal tab is not mentioned in an instruction step, use the left terminal tab.

In the terminal tab, change the directory to the 2-containerized-monolith folder by entering the following command: 

cd ~/environment/2-containerized-monolith


Switch to the Amazon ECR console browser tab.

In the Push commands for mb-repo window, locate the second command and copy the command by choosing the Copy icon.

The command looks like the following example:

docker build -t mb-repo .


Make sure to include the period (.) at the end of the command.

Switch to the AWS Cloud9 IDE browser tab.

In the terminal tab, paste the copied command and run it by pressing ENTER


The build command produces multiples lines of output as it runs the instructions that are in the application's Dockerfile. When it is finished, you see the messages Successfully built nnnnnnnnnn and Successfully tagged mb-repo:latest.

Next, you will tag the image with the repository URI so that it can be pushed the repository.

Switch to the Amazon ECR console browser tab.

In the Push commands for mb-repo window, locate the third command and choose the Copy icon.

The command looks like the following example:

docker tag mb-repo:latest 1234567890.dkr.ecr.us-east-2.amazonaws.com/mb-repo:latest


Switch to the AWS Cloud9 IDE browser tab.

In the terminal tab, paste and run the copied command: example:


docker tag mb-repo:latest 662270786699.dkr.ecr.us-east-1.amazonaws.com/mb-repo:latest


The command does not return anything if it completed successfully.

Finally, you will push the container image to the application's repository.

Switch to the Amazon ECR console browser tab.

In the Push commands for mb-repo window, locate the fourth command and copy it.

The command looks like the following example:


docker push 662270786699.dkr.ecr.us-east-1.amazonaws.com/mb-repo:latest

Switch to the AWS Cloud9 IDE browser tab.

In the terminal tab, paste and run the copied command


The command outputs several messages as each layer of the image is pushed to the repository.

Next, you will verify that the image was successfully uploaded.

Switch to the Amazon ECR console browser tab.

Close the Push commands for mb-repo window.

In the Repositories list, choose mb-repo.

In the Images list, you should see the container image that you pushed, which you can identify by the latest tag.


Record the Image URI. In the Images list, locate the Image URI of the latest version of the image, and choose the Copy icon. Paste the value in a text editor. You will use it in a subsequent step.

You have successfully created a container image for the message board application, and you have also pushed it to an Amazon ECR repository.

662270786699.dkr.ecr.us-east-1.amazonaws.com/mb-repo:latest

Task 4: Deploying the monolith to Amazon ECS

In this task, you deploy the containerized monolithic application to an Amazon ECS runtime environment. Specifically, you use Amazon ECS to create a managed cluster of Amazon Elastic Compute Cloud (Amazon EC2) instances. You will deploy your application container image to this cluster. The cluster is configured as the target group of an Application Load Balancer, which will provide failover and scalability.

The following diagram shows the deployment architecture of the containerized monolithic application. It also displays the resources that you will create in this task.


The steps that you perform in this task are:

Create an Amazon ECS cluster.
Create a task definition for the application container image.
Create the Application Load Balancer.
Deploy the monolithic application as an ECS Service.
Test the containerized monolithic application.

Task 4.1: Creating an Amazon ECS cluster

An Amazon ECS cluster is a logical grouping of EC2 instances where you can run tasks or services that represent your containerized application.

In this subtask, you will create an ECS cluster by using the Amazon ECS console. The console's cluster creation wizard enables you to create all the infrastructure components that are needed to create the ECS cluster environment. These components include the virtual private cloud (VPC), subnets, security groups, internet gateway, and AWS Identity and Access Management (IAM) roles.

Go back to the AWS Management Console browser tab, choose Services, and then select Containers > Elastic Container Service.

In the navigation pane, choose Amazon ECS > Clusters.

In the Clusters page, choose Create Cluster.

In the Select cluster template page, select the EC2 Linux + Networking card.

Choose Next step.

In the Configure cluster wizard, configure the following settings.

Cluster name: mb-ecs-cluster
Provisioning Model: On-Demand Instance
EC2 instance type: t2.micro
Number of instances: 2
VPC: Create a new VPC
CIDR block: 10.32.0.0/16
Subnet 1: 10.32.0.0/24
Subnet 2: 10.32.1.0/24
Security group: Create a new security group
Security group inbound rules: Leave it at the default setting, which allows inbound traffic from all IP addresses on port 80.
Note: The message in the Container instance IAM role section states that you are granting permissions to Amazon ECS to create and use the ecsInstanceRole. This role authorizes the EC2 instances in the cluster to invoke Amazon ECS actions.

Choose Create.

The Launch Status page opens, and shows the tasks that the wizard performs.

Wait until all tasks have a check mark, which indicates that they are complete.

The resources that the wizard creates are listed in the Cluster Resources section.

Choose View Cluster.

The details page for the mb-ecs-cluster opens. The Status field shows a value of ACTIVE.

Choose the ECS Instances tab.

The two EC2 instances for the cluster (which the wizard created) are listed.

Note: It might take a few minutes for the two EC2 instances to show in the list. If you do not see both instances, choose Refresh.

Choose the Tasks tab.

No tasks are deployed to the cluster yet. You will create one next.




Task 4.2: Creating a task definition for the application container image

A task definition is a list of configuration settings for how to run a Docker container on Amazon ECS. It tells Amazon ECS various kinds of information, such as:

What container image to run
How much CPU and memory the container needs
What ports the container listens to traffic on
In this subtask, you will create a task definition for the container image of the message board application.

In the navigation pane of the Amazon ECS console browser tab, choose Task Definitions.

Choose Create new Task Definition.

In the Select launch type compatibility page, choose the EC2 card.

Choose Next step.

The Configure task and container definitions page opens.

In the Task Definition Name box, enter mb-task.

Scroll down to Container Definitions and choose Add container.

The Add container window opens.

Configure the following settings.

Container name: mb-container
Image: Paste the Image URI of the application container image, which you copied to a text editor in a previous step.
Memory Limits: Select Hard limit and enter 256. (This setting defines the maximum amount of memory that the container is allowed to use.)
Port mappings > Container port: 3000 (This setting specifies the port where the container 
receives requests. You do not need to enter a value in Host port.)
The Add container window should look similar to the following example

Choose Add.

Scroll down and choose Create. You can ignore any warnings.

A message displays, indicating that the task definition was successfully created. Notice that the definition is automatically assigned the version number of 1.















