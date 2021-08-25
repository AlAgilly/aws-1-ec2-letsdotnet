# aws-ec2-letsdotnet

Learning about the foundations of getting started in the AWS Cloud.

## Setting up an AWS Webapp
While taking this course, I had a lot of things go wrong. I'm wring down all my solutions to the problems I had in a step by step format so that anyone who might encounter these same issues (including future me) won't have to do as much googling.
1. Create and sign in to your AWS account 
2. Create an instance (click "Launch Instance")
  Instance Configuration:
    * Amazon Linux 2 (62-bit (x86))
    * t2.micro
    * Scroll down to *Advanced Details* and in *User Data* paste:
      ``` bash
      #!bin/bash
      rpm -Uvh https://packages.microsoft.com/config/centos/7/packages-microsoft-prod.rpm
      yum install dotnet-sdk-3.1 -y
      ```
    * 8GB Storage
    * Add tag
      Key: `Name`
      Value: `LetsDotNetv31`
    * Create a new Security Group 
    	> These rules are not good practice but are okay in this tutorial 
    	
    	Name: `launch-wizard-1`
    	Inbound Rules:
    	1. Type: `HTTP`
    	Protocol: `TCP`
		Port: `80`
	    Source: `0.0.0.0/0`
	    2. Type: `HTTP`
    	Protocol: `TCP`
		Port: `80`
	    Source: `::/0`
	    3. Type: `SSH`
	    Protocol: `TCP`
	    Port: `22`
	    Source: `0.0.0.0/0`
    * Create a new KeyPair
	    Name: `YourName.pem`
	    *Download the pem file*
	 * Choose existing KeyPair
	    Name: `YourName.pem`
	    *Accept Acknowledgement*
3. Launch [Rider](https://www.jetbrains.com/rider/)
	* Create *New Solution* (`ASP.NET` Core Webapp)
	Name: `LetsDotNet`
	SDK: `5.0`
	Type: `Web App`
	
	> Build to see if the app is up
	* Add a line to `pages/index.cshtml`
		```html
		<p>App is running on @Environment.MachineName</p>
		<img src="image.png" alt="This is the testing image">
		```
	
	* Add your image into your file system with the name `image.png`
	
	> Build the app again. The app should say "App is running on *NAME*" where *NAME* can be checked in command prompt using the command `hostname`
	* Get public IP of  instance in AWS
	* In Rider, go to `Tools > Start SSH Session`
		Host: *`InstanceIP`* (the IP we got from the last step)
		User: `ec2-user`
		Auth: `KeyPair`
	* In the new Rider terminal that should have opened up, type these commands:
		```bash
		sudo yum update
		sudo yum install dotnet-sdk-5.0
		```
			
		> Type `dotnet --version` in that same terminal to see if the install was successful.
		
		```bash
		mkdir letsdotnet
		```
		> Type `ls` to see if the make directory was successful.

		
