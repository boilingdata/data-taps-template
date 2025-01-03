# Deploying using AWS SAM

BoilingData product called [Data Taps](https://www.taps.boilingdata.com/) are AWS Lambda functions with Function URL and running code as packaged in this repository (i.e. custom C++ runtime and handler developed by BoilingData and distributed as binary packages so that you can deploy them into your own AWS Account)

[BoilingData](https://www.boilingdata.com/) cloud refers to the "SQL over S3" and you can use it to run blazing fast analytics (DuckDB) over the data that your Data Taps collect if you like.

## Pre-requirements

1. You need access to an AWS account with credentials allowing you to create e.g. Lambda functions as this AWS SAM template deploys Data Taps there. If you want to deploy to BoilingData cloud, follow the instructions in the main [README.md](../README.md) of this repository.

2. Install [AWS SAM](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html#install-sam-cli-instructions) to deploy Data Taps to your AWS Account with this [data-tap.aws-sam.yaml](data-tap.aws-sam.yaml) template.

3. Install [bdcli](https://github.com/boilingdata/boilingdata-bdcli/blob/main/ONBOARDING.md) and register to BoilingData with it. You will have your username and password for using BoilingData with various SDKs as well ([JS](https://github.com/boilingdata/node-boilingdata), [Python](https://github.com/boilingdata/py-boilingdata)). Bdcli will store the credentials as well as cache session tokens into your home directory in a file named `~/.bdcli.yaml`. You can use multiple profiles (`BD_PROFILE`) if you like (e.g. if you build Data Mesh or want to test data sharing between users locally from your laptop and thus having multiple profiles in `~/.bdcli.yaml`).

> NOTE: You can use bdcli to run one-off SQL queries, for example over the data that is collected by your Data Taps into your S3 Bucket(s). You can also use BoilingData SDKs for running analytics from your code. We recommend using S3 Express buckets for the Data Taps output, especially if you want to run low latency queries with Boiling, however, normal S3 Buckets work also very fast as Boiling reads the data from S3, decompresses it and indexes it on-the-fly with DuckDB (which incurs initial query latency when this data lifting happens into the in-memory optimised plane).

## Deploy

> You only need to set the `S3_BUCKET_ARN` environment variable, other required parameters are fetched with [bdcli](https://github.com/boilingdata/boilingdata-bdcli). The CloudFormation stack name on your AWS Account is configured in the [samconfig.toml](samconfig.toml) file.

**NOTE**: Clients sending data to the Data Tap also need a valid BoilingData authentication token and set it as `x-bd-authorization` HTTP header when sending newline JSON to Taps.

```shell
# 1. Set S3_BUCKET_ARN env
if [ -z "${S3_BUCKET_ARN}" ];
then
    echo "==> Please set S3_BUCKET_ARN environment variable"
    return
fi

# 2. Set your BoilingData account username as BD_USERNAME env
export BD_USERNAME=`bdcli account config --dump default --disable-spinner | jq -r .credentials.email`

# 3. Fetch your BoilingData Data Taps master secret into BD_TAP_MASTER_SECRET env
export BD_TAP_MASTER_SECRET=`bdcli account tap-master-secret --disable-spinner | jq -r .bdTapMasterSecret`

# 4. Validate and lint the SAM template
sam validate

# 5. Package for deployment
sam package

# 6. Deploy your Data Tap into your AWS Account
sam deploy --parameter-overrides \
    ownerBoilinDataEmail=${BD_USERNAME} \
    dataTapOutputS3BucketArn=${S3_BUCKET_ARN} \
    bdTapTokenSecret=${BD_TAP_MASTER_SECRET} \
```

_NOTE: Currently, if you make the Data Tap Lambda HA (in multiple AZs) and you deploy to VPC and use S3 Express (Directory) Bucket (resides in a single AZ), you may incur cross-AZ data transfer costs when the actual Data Tap Lambda is on another AZ. To avoid that you would need an S3 Express bucket per AZ. However, Data Taps works very well with normal S3 Buckets. However, the analytics over S3 Express is faster due to the minimal latency. So, if you're not concerned with potential cross AZ data transfer costs, using S3 Express is recommended._

## Make clients send Data to the Tap

Get fresh client Tap token as `TAP_TOKEN` environment variable and cached onto `.taptoken` file.

```shell
( printf "export TAP_TOKEN=" && bdcli account tap-token --lifetime 24h --disable-spinner | jq -r .bdTapToken ) > .taptoken
source .taptoken
```

Send data with e.g. curl.

```shell
export TAP_URL='...' # You get the URL from the deployment when the Tap is created
echo -e '{"hello":"world"}\n{"hello":"world2"}' > data.json
curl -H "Content-Type:application/x-ndjson" -H "x-bd-authorization: ${TAP_TOKEN}" --data-binary @data.json ${TAP_URL}
```

## Check the Parquet output files on your S3

By default when a Data Tap has collected 1 MB data or after 1 minute of last sync, it will output the data to your S3 Bucket.

> You can install aws cli tool by following the instructions here: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

```shell
aws s3 ls --recursively s3://YOURBUCKET/YOURPREFIX
```
