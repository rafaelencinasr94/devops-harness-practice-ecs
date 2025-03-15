## Harness Docker Delegate installation
In order for our pipeline to be able to deploy to ECS Fargate we need a Harness Delegate.

1. On AWS, create an EC2 instance in the same region as your project, in our case us-west-1 (N. California)
2. Type in "docker-delegate" for its name
3. For its AMI choose "Aamazon Linux 2023 AMI"
4. For instance type "t2.micro" is fine for testing purposes. For production you might want to choose an instance type with more memory. 
5. Make sure thath "Auto-assign public IP" is Enabled, and to choose a key pair which you have access to.
6. For the security group just make sure SSH traffic is allowed.
7. Create your instance.
8. Once the EC2 instance is ready, connect to it (via SSH from your local computer) and install docker running the `sudo yum install -y docker` command. 
9. Start the docker service by running the `sudo service docker start` command.
For more information about installing docker see here https://docs.aws.amazon.com/es_es/serverless-application-model/latest/developerguide/install-docker.html

10. In Harness, navigate to "Account Settings", found on the left panel.
11. In the Account-level resourcers click on "Delegates"
12. Click on the "Install a Delegate" button
13. Select the "Docker" option
Important: the delegate name MUST match the name of the EC2 instance
14. Copy the command shown and run in it inside your EC2 instance.
15. Wait about 1-2 minutes and then click on the "Verify" button, you should see a message that says: "Great! You have successfully installed the Delegate."

Note: After a while the delegate instance might crash because of how little memory t2.micro instances provide. Just reboot the instance and re-run the "Install your Delegate" command from Harness. 

## Import pipeline from GitHub
1. On Harness "Continous Delivery & GitOps" section, click on the dropdown besides the "+ Create a Pipeline" button, and select "+ Import From Git"
2. Choose the "Third-party Git provider" option
3. On Git Connector, click on the selector and create a new connector.
4. Choose "GitHub Connector", and type in "Devops Harness ECS connector" as its name. Click on "Continue".
5. For "URL Type" choose "Repository", and "HTTP" for its "Connecetion Type"
6. Input the URL the your fork of this repository. Click on "Continue".
7. You may choose a different "Authentication" mode, but for here we will use "Username and Token"
8.  Type in your GitHub username
9. Click on "Create or Select a Secret" and a modal window will appear.
10. Click on "+ New Secret Text"
11. Type in a name for your secret.
12. For the secret value you will need to create a GitHub access token (classic) with the following permissions scope: "admin:repo_hoo", "repo", "user", see here for more information: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic
13. Click on "Save", and then on "Apply Selected"
14. Select "Enable API access", you may use the same access token. Click on "Continue".
15. Choose "Connect through Harness Platform". Click on "Save and Continue".
16. Click on "Finish" after the connection test is successful.
17. The "Repository" and "Git Branch" should autopopulate with the name of the repository and "main" respectively
18. For "YAML Path", the input should look like similar to this: `.harness/Pipeline_file.yaml`.

## ECS service configuration
1. Click on the "Deploy to ECS" stage, navigate to "Service", 
2. Click on "+ Add Service"
3. Name it: "ECS service"
4. Choose inline
5. For the "Task Definition" section choose "Task Definition" option with the little Harness Logo, and then on "+ Add Task Definition"
6. On the "Specify ECS Task Definition Store", choose Harness
7. For Manifest Identifier input "TaskDefinition"
8. Click on the "- Select -" input
9. Create a new folder called "ECS deployment"
10. Inside this folder create a file called "taskDefinition.json". Use the template from the "taskDefinitionTemplate.json" from this GitHub repository.
11. Replace wherever you see <awsAccountId> with your actuall AWS account ID
12. Save and click on "Apply selected"
13. Click on "+ Add Service Definition" to create a new one.
14. Follow the same steps as from the task definition creation instructions, and inside the "ECS deployment" folder create a new file called "CreateServiceRequest.json"
Use the template from the "serviceDefinitionTemplate.json" file from this GitHub repository.
15. As you can see for the "securityGroups" and "subnets" arrays there are the following strings:
- "<+workspace.aws_terraform.ecs_sg_id>"
- "<+workspace.aws_terraform.ecs_subnet_id>"
This are workspace variables that get their values from the output of the terraform provisioning pipeline
16. Save and clcik on "Apply selected"
17. Click on "+ Add Artifact Source"
18. Select "ECR" for the Artifact Repository Type
19. Choose the previously created AWS connector.
20. For the Artifact Source Identifier type in "ECR_artifact_source"
21. Choose the US West (N. California) region
22. For image path type in the name of your private repository, in this case "devops-harness" and click on the option that drops down.
23. Click on "Save"

## ECS Environment
1. Navigate to the "Environment" section and click on "+ New Environment".
2. Type in "ECS enviroment" for its name.
3. Choose "Pre-production" environment type.
4. Select the "Inline" option and click on "Save".
5. Click on "+ New Infrastructure"
6. Type in "ECS infrastructure" for its name
7. Choose the "Inline" option
8. In cluster details choose the "aws connector" we've been using throught the steps
9. Choose the US West (N. California) region
10. For the "Cluster" input paste the following line:
`<+workspace.aws_terraform.ecs_cluster_name>`
This will get the workspace variable from the terraform output

## ECS Execution
1. Choose "ECS Rolling Deployment" or your prefered type of deployment.