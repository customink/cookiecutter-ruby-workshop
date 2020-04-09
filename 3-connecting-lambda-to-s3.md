[◀️ PREV: Setup & Deploy](2-setup-deploy.md)

# Step 3: Connecting Lambda to S3

Time to get familiar with the files in your AWS SAM project. In general, they are similar to how a Ruby gem is organized. There is a `lib` directory containing a file & folder each matching the name of your project. The Ruby file is the entry point for everything required in the directory of the same name. Also present is a `Gemfile` for any additional gem dependencies.

But how did we create our Resources in AWS during the deploy? Also, how does Lambda know where to invoke your Ruby code? 

## Template.yaml

This is the SAM specification to [CloudFormation](https://aws.amazon.com/cloudformation/) on how to build everything needed for your project. Think of it as syntactic sugar to make AWS Lambda development easier. 

❗️An important thing to remember is that [SAM Specification](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-specification.html) is a subset of CloudFormation. Any valid CloudFormation can also be used in this template.

When the deploy script was run, SAM examined your template file, found your code, packaged it into a compressed file, uploaded that file to an S3 bucket, and then instructed CloudFormation with a new packaged template.yaml file to build the needed AWS Resources for you. Take some time to examine your `template.yaml` file. Take note of the following:

- The `Resources` section contains everything you need created in AWS for this project.
- Your function's resources has a `Runtime` property set to Ruby 2.7. It also has memory and timeout settings.
- The `Handler` section of your function is how SAM knows where your code is. It will look for a `lib/image_service.rb` file with a method called `handler` inside of it.
- CloudFormation is actually a semi-dynamic language. It supports runtime reflection via a concept called [Pseudo Parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/pseudo-parameter-reference.html) as well as behaviors called [Intrinsic Functions](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html).

❓What bucket did SAM upload my code too? Great question! During the `bin/bootstrap` part of the process, we created a random bucket name for you. The name of which is stored in a `.bucket-name` file. 

## Adding Your S3 Resource

That was a lot of information above and SAM/CloudFormation certainly has a lot to offer. But lets start slowly and create our Lambda's S3 bucket so it can listen to events and eventually resize our images. Add the following `ImageServiceS3Bucket` to your `template.yaml` file under the `Resources` section. Since indentation is important, we have shown where in relation to the `ImageServiceLambda`. Order is not important though.

⚡️⏩⚡️

```yaml
Resources:
  ImageServiceLambda:
    # ...
  ImageServiceS3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub image-service-SOMEUNIQNAME-${StageEnv}
```

❗️S3 bucket names are global. So replace `SOMEUNIQNAME` above with your own unique name.

## Triggering Lambda Events

Remember how our Lambda had no events connected which required us manually trigger a test? Now that we have a S3 bucket resource declared we can instruct SAM/CloudFormation to trigger our Lambda from any [S3 Notification](https://docs.aws.amazon.com/AmazonS3/latest/dev/NotificationHowTo.html). We are going to choose any `ObjectCreated` event. SAM allows us to provide an `Events` property and this is where we are going to add our. See below for an example of where to add this under the `ImageServiceLambda` `Properties` section.

⚡️⏩⚡️

```yaml
ImageServiceLambda:
  Type: AWS::Serverless::Function
  Properties:
    # ...
    Events:
      ImageServiceS3BucketEvent:
        Type: S3
        Properties:
          Bucket: !Ref ImageServiceS3Bucket
          Events: s3:ObjectCreated:*
```

The `ImageServiceS3BucketEvent` is called a Logical ID. It is defined by us and can be whatever we want, just like `ImageServiceS3Bucket` above. 

Our S3 event does require two things. The name of the bucket and the events we want to trigger our Lambda. The `!Ref` syntax is one of those intrinsic functions I mentioned earlier. In this case we give it the Logical ID of the bucket resource in our template from the step above. This works because the return value of Ref for an S3 bucket is its name.

## Deploy Updated Stack

Time to deploy again. When doing so, watch the SAM output closely. It shows how CloudFormation detects a change to our stack, notices there are new resources, and which resources have to be modified. It happily sorts all this out and even figures out which order events need to happens to get the desired results. Pure magic! ✨

⚡️⏩⚡️

```shell
$ STAGE_ENV=production ./bin/deploy
```

## Test S3 Upload Triggers Lambda

<a href="RubyForGood.png"><img alt="Ruby For Good S3 Test Image" align="right" width="250" src="https://user-images.githubusercontent.com/2381/80309783-159a2a80-87a5-11ea-9c73-d036b683d245.png"></a>Now that our bucket has been created from our stack's deploy, let's upload an image and see if it triggers our Lambda. Here is a fun file called `RubyForGood.png` and we can upload it via the AWS Console.

- Click on "Services" in the toolbar.
- Enter "S3" into the find services field, select.
- Click on your S3 bucket named `image-service-SOMEUNIQNAME-production`. Where `SOMEUNIQNAME` is the name you chose earlier.
- Drag and drop this file into your browser window to start an upload dialog.
- Click "Next" 3 times to use the default values for permissions, etc. Finally click "Upload" to start the upload.

Afterward you should see your file uploaded to your S3 bucket. But did our Lambda get the event? Using the [CloudWatch steps](2-setup-deploy.md#cloudwatch-logs) from the previous chapter, navigate to your log group to see if your Lambda received the event and hence logged it. You should see something like this.

```json
{
  "Records": [
    {
      "eventVersion": "2.1",
      "eventSource": "aws:s3",
      "awsRegion": "us-east-1",
      "eventTime": "2020-04-26T23:40:00.273Z",
      "eventName": "ObjectCreated:Put",
      "userIdentity": {
        "principalId": "AWS:AIDAJIS4TINW4H5GRI3LG"
      },
      "requestParameters": {
        "sourceIPAddress": "72.228.219.152"
      },
      "responseElements": {
        "x-amz-request-id": "B1C967507AE19561",
        "x-amz-id-2": "CwSljyyyFADdm1Cyf06BsYkfWfoojX5CX7ezcT1Y9uWSF/L4mWGjz01cnPpGg+W+fiWEiCj2oQjTkELtfJFny5DlbB80UXkz"
      },
      "s3": {
        "s3SchemaVersion": "1.0",
        "configurationId": "8fe91e68-1782-47d8-93a1-470f89531673",
        "bucket": {
          "name": "image-service-SOMEUNIQNAME-production",
          "ownerIdentity": {
            "principalId": "A4ILFJ7OF8X06"
          },
          "arn": "arn:aws:s3:::image-service-SOMEUNIQNAME-production"
        },
        "object": {
          "key": "RubyForGood.png",
          "size": 659624,
          "eTag": "a2028b43a11dd778720371ceaf53dde5",
          "sequencer": "005EA59B740BD1E80F"
        }
      }
    }
  ]
}
```

In fact this event could be useful to our eventual test suite. I recommend adding it to your project in the `test/events/s3-put.json` file. Later we can use it for some eventual test-driven development.

## 🚀 CI/CD

This feels like another good time to save our work.

⚡️⏩⚡️

```shell
$ git add .
$ git commit -m "Connecting Lambda to S3"
$ git push
```

<br/>
<br/>

[▶️ NEXT: Image Processing & Lambda Layers](4-image-processing.md)

<br/>
<br/>
