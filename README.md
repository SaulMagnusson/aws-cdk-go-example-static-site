<!--BEGIN STABILITY BANNER-->
---

![Stability: Stable](https://img.shields.io/badge/stability-Stable-success.svg?style=for-the-badge)

> **This is a stable example. It should successfully build out of the box**
>
> This example is built on Construct Libraries marked "Stable" and does not have any infrastructure prerequisites to build.
---
<!--END STABILITY BANNER-->

NOTICE: Go support is still in Developer Preview. This implies that APIs may change while we address early feedback from the community. We would love to hear about your experience through GitHub issues.

Throughout the code all property fields are exposed although many contain "nil" values or references to undeclared slices. Some of these fields are deprecated, check the [aws-cdk API reference](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-construct-library.html). By default, go functions take a copy of arguments, thus in aws-cdk-go, references are passed by taking the address of the variable with "&" the address operator. Reference types are indicated by "*" thus &"hello", a reference to the string "hello" is of type *string. jsii.String("") and other functions are helper functions that return references to their argyments. See [Working with the AWS CDK in Go](https://docs.aws.amazon.com/cdk/latest/guide/work-with-cdk-go.html) and the blog post [Getting started with the AWS Cloud Development Kit and Go](https://aws.amazon.com/blogs/developer/getting-started-with-the-aws-cloud-development-kit-and-go/)

This example launches a secure static site hosted in an S3 bucket distributed by CloudFront, protected by a certificate managed by ACM, and with its uri's automatically rewritten by a CloudFront Function (e.g. a user request for example.com is routed to example.com/index.html). 

Constructs used:
- awss3
- awss3-deployments
- awscloudfront
- awscloudfrontorigins
- awscertificatemanager
- awsroute53


##Deploy

Th variables DOMAIN_NAME and ASSET_PATH must be filled in manually. DOMAIN_NAME must be a domain name the user owns (example.com) and it must have a hosted zone with only the original NS and SOA records in it. The certificate will be automatically validated from this zone and a new record will be created in it pointing to the cloudfront distribution.

ASSET_PATH contains the path on the users local directory to the html files for the static site that will be uploaded to an S3 bucket. 

This code requires a bootstrapping step where local files are uploaded into a staging bucket. The staging bucket is named after a hash of the source files, making this a one time step unless the source files change. This bucket will also contain ~10MiB of aws-cli files.

```
cdk bootstrap
cdk synth  
cdk deploy
```




Configuring the CloudFront distribution


Next a CloudFront distribution is created. It will link with several constructs: a bucket origin as a source, a certificate managed by ACM, a Cloudfront Function association which will rewrite the URI's, and an Origin Access Identity that will be used as a Principal for permissions in an S3 bucket policy added to the bucket.

Adding a record to Route53

Finally, a record must be added to the Hosted Zone where the domain is located. A hosted zone lookup can be used for this procedure. Looking up the Hosted Zone by ID does not work for this step because the zone name is obscured in that function.