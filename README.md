# Getting Started with DataBlade

[![DataBlade demo video screenshot](https://github.com/databladeio/getting-started/blob/master/demo-video-screenshot.png)](https://www.youtube.com/watch?v=-sT9P5kHMf8)

## Hello World

DataBlade provides an interactive environment for doing data analysis. After logging in, create a new Project and start typing some code in the Console in the bottom right part of the screen and see what happens.

```python
a = 10
```

```python
b = [1,2,3]
```

```python
c = {"foo":"bar"}
```

```python
print "Hello World!"
```

```python
2 + 2
```

To move any snippets from the Console into your Editor, hover over the code and click the icon that appears to the left of the code.

## Accessing Your Data

- [SQL](#sql)
- [User-Uploaded Files](#user-uploaded-files)
- [S3](#s3)
- [Google Analytics](#google-analytics)
- [Google BigQuery](#google-bigquery)
- [Google AdWords](#google-adwords)
- [MongoDB](#mongodb)
- [Salesforce](#salesforce)
- [FTP](#ftp)
- [Facebook](#facebook)

### SQL
1. Under "Data Integrations", set up your SQL integration. We currently support PostgreSQL, MySQL, Oracle, and Microsoft SQL Server (MySQL, Oracle, and MS SQL may still have some issues, so let us know if you run into any).
2. Currently, we require that your database be accessible from external clients. In the future, we will have a solution for databases that are behind a firewall.
3. Ensure that your database is set up to receive connections from `52.25.129.138/32`.
4. In your code, you can query your SQL using the following snippets:

```python
# You can optionally provide a keyword argument index_col="column name" to set
# the DataFrame index. You'll often want to set this to your ID column name.
r = data.sql.query(INTEGRATION_ID, SQL_QUERY)

# You can also write a DataFrame back to your database
# if_exists takes three parameters:
#   "fail": If table exists, do nothing
#   "replace": If table exists, drop it, recreate it, and insert data
#   "append": If table exists, insert data. Create if does not exist
data.sql.write(INTEGRATION_ID, TABLE_NAME, DATAFRAME, if_exists="fail")

# You can also update specific rows. ID refers to the specific row you
# want to update. VALUES is a dict mapping column name -> updated value.
# The return value is a Boolean indicating whether or not a row was successfully
# updated. False likely means the ID was not found.
row_was_updated = data.sql.update_row(INTEGRATION_ID, TABLE_NAME, ID, VALUES)
```

#### Creating a read-only user (PostgreSQL)
For added security, you may want to create a special, read-only user account to use with DataBlade:

```sql
CREATE ROLE datablade LOGIN PASSWORD 'databladepassword';

GRANT CONNECT ON DATABASE database TO datablade;

GRANT USAGE ON SCHEMA public TO datablade;

GRANT SELECT ON ALL TABLES IN SCHEMA public TO datablade;

ALTER DEFAULT PRIVILEGES IN SCHEMA public
   GRANT SELECT ON TABLES TO datablade;
```

### User-Uploaded Files
1. For your convenience, we allow you to upload files directly to DataBlade and use them in your projects as easily as you might use files locally on your computer. To upload, simply click the Upload button on the "Data Integrations" page
2. In your code, access your files using the following snippets:

```python
# Read a CSV file
my_csv = data.read_csv(FILE_NAME)

# Read a file as a StringIO buffer. Here is an example with line-delimited JSON.
my_file = data.read_file(FILE_NAME)
my_dataframe = [json.loads(line) for line in my_file]
```

### S3
1. Under "Data Integrations", set up your S3 credentials
2. Take note of the `ID` value of your integration
3. In your code, access your files using the following snippets:

```python
# Read a CSV file (replace INTEGRATION_ID, BUCKET_NAME, and KEY as appropriate)
my_csv = data.s3.read_csv(INTEGRATION_ID, BUCKET_NAME, KEY)

# Read a file as a StringIO buffer. Here is an example with line-delimited JSON.
my_file = data.s3.read_file(INTEGRATION_ID, BUCKET_NAME, KEY)
my_dataframe = [json.loads(line) for line in my_file]

# Write a File object to S3
data.s3.write_file(INTEGRATION_ID, BUCKET_NAME, KEY, FILE_OBJECT)
```

### Google Analytics
1. Ensure that you [create or select a project in the Google Developers Console and enable the Analytics API](https://console.developers.google.com//start/api?id=analytics&credential=client_key)
2. If you already have a service account with a P12 key, you can skip this step. Otherwise, in the Developers Console for the chosen project, under **APIs & auth**, select **Credentials**. In the **Add credentials** dropdown, choose **Service account**. For **Key type**, select **P12**, then click **Create**. The private key should automatically download. Click **Close** in the dialog.
3. Copy the email address of the newly generated service account and [add it as a user](https://support.google.com/analytics/answer/1009702) to your Google Analytics account. This user will likely only require **Read & Analyze** permissions.
4. In DataBlade, set up a Google Analytics integration, providing the service email account, the generated `.p12` file, and the associated profile ID. Your profile IDs can be found in your Google Analytics account under Admin -> Select a View in the dropdown -> View Settings -> View ID. If you regularly query multiple profile IDs, set up a separate DataBlade Integration for each (you can still use the same service account for all of them).
5. In your code, query your Google Analytics data using the following snippets:

```python
# See all available query parameters:
# https://developers.google.com/analytics/devguides/reporting/core/v3/reference?hl=en
# You do not need to pass in the `id` parameter, as DataBlade will automatically
# populate it for you.
results = data.ga.query(INTEGRATION_ID,
    start_date='7daysAgo',
    end_date='today',
    dimensions="ga:browser",
    metrics='ga:sessions')
```

### Google BigQuery
1. Ensure that you [create or select a project in the Google Developers Console and enable the BigQuery API](https://console.developers.google.com//start/api?id=bigquery&credential=client_key)
2. If you already have a service account with a P12 key, you can skip this step. Otherwise, in the Developers Console for the chosen project, under **APIs & auth**, select **Credentials**. In the **Add credentials** dropdown, choose **Service account**. For **Key type**, select **P12**, then click **Create**. The private key should automatically download. Click **Close** in the dialog.
3. In DataBlade, set up a BigQuery integration, providing the service email account, the generated `.p12` file, and the associated project ID.
5. In your code, query your BigQuery data using the following snippets:

```python
# Keep in mind that the format of BigQuery table names is <dataset name>.<table name>
results = data.bq.query(INTEGRATION_ID, SQL_QUERY)
```

### Google AdWords
1. If you haven't already done so, [apply for the Google AdWords API](https://developers.google.com/adwords/api/docs/signingup)
2. Once API access is granted on your account, set up a Google AdWords integration in DataBlade, providing the [Developer Token](https://developers.google.com/adwords/api/faq#15113) and [Customer ID](https://support.google.com/adwords/answer/29198?hl=en). Click the **Authenticate with Google** button to complete the authentication process. Make sure that the Google account with authenticate with has been granted AdWords permissions for the provided **Customer ID**. Then click **Create**.
3. Reports for AdWords are defined using [AWQL](https://developers.google.com/adwords/api/docs/guides/awql#adhoc-reports). You can see a full list of report types and queryable fields [here](https://developers.google.com/adwords/api/docs/appendix/reports).
4. In your code, retrieve Google AdWords reports using the following snippets:

```python
# Example AWQL query: "SELECT CampaignId, Clicks FROM CAMPAIGN_PERFORMANCE_REPORT DURING LAST_WEEK"
report = data.adwords.get_report(INTEGRATION_ID, AWQL_QUERY)
```

### MongoDB
1. Under "Data Integrations", set up your MongoDB integration. In **Connection String**, make sure to also include your database name in the URI.
2. In your code, you can query your database using the following snippets:

```python
results = data.mongodb.find(INTEGRATION_ID, COLLECTION_NAME, QUERY)

# You can also access the raw pymongo client
client = data.mongodb.get_client(INTEGRATION_ID)
```

### Salesforce
1. Before accessing Salesforce with DataBlade, you'll have to set up a special user account for DataBlade. Log into Salesforce and in **Setup**, click **Profiles** under **Administer** > **Manage Users**. Then click **New Profile**.
2. For **Existing Profile**, select "Read Only". For **Profile Name**, type "DataBlade". Click **Save**.
3. Once your Profile is created, go down to the section labeled **Login IP Ranges** and click **New**.
4. For both **Start IP Range** and **End IP Range**, enter `52.25.129.138`. Click **Save**.
5. In the left navigation, click **Users** under **Administer** > **Manage Users**. Then click **New User**.
6. For **Username**, type "datablade". For **User License**, select "Salesforce", and the under **Profile**, select "DataBlade". This is the new Profile you created in the previous steps.
7. For the remaining required values, enter as you see fit, making sure that the email you use is one that you have access to.
8. Click **Save**. You should receive an email shortly to set the password for this user.
9. After setting the password, log in as the new DataBlade user. In the top right, click your username, then click **My Settings**.
10. In the **Quick Find** field, type "token" and then click **Reset My Security Token**. Then click the **Reset Security Token** button. You will receive an email with a security token.
11. Back in DataBlade, under "Data Integrations", set up your Salesforce integration by using the username of the account you just created, the password you set, and then the token you received in your email.
12. You can now access your Salesforce objects using SOQL or SOSL using the following snippets:

```python
# Learn more about SOQL here: https://developer.salesforce.com/docs/atlas.en-us.soql_sosl.meta/soql_sosl/
results = data.salesforce.query(INTEGRATION_ID, QUERY)
```

### FTP
1. Create a new FTP integration with the host and the user credentials.
2. In your code, you can access your FTP files using the following snippets:

```python
# List the files in a directory
files = data.ftp.ls(INTEGRATION_ID, DIRECTORY)

# Get info about a specific file
file_info = data.ftp.get_info(INTEGRATION_ID, PATH)

# Get a File object (StringIO)
file_pointer = data.ftp.read_file(INTEGRATION_ID, PATH)
```

### Facebook
1. Create a new Facebook integration and authenticate with an account that has access to your Ad Accounts.
2. In your code, you can access your campaign information using the following snippets:

```python
# Get the Campaigns in your Ad Account
campaigns = data.fb.get_campaigns(INTEGRATION_ID, AD_ACCOUNT_ID)

# Get the Insights for a particular Campaign ID
insights = data.fb.get_campaign_insights(INTEGRATION_ID, CAMPAIGN_ID)

# You can also optionally pass in parameters, as documented here:
# https://developers.facebook.com/docs/marketing-api/reference/ad-campaign-group/insights/
# For example:
insights_actions_only = data.fb.get_campaign_insights(INTEGRATION_ID, CAMPAIGN_ID, fields=["actions", "cost_per_action_type"])
insights_time_range = data.fb.get_campaign_insights(INTEGRATION_ID, CAMPAIGN_ID, time_range={"start": "2016-01-08", "until": "2016-01-10"})
```

## Creating Self-Service Reports

You may find yourself in a situation where you’ve developed a workflow that you’d like other, potentially non-technical team members, to be able to leverage. We can easily do this with a self-service web report page.

Watch our video tutortial for creating self-service reports [here](https://www.youtube.com/watch?v=JUHBqlzrXqY).

DataBlade provides a SDK for receiving parameter values from self-service report runs:

```python
# For all functions, the first parameter is the key name
# (should be entered in "Parameter Keys" in the report config)
# The second parameter is the default value. You'll want to provide one in most cases.
# When you are developing in the IDE, there is no value provided for parameters, so
# the default value will always be used.
str_param = data.params.get_string("some_string_parameter", "default value")
int_param = data.params.get_int("some_int_parameter", 1)
float_param = data.params.get_float("some_float_parameter", 1.0)
bool_param = data.params.get_bool("some_bool_parameter", True)
```

### Setting a PIN code
Since the URLs for self-service reports are publicly available, we allow users to set a PIN code that must be entered before the report can be accessed. The PIN code can be configured in the Web tab in the Report configuration pane. Leaving the PIN field empty will disable PIN protection.
