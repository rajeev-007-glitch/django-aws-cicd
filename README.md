# Django-CICD-Pipeline-with-GitHub-Actions-and-AWS

## How to Deploy Django app using GitHub Actions and AWS

### Solution Overview

    1.	GitHub Actions: Workflow Orchestration tool that will host the Pipeline.
    2.	IAM OIDC identity provider: Federated authentication service to establish trust between GitHub and AWS to allow GitHub Actions to deploy on AWS without maintaining AWS Secrets and credentials.
    3.	Amazon S3: Amazon S3 to store the deployment artifacts.
    4.	AWS Elastic Beanstalk: AWS Elastic Beanstalk is an easy-to-use service for deploying and scaling web applications and services developed with Java, .NET, PHP, Node.js, Python, Ruby, Go, and Docker on familiar servers such as Apache, Nginx, Passenger, and IIS.

### Prerequisites

    1.	An AWS account with permissions to create the necessary resources.
    2.	A Git Client clone the provided source code. In our case we are cloning from following link ( https://github.com/vinDelphini/django-aws_cicd.git ). Tutorial project for Django.
    3.	A GitHub Account with permissions to configure GitHub repositories, create workflows, and configure GitHub secrets.

### Walkthrough

#### The following steps provide a high-level overview of the walkthrough:

    1.	Clone the project from the provided git project link.
    2.	Creating an IAM Role with a full access of S3 & Beanstalk.
    3.	Creating S3 Bucket to store Source Code.
    4.	Setting our Elastic Beanstalk environment.
    5.	Setup GitHub secrets.
    6.	Build CI/CD Pipeline in GitHub Action to build and deploy the code.
    7.	Trigger the GitHub Action to build and deploy the code.
    8.	Verify the deployment.
    9.	Clean up: To avoid incurring future changes, you should clean up the resources that you created.

### Clone GitHub repo

    Open your chosen command-line interface and clone the repository with the below command to get the project files locally.

```
    git clone https://github.com/vinDelphini/django-aws_cicd.git
```

### IAM user access

    Search for IAM users in the AWS console and select [Add user].

### S3 configuration

### Setting our Elastic Beanstalk environment.

    In the Beanstalk dashboard, make sure you have selected the most suitable region (where services to be deployed) in the right top corner. Check [Environments] and choose [Create a new environment]. Then, on the dialogue page, choose [Web server environment].

    Fill in the details and focus on Platform section. Here we need to select Python platform (last version) and last Amazon Linux 2 available.

    Leave Sample application and hit [Create environment]. In a few moments, you should be able to open your environment URL address and see a sample page.

### Setup GitHub secrets

    As we save Slack Webhook URL in our github secrets. Settings->Secrets under the name. Now, also we need to save our AWS credentials (Access key ID, Secret access key) in github secrets. Settings->Secrets under the name MY_AWS_ACCESS_KEY as well as MY_AWS_SECRET_KEY

### Build CI/CD Pipeline in GitHub Action to build and deploy the code

    Now for creating CI/CD Pipeline in GitHub Action firstly we need to setup a workflow by our self as per shown in image Let's create our first workflow that will contain our build and test jobs. We do that by creating a file with a .yml extension. Let's name this file main.yml Add the content below in the yml file you just created:

```
        #Location: .github/workflows/custom_config.yml

        name: CI-CD pipeline to AWS
        env:
        EB_S3_BUCKET_NAME: "test-django-cicd-bucket-1509"
        EB_APPLICATION_NAME: "Aws-cicd-django"
        EB_ENVIRONMENT_NAME: "Aws-cicd-django-env"
        DEPLOY_PACKAGE_NAME: "django-app-${{ github.sha }}.zip"
        AWS_REGION_NAME: "ap-northeast-1"

        on:
        push:
        branches: - master #Use your own branch here (Might be staging or testing)
        jobs:
        build:
        runs-on: ubuntu-latest
        steps: - name: Git clone on our repo
        uses: actions/checkout@v2

            - name: Create zip deployment package
                run: zip -r ${{ env.DEPLOY_PACKAGE_NAME }} ./ -x *.git*

            - name: Configure AWS credentials
                uses: aws-actions/configure-aws-credentials@v1
                with:
                aws-access-key-id: ${{ secrets.aws_access_key_id }}
                aws-secret-access-key: ${{ secrets.aws_secret_access_key }}
                aws-region: ${{ env.AWS_REGION_NAME }}
            - name: Copying file to S3
                run: aws s3 cp ${{ env.DEPLOY_PACKAGE_NAME }} s3://${{ env.EB_S3_BUCKET_NAME }}/
            - name: Print nice message on success finish
                run: echo "CI part finished successfuly"

        deploy:
        runs-on: ubuntu-latest
        needs: [build]
        steps: - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
        aws-access-key-id: ${{ secrets.aws_access_key_id }}
        aws-secret-access-key: ${{ secrets.aws_secret_access_key }}
        aws-region: ${{ env.AWS_REGION_NAME }}

            - name: Create new EBL app ver
                run: |
                aws elasticbeanstalk create-application-version \
                --application-name ${{ env.EB_APPLICATION_NAME }} \
                --source-bundle S3Bucket="${{ env.EB_S3_BUCKET_NAME }}",S3Key="${{ env.DEPLOY_PACKAGE_NAME }}" \
                --version-label "${{ github.sha }}"

            - name: Deploy new app
                run: aws elasticbeanstalk update-environment --environment-name ${{ env.EB_ENVIRONMENT_NAME }} --version-label "${{ github.sha }}"
            - name: Print nice message on success finish
                run: echo "CD part finished successfuly"

```

### Trigger the GitHub Action to build and deploy the code!

    Now, after any commit changes in code the workflow automatically going to be trigger & job will be start. Push any changes to your specified branch and visit Actions tab. If your actions completed successfully, you would see the green mark. If something went wrong, open the logs, and resolve the errors inside.
    Got errors after successful GitHub Actions run? Visit Elastic Beanstalk environment and check the Logs.
    Hopefully, you'll see something like this:

    Verify the deployment!
    Code zip successfully uploaded in S3 Bucket

    App is sucessfully up runing. And as we deploy our app on Beanstalk so it take care of all Health check as well traffic handaling & Sacalibity.

### Clean up!

    To avoid incurring future charges, you should clean up the resources that you created in Aws ( Elastic Bean Stalk,S3 Bucket)
