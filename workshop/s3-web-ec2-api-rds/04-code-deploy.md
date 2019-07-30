# CodeDeploy

[CodeBuild](http://docs.aws.amazon.com/codedeploy/latest/userguide/welcome.html) is a service to automate the deployment of any kind of applications to EC2 instances. The configuration is really simple and easy to adapt. The deployment process is described in an `appspec.yml` file like [this one](/appspec.yml). If you want to know what happens during the deploy, you can also check the implementation of the hooks [here](/infrastructure/aws/codedeploy).

First, we need to create a default role for CodeDeploy so it can have access to other AWS services (like S3).

## Create CodeDeploy Role
1. Go to **IAM** under **Security, Identity & Compliance**.
2. Go to **Role** section and click **Create Role**.
3. Select **CodeDeploy**, in the "Select your use case" choose **CodeDeploy** that has description _Allows CodeDeploy to call AWS services such as Auto Scaling on your behalf. _ and click **Next: Permissions**.
4. Select **Next: Tags**.
4. Select **Next: Review**.
5. Type a name (eg. `CodeDeployWebsiteWorkshopRole`) and description and click **Create Role**.

Now we are ready to start using it.

##  Configure Code Deploy
1. Go to **CodeDeploy** under **Developer Tools**.
2. Click **Deploy** > **Getting Started** and then click **Create application** button.
3. Enter an **Application name** (`WebsiteWorkshopDeploy`) and choose **EC2/On-premises** as **Compute Platform**
4. Click **Deployment group** from the "Application details" page and enter **Deployment group name** (eg. `ProductionDeploy`).
5. On **Service role** select the role created to grant CodeDeploy access to the instances.
6. Select **In-place deployment** on **Deployment Type** section.
7. Select **Amazon EC2 instances** tab on **Environment configuration**.
8. On the first tag group select `environment` as Key and as Value `production`, on the second line select `service` as Key and as Value `api`. This means that CodeDeploy will deploy our application to all the EC2 instances with those tags.
9. On the **Deployment settings** section, select **CodeDeployDefault.OneAtATime** as Deployment Configuration.
10. Uncheck **Enable load balancing** because we will not use it at the moment, but later we will 
11. Click **Create deployment group**.

Now our CodeDeploy application is ready. Let’s try our first deployment.

1. Under **Deployment groups** section tick your group and in **Actions** select **Deploy new revision**.
2. On **Repository type** select **"My application is stored in GitHub"**.
3. In **Connect to GitHub** section type your GitHub account and select **Connect to GitHub**.
4. Allow AWS to access your GitHub account, if needed.
5. Enter your repository name in the form _account/repository_.
6. In **Commit ID** type the commit hash that you want to deploy.
7. Select **Overwrite the content** below.
8. Click Deploy.

During the deploy try **View instances** and then **View events** to follow the progress and see what's happening.

---
**Extra mile:** once the deploy finished:

- Try hitting the API with something like [Postman](https://www.getpostman.com/) or [httpie](https://httpie.org/).
- What effect did the deploy have? Where did all the Python code end up? Is the API connected with the RDS already? `ssh` in to get all those answers, and more.

---
**Next:** we are going to [finish our first deploy](/workshop/s3-web-ec2-api-rds/05-finishing-up.md), only some extra parameters are missing!
