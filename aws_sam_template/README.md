# Deploying using AWS SAM

You need BoilingData account to get Tap Master secret as well as getting authentication token when sending data to the Tap. You can use [bdcli](https://github.com/boilingdata/boilingdata-bdcli) to register and manage your account as well as get tokens and run one-off SQL queries.

The Data Tap master secret that is needed as a parameter for this [SAM](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html#install-sam-cli-instructions) template.

Replace correct values for the parameters below.

**NOTE** Currently, if you make the Data Tap Lambda HA (in multiple AZs) and you use S3 Express (Directory) Bucket (resides in a single AZ), you may incur cross-AZ data transfer costs when the actual Data Tap Lambda is on another AZ.

```shell
sam validate --lint --template-file data-tap.aws-sam.yaml

sam deploy --guided --template-file data-tap.aws-sam.yaml \
    --stack-name data-taps-demo \
    --capabilities CAPABILITY_IAM \
    --parameter-overrides \
        zipCodeUri=uploadedDataTapZipS3Uri \
        extensionS3Bucket=YourS3ExtensionBucket \
        dataTapOutputS3Bucket=YourS3OutputBucket \
        dataTapOutputS3BucketArn=YourS3OutputBucketArn \
        bdTapTokenSecret=yourBdApiTapMasterSecret \
        ownerBoilinDataEmail=yourBoilingDataAccountEmail \
        sgId=yourVpcSgForTheDataTapLambda \
        subnet1=yourAz1SubnetId \
        subnet2=yourAz2SubnetId \
        subnet3=yourAz3SubnetId
```
