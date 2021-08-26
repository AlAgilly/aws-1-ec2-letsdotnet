
# aws-ec2-letsdotnet

Learning about the foundations of getting started in the AWS Cloud.

## Setting up an AWS Webapp
While taking this course, I had a lot of things go wrong. I'm wring down all my solutions to the problems I had in a step by step format so that anyone who might encounter these same issues (including future me) won't have to do as much googling.
1. Create and sign in to your AWS account.
2. Create an instance (click "Launch Instance").
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
3. Launch [Rider](https://www.jetbrains.com/rider/).
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
	* In Rider, go to `Tools > Start SSH Session...`
		Host: *`InstanceIP`* (the IP we got from the last step)
		User: `ec2-user`
		Auth: `KeyPair`
	* In the new Rider terminal that should have opened up, type these commands:
		```bash
		sudo yum update
		```
		```bash
		sudo yum install dotnet-sdk-5.0
		```
			
		> Type `dotnet --version` in that same terminal to see if the install was successful.

		```bash
		mkdir letsdotnet
		```
		> Type `ls` to see if the make directory was successful.
		
	* Edit the build configurations and add a new configuration (*Publish to custom server*)
		Name: `DeploytoEC2` Remote Server: `Add a new deployment (SFTP)`
		> A new deployment window will open.
		
		Name: `MyEC2` SSH Configuration: `Add a new configuration`
		> A new configuration window will open.
		
		Host: *`InstanceIP`* (the IP we got from a couple of steps ago)
		User: `ec2-user`
		Authentication: `KeyPair`
		Root: *`Autodetect`* + `/home/ec2/LetsDotNet`
		URL: `http://` + *`InstanceIP`*
		> Click ok on all the open windows

	* In `program.cs` add the following line before `webBuilder.UseStartup<Startup>();`
		```csharp
		webBuilder.UseKestrel(options => { options.Listen(IPAddress.Any, 80); });
		```
	* Run *DeploytoEC2*
	* Click on `Tools > Start SSH Session...`
	* A new terminal should open up with the title *`InstanceIP`*. Type the following in that terminal:
		```bash
		cd letsdotnet
		```
		> Type `ls` to confirm your connection, build, and deployment was successful.
		
		```bash
		sudo dotnet LetsDotNet.dll
		```
4. In AWS, navigate to S3.
	* Create a Bucket
	Name: `letsdotnet`
	* Upload the testing image (`image.png`) used in step 3
5. To download AWSPowerShell, open PowerShell.
	```bash
	msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
	```
	> For more info on the download or for any other OS, go to: https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html

6. Create an IAM user [here](https://console.aws.amazon.com/iam)
	* Make sure to check the `Programmatic access` box
	* In PowerShell, type `aws configure --profile`*`NAME`*
		> Type the following to check that the user is configured correctly:
		```bash
		cd .aws
		```
		```bash
		aws S3 ls --profile NAME
		```
		```bash
		Get-S3Bucket
		```
		
	* We're going to make the image in the bucket we just made available for 3 days using the following command:
		```bash
		Get-S3PreSignedURL -BucketName letsdotnet -Key image.png -Verb GET -Expire (Get-Date).AddDays(3)
		```
	* Copy the URL that PowerShell returns and go back to the `index.cshtml` code in Rider.
	* Replace `image.png` with the new URL. Your index should look something like this:
		```html
		<p>App is running on @Environment.MachineName</p>
		<img src="https://letsdotnet.s3.us-east-2.amazonaws.com/image1.png?X-Amz-Expires=259200&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAXH3TLZQ/20210824/us-east-2/s3/aws4_request&X-Amz-Date=202104Z&X-Amz-SignedHeaders=host&X-Amz-Signature=237f24c9d7572fbab2" alt="This is the testing image">
		```

There you go! You now have a working AWS instance running your webapp; complete with an image in your bucket that is being called. A little side note for myself: make sure that anytime you want to rebuild the app, you have to:
1. Build in the `DeploytoEC2` configuration
2. Start the SSH session (`Tools > Start SSH Session...`)
3. In the *`InstanceIP`* terminal, always run these two commands:
	```bash
	cd letsdotnet
	```
	
	```bash
	sudo dotnet LetsDotNet.dll
	```
