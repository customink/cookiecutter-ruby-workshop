[◀️ PREV: New Project with SAM Init](1-sam-init.md)

# Step 2: Setup & Deploy

In its current state, the only thing your Lambda does is log whichever event triggers it. Rather than wait for a big release, this is a great time for your first deploy to get familiar with the process. First, we need to setup your project's dependencies and run the tests. These commands are somewhat standard scripts to perform a one-time bootstrap or repeatable setups. For example, if you change dependencies in your Gemfile.

⚡️⏩⚡️

```shell
$ ./bin/bootstrap
$ ./bin/setup
$ ./bin/test
```

If your tests ran fine, you should see output similar to this.

```
# Running:
.
Finished in 0.000992s, 1008.4712 runs/s, 1008.4712 assertions/s.
1 runs, 1 assertions, 0 failures, 0 errors, 0 skips
```

## Your First Deploy

⚡️⏩⚡️

```shell
$ STAGE_ENV=production ./bin/deploy
```

This will use AWS SAM's build, package and deploy CLI commands. Assuming everything went smoothly, you should see output for SAM's CloudFormation deployment tasks ending with something like this. Take some time to examine the full output too.

```
CloudFormation outputs from deployed stack
---------------------------------------------------------------------------------------------
Outputs
---------------------------------------------------------------------------------------------
Key                 ImageServiceLambdaArn
Description         Lambda Function Arn
Value               arn:aws:lambda:us-east-1:012345678912:function:image-service-production
---------------------------------------------------------------------------------------------

Successfully created/updated stack - image-service-production in us-east-1
```

💥Were there problems? Did you make sure to run the `bin/bootstrap` script? It should have made an S3 bucket to store your packaged artifacts. Check that there is a `.bucket-name` file and that bucket exists in your AWS account.

## Invoking Your Lambda

Even without any event handlers, we can still invoke and test that your Lambda is deployed successfully. Log into the [AWS Management Console](https://console.aws.amazon.com/console/home?region=us-east-1):

- Click on "Services" in the toolbar.
- Enter "CloudFormation" into the find services field, select.

From this page you will see your newly deployed image service Lambda listed as a "stack". A stack is a collection of AWS Resources for a project. 

![Your CloudFormation Ruby Stack](https://user-images.githubusercontent.com/2381/80288363-8ab62300-8705-11ea-8725-46fd8c0ec6d4.png)

Click into your stack and click on the "Resources Tab". Your Lambda project only has two resources, the Lambda itself and the IAM Role which it executes under.

- Click the "image-service-productions" Lambda resource under the "Physical ID" column. This will take you to your Lambda within the AWS Console.
- Click on the "Test" button the in the upper right.
- Use the "Hello World" event template.
- Give it a name of "HelloWorld".   
- Click the "Create" button.
- Now click the "Test" button to invoke your Lambda

<img width="1161" alt="Test your Image Service Lambda" src="https://user-images.githubusercontent.com/2381/80289138-c43d5d00-870a-11ea-9798-fa42f96b3851.png">

### 🎉 Congratulations! 

Your new Lambda is working. From the test invoke we just performed, you can see the return value, a Ruby hash containing a status code, an array of headers, and a response body. In this case, the Lambda simply returns the same event the it received as a JSON string. This is the handler method in your project.

```ruby
def handler(event:, context:)
  puts event
  { statusCode: 200,
    headers: [{'Content-Type' => 'application/json'}],
    body: JSON.dump(event) }
end
```

## CloudWatch Logs

So what about that `puts` call in our Ruby handler? Where is standard out written too? How about logging to files? Other than a limited amount of `/tmp` space, you can not write to the filesystem in Lambda. All standard out (logging) is capture by CloudWatch and written to log groups for you. The service is 100% managed. To view the log of our test invocation.

- Click on "Services" in the toolbar.
- Enter "CloudWatch" into the find services field, select.
- Click "Log Groups" from the left hand menu.
- Find the log group named "/aws/lambda/image-service-production" and select it.

From here, CloudWatch organizes logs into streams organized by dates. Click one of these to view logs from our test invocation.

```
START RequestId: 0d6c34b2-42f0-439f-9f65-c35050e8da3e Version: $LATEST
{"key1"=>"value1", "key2"=>"value2", "key3"=>"value3"}
END RequestId: 0d6c34b2-42f0-439f-9f65-c35050e8da3e
REPORT RequestId: 0d6c34b2-42f0-439f-9f65-c35050e8da3e	Duration: 2.93 ms	Billed Duration: 100 ms	Memory Size: 512 MB	Max Memory Used: 49 MB	Init Duration: 255.06 ms	
```

❗️When things go wrong, digging into CloudWatch logs is one of the first places you should go looking to see what happened. 

## 🚀 CI/CD

Because SAM bundled and packaged our Lambda, a newly added `Gemfile.lock` has appeared in our project ensuring our gems are locked down. We can commit this change before moving on.

⚡️⏩⚡️

```shell
$ git add .
$ git commit -m "Add Gemfile.lock"
$ git push
```

<br/>
<br/>

[▶️ NEXT: Connecting Lambda to S3](3-connecting-lambda-to-s3.md)

<br/>
<br/>
