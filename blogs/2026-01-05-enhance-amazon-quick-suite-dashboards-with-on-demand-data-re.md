---
title: "Enhance Amazon Quick Suite dashboards with on-demand data refresh"
url: "https://aws.amazon.com/blogs/business-intelligence/enhance-amazon-quick-suite-dashboards-with-on-demand-data-refresh/"
date: "Mon, 05 Jan 2026 16:36:00 +0000"
author: "Kanwar Bajwa"
feed_url: "https://aws.amazon.com/blogs/business-intelligence/category/analytics/amazon-quicksight/feed/"
---
<p><a href="https://aws.amazon.com/quicksuite/" rel="noopener" target="_blank">Amazon Quick Suite</a> is an agentic AI-powered digital workspace that provides business users with a set of agentic teammates that quickly answer questions at work and turn those answers into action. It also helps in transforming scattered data into strategic insights, helping customers make faster decisions and achieve better business outcomes. As organizations expand analytics workloads in Quick Suite, providing timely data without compromising dashboard performance becomes critical. For structured data sources, Quick Suite offers two primary connection methods: Super-fast, Parallel, In-memory Calculation Engine (<a href="https://docs.aws.amazon.com/quicksuite/latest/userguide/spice.html" rel="noopener" target="_blank">SPICE</a>), which caches data in memory for fast queries, and direct query, which retrieves data live from the source. SPICE delivers subsecond performance but relies on scheduled refreshes. Direct query mode provides real-time results but can introduce latency, especially with large datasets or complex queries during peak usage.</p> 
<p>Many enterprise use cases require both high performance and up-to-date insights. For instance, a retail analytics team might need to update sales, inventory, and engagement data every 10–15 minutes. However, scheduled SPICE refreshes might not align with these frequent updates, and direct queries can slow down dashboards during peak usage when performance is critical.</p> 
<p>On-demand SPICE refresh makes it possible for users to trigger dataset updates directly from the dashboard, providing real-time data without compromising performance. Because the API has usage limits (such as how often refreshes can occur within a day), this feature should be limited to admins or authorized users to prevent overuse. This approach combines SPICE’s fast query performance with the flexibility to update data when needed, providing accurate, timely insights for critical business decisions while maintaining optimal dashboard performance.</p> 
<p>In this post, we show you how to enable on-demand SPICE refresh with a custom button in Quick Suite, as shown in the following screenshot, so you can maintain fast, interactive dashboards while giving users control over when data updates occur.</p> 
<p><img alt="" class="alignnone wp-image-6430 size-full" height="729" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/12/30/bi-4730-1-1.png" style="margin: 10px 0px 10px 0px;" width="1266" /></p> 
<h2>Solution overview</h2> 
<p>The solution is built on a SPICE dataset as the performance layer for the dashboard. The dataset ingests data from a supported source such as <a href="https://aws.amazon.com/rds/" rel="noopener" target="_blank">Amazon Relational Database Service</a> (Amazon RDS), <a href="https://aws.amazon.com/pm/redshift/" rel="noopener" target="_blank">Amazon Redshift</a>, <a href="https://aws.amazon.com/athena/" rel="noopener" target="_blank">Amazon Athena</a>, or <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon Simple Storage Service</a> (Amazon S3). Although these are some of the most commonly used <a href="https://docs.aws.amazon.com/quicksuite/latest/userguide/supported-data-sources.html" rel="noopener" target="_blank">data sources</a>, Quick Suite also supports a wide range of other data sources. A single SPICE dataset can also support many dashboards used across an organization, providing a shared, high-performance data layer that maintains consistency and reduces duplication. Scheduled refreshes maintain baseline data currency, and the on-demand refresh capability extends the architecture to support immediate updates triggered by users at critical moments.</p> 
<p>This solution integrates a refresh control into a Quick Suite dashboard to initiate a SPICE dataset ingestion on demand. The refresh control is implemented as a Quick Suite custom action that sends a request to an <a href="https://aws.amazon.com/api-gateway/" rel="noopener" target="_blank">Amazon API Gateway</a> endpoint. API Gateway invokes an<a href="https://aws.amazon.com/lambda/" rel="noopener" target="_blank"> AWS Lambda</a> function, which calls the Quick Suite <code class="lang-bash">CreateIngestion</code> API to refresh the specified dataset. You can manually refresh datasets in an Enterprise edition account 32 times in a 24-hour period. You can manually refresh datasets in a Standard edition account 8 times in a 24-hour period. Each 24-hour period is measured starting 24 hours before the current date and time. After the ingestion is complete, the dashboard automatically presents the updated data to users.</p> 
<p>The architecture of the solution is shown in the following diagram and includes the following components:</p> 
<ul> 
 <li><strong>Quick Suite</strong> – Contains the dashboard and Refresh custom action</li> 
 <li><strong>API Gateway</strong> – Serves as the secure REST API endpoint that Quick Suite invokes</li> 
 <li><strong>Lambda</strong> – Receives the request and calls the Quick Suite API to start SPICE ingestion</li> 
 <li><strong>Amazon S3</strong> – Is the data source</li> 
