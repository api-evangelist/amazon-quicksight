---
title: "Building a serverless Amazon Quick Suite report delivery solution for non-Quick Sight users"
url: "https://aws.amazon.com/blogs/business-intelligence/building-a-serverless-amazon-quick-suite-report-delivery-solution-for-non-quick-sight-users/"
date: "Fri, 06 Feb 2026 15:48:39 +0000"
author: "Vetri Natarajan"
feed_url: "https://aws.amazon.com/blogs/business-intelligence/category/analytics/amazon-quicksight/feed/"
---
<p><a href="https://aws.amazon.com/quicksight" rel="noopener noreferrer" target="_blank">Amazon Quick Sight</a> is the business intelligence and reporting offering in <a href="https://aws.amazon.com/quicksuite/" rel="noopener noreferrer" target="_blank">Amazon Quick Suite</a>.&nbsp;<a href="https://aws.amazon.com/quicksight/paginated-reports/" rel="noopener noreferrer" target="_blank">Amazon Quick&nbsp;Sight pixel-perfect reports</a> enables the creation and sharing of custom- formatted, personalized reports containing business-critical data with hundreds of thousands of users without requiring infrastructure setup or maintenance, up-front licensing, or long-term commitments.&nbsp;Quick&nbsp;Sight supports built-in scheduling and distribution of pixel-perfect reports as PDF, CSV, or XLS files to users registered in Quick&nbsp;Sight. However, there are scenarios where these reports need to be delivered to users who aren’t registered in Quick&nbsp;Sight. Snapshot APIs from Quick&nbsp;Sight help achieve this use case. The blog post&nbsp;<a href="https://aws.amazon.com/blogs/business-intelligence/deliver-amazon-quicksight-paginated-reports-to-non-quicksight-users/" rel="noopener noreferrer" target="_blank">Deliver Amazon Quick&nbsp;Sight pixel-perfect reports to non-Quick&nbsp;Sight users</a> discusses a use case where Quick&nbsp;Sight APIs can be used to achieve this.&nbsp;In this post, we show you how to implement a serverless solution to deliver Quick&nbsp;Sight reports to non-Quick Sight users using AWS services. The solution tracks and persists the snapshot request details and output outside of Quick&nbsp;Sight.</p> 
<h2>Use case overview</h2> 
<p>Let’s consider a fictional software as a service (SaaS) startup company—AnyCompany—that provides call center management service to organizations. The company has been using Amazon Quick&nbsp;Sight to generate insights for their customers using anonymous embedding. The company wants to provide a solution to generate scheduled reports or one-time downloads for its customers that can be accessed anytime.</p> 
<h2>Solution overview</h2> 
<p>This post explains the architecture and technical implementation of a solution that helps achieve the following using APIs. These APIs can be called from a web application or from a batch process depending on the use case.&nbsp;The figure that follows illustrates the architecture of our serverless report delivery solution. It showcases how various AWS services interact to process report requests, generate reports using Amazon Quick&nbsp;Sight, and deliver them to users. As depicted in the figure, the solution follows these key steps:</p> 
<ol> 
 <li>The customer application submits a report request through <a href="https://aws.amazon.com/api-gateway" rel="noopener noreferrer" target="_blank">Amazon API Gateway</a>.</li> 
 <li>An <a href="https://aws.amazon.com/lambda" rel="noopener noreferrer" target="_blank">AWS Lambda</a> function processes the request and initiates the Quick&nbsp;Sight snapshot job.</li> 
 <li>Another Lambda function periodically polls for job status updates. Alternatively, the target <a href="https://aws.amazon.com/s3/" rel="noopener noreferrer" target="_blank">Amazon Simple Storage Service (Amazon S3)</a> bucket can be monitored using&nbsp;<a href="https://aws.amazon.com/eventbridge" rel="noopener noreferrer" target="_blank">Amazon EventBridge</a> to detect when a file is uploaded to the S3 bucket.</li> 
 <li>When complete, the report is stored in Amazon&nbsp;S3 and made available for download.</li> 
