# Sharing Budget

An user-friendly platform allows you to effortlessly track, manage, and share budgets in real-time:
- List a summary of all budgets and filter by title
- Show all club sessions and explore various options: Add a new participant, new session, category, and more.
- Just click to view the amount contributed by each club member for every period.

[See the demo video](https://www.youtube.com/watch?v=4wPtLNOpups)

# Deploy the applicaton in AWS

## Launch new EC2 Instance

### User data (bootstrap):

```bash
sudo apt update &&
sudo apt install python3-virtualenv &&

sudo apt install python3-pip &&

mkdir club_budget &&

cd club_budget &&

virtualenv venv &&

git clone https://github.com/MinhHiepPHAM/backend_projects.git &&

cp -r backend_projects/club_budget/backend/project . &&
```

### Install requirements:
>pip install -r requirements.txt

### requirements.txt
```bash
django==4.2.9
django-cors-headers==4.3.1

djangorestframework==3.14.0

djangorestframework-simplejwt==5.3.1

python-decouple==3.8

requests==2.31.0

psycopg2-binary
```
### Install nginx as an API gateway:
```bash
sudo apt install nginx

sudo systemctl start nginx
```
Useful Link:
- To Securely deploy a Django App with Gunicorn, Nginx, & HTTPS, see [link](https://realpython.com/django-nginx-gunicorn/)
- See [Elastic IP address](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html) for EC2 configuration

## Launch new AWS Postgres:

Using tier for free deployement.

### .env
```env
DB_NAME=< database name > </br>
DB_USER=< master name > </br>
DB_PASSWORD= < master password > </br>
HOST=< database endpoint > </br>
SECRET_KEY=< django secret key > </br>
```
## Connect EC2 instance to RDS:
See [Tutorial: Connect an Amazon EC2 instance to an Amazon RDS database](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/tutorial-connect-ec2-instance-to-rds-database.html) </br>

**Security rule summary**:
- Security group **ec2-rds-x** is created and added to the EC2 instance. It has one outbound rule that references the **rds-ec2-x** security group as its destination. This allows traffic from the EC2 instance to reach the RDS database with the **rds-ec2-x** security group.
- Security group **rds-ec2-x** is created and added to the RDS database. It has one inbound rule that references the **ec2-rds-x** security group as its source. This allows traffic from the EC2 instance with the **ec2-rds-x** security group to reach the RDS database
- Allow all inbound traffic in ec-rds-x security group to allow the regquest from client

## Use S3 and cloudfront to host react app
Run *npm run build* then upload files and assets folder in **dist** repo to S3 (Block all public options in the S3 creation).

Update the S3 policy to allow only cloudfront that can be access to it
```json
{

   "Version": "2012-10-17",

    "Statement": [ 

        {

            "Sid": "AllowCloudFrontServicePrincipalReadOnly",

            "Effect": "Allow",

            "Principal": {

                "Service": "cloudfront.amazonaws.com"

            },

            "Action": "s3:GetObject",

            "Resource": "arn:aws:s3:::<bucket name>/*",

            "Condition": {

                "StringEquals": {

                    "AWS:SourceArn": "arn:aws:cloudfront::xxxx"

                }

            }

        }

     ]

}
```


Create a cloudfront distribution and chose S3 bucket as an origin name and allow all http methods

Set **Default root object** to '*index.html*' (not */index.html*). Also handle the error page by setting response path to '*/index.html*'.

Check the app with cloudfront distribution domain name

To support the DNS, we can use [AWS Route53](https://aws.amazon.com/route53/) (the dommain name can be registered from AWS or other platform, for example, [NameCheap](namecheap.com)).

## Summary

To deploy the application in AWS
- Use EC2 instance for backend API.
- Use S3 for frontend (store react pages).
- Use Cloudfront for CDN and Route53 for DNS