</ul> 
<p><img alt="" class="aligncenter wp-image-6435 size-full" height="484" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/12/30/b1-4730-2-1.png" width="569" /></p> 
<h2>Prerequisites</h2> 
<p>The following prerequisites are required for this solution:</p> 
<ul> 
 <li>AWS account</li> 
 <li>Basic knowledge of AWS services</li> 
 <li>Quick Suite Enterprise edition</li> 
 <li>SPICE capacity under Quick Suite</li> 
</ul> 
<h2>Deployment</h2> 
<p>In the following sections, we walk through the steps needed to implement an on-demand SPICE ingestion pipeline using AWS services.</p> 
<h3>Create S3 bucket and upload dataset</h3> 
<p>In this first step, you create an S3 bucket for data storage and upload a dataset.</p> 
<h3>Create S3 bucket</h3> 
<ul> 
 <li>Create an S3 bucket with the following steps:</li> 
 <li>On the Amazon S3 console, choose <strong>Create bucket</strong>.</li> 
 <li>Enter a bucket name, such as <code>refresh-iris-data</code>.</li> 
 <li>Select the AWS Region where you want to create the bucket.</li> 
 <li>Leave other settings as default or modify them based on your security or policy needs.</li> 
 <li>Choose <strong>Create bucket</strong>.</li> 
</ul> 
<h3>Upload dataset</h3> 
<p>Complete the following steps to upload your dataset:</p> 
<ul> 
 <li>On the Amazon S3 console, go to the S3 bucket you just created.</li> 
 <li>Choose <strong>Upload</strong> and choose <strong>Add files</strong>.</li> 
 <li>Select your dataset file (in this example, <code>iris-parquet.csv</code>).</li> 
 <li>Choose <strong>Upload</strong>.</li> 
</ul> 
<p><img alt="" class="aligncenter wp-image-6416 size-full" height="306" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/12/30/bi-4730-3.png" style="margin: 10px 0px 10px 0px;" width="936" /></p> 
<h3>Create a manifest file for Quick Suite</h3> 
<p>Quick Suite uses a manifest file to understand the structure of files in Amazon S3. Use the following code to create a manifest file with the name <code>iris-parquet.csv</code>:</p> 
<pre><code class="lang-bash">{
  "fileLocations": [
    {
      "URIs": [
        "s3://refresh-iris-data/iris-parquet.csv"
      ]
    }
  ],
  "globalUploadSettings": {
    "format": "CSV",
    "delimiter": ",",
    "textqualifier": "\"",
    "containsHeader": "true"
  }
}
</code></pre> 
<h3>Add Amazon S3 permissions to Quick Suite</h3> 
<p>Complete the following steps to add Amazon S3 permissions to Quick Suite:</p> 
<ul> 
 <li>On the Quick Suite console, choose your user name on the top right and choose <strong>Manage Quick Suite</strong>.</li> 
 <li>Under <strong>Permissions</strong> in the navigation pane, choose <strong>AWS resources</strong>.</li> 
 <li>In the <strong>Select Amazon S3 buckets section</strong>, select <strong>Select buckets</strong>, select the bucket you created (in this example, <code>refresh-iris-data</code>), then choose <strong>Finish</strong>.</li> 
 <li>Choose <strong>Save</strong>.</li> 
