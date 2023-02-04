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




