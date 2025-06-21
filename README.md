<h1>☁️ Migrating a MariaDB Database from EC2 to Amazon RDS</h1>

This project walks through how to migrate a **MariaDB database** running on an **Amazon EC2 instance** to a **fully managed Amazon RDS MariaDB instance**. The web application is then updated to connect to RDS, ensuring data reliability, backups, and reduced manual maintenance.

---
At the end of this project, your architecture will look like the following example:
<br/>
<img src="first db.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
---

<h2>📖 Scenario</h2>
The café initially hosted its website, database, and application on a single EC2 instance. As the business grew:

- Maria and Covenant needed reliable access to order history
- Sofia wanted automatic backups, patching, and easier administration
- The team wanted to reduce labor costs and technical complexity

✅ Solution: Move the database to **Amazon RDS (MariaDB)** and connect the existing app to it.
<h2>🎯 Objectives</h2>

- Export the existing MariaDB database from EC2
- Create a new Amazon RDS MariaDB instance
- Import the database into RDS
- Update the café’s web app to use RDS
- Test and secure the solution

---

<h3>🔧 AWS Services Used</h3>

| Service     | Role in Project                            |
|-------------|---------------------------------------------|
| EC2         | Hosted the original MariaDB + web server    |
| RDS         | New managed MariaDB database                |
| VPC         | Ensured secure networking between resources |
| Security Groups | Controlled database access              |

---

<h3>🧱 Step-by-Step Guide</h3>

✅ Step 1: Prepare the EC2 Instance (Source Database)
🧰 What you need:
A running EC2 instance (Amazon Linux or Ubuntu)

MariaDB already installed and holding the café’s data

<h4>✅ Step 2: Create Amazon RDS MariaDB Instance (Target Database)</h4>

- Go to the Amazon RDS console
- In the Engine options section, for Engine type, choose MariaDB.
- For Templates, choose Dev/Test
- In the Settings section, configure the following options:
- DB instance identifier: Enter CafeDatabase.
- Master username: Enter admin.
- Master password and Confirm master password: Enter Caf3DbPassw0rd!.
- In the Instance configuration section, for DB instance class, choose Burstable classes (includes t classes), and then choose db.t3.micro
- In the Storage section, configure the following options:
- Storage type: Choose General Purpose SSD(gp2).
- Allocated storage: Enter 20 GiB
- For Availability & durability, choose Do not create a standby instance.
- VPC: Select your existing VPC (where EC2 also lives)-
- Subnet group: Leave as default or choose one with private subnets
- Public access: ❌ No (recommended — only EC2 can reach it inside the VPC)
- VPC security group: Select existing security group (or create one)
- This security group must allow port 3306 from your EC2 instance (This means you have already created a VPC already)
- Choose Create database.

<h4> you connect to the existing EC2 instance that runs the current café application.</h4>
Next, you connect to the EC2 instance by using AWS Systems Manager to access a terminal session in the browser.

- In the Amazon EC2 console, in the left navigation pane, choose Instances, and then choose the name of your EC2 instance. (We will call it cafeserver just for this project)
- To open the Connect to instance menu, choose Connect.
- Choose Session Manager.
- Choose Connect.(You should now have a new browser tab open with a terminal session that is connected to the EC2 instance.)
- At the prompt, enter the following commands:
  <br/>
<img src="1 db.jpeg" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
 Analysis: The first command gives you a bash shell. The second command switches your session to use the root user account on the EC2 instance. The third command switches you to use the ec2-user account. The fourth command should return output that confirms that you are connected as the ec2-user. The last command switches your terminal to the home directory of the ec2-user
 <br/>
 <br/>
<img src="2 db.jpeg" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
Note: The Systems Manager agent (ssm agent) is installed by default on all Amazon Linux 2 instances (and some other operating system types)

<h4>✅Description For Next Step</h4>
Now that you have created a new RDS instance, you can move on to the next step in the café's database migration plan. Next, you export the data from the database that the café application currently uses. You also establish a network connection from the EC2 instance (where the application runs) to the new RDS database instance.

You then export the existing data from the database by using the mysqldump utility.
To observe details of the database that runs on the EC2 instance, in the terminal, run the following commands:
 <br/>
<img src="3 db.jpeg" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

- Leave this browser tab open. You use it throughout this lab.

- On the AWS Management Console, in the search box, enter and choose Secrets Manager to open the AWS Secrets Manager console.

- In the left navigation pane, choose Secrets.

- There are seven secrets stored here. The café application PHP code references these values (for example, to retrieve the connection information for the database).

