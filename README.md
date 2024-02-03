# Data Taps Getting Started

Step by step instructions on starting to send streaming data to Data Taps with processed data output to your S3 Bucket.

You will create an IAM Role into your AWS Account and connect Data Taps to your S3 Bucket, so that when you HTTP POST newlined delimited JSON data to the Tap URL, the data ends up into your S3 Bucket with nicely named and partitioned.

> You can see more detailed instructions [here](https://github.com/boilingdata/boilingdata-bdcli/blob/main/ONBOARDING.md).

## 1. Create account with your email

```shell
yarn install
bdcli account config
bdcli account register
# Check your email for verification code
bdcli account register --confirm <codeFromEmail>
```

## 2. Connect your account with your S3 Bucket

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
