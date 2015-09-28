# Getting Started with DataBlade

## Accessing Your Data

- [SQL](#sql)
- [User-Uploaded Files](#user-uploaded-files)
- [S3](#s3)
- [Google Analytics](#google-analytics)


### SQL
1. Under "Data Integrations", set up your SQL integration. We currently support PostgreSQL, MySQL, Oracle, and Microsoft SQL Server. 
2. Currently, we require that your database be accessible from external clients. In the future, we will have a solution for databases that are behind a firewall.
3. Ensure that your database is set up to receive connections from `52.25.129.138/32`.
4. In your code, you can query your SQL using the following snippets:

```python
import datablade as dbl

# You can optionally provide a keyword argument index_col="column name" to set
# the DataFrame index. You'll often want to set this to your ID column name.
r = dbl.sql.query(INTEGRATION_ID, SQL_QUERY)
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
import datablade as dbl
import json

# Read a CSV file
my_csv = dbl.read_csv(FILE_NAME)

# Read a file as a StringIO buffer. Here is an example with line-delimited JSON.
my_file = dbl.read_file(FILE_NAME)
my_dataframe = [json.loads(line) for line in my_file]
```

### S3
1. Under "Data Integrations", set up your S3 credentials
2. Take note of the `ID` value of your integration
3. In your code, access your files using the following snippets:

```python
import datablade as dbl
import json

# Read a CSV file (replace INTEGRATION_ID, BUCKET_NAME, and KEY as appropriate)
my_csv = dbl.s3.read_csv(INTEGRATION_ID, BUCKET_NAME, KEY)

# Read a file as a StringIO buffer. Here is an example with line-delimited JSON.
my_file = dbl.s3.read_file(INTEGRATION_ID, BUCKET_NAME, KEY)
my_dataframe = [json.loads(line) for line in my_file]
```

### Google Analytics
1. Ensure that you [create or select a project in the Google Developers Console and enable the Analytics API](https://console.developers.google.com//start/api?id=analytics&credential=client_key)
2. In the Developers Console for the chosen project, under **APIs & auth**, select **Credentials**. In the **Add credentials** dropdown, choose **Service account**. For **Key type**, select **P12**, then click **Create**. The private key should automatically download. Click **Close** in the dialog.
3. Copy the email address of the newly generated service account and [add it as a user](https://support.google.com/analytics/answer/1009702) to your Google Analytics account. This user will likely only require **Read & Analyze** permissions.
4. In DataBlade, set up a Google Analytics integration, providing the service email account, the generated `.p12` file, and the associated profile ID. Your profile IDs can be found in your Google Analytics account under Admin -> Select a View in the dropdown -> View Settings -> View ID. If you regularly query multiple profile IDs, set up a separate DataBlade Integration for each (you can still use the same service account for all of them).
5. In your code, query your Google Analytics data using the following snippets:

```python
import datablade as dbl

# See all available query parameters:
# https://developers.google.com/analytics/devguides/reporting/core/v3/reference?hl=en
# You do not need to pass in the `id` parameter, as DataBlade will automatically
# populate it for you.
results = dbl.ga.query(INTEGRATION_ID,
    start_date='7daysAgo',
    end_date='today',
    dimensions="ga:browser",
    metrics='ga:sessions')
```
