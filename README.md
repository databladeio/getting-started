# Getting Started with DataBlade

[![DataBlade demo video screenshot](https://github.com/databladeio/getting-started/blob/master/demo-video-screenshot.png)](https://www.youtube.com/watch?v=-sT9P5kHMf8)

## Overview Videos
* [General Overview](https://www.youtube.com/watch?v=-sT9P5kHMf8)
* [Creating Automated Email Reports](https://www.youtube.com/watch?v=ZF48QJJH_pY)
* [Creating Self-Service Dashboards](https://www.youtube.com/watch?v=JUHBqlzrXqY)

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

```python
data  # Show documentation for the all DataBlade libraries
```

```python
data.fb  # Show documentation for the DataBlade Facebook library
```

To move any snippets from the Console into your Editor, hover over the code and click the icon that appears to the left of the code.

## Before You Begin
Before you begin using DataBlade, it's important to note that nearly all of the data querying libraries provided by DataBlade (`data.*`) return results as [DataFrames](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.html). DataFrames are part of the popular pandas library and provide a very simple interface to common data manipulation tasks like filtering, mapping, merging, etc.

If you are not familiar with pandas, check out this [introduction](http://synesthesiam.com/posts/an-introduction-to-pandas.html
).

With that said, DataBlade provides you all the power and flexibility of a full-featured Python environment, so if you'd rather define your workflows using other techniques or libraries, you are absolutely free to do so. Here is a [list](https://gist.github.com/allenpc/0dcddadd121a778c7aadc158bafb901e) of Python modules that are currently available to import in the environment. If you have additional libraries you'd like to see added, [let us know](mailto:support@datablade.io).

## Accessing Your Data

- [SQL](#sql)
- [User-Uploaded Files](#user-uploaded-files)
- [S3](#s3)
- [Google Analytics](#google-analytics)
- [Google BigQuery](#google-bigquery)
- [Google AdWords](#google-adwords)
- [Google Sheets](#google-sheets)
- [MongoDB](#mongodb)
- [RethinkDB](#rethinkdb)
- [Salesforce](#salesforce)
- [FTP](#ftp)
- [Facebook](#facebook)
- [Shopify](#shopify)
- [Other Data Sources](#other-data-sources)

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

# You can also query any existing DataFrames that are in memory as if they were tables
# The DataFrames you refer to in your query must be declared prior to the `query_local`
# call.
my_df = pd.DataFrame(np.random.randn(6,4), columns=list('ABCD'))
result = data.sql.query_local("SELECT A, B FROM my_df")
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

# Read a single-sheet Excel file (.xls or .xlsx) (or the first sheet of a workbook)
my_excel_file = data.read_excel(FILE_NAME)

# Read an entire Excel workbook
my_excel_file = data.read_excel(FILE_NAME, sheetname=None)

# Read a specific sheet from an Excel workbook
my_excel_file = data.read_excel(FILE_NAME, sheetname='Sheet Name')  # May also be a numerical index

# Read a file as a StringIO buffer. Here is an example with line-delimited JSON.
my_file = data.read_file(FILE_NAME)
my_dataframe = [json.loads(line) for line in my_file]

# Write back to your user file store
# (creates a new file or overwrites existing)
# data.write_file(FILE_NAME, FILE_OBJECT)
# Example:
import StringIO
my_df = pd.DataFrame(np.random.randn(6,4), columns=list('ABCD'))
my_df_csv = StringIO.StringIO(my_df.to_csv(index=False))
data.write_file("random.csv", my_df_csv)
```

### S3
1. Under "Data Integrations", set up your S3 credentials
2. Take note of the `ID` value of your integration
3. In your code, access your files using the following snippets:

```python
# For all: Replace INTEGRATION_ID, BUCKET_NAME, and KEY as appropriate

# Read a CSV file
my_csv = data.s3.read_csv(INTEGRATION_ID, BUCKET_NAME, KEY)

# Read a single-sheet Excel file (.xls or .xlsx) (or the first sheet of a workbook)
my_excel_file = data.s3.read_excel(INTEGRATION_ID, BUCKET_NAME, KEY)

# Read an entire Excel workbook
my_excel_file = data.s3.read_excel(INTEGRATION_ID, BUCKET_NAME, KEY, sheetname=None)

# Read a specific sheet from an Excel workbook
my_excel_file = data.s3.read_excel(INTEGRATION_ID, BUCKET_NAME, KEY, sheetname='Sheet Name')  # May also be a numerical index

# Read a file as a StringIO buffer. Here is an example with line-delimited JSON.
my_file = data.s3.read_file(INTEGRATION_ID, BUCKET_NAME, KEY)
my_dataframe = [json.loads(line) for line in my_file]

# Write a File object to S3
data.s3.write_file(INTEGRATION_ID, BUCKET_NAME, KEY, FILE_OBJECT)
```

### Google Analytics
1. In DataBlade, set up a Google Analytics integration by providing a name for the integration and then clicking **Authenticate with Google**. Follow the prompts that appear to log in.
2. In your code, query your Google Analytics data using the following snippets:

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

### Google Sheets
1. In DataBlade, set up a Google Sheets integration by providing a name for the integration and then clicking **Authenticate with Google**. Follow the prompts that appear to log in.
2. In your code, query your Google Sheets data using the following snippets:

```python
# The spreadsheet ID can be found in the URL of the spreadsheet
# Example: https://docs.google.com/spreadsheets/d/1AFtzJQxZ_Vhr2ikzwnBpstnf-tM9V-MMus9wT7LqXXX/edit
# The ID is located between /d/ and /edit, i.e. 1AFtzJQxZ_Vhr2ikzwnBpstnf-tM9V-MMus9wT7LqXXX
sheet = data.sheets.get_spreadsheet(INTEGRATION_ID, SPREADSHEET_ID)
```

### MongoDB
1. Under "Data Integrations", set up your MongoDB integration. In **Connection String**, make sure to also include your database name in the URI.
2. In your code, you can query your database using the following snippets:

```python
results = data.mongodb.find(INTEGRATION_ID, COLLECTION_NAME, QUERY)

# You can also access the raw pymongo client
client = data.mongodb.get_client(INTEGRATION_ID)
```

### RethinkDB
1. Under "Data Integrations", set up your RethinkDB integration
2. In your code, you can query your database using the following snippets:

```python
import rethinkdb as r

# At the moment, you must execute all your commands within a with block
with data.rethinkdb.get_connection(INTEGRATION_ID) as conn:
    result = r.table("mytable").run(conn)
    
# Alternatively, you can bypass the managed data integrations and use the driver directly
conn = r.connect(...)
```

### Salesforce
1. Under "Data Integrations", set up your Salesforce integration by giving your integration a name, and then click **Authenticate with Salesforce** and follow the prompts that appear.
2. You can now access your Salesforce objects using SOQL or SOSL using the following snippets:

```python
# Learn more about SOQL here: https://developer.salesforce.com/docs/atlas.en-us.soql_sosl.meta/soql_sosl/
results = data.salesforce.query(INTEGRATION_ID, QUERY)
```

**NOTE:** Your organization must be using an edition of Salesforce that has API access enabled. See [here](https://help.salesforce.com/apex/HTViewSolution?urlname=Enabling-API&language=en_US) for more info.

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

# Get Insights for all Campaigns in your Ad Account
all_insights = data.fb.get_all_campaign_insights(INTEGRATION_ID, AD_ACCOUNT_ID)

# You can also optionally pass in parameters, as documented here:
# https://developers.facebook.com/docs/marketing-api/reference/ad-campaign-group/insights/
# These parameters will work for both get_campaign_insights and get_all_campaign_insights
# For example:
insights_actions_only = data.fb.get_campaign_insights(INTEGRATION_ID, CAMPAIGN_ID, fields=["actions", "cost_per_action_type"])
insights_time_range = data.fb.get_campaign_insights(INTEGRATION_ID, CAMPAIGN_ID, time_range={"start": "2016-01-08", "until": "2016-01-10"})
```

### Shopify
1. Create a new Shopify integration by providing it a name and your shop. Click **Authenticate with Shopify** and follow the prompts that appear.
2. In your code, you can access your Shopify store information using the following snippets:

```python
# Get your Session and set up the Shopify Python SDK
import shopify
session = data.shopify.get_session(INTEGRATION_ID)
shopify.ShopifyResource.activate_session(session)

# You can now query your Shopify objects
# See additional documentation: http://shopify.github.io/shopify_python_api/
all_products = shopify.Product.find()

# Coming soon: APIs to load your Shopify objects directly into DataFrames
```

### Other Data Sources

If you want to connect to other data sources that DataBlade doesn't support yet, please [send us a data source addition request at support@datablade.io](mailto:support@datablade.io?subject=New%20data%20source%20request)! We have a bunch of new data sources in our pipeline, but we're happy to prioritize what's important for you.

If you need to access an unsupported data source more quickly than we can get back to you and don't mind getting your hands dirty, you can try using `pip install` directly. Here's an example of installing the [Stripe](https://stripe.com/docs/api/python) API using `pip install`:

```
import subprocess
subprocess.check_output(["pip", "install", "stripe"])
import stripe
```


## Creating Self-Service Dashboards

You may find yourself in a situation where you’ve developed a workflow that you’d like other, potentially non-technical team members, to be able to leverage. We can easily do this with a Self-Service Dashboard.

[Watch our video tutorial for creating Self-Service Dashboards here](https://www.youtube.com/watch?v=JUHBqlzrXqY).

DataBlade provides a SDK for receiving parameter values from Self-Service Dashboard runs:

```python
# For all functions, the first parameter is the key name
# (should be entered in "Parameter Keys" in the Dashboard config)
# The second parameter is the default value. You'll want to provide one in most cases.
# When you are developing in the IDE, there is no value provided for parameters, so
# the default value will always be used.
str_param = data.params.get_string("some_string_parameter", "default value")
int_param = data.params.get_int("some_int_parameter", 1)
float_param = data.params.get_float("some_float_parameter", 1.0)
bool_param = data.params.get_bool("some_bool_parameter", True)
```

### Setting a PIN code
Since the URLs for Self-Service Dashboards are publicly available, we allow users to set a PIN code that must be entered before the report can be accessed. The PIN code can be configured in the Web tab in the Dashboard configuration pane. Leaving the PIN field empty will disable PIN protection.

## API Keys

API Keys can be created and managed from the **Settings** page. Note that you must be a part of an organization before creating an API key and that the keys you create are specific to an organization, which will limit what resources each API key will have access to.

## Metadata Store

Each project has a simple key-value store that can hold arbitrary JSON, making it very easy to store and retrieve state. The metadata store is also a handy place to store values that need to be accessed by an external tool, since metadata values are accessible via our API.

*Keep in mind that the metadata store should not be used to store large amounts of raw data. For that, use your [user file store](#user-uploaded-files) or [S3](#s3).*

### Setting and Retrieving Metadata

```python
data.set_metadata(KEY, VALUE)
data.get_metadata(KEY, VALUE)
```

### Querying Project Metadata via API

**Getting all keys & values**
```
https://app.datablade.io/api/v1/projects/<PROJECT_ID>/metadata?access_token=<API_KEY>
```

**Getting specific key**
```
https://app.datablade.io/api/v1/projects/<PROJECT_ID>/metadata?access_token=<API_KEY>&key=<KEY>
```

## Keyboard shortcuts

### Editor and Console

- Run main code: `Ctrl-r` (Windows), `Cmd-r` (Mac)
- Save (Auto-save works, too): `Ctrl-s` (Windows), `Cmd-s` (Mac)
- Outdent lines: `Ctrl-[` (Windows), `Cmd-[` (Mac)
- Indent lines: `Ctrl-]` (Windows), `Cmd-]` (Mac)
- Go to beginning of line: `Ctrl-a`
- Go to end of line: `Ctrl-e`
- Clear to beginning of line: `Ctrl-u`
- Clear to end of line: `Ctrl-k`
- Search: `Ctrl-f` (Windows), `Cmd-f` (Mac)
- Clear error and output popups: `Esc`

### Console only

- Previous item in history: `Ctrl-p` (Mac)
- Next item in history: `Ctrl-n` (Mac)
