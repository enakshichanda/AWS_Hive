# AWS_Hive

Running a Hive Job for Log Analysis
Suppose you host a popular e-commerce website and you want to analyse
your Apache web logs to see how people find your site. You want to
determine which of your online ad campaigns drive the most traffic to your
online store.
The web server logs, however, are too large to import into a MySQL
database, and they are not in a relational format. You need another way to
analyse them.
AWS EMR integrates open-source applications such as Hadoop and Hive with
Amazon Web Services to provide a scalable and efficient architecture for
analysing large-scale data, such as Apache web logs.
In this tutorial, we'll import data from Amazon S3 and create an Amazon
EMR cluster. Then we'll run Hive script to query the Apache logs using a
simplified SQL syntax. To learn more about Hive, see
http://hive.apache.org/ .
1. Background: Sample Data Overview
The sample data is a series of Amazon CloudFront web distribution log files,
collected between 7th-8th May 2014. The data is stored in Amazon S3 at
s3://us-east-1.elasticmapreduce.samples where us-east-1 is the N.
Virginia region.
Each entry in the CloudFront log files provides details about a single user
request in the following format:
2014-07-05 20:00:00 LHR3 4260 10.0.0.15 GET eabcd12345678.cloudfront.net /testimage-
1.jpeg 200 - Mozilla/5.0%20(MacOS;%20U;%20Windows%20NT%205.1;%20en-
US;%20rv:1.9.0.9)%20Gecko/2009040821%20IE/3.0.9
Sample Hive Script Overview
The sample script calculates the total number of requests per operating system over a specified timeframe.
The script uses HiveQL, which is a SQL-like scripting language for data warehousing and analysis. The
script is stored in Amazon S3 at
s3://region.elasticmapreduce.samples/cloudfront/code/Hive_CloudFront.q where
region is your region.
Amazon Elastic MapReduce Management Guide
Sample Hive Script Overview
2. Background: Sample Hive Script Overview
In order for Hive to interact with data, it must translate the data from its
current format (in the case of Apache web logs, a text file) into a format that
can be represented as a database table. Hive does this translation using a
serializer/deserializer (SerDe). SerDes exist for a variety of data formats. For
information about how to write a custom SerDe, see the Apache Hive
Developer Guide. The SerDe we'll be using in this example uses regular
expressions to parse the log file data. It comes from the Hive open-source
community. Using this SerDe, we can define the log files as a table, which
we'll query using SQL-like statements later in this tutorial.
The sample script below calculates the total number of requests per
operating system over a specified timeframe. The script uses HiveQL, which
is a SQL-like scripting language for data warehousing and analysis. As
discussed in lectures, the Hive code below creates a table, reads the log file
and queries the number of web requests grouped by operating systems.
-- Summary: This sample shows you how to analyze CloudFront logs stored in S3 using
Hive
-- Create table using sample data in S3.
CREATE EXTERNAL TABLE IF NOT EXISTS cloudfront_logs (
DateLog DATE,
Time STRING,
Location STRING,
Bytes INT,
RequestIP STRING,
Method STRING,
Host STRING,
Uri STRING,
Status INT,
Referrer STRING,
OS STRING,
Browser STRING,
BrowserVersion STRING
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
WITH SERDEPROPERTIES ("input.regex" = "^(?!#)([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^
]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^
]+)\\s+[^\(]+[\(]([^\;]+).*\%20([^\/]+)[\/](.*)$"
) LOCATION '${INPUT}/cloudfront/data';
-- Total requests per operating system for a given time frame
INSERT OVERWRITE DIRECTORY '${OUTPUT}/os_requests/' SELECT OS, COUNT(*) FROM
cloudfront_logs WHERE DateLog BETWEEN '2014-07-05' AND '2014-08-05' GROUP BY OS;
Next you will be running the Hive script in an Amazon EMR cluster in order
to process the sample data.
3. Sign-in to the AWS Console as before
If you have AWS Educate Starter Account
(without credit card), use the link (button)
on the welcome email you have received to
sign in.
If you have regular AWS account (with
credit card) and promo code for $100 credit
redeemed (installed) on it, sign in by
https://console.aws.amazon.com/
If you still don’t have own account with
credit, for this tutorial you will be using the
tutor's account. Sign in by:
https://pl-dsbda-
2020.signin.aws.amazon.com/console
Account ID: pl-dsbda-2020
IAM user name: dsbda
Password: ***** (password provided in class)
4. Prepare your S3 Bucket
Find the Hive script file Hive_CloudFront.q on the Blackboard (under
Hands-on 2) and store it locally on your U: drive (or your laptop). Go to S3
and create a new folder in your bucket, e.g. hive-job, with two subfolders,
e.g. script and output. Upload the script from your U: drive to the
script folder.
Students who use the tutor’s account should skip the next
section 5. as the tutor has setup a cluster for them. Those
students should NOT create new clusters on the tutor’s account.
5. Create a Cluster
Make sure you are in the US N. Virginia region. Create a new cluster as you
did it before, but now with Hive installed on it – you must have Hive and
Hadoop selected in the 'Software configuration' step, any other software is
optional. Continue with the next steps as before and create cluster. After a
while the cluster gets in 'Waiting' state, ready to be used.
Open the cluster.
6. Run a Hive Script on the Cluster
To run a job (a.k.a. step) on the cluster, select the Steps tab and click the
Add Step button:
In the Add Step dialog:
o For Step type select Hive program.
o For Name, name your step, e.g. anatoli-step1
o For Script S3 location, navigate to the script file Hive_CloudFront.q
o For Input S3 location, type s3://us-east-1.elasticmapreduce.samples
o For Output S3 location, navigate to the output folder.
o For Arguments, leave the field blank.
o For Action on failure, accept the default option Continue.
Click Add.
Alternatively, type what you see in the screenshot below using your bucket
name instead of a.nachev2020.test .
7. View the Result
When the step is completed, find the output in your bucket under the
output folder specified in the step. Download the file and open it using
text editor.
Output from the Hive script:
1. You should see how many requests for web pages
have arrived to the server in the time frame
specified. The output is grouped by client’s
operating system (e.g. 855 from Android, 883 from
Windows, etc).
8. Task1: Modify the Script Output
Modify the script in order to separate the OS name and number of requests
in the output, i.e.:
Hint: Create a copy of the previous script on your U: drive and rename it. Modify
the code by adding a string (e.g. ,’-> ‘,) between OS and COUNT in the SELECT
query and save the script. Upload the updated version to the S3 script folder.
Optionally, create a new output folder, although you can use the existing one. Run
a new step with the updated script and when completed, see the result.
9. Task2: Modify the Hive Script to Calculate the Outbound
Traffic
You are to modify the Hive Script in order to calculate the outbound traffic
that the web server generates between 2014-07-05 and 2014-07-06 (daily
traffic). In fact, the outbound traffic is the sum of bytes that the server has
sent back replying to browser requests in that period. Apparently, the table
fields Bytes, DateLog, and the aggregate function SUM( ) will be involved in
the query.
10. Should you have time left, create new SELECT queries
that you believe can analyse the data.
(Students who use the tutor’s account should
skip section 11; all other students MUST do it)
11. Terminate Your Cluster:
Terminating your cluster will stop charging on your account for using it.
As before, to terminate your Amazon EMR cluster
1. On the Cluster List, select your cluster.
2. Choose Terminate.
Make sure you see your cluster in Terminating / Terminated state
and after a while:
