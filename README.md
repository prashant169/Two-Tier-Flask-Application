# Two-Tier-Flask-Application 
(2 Tier Application with Flask and MYSQL!)

1.	Project Overview
   
The Two-Tier Flask Application is a demonstration of a simple web application architecture using Flask. The application has two main layers:

1.	Presentation Layer: Handles user interactions and presents data to users. 

2.	Data Layer: Manages data storage and retrieval.

This separation makes the application modular and easier to maintain. 

 2. Project Architecture

• Two-Tier Architecture

Tier 1 - Presentation Layer: This includes the Flask web server that processes HTTP requests, routes them to appropriate handlers, and serves responses.

Tier 2 - Data Layer: This layer manages data operations. For this project, it involves a database to store and retrieve information. It can be any database like SQLite, PostgreSQL, or MySQL 

![image](https://github.com/user-attachments/assets/f7a14063-38bc-4f66-ae1c-df385a407ddb)


3. Key Components

  1. Flask Application (Presentation Layer)

  • Routes and Views: Defines the endpoints and handles HTTP requests.

  • Templates: HTML files using Jinja2 templating to render the data.

  • Static Files: CSS, JavaScript, and images to enhance the user interface.

  2. Database (Data Layer)

  • Model Definition: SQL to define the schema and interact with the database.

  • CRUD Operations: Functions to Create, Read, Update, and Delete data.

Here's a step-by-step guide to deploying a two-tier Flask application on an AWS Ubuntu instance using Docker containers: 

![image](https://github.com/user-attachments/assets/5a05bad1-ed2a-47da-a40b-b3b869e119cb)


4. Deployment Steps

Step 1: Set Up the AWS Ubuntu Instance

        • Launch an EC2 Instance:

       -Go to the AWS Management Console and navigate to EC2.

       -Click "Launch Instance" and select an Ubuntu Server 22.04 LTS AMI.
  
       -Choose an instance type (e.g., t2.micro for free tier eligibility).
  
       -Configure instance details, add storage, and tags if needed.
  
       -Configure the security group to allow HTTP (port 80), HTTPS (port 443), and SSH (port 22) access.
  
      -Review and launch the instance.

       • Connect to the EC2 Instance:
   
        ssh -i your-key-pair.pem ubuntu@your-instance-public-ip

Step 2: Install Docker on Ubuntu
 
        • Update the package index:
 
         sudo apt-get update
 
        • Install Docker:
 
         sudo apt-get install -y docker.io
 
        • Verify Docker Installation:

         docker ps

 Alternatively, 

![image](https://github.com/user-attachments/assets/047e81b0-7e1d-4084-b110-2c3b3208e2e6)


Sudo chown $USER /var/run/docker.sock 

Using sudo chown $USER /var/run/docker.sock changes the ownership of the Docker socket file to the current user. This allows the user to interact with the Docker daemon without requiring sudo. However, this change is temporary because: 

1.	Temporary Change: The ownership change applies only until the Docker service restarts or the system reboots. Once either happens, the ownership reverts to the default settings, and the user will encounter permission issues again. 

2.	Security Implications: Changing the ownership of system files can have security implications, as it grants potentially unnecessary permissions to a single user. 

#Recommended Approach: Add User to Docker Group 

Adding your user to the Docker group is a more permanent and secure solution. Here’s why: 

1.	Permanent Fix: By adding your user to the Docker group, you ensure that the user always has the necessary permissions to interact with the Docker daemon, regardless of system reboots or service restarts. 

2.	Security and Best Practices: This method adheres to security best practices by using groupbased permissions rather than changing ownership of system files. It maintains system integrity and ensures that Docker permissions are managed appropriately. 

#By following these steps, you should be able to resolve the permission issue and successfully run Docker commands without needing to use sudo. 

2.4. Resolve Docker Permission Issues:

    • Add Your User to the Docker Group:

      sudo groupadd docker
    
      sudo usermod -aG docker $USER
    
      newgrp docker

    • Start and Enable Docker Service:
    
      sudo systemctl start docker
    
      sudo systemctl enable docker

Step 3: Prepare the Flask Application
    
      • Install Git (if not already installed):
    
      sudo apt update
    
      sudo apt install git -y
    
      • Clone the Repository:

       git clone https://github.com/saiguda654/2-tier-flask-app.git
   
       cd 2-tier-flask-app
----------------------------------------------------------------------------
Step 4: Launch the MySQL Database Container
     
     •launch the MySQL database container with the volume mounted to it as below: 

 • Run MySQL Container: 
  
  docker run -d -p 3306:3306 --name mysql \
  
  -e MYSQL_ROOT_PASSWORD=admin \

  -e MYSQL_DATABASE=testdb \
  
  -e MYSQL_USER=admin \
  
  -e MYSQL_PASSWORD=admin \
  
  mysql:latest
  
----------------------------------------------------------------

 • Verify Database Creation:

   docker ps
   
   docker exec -it <container id> bash
   
   mysql -u root -p
   
   show databases;

-----------------------------------------------------------------------------

Step 5: Create and Run Flask App Container(Create your frontend )
  
     •	At this moment we have the mysql image. To create the flask-app image (make sure you are in the path that contains your application + Dockerfile) run: 

     •  Build Flask App Image:
   
       docker build -t flask-app .
   
       docker images    

---------------------------------------------------------

   • Run Flask App Container:

 docker run -d -p 5000:5000 --name flask-app \
 
  -e MYSQL_HOST=mysql \
  
  -e MYSQL_USER=admin \
  
  -e MYSQL_PASSWORD=admin \
  
  -e MYSQL_DB=testdb \
  
  flask-app:latest

-----------------------------------------------------------------------------------
Open Port 5000 on EC2 Security Group:

Ensure that port 5000 is open in the security group settings of your EC2 instance.

 • Verify Application Access:

Navigate to http://<your-ec2-public-ip>:5000 in your web browser to access the Flask application.

--------------------------------------------------------------------------------------------------

Step 6: Create a Docker Network for Communication
     
        docker network ls 
        
        Create a Docker Network:

        docker network create -d bridge two-tier-app-nw

----------------------------------------------------------------------------------------------------------

     6.2  Re-launch Containers with Network Configuration:

•	To achieve this goal, edit the run commands for both MySql and FLask to include --network <name of newly created network> , as shown below: 

• Destroy the previously created MySQL and flask-app containers $ docker kill <container id> , then docker rm <container id>, otherwise you will get an error message when containers are relaunch as ports are in use. 


 • MySQL Container:

docker run -d -p 3306:3306 --name mysql --network two-tier-app-nw \
  
  -e MYSQL_ROOT_PASSWORD=admin \
  
  -e MYSQL_DATABASE=testdb \
  
  -e MYSQL_USER=admin \
  
  -e MYSQL_PASSWORD=admin \
  
  mysql:latest


• Flask App Container:

docker run -d -p 5000:5000 --name flask-app --network two-tier-app-nw \

  -e MYSQL_HOST=mysql \
  
  -e MYSQL_USER=admin \
  
  -e MYSQL_PASSWORD=admin \
  
  -e MYSQL_DB=testdb \
  
  flask-app:latest

• Verify Containers on Network:

  docker inspect two-tier-app-nw

•	At this point, both the front-end and back-end containers have been launched in the same network. 

•	Take your ec2 IP address:5000 and run this URL in your browser. 

•	Congratulations!! Your app is up and running   

------------------------------------------------------------------------------------------------------------------

Step 7: Verify Application and Data Access Application:
      
       Access Application:

      Open http://<your-ec2-public-ip>:5000 in your web browser.

 • Check Database Entries:

  docker exec -it <mysql container id> bash

  mysql -u root -p

  use testdb;

  select * from messages;

•	The input data has been effectively saved in the database, as evident from the displayed tables. 

OutPut

![Instances _ EC2 _ us-east-1 - Google Chrome 07-08-2024 20_18_02](https://github.com/user-attachments/assets/dae5b4c2-2443-4ab6-a348-81fc91a411ae)

![SecurityGroup _ EC2 _ us-east-1 - Google Chrome 07-08-2024 20_20_56](https://github.com/user-attachments/assets/34f44148-f25d-4450-bb2c-d28833805f09)

![3 85 196 93 (ubuntu) 07-08-2024 20_18_45](https://github.com/user-attachments/assets/ffac32c8-4e3c-4123-9364-1a840bb3fe88)


Thank you @Sai_Guda Sir 


