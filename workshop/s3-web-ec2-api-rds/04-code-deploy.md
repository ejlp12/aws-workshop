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

1. Under **Deployments** tab, click **Create Deployment** button.
2. On **Deployment group** select `ProductionDeploy`
3. On **Revision type** select **"My application is stored in GitHub"**.
4. Type your Github username (or can be anything to identify your GitHub account) on **GitHub token name** 
5. Click **Connect to GitHub** button.
6. Allow AWS to access your GitHub account, if needed. A popup browser window will appear to ask your permission.
7. Enter your repository name in the form _account/repository_ (eg. `ejlp12/aws-workshop`).
8. In **Commit ID** type the commit hash that you want to deploy. You can check on your own repo commit page which is a fork  from this repo, the page to check latest commit is similar to `https://github.com/ejlp12/aws-workshop/commits/master`. Just change `ejlp12` to your Github username. Click the "copy" icon to copy the commit hash (eg. `0d5ea42e785f29d66a8dcd3192390319d7cdb138`) 
9. Keep default the rest of fields.
8. Click **Create deployment** button.

During the deploy you can see the Deployment status progress, try to click **View events** to follow the detailed progress and see what's happening.

After sucessful deployment, let's try to test the backend application.

1. From the Deployment page, click the _Instance ID_ listed in the table. This is the EC2 Instance ID where the backend application was deployed.
2. On the EC2 dashboard take a loook at instance description, copy the value of **Public DNS (IPv4)**
3. Try to access from the browser to `http://<Public DNS (IPv4)>:9000/api`

---
**Extra mile:** once the deploy finished:

- Try hitting the API with something like [Postman](https://www.getpostman.com/) or [httpie](https://httpie.org/). Explore more [about the backend application here](/backend/README.md)
- What effect did the deploy have? Where did all the Python code end up? Is the API connected with the RDS already? `ssh` in to get all those answers, and more.

---
**Next:** we are going to [finish our first deploy](/workshop/s3-web-ec2-api-rds/05-finishing-up.md), only some extra parameters are missing!
