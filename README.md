# WildRydes
Project : Serverless Web Application -AWS

Introduction: Wildrydes is a innovative project which makes the life simpler by choosing the fastest mode of transport.

AWS Services Used:
. Amplify
. Lambda
. DynamoDB
. IAM
. API Gateway
. CodeCommit

I would like to share step by step instructions,

Step1: Pull the source code and push it new repository.

* Select the region in which you want to work on.
* Create an empty repository.
* Add policy to IAM user so that I can use codecommit.
* Create Git Credentials to the IAM User to allow HTTPS connections to codecommit.
* Clone the repository(create an empty folder for future code)
* Copied the project code from aws s3 cp s3://ttt-wildrydes/wildrydes-site ./ --recursive because AWS locked or deleted the source code from their s3 bucket and then commit 
  it to new repo.

Step2: Host website and make updates to it.

* Goto Amplify ,select aws codecommit and select continue.
* Select the repository wildrydes-site from the drop down menu.
* On the Build settings page, leave all the defaults, select Allow AWS Amplify to automatically deploy all files hosted in the project root directory and choose Next.
* On the Review page select Save and deploy.
* The process took nearly 2-3 minutes for Amplify Console to create the necessary resources and to deploy the code.
* Once completed,select the link for master you'll see the build and deployment details related to the branch.

  ![Screenshot 2024-05-15 115430](https://github.com/RoginiMano/WildRydes/assets/164807520/f1a48995-3ec0-4e9f-b34b-b7a0081ea8ea)