</ul> 
<h3>Create SPICE dataset in Quick Suite</h3> 
<p>In this step, you create a SPICE dataset with the data you have uploaded to Amazon S3.</p> 
<h3>Create dataset</h3> 
<p>Complete the following steps to create a dataset:</p> 
<ul> 
 <li>On the Quick Suite console, choose <strong>Datasets</strong> in the navigation pane.</li> 
 <li>Choose <strong>Create dataset</strong>.</li> 
 <li>Choose <strong>Create data source</strong>.</li> 
 <li>Choose <strong>S3</strong> as the data source.</li> 
 <li>For <strong>Data source name</strong>, enter a name (for example, <code>IrisDataFromS3</code>).</li> 
 <li>For <strong>Upload a manifest file</strong>, enter the S3 URI of the manifest file (for example, <code>s3://refresh-iris-data/manifest.json</code>.)</li> 
 <li>Choose <strong>Connect</strong>.</li> 
</ul> 
<h3>Import dataset to SPICE</h3> 
<p>Quick Suite will read the file and show a preview. Complete the following steps to <a href="https://docs.aws.amazon.com/quicksuite/latest/userguide/spice.html" rel="noopener" target="_blank">import the dataset to SPICE</a>:</p> 
<ul> 
 <li>In the Data source details section, choose Visualize.</li> 
</ul> 
<p><img alt="" class="aligncenter wp-image-6417 size-full" height="458" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/12/30/bi-4730-4.png" style="margin: 10px 0px 10px 0px;" width="932" /></p> 
<ul> 
 <li>Note the dataset ID. You can get it by opening the dataset and copying the URL. In the following example URL, the dataset ID is <code>2423b7</code>: <code>https://us-east- 1.quicksight.aws.amazon.com/data-sets/2423b7xxxx/view</code>.</li> 
</ul> 
<h3>Create Lambda function to trigger ingestion</h3> 
<p>In this step, you create a Lambda function to trigger ingestion of the latest data.</p> 
<h3>Create Lambda function</h3> 
<p>Complete the following steps to create a Lambda function:</p> 
<ul> 
 <li>On the Lambda console, choose <strong>Create function</strong>.</li> 
 <li>For <strong>Function name</strong>, enter a name (for example, <code>quicksuite_refresh</code>).</li> 
 <li>For <strong>Runtime</strong>, choose <strong>Python 3.12</strong> or newer.</li> 
 <li>Choose <strong>Create a new role with basic Lambda permissions</strong>.</li> 
 <li>&nbsp;Choose <strong>Create function</strong>.</li> 
 <li>Create the Lambda function by entering the following code. This Lambda function is written in Python to trigger a <a href="https://docs.aws.amazon.com/quicksight/latest/APIReference/API_CreateIngestion.html" rel="noopener" target="_blank">SPICE ingestion</a> refresh API for a Quick Suite dataset.</li> 
</ul> 
<pre><code class="lang-bash">import json
import boto3
import uuid
def lambda_handler(event, context):
&nbsp;&nbsp;&nbsp; print("Full event:", event)
&nbsp;&nbsp;&nbsp; qs_params = event.get("queryStringParameters") or {}
&nbsp;&nbsp;&nbsp; account_id = qs_params.get("QUICKSUITE_ACCOUNT_ID")
&nbsp;&nbsp;&nbsp; dataset_id = qs_params.get("DATASET_ID")
&nbsp;&nbsp;&nbsp; ingestion_type = "FULL_REFRESH"&nbsp; # or "INCREMENTAL_REFRESH"
&nbsp;&nbsp;&nbsp; if not account_id or not dataset_id:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; return {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "statusCode": 400,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "body": json.dumps({
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "message": "Missing required parameters: QUICKSUITE_ACCOUNT_ID and/or DATASET_ID"
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; })
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; ingestion_id = str(uuid.uuid4())
&nbsp;&nbsp;&nbsp; client = boto3.client("quicksight")
&nbsp;&nbsp;&nbsp; try:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; response = client.create_ingestion(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AwsAccountId=account_id,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; DataSetId=dataset_id,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; IngestionId=ingestion_id,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; IngestionType=ingestion_type
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; )
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; print("Full response:", response)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; return {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "body": json.dumps({
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "StatusCode" : response.get("ResponseMetadata", {}).get("HTTPStatusCode"),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "Arn": response.get("Arn"),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "IngestionId": response.get("IngestionId"),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "IngestionStatus": response.get("IngestionStatus"),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "RequestId": response.get("RequestId")
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; })
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; except Exception as e:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; print("Error:", str(e))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; return {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "statusCode": 500,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "body": json.dumps({
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "message": "Failed to start ingestion",
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "error": str(e)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; })
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }</code></pre> 
<ul> 
 <li>Choose <strong>Deploy</strong>.</li> 
</ul> 
<p><img alt="" class="aligncenter wp-image-6418 size-full" height="632" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/12/30/bi-4730-5.png" style="margin: 10px 0px 10px 0px;" width="584" /></p> 
<h3>Configure IAM role for Lambda</h3> 
<p>Complete the following steps to configure an <a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> role for Lambda:</p> 
<ul> 
 <li>On the <strong>Configurations</strong> tab, choose <strong>Permissions</strong>.</li> 
 <li>Under <strong>Execution role</strong>, choose the <strong>Role name</strong> link.</li> 
 <li>Attach the following inline policy to your Lambda function’s execution role:</li> 
</ul> 
<pre><code class="lang-bash">{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "logs:CreateLogGroup",
            "Resource": "arn:aws:logs:us-east-1:&lt;AccountID&gt;:*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": [
                "arn:aws:logs:us-east-1: &lt;AccountID&gt;:log-group:/aws/lambda/quicksuitet_refresh:*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "quicksight:CreateIngestion",
                "quicksight:DescribeIngestion"
            ],
            "Resource": "arn:aws:quicksight:us-east-1: &lt;AccountID&gt;:dataset/*"
        }
    ]
}
</code></pre> 
<h3>Test Lambda function</h3> 
<p>Complete the following steps to test your function:</p> 
<ul> 
 <li>On your function details page, choose <strong>Test</strong>.</li> 
 <li>Create a new event and enter a name, such as <code>Refresh</code>.</li> 
 <li>Provide the account ID and dataset ID as query parameters.</li> 
 <li>Choose <strong>Test</strong>.</li> 
</ul> 
<p>Lambda should have a successful execution, as shown in the following screenshot.</p> 
<p><img alt="" class="aligncenter wp-image-6420 size-full" height="302" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/12/30/bi-4730-6.png" style="margin: 10px 0px 10px 0px;" width="842" /></p> 
<h3>Create API Gateway endpoint integrated with Lambda function</h3> 
<p>Use the integrated endpoint to trigger your Lambda function (which starts Quick Suite ingestion) using a <code>REST GET</code> request.</p> 
<h3>Create new REST API</h3> 
<p>Complete the following steps to create a new REST API:</p> 
<ul> 
 <li>On the API Gateway console, in the navigation pane, choose <strong>APIs</strong>.</li> 
 <li>Choose <strong>Create API</strong>.</li> 
 <li>Select <strong>REST API</strong>.</li> 
 <li>Choose <strong>Build</strong>.</li> 
 <li>For <strong>API name</strong>, enter a name (for example, <code>quicksuite_refresh</code>).</li> 
 <li>For <strong>Endpoint type</strong>, choose <strong>Regional</strong> (the default value).</li> 
 <li>Choose <strong>Create API</strong>.</li> 
</ul> 
<p><img alt="" class="aligncenter wp-image-6436 size-full" height="376" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/12/30/bi-4730-14.png" style="margin: 10px 0px 10px 0px;" width="846" /></p> 
<ul> 
 <li>In the navigation pane, under your new API, choose <strong>Create resource</strong>.</li> 
 <li>For the <strong>Resource name</strong>, enter refresh.</li> 
 <li>Choose <strong>Create resource</strong>.</li> 
</ul> 
<p><img alt="" class="aligncenter wp-image-6437 size-full" height="208" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/12/30/bi-4730-15.png" style="margin: 10px 0px 10px 0px;" width="936" /></p> 
<h3>Create GET method for resource</h3> 
<p>Complete the following steps to create a GET method for the resource:</p> 
<ul> 
 <li>On the <strong>Resources</strong> page, choose the <code>/refresh</code> resource.</li> 
 <li>Choose <strong>Create method</strong>.</li> 
</ul> 
<p><img alt="" class="alignnone size-full wp-image-6438" height="212" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/12/30/bi-4730-16.png" style="margin: 10px 0px 10px 0px;" width="936" /></p> 
<ul> 
 <li>Choose <strong>GET</strong> on the dropdown menu.</li> 
 <li>For Integration type, select <strong>Lambda function</strong>.</li> 
 <li>Enable <strong>Lambda proxy integration</strong>.</li> 
 <li>For <strong>Lambda function</strong>, enter the Lambda function you created earlier.</li> 
 <li>Choose <strong>Create method</strong>.</li> 
</ul> 
<h3>Deploy API</h3> 
<p>Complete the following steps to deploy the API:</p> 
<ul> 
 <li>On the API Gateway console, in the navigation pane, choose <strong>Deploy API</strong>.</li> 
 <li>For <strong>Deployment stage</strong>, select <strong>New stage</strong>.</li> 
 <li>For <strong>Stage name</strong>, enter <code>prod</code> (or <code>dev</code>).</li> 
 <li>Choose <strong>Deploy</strong>.</li> 
</ul> 
<h3>Test API</h3> 
<p>Complete the following steps to test the API:</p> 
<ul> 
 <li>On the API details page, choose the <strong>Test</strong> tab.</li> 
 <li>Enter the account ID and dataset ID as query parameters.</li> 
 <li>Choose <strong>Test</strong>.</li> 
</ul> 
<p>The API should have a successful execution, as shown in the following screenshot.</p> 
<p><img alt="" class="aligncenter wp-image-6421 size-full" height="182" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/12/30/bi-4730-7.png" style="margin: 10px 0px 10px 0px;" width="936" /></p> 
<h3>Add refresh dataset button in Quick Suite</h3> 
<p>To complete the solution, add a button to refresh the dataset.</p> 
<h3>Create parameter</h3> 
<p>Complete the following steps to create a parameter:</p> 
<ul> 
 <li>In Quick Suite, go to your analyses.</li> 
 <li>Under <strong>Parameters</strong>, choose <strong>Add</strong>.<br /> <img alt="" class="aligncenter wp-image-6422 size-full" height="570" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/12/30/bi-4730-8.png" style="margin: 10px 0px 10px 0px;" width="936" /></li> 
 <li>Name the parameter refresh, set Data type to String, and enter Click here to refresh data for the <strong>Static default value</strong>.</li> 
 <li>Choose <strong>Create</strong>.</li> 
</ul> 
<p><img alt="" class="aligncenter wp-image-6423 size-full" height="558" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/12/30/bi-4730-9.png" style="margin: 10px 0px 10px 0px;" width="568" /></p> 
<h3>Create calculated field</h3> 
<p>Complete the following steps to create a calculated field:</p> 
<ul> 
 <li>Choose <strong>Calculated field</strong> to create a new calculated field with the expression<code> ${refresh}</code>.</li> 
 <li>Name it <code>refresh_field</code>.</li> 
</ul> 
<p><img alt="" class="aligncenter wp-image-6425 size-full" height="274" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/12/30/bi-4730-11.png" style="margin: 10px 0px 10px 0px;" width="342" /></p> 
<h3>Add table visual</h3> 
<p>Complete the following steps to add a table visual:</p> 
<ul> 
 <li>In the <strong>Visuals</strong> pane, add a new table visual to your analysis.</li> 
 <li>Drag <code>refresh_field</code> into the <strong>Group by</strong> section.</li> 
</ul> 
<p>This creates a single-row table that displays your parameter value.</p> 
<p><img alt="" class="aligncenter wp-image-6426 size-full" height="232" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/12/30/bi-4730-12.png" style="margin: 10px 0px 10px 0px;" width="468" /></p> 
<h3>Add custom action</h3> 
<p>Complete the following steps to add a custom action:</p> 
<ul> 
 <li>Choose the visual, choose <strong>Actions</strong>, and select <strong>Add URL action</strong>.</li> 
 <li>In the URL field, enter your API Gateway endpoint that triggers the ingestion, for example: <code>https://your-api-url.amazonaws.com/dev/refresh?QUICKSUITE_ACCOUNT_ID=&lt;AccountID&gt;&amp;DATASET_ID=&lt;DataSetID&gt;</code>.</li> 
 <li>For <strong>Open in</strong>, select <strong>New browser tab</strong>.</li> 
 <li>Choose <strong>Save</strong>.</li> 
</ul> 
<h3>Customize button</h3> 
<p>You can customize the button you just created as needed:</p> 
<ul> 
 <li>Hide column headers if needed.</li> 
 <li>Resize the visual to make it look like a button.</li> 
 <li>Customize formatting to make it more user-friendly (for example, change the font size, color, border, and so on).</li> 
</ul> 
<h3>Secure API gateway with resource policy</h3> 
<p>To help make sure only authorized users (for example, those accessing the solution from your organization’s proxy or specific IP addresses) can trigger the on-demand SPICE ingestion, use a resource policy on the API gateway to restrict access by IP address. Complete the following steps:</p> 
<ul> 
 <li>Identify the trusted IP address or range.</li> 
 <li>Identify the public IP address or CIDR block that your users will be coming from. For example, 22.01.13.11 /32 (single IP) or 22.01.13.11/24 (range).</li> 
 <li>On the API Gateway console, go to the API you created (for example, <code>quicksuite_refresh</code>).</li> 
 <li>In the navigation pane, choose Resource policy.</li> 
 <li>Enter the following policy:</li> 
</ul> 
<pre><code class="lang-bash">{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowFromCorporateProxy",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "execute-api:Invoke",
      "Resource": "arn:aws:execute-api:us-east-1:&lt;AccountID&gt;:4muro2t76a/*/GET/refresh",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": [
            "&lt;IP Address&gt;”,
            "&lt;IP Address Range&gt;"
          ]
        }
      }
    },
    {
      "Sid": "DenyAllOthers",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "execute-api:Invoke",
      "Resource": "arn:aws:execute-api:us-east-1:&lt;AccountID&gt;:4muro2t76a/*/GET/refresh",
      "Condition": {
        "NotIpAddress": {
          "aws:SourceIp": [
            "&lt;IP Address&gt;”
          ]
        }
      }
    }
  ]
}
</code></pre> 
<ul> 
 <li>Choose Deploy to deploy your API.</li> 
