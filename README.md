# Data Taps

## Tailor made C++ runtime, bootstrap, handler, and extension

Data Taps are tailor made AWS Lambda functions with Function URL as the Tap ingestion point. Taps are made of custom C++ runtime and handler code with embedded DuckDB. They run efficiently with the smallest ARM64 Lambda (128MB) and provide unparalleled scalability, cost efficiency, and stable low latency. A Data Tap collects data into the Lambda by running atomic filesystem append commands and depending on the incoming data packet size completes even below `2ms`. Accruing and buffering the data does not require processing power, except for verifying the JWT token for authentication and access control purposes.

When thresholds are reached, DuckDB is used to stream process the newline delimited JSON files into S3 as ZSTD compressed Parquet files. This is where the data processing and when data upload happens. A single SQL statement is used to process the data. Like for an example below, with no actual transformations but just data format conversion from NDJSON to Parquet. This network upload is also the most time consuming part with small AWS Lambda functions and dominating factor when calculating costs. There are ways to optimise this part.

```sql
COPY (
    SELECT CAST(filename[17:27] AS INT32) AS __bd_ts, * EXCLUDE(filename)
      FROM read_json_auto('/tmp/json_files/*.json', ignore_errors=true, auto_detect=true, format=newline_delimited, filename=true, maximum_depth=2, union_by_name=true)
)
TO '%s' WITH ( FORMAT 'Parquet', COMPRESSION 'ZSTD' )
```

The output filename is a `strftime()` path on S3 to get hive partitioned output by `year`/`month`/`day`/`hour`.

> NOTE: An accompanying AWS Lambda extension is used to hook into the AWS Lambda lifecycle events to flush remaining data when AWS Lambda shuts down.

## Getting Started

Step by step instructions on starting to send streaming data to Data Taps with processed data output to your S3 Bucket.

You will create an IAM Role into your AWS Account and connect Data Taps to your S3 Bucket, so that when you HTTP POST newlined delimited JSON data to the Tap URL, the data ends up into your S3 Bucket nicely named and partitioned.

> You can see more detailed instructions [here](https://github.com/boilingdata/boilingdata-bdcli/blob/main/ONBOARDING.md).

> NOTE! Data Taps is deployed to the following AWS Regions: `eu-west-1`, `eu-north-1`, `us-west-2`, and `us-east-2`. You can deploy to any of these regions by changing the template `region:` key. However, your data output S3 Bucket must reside on the same AWS Region.

## 1. Create Boiling account with your email

```shell
yarn install
npx bdcli account config
npx bdcli account register
# Check your email for verification code
npx bdcli account register --confirm <codeFromEmail>
```

## 2. Connect your Boiling account with your S3 Bucket

Open the example IaC template [data-taps-iac.yaml](data-taps-iac.yaml) and update the following:

- Replace `YOURBUCKET` with your S3 Bucket name
- Replace `PREFIX` with your S3 prefix ("folder")
- Replace `YOUREMAIL` with your email you used for registering

```shell
yarn validate
yarn connect
yarn upload
yarn plan
yarn deploy
```

## 3. Get token and ingestion URL and send data

```shell
yarn taptoken
yarn tapurl
yarn test
yarn test:loop
```

## 4. Check data output to YOURBUCKET/PREFIX

The tumbling window in the example template is 1 minute. After couple of minutes you should start seeing data in your S3 Bucket.

```shell
aws s3 ls --recursive s3://YOURBUCKET/PREFIX
```

## 5. You can also use Boiling Data to query your S3 Data at rest or run Streaming SQL

You can `SUBSCRIBE` live to your Data Taps with `SQL` if you like, or query your data from S3.

For more information, check out [https://www.boilingdata.com/](https://www.boilingdata.com).
