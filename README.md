# CountAPI-Serverless
CountAPI deployed on AWS Lambda, Elasticache, and API Gateway

### Whats the difference between this repo and [CountAPI](https://github.com/philip306/countapi)

This repo is specific for deploying this app to AWS Lambda and Elasticache.  [Mangum](https://github.com/jordaneremieff/mangum) is used to wrap the app as the handler for Lambda.  There seems to be some issue with mangum and lambda runtime 3.7, so you'll see later a 3.6 runtime specified for the lambda runtime.  Will update if i find the cause.  This repo contains no images for a redis database. Elasticache is used here instead.

## Prerequisites

You'll need to have an AWS account with a VPC containing multiple subnets.  You'll also need the ***aws-cli*** if you want to use some of the commands below.  Optionally, you could use AWS SAM or zappa to automate this process even further, but this deployment is simple enough to where the install for SAM would take longer than doing it manually.  You choose. 

### Elasticache Setup
* Using either the AWS console or ***aws-cli*** to create a redis instance. For testing purposes a t2.micro is more than enough for this app
* Take note of the VPC used and the URL for you redis instance.  You'll need these later
* Make sure the security group you use allows for http traffic to your app and traffic on the redis port is allowed within the VPC

### Create an IAM Role for Lambda

* In IAM create a new roll "fastapiLambdaRole"
* Attach the ***AWSLambdaBaswicExecutionRole*** to the policy

### Create a security group

* Create a security group with a rule that allows traffic within you VPC over the redis port(6379).  
* Create another rule that allows http traffic from wherever you intend to access the app from. 

### Clone this repo and update

* Clone this repro locally 
```
git clone https://github.com/philip306/CountAPI-Serverless
```
* update config.py with the url to your redis elasticache host you created above. 


### Package and Deploy
* install the python requirements to the local directory so they can be packaged for lambda
```
pip install -r requirements.txt -t .
```
* Zip the current directory(Update this command to exclude any other direcotries or files that aren't necessary for the app to function. I've included common git files and virtual env dirs in the command below)
```
zip -r function.zip * -x ".git" ".gitignore" venv/\*
```
* Deploy the zip file to lambda using the ***aws-cli*** using the arn fro the role you created and your VPC subnetIds and security group Ids
```
aws lambda create-function --function-name CountAPI --timeout 30 --memory-size 1024 \
--zip-file fileb://function.zip --handler app.main.handler --runtime python3.6 \
--role arn:aws:iam::[youraccount#]:role/fastapiLambdaRole \
--vpc-config SubnetIds=[subnetId],SecurityGroupIds=[security group Id]
```

* Create the HTTP api gateway to point to your Lambda function.  I did this via the console, but it would be just as easy via aws-cli
* Make sure to add the Lamda you created as an integration to your HTTP api gateway
* For routes, allow any method to the resource path ***/{proxy+}***. This will pass everythin to the app and allow it to handle http response codes. 

Once this is complete you should be able to access CountAPI by navigating to the URL provided after the creation of your API gateway. 
