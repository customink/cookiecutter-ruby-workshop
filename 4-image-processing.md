[◀️ PREV: Connecting Lambda to S3](3-connecting-lambda-to-s3.md)

# Step 4: Image Processing & Lambda Layers

In order to resize images we need to select a Ruby gem to do the work for us. Thankfully Rails [ActiveStorage](https://edgeapi.rubyonrails.org/classes/ActiveStorage/Variant.html) has elected a wonderful gem called [ImageProcessing](https://github.com/janko/image_processing) that leverages either ImageMagick or Libvips to resize images. 

Add this to your `Gemfile`.

⚡️⏩⚡️

```ruby
gem 'image_processing'
```

## Adding Libvips

When talking about Lambda we often refer to it as "serverless" when in fact there are technically... servers involved. Specifically they are [Micro VMs](https://firecracker-microvm.github.io) that execute your code. You can think of them as managed containers which we refer to as [Runtimes](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html). If you look in your project's `template.yaml` file you would see that our function's resource `Runtime` property is set to `ruby2.7`. As the name suggests, these Micro VMs only have the bare minimal software to perform their needed task, for example, having Ruby installed. This ensures they are small and execute our code fast. 

Often times Ruby gems with native C extensions will need headers or shared objects to compile or link to. ImageMagick and Libvips both require other shared objects. None of which are pre-installed on AWS Lambda's modern Runtimes. That work is left to you using a technique called [Lambda Layers](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html). The most simple explanation of how Layers work is they add code to your runtime's `/opt` directory. We are going to use a Lambda layer that adds [Libvips](http://libvips.github.io/libvips/) to our runtime in a way that allows the ImageProcessing gem to use it.

Add this public Libvips layer resource [ARN](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html) under the functions's `Properties` section in your `template.yaml` file.

⚡️⏩⚡️

```yaml
ImageServiceLambda:
  Type: AWS::Serverless::Function
  Properties:
    # ...
    Layers:
      - arn:aws:lambda:us-east-1:589405201853:layer:rubyvips892-27:13
```

❓Can I learn more about how this Libvips layer was made? Yes, here is the [ruby-vips-lambda](https://github.com/customink/ruby-vips-lambda) project used to make the layer. Also the [Yumda](https://github.com/lambci/yumda) project is a great way for other common layer use cases.

## Permissions

Remember when we [first deployed](2-setup-deploy.md) our Lambda and looked at the CloudFormation stack in the AWS Console? Within the Resources tab was a type of `AWS::IAM::Role`. SAM made this role for us automatically. When invoked, our Lambda executes with any managed or inline policies attached to this role. If our Lambda is going to read & write to an S3 bucket, it needs permission to do so. Thankfully this is super easy with the AWS SAM framework using the `Policies` property.

Any valid AWS [Identity & Access Management](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html) statement can be expressed in this section as a collection. Each one will be attached to your Lambda function's role. Here is a simple one that states this Lambda can perform any operation on the S3 bucket it owns.

⚡️⏩⚡️

```yaml
ImageServiceLambda:
  Type: AWS::Serverless::Function
  Properties:
    # ...
    Policies:
      - Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Action:
              - s3:*
            Resource:
              - !Sub arn:aws:s3:::image-service-SOMEUNIQNAME-${StageEnv}
              - !Sub arn:aws:s3:::image-service-SOMEUNIQNAME-${StageEnv}/*
```

❗️S3 bucket names are global. So replace `SOMEUNIQNAME` above with your own unique name. Make sure it matches from the name you used in the `ImageServiceS3Bucket` resource.

## 🚀 CI/CD

We are not yet finished with all the code needed to read and write to our S3 bucket. So no need to commit our work now.


<br/>
<br/>

[▶️ NEXT: Resizer Code & Permissions](5-resizer-code.md)

<br/>
<br/>
