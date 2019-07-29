# Serve a static website from S3

## Creating the bucket

First we need to create a bucket from where we are going to serve the website.

1. On your AWS Console, go to **[S3](https://console.aws.amazon.com/s3/)** under "Storage" section and click on **+ Create bucket** button.
2. Enter the name of the bucket. Remember, bucket names must be unique across all existing accounts and regions in AWS. You cannot rename a bucket after it is created, so chose the name wisely. Amazon suggests using DNS-compliant bucket names. You should read more about this [here](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html#bucketnamingrules).
3. Pick a region for the S3 bucket. You can chose any region you like, but beware that Amazon has [different pricing](https://aws.amazon.com/s3/pricing/) for storage in different regions. In this case (though it won't matter too much) we will pick `US East (N. Virginia)`.
4. Click Next until the Review section then click **Create bucket**. We will configure the properties later.
5. Once created, click on the name of your bucket, go to **Properties**, click **Static website hosting** check the option **Use this bucket to host a website**
6. As index and error document put: `index.html`. Later, we will go to the **endpoint url** specified at the top to access our website.
7. Click **Save** button.
8. Go to **Permissions** tab.
9. On the **Block Public Access** section, click **Edit**, uncheck **Block _all_ public access**, **Save** and **Confirm**.
10. Then go to **Bucket Policy** section and add the following policy to make every object readable. Change `<your-bucket-name>` with your real bucket name:
  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Sid": "AddPerm",
              "Effect": "Allow",
              "Principal": "*",
              "Action": "s3:GetObject",
              "Resource": "arn:aws:s3:::<your-bucket-name>/*"
          }
      ]
  }
  ```
11. Click **Save**


## Add `WEBSITE_BUCKET_NAME` to the Parameters Store

Every application needs to have some configurations that inherently will vary between different deployments: whether to enable debug or not, the address of the database server, secret keys or access tokens for third party services, etc. Some of these need to be stored securely (ie. keys API tokens). Many people use [environment variables](https://en.wikipedia.org/wiki/Environment_variable) for this, but it is not secure enough.

[AWS Parameters Store](http://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-paramstore.html) is a service designed for just this, and we will use it to store variables of our system. This will enable us to store constants and later use them during other steps of the deployment. We will start by storing the bucket name.

1. Go to **[S3](https://console.aws.amazon.com/s3/)** under "Storage" section of Services list.
2. See details of the bucket you just created and copy its name.
3. Go to **[Systems Manager](https://console.aws.amazon.com/systems-manager/)** under "Management & Governance" section of Services list.
4. On the left menu select **Parameter Store**.
5. Click **Create Parameter**.
6. Enter `/prod/codebuild/WEBSITE_BUCKET_NAME` as name and a meaningful description of what the parameter means (ie. "name of the website bucket").
7. Enter `s3://<your-bucket-name>` as value.
8. Click "Create parameter" button.

Now we can retrieve the bucket name with `aws ssm get-parameter` like we did [here](/buildspec.frontend.yml). Also, we can use [AWS SSM Agent](http://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent.html) to manage our instances' configuration from the AWS web console.


## Create a policy to get full access to the S3 website bucket

With [AWS Policies](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html), you can specify different permissions regarding every AWS resource you use. For example, you can create a policy for enabling full access to a specific S3 bucket, and that is what we are going to do. We will need this in the future to build the project programmatically and store it on S3.

1. Go to **[IAM](https://console.aws.amazon.com/iam/)** under "Security, Identity & Compliance" section of Service list.
2. Click in **Policies**.
3. Click in **Create Policy**.
4. Click **Import managed policy** link.
5. Search and select `AmazonS3FullAccess` (this is a premade policy by AWS, but you can also build your own).
6. Click the **JSON** tab and change the `Resource` value to `["arn:aws:s3:::<your-bucket-name>", "arn:aws:s3:::<your-bucket-name>/*"]` in the JSON content.
7. Click **Review policy**
8. Choose a name for the policy (eg. `S3WebsiteFullAccess`) and click in Create Policy.

Now we have a policy that allows full access (list, write, update, delete, etc) to our website bucket. Let’s see how we can use it in the following section.


## Create a project in CodeBuild to build and deploy the frontend

As we mentioned earlier, [AWS CodeBuild](https://aws.amazon.com/codebuild/) is an AWS service to build projects. In order to instruct CodeBuild on what to do, we have created the `buildspec.frontend.yml`. CodeBuild will first pull the repository, and then run the commands specified on [that file](/buildspec.frontend.yml). If you see, we have specified what is needed for a fresh install of the project, which ends up in a build using `npm build` and `aws s3 sync` for uploading the resulting files to S3.

Follow these steps to get it ready:

1. Go to **[CodeBuild](https://console.aws.amazon.com/codesuite/codebuild/)** under the "Developer Tools" section of Service list.
2. Click on **Getting Started** (or **Create Project** if you had other projects).
3. Choose a project name (eg. `aws-workshop`)and write an description (optional).
4. On the Source section:
   1. Choose **Github** as the source provider.
   2. Select an option for the repository.
   3. Connect Github with AWS if neccesary.
   4. Fill the **Repository URL** or choose one repository from your Github account.
5. On the **Environment** section:
   1. Choose Ubuntu as the OS as the Operating system
   2. Choose **Standard** as **Runtime(s)**
   3. Choose `aws/codebuild/standard:2:0` as the **Image**
   4. Choose **Always use the latest image for this runtime version** as **Image version** 
   5. Select **New service role** in your account.
   6. Choose a name for the Role and name it `codebuild-aws-workshop-service-role`.   
6. In the **Buildspec** section, choose **Use buildspec file** and change the BuildSpec name to `buildspec.frontend.yml` (our yaml file with the steps to follow).
6. In the **Artifacts** section select _No artifacts_.
7. In the Service Role section:

8. Click on Continue.
8. Click **Create build project**
9. Click on Save. ** Create build project**

Now, we have created a CodeBuild application. We won’t be able to run it though, because we don’t have permissions to add files to our S3 bucket. That is why earlier we created the policy and also something called a "role". For everything to work, we need to attach the policy to the role.

## Attach policies to the CodeBuild role

A [Role](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) defines permissions inside AWS. Those permissions come in the form of policies, just like in the case of your AWS user. Things like certain EC2 services (and even instances) which need to execute some actions can run attached to a certain role, and will thus get whatever permissions the role has.

Earlier, we created a policy to allow full access to our S3 bucket and assigned a new role to our CodeBuild application. Now we will attach the policy to the role, effectively giving CodeBuild access to our S3 bucket. Moreover, we will need to attach another policy to give it access to SSM, so that it can query the values of the variables we setup in the Parameter Store.

**Website full access**

1. Go to **[IAM](https://console.aws.amazon.com/iam/)** under "Security, Identity & Compliance".
2. Click in **Roles**.
3. You should see the role created in the CodeBuild project creation, select it. You can also search from the search box using `codebuild-aws-workshop-service-role`. Click role name shown in the list
4. In the **Permission** tab, click **Attach Policies** button.
5. Search for the Policy for full access to the S3 website bucket (`S3WebsiteFullAccess`), select it and then click **Attach Policy**.

**AWS Systems Manager (SSM) read access**

1. Click **Attach Policy** again.
2. Search for `AmazonSSMReadOnlyAccess` and select it then click on **Attach Policy**.

---
**Extra mile:** try get the value of `WEBSITE_BUCKET_NAME` from the command line.

---

**Next:** [EC2 instances](/workshop/s3-web-ec2-api-rds/02-EC2-instances.md).