</ol> 
<p><img alt="" class="alignnone wp-image-6553 size-full" height="619" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/24/qs-ppr-image-1.jpg" width="1285" /></p> 
<h2>Prerequisites</h2> 
<p>Before you begin, ensure you have the following components in place.</p> 
<ul> 
 <li>A mechanism to get inputs from users and make API calls. For one-time reports, this mechanism would typically be a web page with an input form. For batch reports, the inputs are typically stored in a repository and a script fetches the input parameters and makes the API call.</li> 
 <li><a href="https://aws.amazon.com/cli" rel="noopener noreferrer" target="_blank">AWS Command Line Interface (AWS CLI)</a> configured with appropriate permissions.</li> 
 <li><a href="https://aws.amazon.com/serverless/sam/" rel="noopener noreferrer" target="_blank">AWS Serverless Application Model (AWS SAM)</a> CLI installed using the command <code>install aws-sam-cli</code>.</li> 
 <li>Python 3.9 or later.</li> 
</ul> 
<h2>Solution walkthrough</h2> 
<p>With the prerequisites in place, you’re ready to start building and using the solution.</p> 
<h3>Build and deploy</h3> 
<p>Download the ZIP file from <a href="https://github.com/aws-samples/sample-serverless-quick-sight-report-to-non-quick-sight-users" rel="noopener noreferrer" target="_blank">GitHub</a> and extract the contents of the file into a folder. Execute the following commands from the folder <code>sample-serverless-quicksight-report-to-non-quicksight-users-main</code>. By the end of this step, the necessary components will be deployed into the AWS account.</p> 
<div class="hide-language"> 
 <pre><code class="lang-code"># Set up the necessary configuration for interacting with AWS services.
aws configure
&nbsp; &nbsp; AWS Access Key ID [****************JJMR]:&nbsp;Enter&nbsp;AWS Access Key ID&nbsp;
&nbsp; &nbsp; AWS Secret Access Key [****************gjv7]:&nbsp;Enter AWS&nbsp;Secret Access Key
&nbsp; &nbsp; Default region name [us-east-1]:&nbsp;Enter&nbsp;AWS region name
&nbsp; &nbsp; Default output format [None]:&nbsp;Leave&nbsp;as&nbsp;None&nbsp; &nbsp;&nbsp;
# Build the application
sam build
# Deploy with default settings (5 concurrent jobs)
sam deploy --guided
# Or deploy with custom concurrent job limit
sam deploy --parameter-overrides MaxConcurrentJobs=10
</code></pre> 
</div> 
<h3>Get API endpoints</h3> 
<p>After deployment, the <a href="https://aws.amazon.com/cloudformation" rel="noopener noreferrer" target="_blank">AWS CloudFormation</a> outputs will display:</p> 
<ul> 
 <li><strong>QSAnonymoustSnapShotAPI</strong>: Base API URL</li> 
 <li><strong>APIEndpoints</strong>: All available endpoints with methods</li> 
</ul> 
<h3>Get API key value</h3> 
<p>The API key is automatically created but the value must be retrieved separately. This can be retrieved using the AWS Management Console for API Gateway.</p> 
<ol> 
 <li>Go to API Gateway and choose <strong>API Keys</strong>.</li> 
 <li>Search for and select <strong>QSAnonymousSnapshotApiKey</strong>.</li> 
 <li>Choose <strong>Show</strong> to reveal the key value.</li> 
</ol> 
<h3>List of APIs</h3> 
<p>This AWS SAM application will create</p> 
<ol> 
 <li>An S3 bucket to store output.</li> 
 <li>An <a href="https://aws.amazon.com/dyanmodb" rel="noopener noreferrer" target="_blank">Amazon&nbsp;DynamoDB</a> table to store job request details.</li> 
 <li>A Lambda function with an API Gateway as an interface to submit, list, and delete snapshot requests and download the output of a successfully completed snapshot job. The following table lists the API endpoints that will be generated.</li> 
