# Cookiecutter Ruby - Microservice Workshop

Interested in learning AWS Lambda using Ruby? This workshop leverages Docker & [AWS SAM Cookiecutter templates](https://technology.customink.com/blog/2020/03/13/using-aws-sam-cookiecutter-project-templates-to-kickstart-your-ambda-projects-copy/) to provide a fast-paced curriculum with little configuration fuss. Expect to learn the following while making a Lambda that responds to S3 events and resize source images.

- Introductory concepts of [AWS Lambda](https://aws.amazon.com/lambda/).
- Using [AWS SAM CLI](https://aws.amazon.com/serverless/sam/) to locally develop & deploy Lambda.
- Practicing Infrastructure as Code (IaC) via [CloudFormation](https://aws.amazon.com/cloudformation/).

## Workshop Steps

Want to go fast, make excellence, and read later? As you follow each section, find code sections with the ⚡️⏩⚡️symbols before them, type them in, and breeze thru the workshop or play with the code. However, do take time to read the notes for other useful learnings.

1. [New Project with SAM Init](1-sam-init.md)
2. [Setup & Deploy](2-setup-deploy.md)
3. [Connecting Lambda to S3](3-connecting-lambda-to-s3.md)
4. [Image Processing & Lambda Layers](4-image-processing.md)
5. [Resizer Code & Permissions](5-resizer-code.md)
6. [Next Steps & Resources](6-resources.md)

## Preparation

#### 🚢 Install Docker

AWS SAM uses Docker to simulate the Lambda Runtime environment. We also use Docker to avoid installing both the AWS CLI & SAM CLI. Both can be problematic to install since each use Python. No worries, Docker will make all this easy and it is [super easy to install](https://docs.docker.com/get-docker/). Is it working?

```shell
$ docker --version
Docker version 19.03.8, build afacb8b
```

#### ⛅️ AWS Account

We are after all, in the cloud, from this point on - so you are going to need an AWS Account. [Creating one](https://aws.amazon.com/resources/create-account/) is pretty straightforward. Afterward you will need to configure programatic access using your account's `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`. If you do not have these already, you can create them by doing the following within the [AWS Management Console](https://console.aws.amazon.com/console/home?region=us-east-1).

- Click on "Services" in the toolbar.
- Enter "IAM" into the find services field, select.
- Click on "Users" from the left hand navigation, select your username.
- Click the "Security credentials" tab.
- Click the "Create access key" button.
- Copy your key id and secret to a secure place.

From here you need to configure CLI programatic access. This is easy to do with the AWS CLI Docker container.

```shell
$ docker run \
  --interactive \
  --tty \
  --rm \
  --volume "${HOME}/.aws:/root/.aws" \
  "amazon/aws-cli" \
  configure
```

When prompted, paste in your key id and secret key from the steps above. I recommend using `us-east-1` as the default region and `json` as the output format. To make all `aws` CLI commands easier, I recommend using the following alias.

```shell
alias aws='docker run --rm -it --tty -v "${HOME}/.aws:/root/.aws" -v "${PWD}:/aws" "amazon/aws-cli"'
```

Alternatively, you can install AWS CLI & SAM via Homebrew.

```shell
$ brew install awscli
$ brew tap aws/tap
$ brew install aws-sam-cli
```

Assuming you have run `configure` and set the alias. Is it working?

```shell
$ aws s3 ls
```

#### 🚀 CI/CD

This part is optional. But if you have `git` installed and are using GitHub, committing our work along the way is both fun and makes code exploration safe. If you have neither, that is fine and you can safely skip these steps.

But if you do have a GitHub account, our starter project includes a [GitHub Action Workflow](https://technology.customink.com/blog/2019/09/02/from-travis-ci-to-github-actions/) to help you practice Continuous Integration & Continuous Delivery (CI/CD) in a fully automated way. More on that later.
