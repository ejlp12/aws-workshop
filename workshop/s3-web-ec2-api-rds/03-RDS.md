# RDS

## Create a PostgreSQL instance in RDS
1. Go to **[RDS](https://console.aws.amazon.com/rds/)** under **Database** section.
2. Click on **Create Database**.
3. Click on **PostgreSQL** logo, tick the _"Only enable options eligible for RDS Free Usage Tier"_ checkbox at the bottom and click **Next**.
7. In the "Specify DB details" page
   1. Keep as it is all the setting in the section **Instance specifications**
   2. Enter a name (eg. `WebsiteWorkshopDB`) on **DB Instance identifier** (we will need it later, so don’t forget it).
   3. Enter a username (eg. `pguser`) and password (eg. `P455w0rd`) and click Next (again, we will need these later).
8. In the "Configure advanced settings" page
   1. On the **Network & Security**, select **No** on **Publicly Accessible**.
   2. Availability Zone: `us-east-1a`.
   3. On **VPC security groups** select **Choose existing VPC security groups and select the security group you create when [launching the EC2 instance](/workshop/s3-web-ec2-api-rds/02-EC2-instances.md#launch-your-first-ec2-instance).
   4. On the **Database options**, pick a db name (eg. `WebsiteWorkshopDB`) (again, we will need the database name later)
   5. Examine the other section but you can keep it as it is.
9. Click **Create database** .
12. Click **View DB instances details**.

Now our instance is created. We configure its access, allowing every instance under the security group that was created in the previous section to connect.

## Add DB parameters on Parameters Store

As before, we will need some variables stored in the parameter store, including the database name, username, password and endpoint. These variables are referenced in [this file](/backend/conduit/settings/ec2.py), so Django can access the database.

1. Go to **RDS** under **Database** section.
2. Click on **Databases** from the left menu.
3. See details of your db and copy the **Endpoint**. This will be the value for `DATABASE_HOST`. If you don't see the value of Endpoint, probably it is because the database still in "Creating" status. Take a look for the status at the "info" in the Summary box.
4. Go to **Systems Manager** under "Management & Governance" section of Services list.
5. On the left menu select **Parameter Store**.
6. Click **Create Parameter**.
7. Enter  `/prod/api/DATABASE_NAME` as the name and a meaningful description like "Name of the PostgreSQL database".
8. Enter the DB name you selected before on the value attribute (`WebsiteWorkshopDB`).
9. Click **Create parameter** button.
10. Now we will need to do the same thing for the username and host
    1. For the username enter `/prod/api/DATABASE_USER` as the name and your database username and as the value (`pguser`)
    2. For the host enter `/prod/api/DATABASE_HOST` as the name and the hostname (Endpoint) you copied earlier as the value
    3. For `/prod/api/DATABASE_PASSWORD` do the same steps but select as **Type: Secure String** and as KMS Key ID the key `workshopkey`.

Now we have our database parameters set, and the password encrypted. Only our EC2 instances will be able to decrypt it.

---
**Extra mile:**

- Can you `ping` the Postgres instance?
- Try to connect to the DB through your running EC2 instance.

---

**Next:** create a [CodeDeploy project to deploy your API](/workshop/s3-web-ec2-api-rds/04-code-deploy.md).