</ol> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">API method</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">API request path</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">API description</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">GET</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">https://{api-id}.execute-api.{region}.amazonaws.com/Prod/qs-anonymous-snapshot</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">List existing shapshot jobs</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">POST</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">https://{api-id}.execute-api.{region}.amazonaws.com/Prod/qs-anonymous-snapshot</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Submit a snapshot job request</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">DELETE</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">https://{api-id}.execute-api.{region}.amazonaws.com/Prod/qs-anonymous-snapshot/{dashboardId}/{jobId}</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Delete the snapshot job after completion</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">GET</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">https://{api-id}.execute-api.{region}.amazonaws.com/Prod/qs-anonymous-snapshot/describe_job/{dashboardId}/{jobId}</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Describe the snapshot job</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">GET</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">https://{api-id}.execute-api.{region}.amazonaws.com/Prod/qs-anonymous-snapshot/describe_job_result/{dashboardId}/{jobId}</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Describe the result of a snapshot job</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">GET</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">https://{api-id}.execute-api.{region}.amazonaws.com/Prod/qs-anonymous-snapshot/download_output/{dashboardId}/{jobId}</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Download the output of the snapshot job</td> 
  </tr> 
 </tbody> 
</table> 
<ol start="4"> 
 <li>A Lambda function to constantly poll Amazon Quick&nbsp;Sight to check for status changes of currently running snapshot jobs.</li> 
</ol> 
<h3>Submit a snapshot job request</h3> 
<p>The following are detailed steps with an appropriate sample payload to submit a snapshot request:</p> 
<ol> 
 <li>Create a Python job that executes the API call for a snapshot report with a set of parameters.</li> 
 <li>Execute the Python job. The following is the typescript definition of the payload and sample of the POST request API call <code>/qs_anonymous_snapshot</code>.</li> 
</ol> 
<p>The below code shows the base url and the structure of the input that needs to be given to the snapshot job and the location where the output needs to go. The bucket prefix, as the name suggests, is the path of an S3 bucket that’s configured at the time of installation. Note that you can’t define the file name of the output because Amazon Quick&nbsp;Sight is designed to enforce that the output file has a unique name and doesn’t accidentally overwrite an existing file in Amazon&nbsp;S3. The input from users is set as parameters.</p> 
<div class="hide-language"> 
 <pre><code class="lang-css">base_url = "https://{api-id}.execute-api.{region}.amazonaws.com/Prod/qs-anonymous-snapshot"
api_key = "{api_key}"

dashboard_id = "{dashboard_id}"
sheet_id = "{sheet_id}"
visual_id = "{visual_id}"&nbsp;#Only&nbsp;required&nbsp;for&nbsp;CSV

headers = {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;'x-api-key': api_key,
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;'Content-Type': 'application/json'
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;}

