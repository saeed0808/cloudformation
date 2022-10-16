## Assumptions:
1. You have an AWS Console access and Programmatic access and you have configured aws-cli default profile with region eu-central-1 on your local /laptop.

## Download/unzip code and go to code path

            git clone https://github.com/saeed0808/cloudformation.git; cd cloudformation

## Your mission:
The Development Team needs a test server.
The system should respond to port 80 and deliver content from a database via HTTP. The
content should contain a simple text message "Welcome to your new test system!" read
from the database.

## Let’s use the AWS CLI to create a key pair and save it. in order to use key set persmission 400

       aws ec2 create-key-pair --key-name my-key-pair --query 'KeyMaterial' --output text > my-key.pem
       chmod 400 my-key.pem

## Provision: SNS, EC2 and RDS instances running in your AWS
1: Provision SNS

       aws cloudformation deploy --template-file sns.yaml --stack-name sns

2: Provision: Webserver and jumpuserver with allowed SG port 22 and 80 any where
   (if required replace vpc_id and subnet_id with your one in file ec2-web-or-jump.yaml)
   
2.1: This will provision Two ec2 one web-server 2nd jump-server

2.2: On web-server it will install apache, git, php and set coronjob to pull code every monday 

2.3: On jump-server it will install mysql 
   
       aws cloudformation deploy --template-file ec2-web-or-jump.yaml --stack-name webserver            

3: Provision RDS: varible you can replace with your one VPC, subnets, MasterPassword

3.1: This will create RDS mysql instalnce or SG and set user rdsmysql and password defined

       aws cloudformation deploy --template-file rds.yaml --stack-name rdsmysql --parameter-overrides VPC=vpc-0313e9c34ece73f28 PrivateSubnet1=subnet-0e344bd2a8859db1e MasterUserPassword=password PrivateSubnet2=subnet-07a6e5e16a49b51be PrivateSubnet3=subnet-0b8f3e11f21b00e49

4: To run some query I did setup of jumpserver you can use any tools to connect form local.

4.1: Go to aws console search RDS select your rds copy endpoint then edit Security group add web-server subnet-id/IP port 3306 for incoming traffic.

4.2: Go to ec2 copy public ip of you jump-server then login to jump server then from there connect RDS and run mysql-server querys create db, create table, insert values, select etc for our result.
   
        ssh -i my-key.pem ec2-user@3.66.171.84                  ## login jump
        mysql -hstatic.chtgu2gyidxg.eu-central-1.rds.amazonaws.com -urdsroot -p'password' -A           ## connect RDS

4.1 RUN below query for message

          create database saeed;
          use saeed;
          create table info(data varchar(500));
          insert into info values("Welcome to your new test system!");
          select * from info;

6: Web-server configuration:

6.1: Edit index.php file with correct RDS credentails endpints then send it to web-server using rsync command to code path.


            vim index.php
            $servername = "localhost";
            $username = "username";
            $password = "password";
            $dbname = "myDB";
            
            sudo rsync -e 'ssh -i my-key.pem' -avz ec2-user@IP:/var/www/html/index.php


refresh url in browser http://IP  you will get output Welcome to your new test system!

7: Cleanup infra created run below command one by one

             aws cloudformation delete-stack --stack-name sns
             aws cloudformation delete-stack --stack-name webserver 
             aws cloudformation delete-stack –stack-name rdsmysql
