---
layout: post
title: Setting up AWS API Gateway using Chalice, SQS and Celery
date: 2018-01-31 05:35:48
categories: aws api gateway chalice sqs celery python lambda linux
---
### End Goal
To have an api that sends SQS messages to a queue to be processed by workers. 

### Flow
api request -> AWS API Gateaway -> SQS -> Celery

#### Why?
Why manage an api for yourself when chalice and AWS can handle it quite easily for you. Yes the cloud is someone else's computer, but I don't want to be woken up because a server is down. I would rather be notified because a behemouth like Amazon is down. 

### Setup

#### Create Chalice Appliction
I use python3 for everything. 3 > 2 so python3 is obviously better. We will also use the default project names. Keeps it simple. A lot taken from their README
* Create virtualenv for program
{% highlight bash %}
python3.6 -m venv chalice-demo
cd chalice-demo
. ./bin/actiavate
pip install chalice==1.1.0
pip install celery==4.1.0
{% endhighlight %}

Since Chalice is for Amazon, you need to setup your Amazon credentials

{% highlight bash %}
$ mkdir ~/.aws
$ cat >> ~/.aws/config
[default]
aws_access_key_id=YOUR_ACCESS_KEY_HERE
aws_secret_access_key=YOUR_SECRET_ACCESS_KEY
region=YOUR_REGION (such as us-west-2, us-west-1, etc)
{% endhighlight %}

{% highlight bash %}
chalice new-project chalice_project
cd chalice_project
{% endhighlight %}

Then create app.py inside the chalice_project directory with the following contents:
{% highlight python %}
from chalice import Chalice, IAMAuthorizer
import json
import boto3
import base64
from celery import Celery


broker_url='sqs://<user api key id>:<user api secret key>@'
app = Chalice(app_name='chalice_project')

authorizer = IAMAuthorizer()

@app.route('/', api_key_required=True, authorizer=authorizer)
def index():
    return {'hello': 'world'}

@app.route('/sendtosns', api_key_required=True, authorizer=authorizer)
def sendto_sqs():

    sqs = boto3.resource('sqs')
    # assuming you are using a fifo queue
    queue_url = 'https://sqs.us-east-1.amazonaws.com/<aws account id>/<queue name>.fifo'
    
    try:
        celery=Celery(broker=broker_url)
        celery.conf.task_default_queue ='<queue name>.fifo'
        celery.conf.update(CELERY_ACCEPT_CONTENT=["json"],
                CELERY_TASK_SERIALIZER="json",
                CELERY_RESULT_SERIALIZER="json",
                CELERY_DEFAULT_QUEUE='environment_queue.fifo')

        celery.send_task("tasks.add", (2,2))
    except Exception as e:
        print(e)
        return '{"error occurred":"check cloudwatch logs"}'

    return '{"message_status":"success"}'
    
{% endhighlight %}

Also add the following to requirements.txt

{% highlight bash %}
celery==4.1.0
{% endhighlight %}

Notice a few things here. We need to create a SQS queue, and we need our aws credentials. For testing that is fine to leave those in there, but I would HIGHLY recommend using AWS KMS Keys for the data. This is just to get things working.

You can also use your AWS "master" account, but below I will show the polcies that I used to make it work.

Go to your aws dashboard (or use cloudformation) and make your SQS FIFO queue.

Now it is time to deploy your chalice app!

{% highlight bash %}
chalice deploy
{% endhighlight %}

Any credentials or permissions you are missing for that user are made clear here. If you are using your master account, it should work just fine. It is creating the lambda, creating permissions, and making the api gateay. Really, it is setting up the API Gateway -> Lambda function method execution for you. Anything I do not have to do manually is a win. 

In addition, it uploads any depencies you installed via pip, amazing!

Something else to notice as well, we added 'api_key_required=True, authorizer=authorizer' to the constructor, so yes, we will need to auth, TWICE! Why not, security rocks. You can remove those from the constructor and have it be unauthenticated, but whats the fun in that!

#### Adding api key to endpoint 
You can probably do this with cloudformation or aws api, but for this tutorial we will use the console

* In your AWS Console (web interface) go to Services and choose API Gateway
* Go to API Keys
* Under actions, select "Create API Key"
* Give it a name. I let is autogenerate but if I am very paranoid I will generate my own.
* Hit Save
* Go to  "Usage Plans"
* Hit Create
* Give it a name (Basic is fine for this demo)
  * Usage plans let you rate limit, throttle, etc... based on api key
* After hitting save, select your newly created Usage Plan
* Select "Add API Stage"
  * An API Stage lets you deploy code to different "Stages". You could have a stage for dev, pre-prod, prod, hamster, etc.. 
  * If you wanted a new stage, its as easy as "chalice deploy --stage penguin"
* Under api you should have your api selected
* Under stage, if its default you can enter in api, or choose whatever stage you want it to be.
* Hit that tiny checkmark beside it

There, now you have an api key to hit this API. We also chose IAM authentication. That means we have to use AWS Signature Version 4 singing. Don't worry, smart people already made a python module for this.
{% highlight bash %}
pip install aws-requests-auth==0.4.1
pip install requests==2.18.4
{% endhighlight %}
And then to test the api, we have this snazzy little python program. Let's name is snazzy.py

{% highlight python %}
import requests
from aws_requests_auth.aws_auth import AWSRequestsAuth

