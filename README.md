# serverless CI/CD pipelines for docker images with AWS
When you look at AWS services like CodeBuild, CodeCommit, CodePipeline and ECR, you would think it is very easy to
create a simple CI/CD build pipeline for a docker image. But it is not. In this repository you will find a CloudFormation 
template which creates a serverless CI/CD pipeline for Docker images. The template allows you to create CI/CD pipelines
for Docker images in minutes!

<!--more-->
### How do I do it?
It is simple, you just download the CloudFormation template:

```sh
git clone https://github.com/binxio/blog-serverless-docker-image-ci-cd-with-aws.git
cd blog-serverless-docker-image-ci-cd-with-aws
```
and deploy it:
```sh
aws --region eu-central-1 \
          cloudformation create-stack \
       --stack-name paas-monitor-ci-cd \
       --template-body file://./serverless-docker-image-ci-cd.yaml \
       --capabilities CAPABILITIES_IAM

aws cloudformation wait stack-create-complete --stack-name paas-monitor-ci-cd
```
This creates:
- a git repository named `paas-monitor`
- a Docker image repository named `mvanholsteijn/paas-monitor`
- a build project named `paas-monitor`
- an IAM policy controlling access of the build project
- a lambda which is starts the project on repository commits 

### cloning the source repository
To see this in action, you are going to push our [paas-monitor](https://github.com/mvanholsteijn/paas-monitor.git) repository
to the newly created CodeCommit repository:
```sh
git clone https://github.com/mvanholsteijn/paas-monitor.git
cd paas-monitor
```
In this repository you will find a `.buildspec.yaml` which tells CodeBuild
how to build this image.

### installing the git remote helper
Next you install the [git remote helper for CodeCommit](https://github.com/awslabs/git-remote-codecommit) which allows you to push with our AWS credentials: No ssh keys needed!
```sh
pip install git-remote-codecommit
```
This remote helper, allows you to use `codecommit` as a protocol in a git
repository url. A full CodeCommit git url has the following format: `codecommit:<profile>/<name>`,
where `profile` is the AWS profile which contains the credentials to use and `name` the name of 
the intended repository.

### pushing to remote
Now, you can push the git repository to the CodeCommit repository:
```sh
git remote add aws codecommit:${AWS_PROFILE:-default}/paas-monitor
git push aws --tags
git push aws
```
In order to view the build process, go to the [CodeBuild console](https://eu-central-1.console.aws.amazon.com/codesuite/codebuild/projects/paas-monitor/history).

### Reuse
If you want to reuse this template for your own build pipelines, just specify your repository name and user when creating 
the stack:

```sh
read -p 'repository name: ' REPOSITORY_NAME
read -p 'repository user: ' REPOSITORY_USER

aws cloudformation create-stack \
       --stack-name ${REPOSITORY_NAME}-ci-cd \
       --template-body file://./serverless-docker-image-ci-cd.yaml \
       --capabilities CAPABILITIES_IAM \
       --parameters ParameterKey=RepositoryName,ParameterValue=${REPOSITORY_NAME} \
                     ParameterKey=RepositoryUser,ParameterValue=${REPOSITORY_USER}

```
## Conclusion
With this CloudFormation template we combined AWS CodeBuild, CodeCommit, Lambda and ECR to create a serverless CI/CD pipeline for Docker images. 
The same template can be reused to create pipelines for other images. The same setup can also be used to build other artifacts based on a git
repository or deploy stuff to AWS. It is not simple, but it completely serverless!