- Choose the /cafe/dbPassword secret.

- In the Secret value section, choose Retrieve secret value, and copy the Value to your clipboard. 

- You use this value in a moment.

To connect to the database that is running on the EC2 instance, in the browser tab with the bash terminal, run the following command:
 <br/>
<img src="4 db.jpeg" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

When prompted for the database password, paste the dbPassword parameter value that you copied a moment ago.
<br/>
<br/>
<img src="5 db.jpeg" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

To observe the data in the existing database, run the following commands:
<br/>
<br/>
<img src="6 db.jpeg" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

To exit the SQL client, enter the following command:

<br/>
<br/>
<img src="8 db.jpeg" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

To capture existing data in a file by using the mysqldump utility, run the following command:

<br/>
<br/>
<img src="9 db.jpeg" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

When prompted for the database password, paste the dbPassword value from the Secrets Manager secrets.
To confirm that mysqldump succeeded, run the "ls" command in the terminal. 
To see the contents of the file, run the "cat CafeDbDump.sql" command.

<h3>In the next step, you import this data into the new RDS database.</h3>

- Establish a network connection from the terminal running on the EC2 instance to the new RDS instance.
- Run the following command (mysql -u admin -p --host cafedatabase.c1gc0eas4zav.us-east-1.rds.amazonaws.com)
- Go to RDS → Databases

Click your instance name

Copy the Endpoint (e.g., cafe-db.abcd1234.us-west-2.rds.amazonaws.com)

To allow your EC2 instance to connect to your Amazon RDS (MariaDB) database, you need to set a security group rule that allows inbound access on port 3306.
- ✅ The Rule You Need:
- 🔒 Inbound Rule for RDS Security Group
- Type	Protocol	Port Range	Source	Description
- MySQL/Aurora	TCP	3306	The EC2 instance's security group ID or its private IP	Allow EC2 to connect to RDS

<h2>💡 How to Add It (Console):</h2>
Go to the VPC → Security Groups in AWS Console.

Find and select the security group attached to your RDS instance.

Go to Inbound Rules → Edit inbound rules.

Add this:

Type: MySQL/Aurora

Protocol: TCP (auto-filled)

Port Range: 3306

Source: put the security group of the EC2

Save the rule.

It's important to confirm that you can connect to the RDS MariaDB instance before you go to the next step. Run the show databases; command. 

  It should show the following output:
  <br/>
<img src="10 db.jpeg" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
Notice that cafe_db database is not in the list yet. This situation is expected because you haven't imported any data. To disconnect, run the exit; command.

<h2>✅ Step 4: Import the Database Dump into Amazon RDS</h2>

In the previous step, you exported the data from the database that the café application currently uses. You also established a network connection from the EC2 instance to the RDS instance.

To import the data that you exported in task 3 to the RDS database instance, in the following command, replace  with the actual endpoint, and run your adjusted command
 <br/>
<img src="11 db.jpeg" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
At the password prompt, enter the password for the RDS instance.

To confirm that the data was imported and connect to the RDS database, run the following command:
 <br/>
<img src="12 db.jpeg" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
At the password prompt, enter the password for the RDS instance.

To confirm that the data was imported, run the following commands:
<br/>
<img src="13 db.jpeg" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

To exit the SQL client, run the following command:
<br/>
<img src="14 db.jpeg" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

<h2>✅ Step 3: Update the Web App to Use Amazon RDS</h2>

In this last task in the lab, you connect the café application to the new database. You also stop the database that runs locally on the EC2 instance.

- Return to the Secrets Manager console browser tab.
- In the left navigation pane, choose Secrets. Recall from an earlier challenge lab that the café application's PHP code references these values. For example, it uses the values to retrieve the connection  information for the database.
- Connect the café application to the RDS instance.
Because the database connection information has changed, you must update these values to connect the application to the new RDS database instance instead of to the database running on the EC2 instance.

- The currency, dbName, timeZone, and showServerInfo values don't need to be updated.
- The dbUrl should be the RDS endpoint value.
- The dbuser should be the master username you used for the RDS DB
- The dbpassword should be the password you used for the RDS DB

To confirm that your web application now uses the new database and to stop the database that's still running on the EC2 instance, in the terminal, run the following command:
<br/>
<img src="15 db.jpeg" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

<h2>🎉 Success!</h2>
You’ve now:

Migrated your data to RDS
Improved reliability, backups, and scalability

Updated your app to connect to it

Verified that everything works

<h1>🙋 Author</h1>
Covenant Urch
AWS Cloud Solutions Builder | Lagos, Nigeria