auth = AWSRequestsAuth(aws_access_key='<your aws access key id>',
                       aws_secret_access_key='<your aws access secrey key>,
                       aws_host='123456abcde.execute-api.us-east-1.amazonaws.com',
                       aws_region='us-east-1',
                       aws_service='execute-api')
headers = {'x-api-key': '<api key you generated>'}
response = requests.get('https://123456abcde.execute-api.us-east-1.amazonaws.com/api/sendtosqs',
                          auth=auth, headers=headers)
print(response.content)

{% endhighlight %}
You can get the api path by going to your api, selecting stages, your stage, then the url at the top is how to get to your url (also the end of chalice deploy tells you what it is as well)

Now, if things are good, we are almost done. We need to enable cloudwatch logs for the api.

[ AWS Documentation](https://aws.amazon.com/premiumsupport/knowledge-center/api-gateway-cloudwatch-logs/). This is where I got my info.

* Under services, select IAM
* Select Roles
* You should see something like chalice_project-dev, select that
* Under trust relationship, select "Edit trust relationship"
* It should look something like this:
{% highlight json %}
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    },
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Service": "apigateway.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
{% endhighlight %}
* Select "Update Trust Policy"
* Back at the policy summary for chalie_project-dev, select "Attach Policy"
* Choose "AmazonAPIGatewayPushToCloudWatchLogs" and select "Attach Policy"
* Back at the policy summary page, copy the Role ARN
* Go to services and select API Gateay
* Select your API
* Select Stages
* Select your stage (api is default)
* Under logs select "Enable Cloudwatch logs" and hit save changes
* Under settings on the left hand side, under "Cloudwatch log role ARN", paste the ARN you copied earlier and hit save
If all goes well, there should be no errors, and now your application is logging to cloudwatch!

Now you can test your api, and look at cloudwatch to see if there are any issues. Just run:
{% highlight bash %}
python3.6 snazzy.py
{% endhighlight %}
It should hit the sendtosqs endpoint and put a message on the queue! WOW!

### Celery setup
* Create another virtual environment somewhere
{% highlight bash %}
python3.6 -m venv workers
cd workers
. ./bin/activate
pip install celery==4.1.0
export PYCURL_SSL_LIBRARY=nss
pip install pycurl --no-cache-dir
pip install pycurl
{% endhighlight %}
* NOTE: there may be funny things with pycurl. This is the worst part of everything. YMMV, may have to use google to figure out why it wont install. Again, so stupid.
* Create a file called tasks.py
{% highlight python %}
from celery import Celery
import uuid
import time
import random
broker_url='sqs://<aws key id>:<aws secret key>@'

broker_transport_options = {'queue_name_prefix': '<queue_name>.fifo'}
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
app = Celery('tasks', broker=broker_url, broker_transport_options=broker_transport_options )
app.conf.task_default_queue ='<queue_name>.fifo'
app.conf.update(CELERY_ACCEPT_CONTENT=["json"],
                CELERY_TASK_SERIALIZER="json",
                CELERY_RESULT_SERIALIZER="json",
                CELERY_DEFAULT_QUEUE='<queue_name>.fifo')

@app.task(serializer='json')
def add(message,duo):
    my_uuid = uuid.uuid4()
    sleep_time = random.randint(1,10)
    print('I got a task, and my uuid is: {} and I am sleeping {}'.format(my_uuid, sleep_time))
    time.sleep(sleep_time)
    print("My uuuid is %s and I am done sleeping" % my_uuid)
    print(message)
    return message

{% endhighlight %}

The point here is not to do actual work, its to show that it is reading SQS messages, and executing child tasks that are run in paralell.

Run the program with:
{% highlight bash %}
celery -A tasks worker --loglevel=info --concurrency=10
{% endhighlight %}

If thinks work right, you should see things coming on in!



### Permissions
If you do everything with an account that has full access to everything, there should be zero issues. However, that is NOT how you want to do it in production.
* Here is the policy json that worked for me to have a user I create have access to deploy, update and delete a chalice app.
{% highlight json %}
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "lambda:CreateFunction",
                "apigateway:DELETE",
                "lambda:ListTags",
                "apigateway:PUT",
                "lambda:UpdateFunctionConfiguration",
                "apigateway:POST",
                "sqs:*",
                "apigateway:GET"
            ],
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "iam:GetRole",
                "lambda:GetFunction",
                "iam:CreateRole",
                "iam:DeleteRole",
                "lambda:GetFunctionConfiguration",
                "iam:PutRolePolicy",
                "lambda:UpdateFunctionCode",
                "iam:PassRole",
                "lambda:AddPermission",
                "iam:DeleteRolePolicy",
                "lambda:DeleteFunction",
                "iam:ListRolePolicies",
                "lambda:GetPolicy"
            ],
            "Resource": [
                "arn:aws:iam::<account id>:role/chalice_project-dev",
                "arn:aws:lambda:us-east-1:<account id>:function:chalice_project-dev"
            ]
        }
    ]
}
{% endhighlight %}

#### API user
The nice thing if you are using IAM authentication, you can make a user that can only hit certain endpoints. (There are OAUTH and custom authenticators you can use with API Gateway)
* I created a user, then a group to contain that user. I then created then attached the following policy:
{% highlight json %}
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "execute-api:Invoke",
            "Resource": "arn:aws:execute-api:us-east-1:<account id>:12345abcde/api/GET/sendtosqs"
        }
    ]
}
{% endhighlight %}
* This user could only hit the sendtosns endpoint using Get. 
