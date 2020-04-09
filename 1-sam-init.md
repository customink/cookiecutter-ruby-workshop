[◀️ PREV: Preparation](README.md)

# Step 1: New Project with SAM Init

Most developers are familiar with the [Serverless](https://serverless.com) framework since it was first on the scene. Think of it as the ActiveRecord adapter for every cloud provider's "serverless" offering. It is a higher order abstraction that is powered by community plugins.

To ease the development cycle for their own Lambda offering, AWS developed a tool called 🐿 SAM which is short for the [Serverless Application Model](https://aws.amazon.com/serverless/sam/). SAM encompasses both the name of the YAML syntax to express your function & the command line tooling to run, build, and deploy it.

❓Why use SAM vs. Serverless? Both tools are open source. However, I recommend SAM if you are new to serverless and using AWS. Learning too many layers of abstraction can be frustrating. So stick with SAM and learn it before move higher up, if ever needed at all.

## SAM Init

To start a new SAM project, we are going to use a Docker container to run [`sam init`](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-init.html) which leverages this Custom Ink [Cookiecutter Project Template](https://technology.customink.com/blog/2020/03/13/using-aws-sam-cookiecutter-project-templates-to-kickstart-your-ambda-projects-copy/) to kickstart a new project folder for you.

⚡️⏩⚡️

```shell
$ docker run \
  --interactive \
  --volume "${PWD}:/var/task:delegated" \
  lambci/lambda:build-ruby2.7 \
  sam init --location "gh:customink/cookiecutter-ruby"
```

It will prompt you for a project name and ask if your Lambda needs an HTTP API. Use **"image_service"** for this name. Also we do NOT want HTTP events for this Lambda, so enter **"2"** for the second question.

- project_name [my_awesome_lambda]: **image_service**
- Select http_api 1 - yes 2 - no: 2️⃣

Now change to your newly created project directory. All other commands assume you are in this working directory.

⚡️⏩⚡️

```shell
$ cd image_service
```

## 🚀 CI/CD

Assuming you are using git and have created a GitHub repo while working on this project, create your first commit and push. Remember, make a new repo in GitHub first.

⚡️⏩⚡️

```shell
$ git init && git add .
$ git commit -m "Initial commit"
$ git remote add origin git@github.com:YOURUSERNAME/image_service.git
$ git push -u origin master
```

❗️Replace `YOURUSERNAME` with your own GitHub username.

## What Happened?

Your new SAM project folder has everything you need for your new Ruby Lambda. Take some time to explore its contents. Specifically the SAM `template.yaml` file which describes your Lambda function. That file designates a single handler method within `lib/image_service.rb` that receives events. From a higher level, here is what has been generated in your new project.

- Docker setup using both a `Dockerfile` and `docker-compose`.
- A working Ruby project with a lib directory, bundler, and tests.
- SAM fully integrated into all setup and test scripts.

All of which you will have a chance to become familiar with throughout the workshop.

<br/>
<br/>

[▶️ NEXT: Setup & Deploy](2-setup-deploy.md)

<br/>
<br/>
