# Encrypting Simple Notification Service(SNS) Topic using Key Management Service(KMS)

AWS Key Management Service (KMS) is an encryption and key management service scaled for the cloud. KMS keys and functionality are used by other AWS services, and you can use them to protect data in your own applications that use AWS.

Amazon SNS provides message encryption in transit, based on Amazon Trust Services (ATS) certificates, as well as message encryption at rest, using AWS Key Management Service (AWS KMS) keys.

# Message encryption in transit

 Amazon SNS API is served through Secure HTTP (HTTPS) and encrypts all messages in transit with Transport Layer Security (TLS) certificates issued by ATS(Amazon Trust Services). These certificates verify the identity of the Amazon SNS API server whenever an encrypted connection is established.

# Message encryption at rest
  Amazon SNS supports encrypted topics. When you publish messages to encrypted topics, Amazon SNS uses customer master keys (CMK), powered by AWS KMS, to encrypt your messages. Amazon SNS supports customer-managed as well as AWS-managed CMKs. As soon as Amazon SNS receives your messages, the encryption takes place on the server.

  The messages are stored in encrypted form across multiple Availability Zones (AZs) for durability and are decrypted just before being delivered to subscribed endpoints, such as Amazon Simple Queue Service (Amazon SQS) queues, AWS Lambda functions, and HTTP and HTTPS webhooks.

#### General Points
 - Amazon SNS encrypts only the body of the messages that you publish. It doesn’t encrypt message metadata (identifier, subject, timestamp, and attributes), topic metadata (name and attributes), or topic metrics. Thus, encryption doesn’t affect the operations of Amazon SNS, such as message fanout and message filtering.
 - Amazon SNS uses envelope encryption internally. It uses your configured CMK to generate a short-lived data encryption key (DEK) and then reuses this DEK to encrypt your published messages for 5 minutes. When the DEK expires, Amazon SNS automatically rotates to generate a new DEK from AWS KMS.

# Creating, subscribing and publishing to encrypted topics
You can create an Amazon SNS encrypted topic by setting its attribute KmsMasterKeyId, which expects an AWS KMS key identifier. The key identifier can be a key ID, key ARN, or key alias. You can use the identifier of either a customer-managed CMK, such as alias/MyKey, or the AWS-managed CMK in your account, whose alias is alias/aws/sns.

If you want to use a customer-managed CMK, a CMK needs to be created and secured by granting the publisher access to the same AWS KMS operations ```GenerateDataKey``` and ```Decrypt```. The access permission is granted using KMS key policies. The following JSON document shows an example policy statement for the customer-managed CMK

# Enabling compatibility between encrypted topics and event sources
Several AWS services publish events to Amazon SNS topics. To allow these event sources to work with encrypted topics, you must first create a customer-managed CMK and then add the following statement to the policy of the CMK.
```
Json
{
   "Version": "2012-10-17",
      "Statement": [{
         "Effect": "Allow",
         "Principal": {
            "Service": "s3.amazonaws.com"
         },
         "Action": [
            "kms:GenerateDataKey*",
            "kms:Decrypt"
         ],
         "Resource": "*"
       }]
}
```

You can use the following example service principals in the statement:

| Event Source | Service Principal |
|------|-------------|
| Amazon CloudWatch Events | events.amazonaws.com |
| Amazon DynamoDB |	dynamodb.amazonaws.com |
| Amazon S3 Glacier	| glacier.amazonaws.com |
| Amazon Redshift |	redshift.amazonaws.com |
| Amazon Simple Email Service |	ses.amazonaws.com |
| Amazon Simple Storage Service |	s3.amazonaws.com |
| AWS CodeCommit	| codecommit.amazonaws.com |
| AWS Database Migration Service |	dms.amazonaws.com |
| AWS Directory Service	| ds.amazonaws.com |
| AWS Snowball	| importexport.amazonaws.com |

Some Amazon SNS event sources require you to provide an IAM role (rather than the service principal) in the AWS KMS key policy:

- Amazon EC2 Auto Scaling events
- Amazon Elastic Transcoder events
- AWS CodePipeline events
- AWS Config events
- AWS Elastic Beanstalk events
- AWS IoT events

Once the CMK key policy has been configured, you can enable encryption on the topic using the CMK, and then provide the encrypted topic’s ARN to the event source.
