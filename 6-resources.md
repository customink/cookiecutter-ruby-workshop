[◀️ PREV: Resizer Code & Permissions](5-resizer-code.md)

# Next Steps

Here are some things I recommend doing to keep learning or digging into the AWS eco-system.

### 🚀 Automatic CI/CD

Did you get tired of doing `bin/deploy` all the time? Want to automate the process with GitHub Actions? This project includes a workflow to get your started. Open your project's README for steps on how to do that.

### HTTP Web Service

Use the HTTP option of the cookie cutter and make a small web service. This will help you learn API Gateways [HTTP API](https://aws.amazon.com/blogs/compute/announcing-http-apis-for-amazon-api-gateway/)

### Expire CloudWatch Logs

Set your CloudWatch logs to expire. That's right, you pay for them till fore ever. You can set them to expire past whatever makes sense for you.

### S3 CloudFormation Bucket

During the `bin/bootstrap` process we made a dynamic bucket name for you and stored it in the `.bucket-name` file. It is usually a good idea to use the same S3 bucket across different projects.

## Resources

Find your next steps and knowing where to look for help can be hard. Here are some places I reference when writing AWS Lambda code using SAM.

**[AWS Serverless Application Model (AWS SAM) Specification](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-specification.html)** - This is everything you need to know about `template.yaml` from property references to intrinsic functions. Remember, the Serverless Application Model (SAM) specification is a tightly focused and more succinct subset of CloudFormation to ease Lambda development.


**[AWS Resource and Property Types Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)** When passing AWS Resources around in your template it is helpful to know the return value of the `Ref` intrinsic function. Each resource is different. Sometimes it is a name, sometimes an ARN.


<br/>
<br/>

[⬆️ START OVER](README.md)

<br/>
<br/>
