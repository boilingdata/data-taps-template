# Deploying using AWS SAM

You can use [AWS SAM](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html#install-sam-cli-instructions) to deploy Data Taps to your AWS Account with this [data-tap.aws-sam.yaml](data-tap.aws-sam.yaml) template. Parameters are documented inside the file.

You need BoilingData account to get Tap Master secret as well as getting authentication token when sending data to the Tap. You can use [bdcli](https://github.com/boilingdata/boilingdata-bdcli) to register and manage your account as well as get tokens and run one-off SQL queries. This template requires the Data Tap master secret parameter. for this template.

To fetch the Tap master secret, you can run:

```shell
bdcli account tap-master-secret --disable-spinner | jq -r .bdTapMasterSecret
```

Replace correct values for the parameters below, as this template does not create S3 Buckets or networking configurations for you (e.g. VPC, SG, subnets, routing tables, S3 VPC Gateways, etc.).

```shell
sam validate
sam package
sam deploy --parameter-overrides \
    dataTapOutputS3Bucket=YourS3OutputBucket \
    dataTapOutputS3BucketArn=YourS3OutputBucketArn \
    bdTapTokenSecret=`bdcli account tap-master-secret --disable-spinner | jq -r .bdTapMasterSecret` \
    ownerBoilinDataEmail=yourBoilingDataAccountEmail \
    sgId=yourVpcSgForTheDataTapLambda \
    subnet1=yourAz1SubnetId \
    subnet2=yourAz2SubnetId \
    subnet3=yourAz3SubnetId
```

> **NOTE** Currently, if you make the Data Tap Lambda HA (in multiple AZs) and you use S3 Express (Directory) Bucket (resides in a single AZ), you may incur cross-AZ data transfer costs when the actual Data Tap Lambda is on another AZ.