</ul> 
<h3>Test the flow</h3> 
<p>With the solution complete, you can test it by choosing <strong>Refresh</strong>. You should see a JSON response with the ingestion status.</p> 
<p>If you correctly configured the resource policy, the allowed environment receives an HTTP 201 OK response. Rerun the same command from an environment without an allowed IP address. A denied environment will receive an HTTP 403 Forbidden error.</p> 
<p>You can view your refresh history on the console, as shown in the following screenshot.</p> 
<p><img alt="" class="aligncenter wp-image-6427 size-full" height="404" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/12/30/bi-4730-13.png" style="margin: 10px 0px 10px 0px;" width="718" /></p> 
<h2>Conclusion</h2> 
<p>By implementing an on-demand SPICE refresh solution, you can maintain high-performance dashboards in Quick Suite while helping make sure data reflects the most recent updates when it matters most, all while providing end-users with direct control over when data is refreshed. This architecture gives decision-makers immediate access to accurate insights without compromising responsiveness or scalability. With the ability to trigger dataset refreshes directly from the dashboard, teams can align data updates with critical decision points, optimize resource usage, and reduce unnecessary ingestion jobs.</p> 
<p>Adopt this solution to improve the timeliness and relevance of analytics, enhance the user experience, and increase the operational efficiency of enterprise BI workloads on AWS.</p> 
<p>If you have any questions, comments, or suggestions, leave a comment. You can also visit <a href="https://repost.aws/tags/TAtoF6BGmuRdGfw2e14IBwEQ/amazon-quick-sight" rel="noopener" target="_blank">AWS re:Post.</a></p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-4541 alignleft" height="115" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/04/03/bajwkanw.png" width="100" />Kanwar Bajwa</strong> is a Principal Enterprise Account Engineer at AWS who works with customers to optimize their use of AWS services and achieve their business objectives.</p> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-6454 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/12/31/syedazn.jpeg" width="100" />Zainab Syeda</strong> is a Technical Account Manager at Amazon Web Services in Toronto. She works with customers in the Financial Services segment, helping them leverage cloud-native solutions at scale.</p>