// PDF format (visualId not required)
payload = {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"dashboardId": dashboard_id,
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"sheetId": sheet_id,
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"format": 'PDF',
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"parameters": &nbsp;None,
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"bucketConfiguration": {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"BucketPrefix": f"snapshots/{dashboard_id}/"
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;
response = requests.post(base_url, json=payload, headers=headers) &nbsp; &nbsp; &nbsp; &nbsp;

// CSV/EXCEL format (visualId required)
payload = {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"dashboardId": dashboard_id,
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"sheetId": sheet_id,
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"format": 'CSV',
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"parameters": &nbsp;None,&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"visualId" : [visual_id],
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"bucketConfiguration": {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"BucketPrefix": f"snapshots/{dashboard_id}"
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;
response = requests.post(base_url, json=payload, headers=headers) &nbsp; &nbsp; &nbsp; &nbsp;
</code></pre> 
</div> 
<ol start="3"> 
 <li>The preceding payload details will be used to trigger the <a href="https://docs.aws.amazon.com/cli/latest/reference/quicksight/start-dashboard-snapshot-job.html" rel="noopener noreferrer" target="_blank"><code>start_dashboard_snapshot_job</code></a> API call. This job takes the dashboard, the sheet, visual (for an XLS or CSV file), and parameter name and values that are passed as indicated in the following code. You can also optionally pass session tags to apply row-level security. The details of the session tags must be determined by the application to help ensure that users don’t manipulate the row level security. The <code>start_dashboard_snapshot_job</code> job is an asynchronous job, and so the details of this request need to be available at a later point in time to refer to the job and check the status. To support this, the details of the job are stored in the DynamoDB table. The details stored include dashboard ID, snapshot job ID, and job status (initially set as <code>SUBMITTED</code>).</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-css">&nbsp;&nbsp; &nbsp;response = QuickSight.start_dashboard_snapshot_job(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; AwsAccountId = awsAccountId,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; DashboardId = dashboardId,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; SnapshotJobId = snapshotJobId,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; SnapshotConfiguration = snapShortConfigurationTemplate,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; UserConfiguration = {
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"AnonymousUsers": [
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;{
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"RowLevelPermissionTags": [
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;{
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"Key": "Tag1",
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"Value": "123"
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;]
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;]
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; )
 &nbsp; &nbsp;dyn_resource = boto3.resource('dynamodb')
 &nbsp; &nbsp;table = dyn_resource.Table(os.environ['TABLE_NAME'])
 &nbsp; &nbsp;dashboardId = event["body"]["dashboardId"]
 &nbsp; &nbsp;table.put_item(Item={
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;'dashboardId': dashboardId,
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;'jobId': response["SnapshotJobId"],
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;'jobStatus': "SUBMITTED"
 &nbsp; &nbsp; &nbsp; &nbsp;})&nbsp; &nbsp; &nbsp; &nbsp;</code></pre> 
</div> 
<h3>Queue the snapshot job request</h3> 
<p>Although Quick Sight can scale automatically, a limited number of concurrent jobs can be submitted on a per account basis. A simple yet robust queuing solution can simplify operations at scale significantly. By adding a queuing solution layer, you can manage user variance in expectation of SLAs for reports and clear the queue if there’s a delay or an exception scenario that invalidates a set of delayed requests.Whenever the number of running jobs exceeds the maximum concurrent jobs limit,&nbsp;the polling function processes the queue and submits the job to Quick&nbsp;Sight when the number of running jobs is below the limit.</p> 
<div class="hide-language"> 
 <pre><code class="lang-css">#Queue the job&nbsp;
max_concurrent_jobs = int(os.environ.get('MAX_CONCURRENT_JOBS', '5'))
running_count = get_running_jobs_count()

if running_count &gt;= max_concurrent_jobs:
 &nbsp; &nbsp;# Queue the job
 &nbsp; &nbsp;dyn_resource = boto3.resource('dynamodb')
 &nbsp; &nbsp;table = dyn_resource.Table(os.environ['TABLE_NAME'])
 &nbsp; &nbsp;table.put_item(Item={
 &nbsp; &nbsp; &nbsp; &nbsp;'dashboardId': dashboardId,
 &nbsp; &nbsp; &nbsp; &nbsp;'jobId': snapshotJobId,
 &nbsp; &nbsp; &nbsp; &nbsp;'jobStatus': 'QUEUED',
 &nbsp; &nbsp; &nbsp; &nbsp;'sheetId': sheetId,
 &nbsp; &nbsp; &nbsp; &nbsp;'visualId': visualId,
 &nbsp; &nbsp; &nbsp; &nbsp;'parameters': parameters,
 &nbsp; &nbsp; &nbsp; &nbsp;'outputFormat': outputFormat,
 &nbsp; &nbsp; &nbsp; &nbsp;'outputDestination': outputDestination,
 &nbsp; &nbsp; &nbsp; &nbsp;'timestamp': int(time.time())
 &nbsp; &nbsp;})
 &nbsp; &nbsp;return {'SnapshotJobId': snapshotJobId, 'Status': 'QUEUED'}</code></pre> 
</div> 
<h3>Poll and update the status of the running snapshot jobs</h3> 
<p>A Lambda function is invoked every minute to check the status of snapshot jobs that aren’t complete. The status of the initiated job can be polled by invoking <code>describe_dashboard_snapshot_job</code> by passing in the <code>request_snapshot</code> job ID and the dashboard ID, which can be retrieved from DynamoDB as shown in the following code.</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">response = QuickSight.describe_dashboard_snapshot_job(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; AwsAccountId = awsAccountId,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; DashboardId = dashboardId,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; SnapshotJobId = snapshotJobId&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 
&nbsp; &nbsp; &nbsp; &nbsp; )</code></pre> 
</div> 
<p>After the job completes with a status of success or failure, the <code>describe_dashboard_snapshot_job_result</code> API is called to retrieve the details. If the job is successful, the Amazon&nbsp;S3 location of the output is stored in DynamoDB. If the job fails, the failure reason is stored in DynamoDB.</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">response = QuickSight.describe_dashboard_snapshot_job_result(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; AwsAccountId = awsAccountId,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; DashboardId = dashboardId,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; SnapshotJobId = snapshotJobId&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 
&nbsp; &nbsp; &nbsp; &nbsp; )
Following&nbsp;code checks for&nbsp;the status of&nbsp;the job by&nbsp;calling Quick&nbsp;Sight&nbsp;API&nbsp;DescribeDashboardSnapshot&nbsp;API. .&nbsp;If&nbsp;the job status is&nbsp;COMPLETED in&nbsp;Amazon&nbsp;Quick&nbsp;Sight,&nbsp;then&nbsp;Quick&nbsp;SightDashboardSnapshotResult&nbsp;API is&nbsp;called to get&nbsp;the details.

if job["jobStatus"] == 'SUBMITTED' or job["jobStatus"]=='RUNNING':
 &nbsp; &nbsp;r= describeSnapshotJob (awsAccountId,job["dashboardId"], job["jobId"])
 &nbsp; &nbsp;print(r)

 &nbsp; &nbsp;url = ""
 &nbsp; &nbsp;completedTime = ""

 &nbsp; &nbsp;if r["JobStatus"] == 'COMPLETED' :
 &nbsp; &nbsp;r_result = describeSnapshotJobResult (awsAccountId,job["dashboardId"], job["jobId"])
 &nbsp; &nbsp;print(r_result)
 &nbsp; &nbsp;url = r_result["Result"]["AnonymousUsers"][0]["FileGroups"][0]["Amazon&nbsp;S3Results"][0]["Amazon&nbsp;S3Uri"]
 &nbsp; &nbsp;completedTime = r_result["LastUpdatedTime"]
 &nbsp; &nbsp;# print(url) &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;

 &nbsp; &nbsp;current_response = table.update_item(
 &nbsp; &nbsp;Key={'dashboardId': job["dashboardId"], 'jobId': job["jobId"]},
 &nbsp; &nbsp;UpdateExpression="set jobStatus=:r, outputLocation= :u, jobParameters = :p, createdTime = :c, completedTime = :f",
 &nbsp; &nbsp;ExpressionAttributeValues={
 &nbsp; &nbsp; &nbsp; &nbsp;':r':r["JobStatus"], ':u': url, ':p': json.dumps(r["SnapshotConfiguration"]["Parameters"], indent=4, sort_keys=True, default=str), ':c': json.dumps(r["CreatedTime"], indent=4, sort_keys=True, default=str), ':f': json.dumps(completedTime, indent=4, sort_keys=True, default=str) &nbsp; },
 &nbsp; &nbsp;ReturnValues="UPDATED_NEW")</code></pre> 
</div> 
<h3>List snapshot jobs with status</h3> 
<p>The following API call returns a list of jobs along with their status.<code>POST https://{api-id}.execute-api.{region}.amazonaws.com/Prod/qs-anonymous-snapshot/:dashboardId</code>The preceding API call takes two query string parameters:</p> 
<ol> 
 <li><code>dashboardId</code>: The API returns all jobs when <code>dashboardId</code> isn’t provided. When <code>dashboardId</code> is provided, the jobs for that specific dashboard is returned. In this sample implementation, the DynamoDB table is partitioned using <code>dashboardId</code>.</li> 
 <li><code>nextToken</code>: This parameter is used to support pagination. It contains the base64-encoded token from the previous request, which is used to navigate to different pages of the results if the number of jobs in the DynamoDB table is more than a single page.</li> 
</ol> 
<p>The following is the typescript definition and typical usage:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">base_url = "https://{api-id}.execute-api.{region}.amazonaws.com/Prod/qs-anonymous-snapshot"
api_key = "{api_key}"

dashboard_id = "{dashboard_id}"
sheet_id = "{sheet_id}"
visual_id = "{visual_id}" #Only required for CSV

headers = {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;'x-api-key': api_key,
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;'Content-Type': 'application/json'
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;}

payload = {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"dashboardId": dashboard_id,
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"sheetId": sheet_id,
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"format": 'PDF',
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"parameters": &nbsp;None,
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"bucketConfiguration": {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"BucketPrefix": f"snapshots/{dashboard_id}/"
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}

status_url = f"{base_url}?dashboardId={dashboard_id}"
status_response = requests.get(status_url, headers=headers)

print('status_response = ',status_response.json())</code></pre> 
</div> 
<p>The following is the code that is executed against the DynamoDB table and returns results. The DynamoDB table is partitioned by dashboard ID. When the dashboard ID is given, then the API returns the list of snapshot jobs that have been invoked for the dashboard along with the details of sheets used as well as statuses.</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">def listSnapshots ():
 &nbsp; &nbsp;print("inside listSnapshots")
 &nbsp; &nbsp;dyn_resource = boto3.resource('dynamodb')
 &nbsp; &nbsp;table = dyn_resource.Table(os.environ['TABLE_NAME'])
 &nbsp; &nbsp;dashboardId = event.get("queryStringParameters", {}).get("dashboardId") if event.get("queryStringParameters") else None
 &nbsp; &nbsp;if not dashboardId:
 &nbsp; &nbsp; &nbsp; &nbsp;return {'error': 'dashboardId parameter required'}
 &nbsp; &nbsp;key_condition_expression = \
 &nbsp; &nbsp;Key('dashboardId').eq(dashboardId) 
 &nbsp; &nbsp;response=table.query(KeyConditionExpression=key_condition_expression)
 &nbsp; &nbsp;return response</code></pre> 
</div> 
<h3>Download the output</h3> 
<p>The users who access these reports won’t have AWS credentials to interact directly with Amazon&nbsp;S3 to download the report. So, a pre-signed URL is generated to provide temporary, controlled access to the report stored in Amazon&nbsp;S3. With this change, users can download and view the report, which has been exported as a PDF, XLS, or CSV file. The S3 bucket also enables setting up retention rules for reports using the Amazon S3 lifecycle management tools.The following is the API call to receive the pre-signed URL of the output:</p> 
<div class="hide-language"> 
 <pre><code class="lang-css">GET &nbsp;https://{api-id}.execute-api.{region}.amazonaws.com/Prod/qs-anonymous-snapshot/download_output/{dashboardId}/{jobId}</code></pre> 
</div> 
<p>The following is the typescript definition and typical usage:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">while status_response.json()['JobStatus'] != 'COMPLETED':
&nbsp;&nbsp; &nbsp;status_response = requests.get(status_url, headers=headers)
&nbsp;&nbsp; &nbsp;if status_response.json()['JobStatus'] == 'FAILED': 
&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;print(status_response.json())
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;exit()

download_url = f"{base_url}/download_output/{dashboard_id}/{job_id}"
download_response = requests.get(download_url, headers=headers)

print("Output URL = ",download_response.json()['presignedUrl'])</code></pre> 
</div> 
<p>Following code is executed in lambda to generate the pre-signed URL for the output file of a snapshot job.</p> 
<div class="hide-language"> 
 <pre><code class="lang-css">response = AmazonS3_client.generate_presigned_url('get_object',
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Params={'Bucket': bucket_name,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 'Key': object_name},
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ExpiresIn=expiration)</code></pre> 
</div> 
<h3>Sample Python test script to generate the URL for&nbsp;PDF output</h3> 
<div class="hide-language"> 
 <pre><code class="lang-python">import requests
import json

base_url = "https://{api_id}.region.amazonaws.com/dev/qs-anonymous-snapshot"

api_key = "{api_key}"

dashboard_id = "{dashboard_id}"
sheet_id = "{sheet_id}"

headers = {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;'x-api-key': api_key,
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;'Content-Type': 'application/json'
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;}

payload = {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"dashboardId": dashboard_id,
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"sheetId": sheet_id,
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"format": 'PDF',
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"parameters": &nbsp;None,
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"bucketConfiguration": {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"BucketPrefix": f"snapshots/{dashboard_id}/"
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;
response = requests.post(base_url, json=payload, headers=headers) &nbsp; &nbsp; &nbsp; &nbsp;

print(response.json())

job_id = response.json()['SnapshotJobId'] &nbsp;

status_url = f"{base_url}/describe_job/{dashboard_id}/{job_id}"
status_response = requests.get(status_url, headers=headers)


while status_response.json()['JobStatus'] != 'COMPLETED':
&nbsp;&nbsp; &nbsp;status_response = requests.get(status_url, headers=headers)
&nbsp;&nbsp; &nbsp;if status_response.json()['JobStatus'] == 'FAILED': 
&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;print(status_response.json())
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;exit()

download_url = f"{base_url}/download_output/{dashboard_id}/{job_id}"
download_response = requests.get(download_url, headers=headers)

print("Output URL = ",download_response.json()['presignedUrl'])</code></pre> 
</div> 
<p>If the number of concurrent invocations is more than the concurrent job limit, the job will be queued as seen in the following status.</p> 
<div class="hide-language"> 
 <pre><code class="lang-css">{'SnapshotJobId': 'PDF_1765307746_6813369', 'Status': 'QUEUED'}</code></pre> 
</div> 
<h2>Clean up</h2> 
<p>To avoid ongoing charges, delete the following resources if they’re no longer needed, delete the Cloud Formation stack by using the following AWS SAM command.<code>sam delete --stack-name &lt;your-stack-name&gt;</code></p> 
<h2>Conclusion and next steps</h2> 
<p>This post illustrates how an SaaS startup company can use Amazon Quick&nbsp;Sight to generate scheduled reports or one-time downloads for its customers that can be accessed anytime. Depending on the preferred delivery method, the generated report can also be transferred to other systems using Secure File Transfer Protocol (SFTP) or emailed to permitted users and groups within and outside the company for customers and partners. A separate Amazon&nbsp;S3 bucket per customer or partner can be used to store report outputs to help ensure tighter delineation of output data.By default, the solution implements basic API key-based authorization. Depending on the use case, the authorization can be implemented to suit customers’ preferred authorization method.</p> 
<p>To learn more, see <a href="https://aws.amazon.com/quicksight/pixel-perfect-reports/" rel="noopener noreferrer" target="_blank">Quick Sight pixel-perfect reports</a>.</p> 
<hr /> 
<h3>About the authors</h3> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-6558 alignleft" height="131" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/28/Vetri-1.jpg" width="102" />Vetri Natarajan</strong> is a&nbsp;Senior Specialist Solutions Architect for Amazon Quick Suite. Vetri has 15 years of experience implementing enterprise business intelligence (BI) solutions and greenfield data products. Vetri specializes in integration of BI solutions with business applications and enabling data-driven decisions.</p> 
<p style="clear: both;"><strong><img alt="" class="wp-image-6570 size-full alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/02/02/vaidya.jpeg" width="100" />Vaidy Janardhanam</strong> is a Specialist Solutions Architect for Amazon Quick Suite. He specializes in empowering customers to transform their analytics landscape, unifying traditional business intelligence dashboards with cutting-edge agentic AI workflows that turn insights into automated action.</p> 
<p style="clear: both;"><strong><img alt="" class="wp-image-6571 size-full alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/02/02/ljiahuan.jpeg" width="100" />Leona Li</strong>&nbsp;is a Specialist Solutions Architect for Amazon Quick Suite. She focuses on enabling data intelligence and helping enterprise customers become more data-driven by designing, building, and modernizing their solutions in the cloud.</p>
