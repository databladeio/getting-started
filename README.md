# Getting Started with DataBlade

## Accessing Your Data

### User Uploaded Files
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
