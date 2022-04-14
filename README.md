# Certificate Validator

[![Travis CI](https://travis-ci.org/NickolasHKraus/certificate-validator.svg?branch=master)](https://travis-ci.org/NickolasHKraus/certificate-validator)
[![Codecov](https://img.shields.io/codecov/c/github/NickolasHKraus/certificate-validator)](https://codecov.io/gh/NickolasHKraus/certificate-validator)
[![Releases](https://img.shields.io/github/v/release/NickolasHKraus/certificate-validator?color=blue)](https://github.com/NickolasHKraus/certificate-validator/releases)
[![MIT License](https://img.shields.io/badge/License-MIT-blue.svg)](https://github.com/NickolasHKraus/certificate-validator/blob/master/LICENSE)

Certificate Validator is an AWS CloudFormation custom resource which facilitates AWS Certificate Manager (ACM) certificate validation via DNS.

## Overview

Certificate Validator solves a common problem:

>*AWS CloudFormation does not provide a means for automatically validating AWS Certificate Manager (ACM) certificates.*

From the [`AWS::CertificateManager::Certificate`](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-certificatemanager-certificate.html) documentation:

>**Important**
>
>When you use the `AWS::CertificateManager::Certificate` resource in an AWS CloudFormation stack, the stack will remain in the `CREATE_IN_PROGRESS` state. Further stack operations will be delayed until you validate the certificate request, either by acting upon the instructions in the validation email, or by adding a CNAME record to your DNS configuration.

## Getting Started

Check out the [Getting Started](https://github.com/NickolasHKraus/certificate-validator/blob/master/docs/getting-started.md) documentation to start using Certificate Validator.

## Validating a certificate with DNS

When you use the `AWS::CertificateManager::Certificate` resource in an AWS CloudFormation stack, the stack will remain in the `CREATE_IN_PROGRESS` state and any further stack operations will be delayed until you validate the certificate request. Certificate validation can be completed either by acting upon the instructions in the certificate validation email or by adding a CNAME record to your DNS configuration.

The **Status Reason** for your CloudFormation deploy will contain the following:

```
Content of DNS Record is: {Name: _x1.<domain-name>.com.,Type: CNAME,Value: _x2.acm-validations.aws.}
```

Where `x1` and `x2` are random hexadecimal strings.

To automate DNS validation, you can use [this](https://github.com/NickolasHKraus/cloudformation-templates/blob/master/static-website/dns-validation.sh) script.

```bash
./dns-validation.sh $DOMAIN_NAME $STACK_NAME
```

However, this is an inelegant solution.

## Automation limitations with DNS validation

Since CloudFormation only outputs the **Name** and **Value** for the validation of the root domain name (`DomainName`), any other subdomain (`SubjectAlternativeNames`) that you wish to validate (ex. www), must be manually validated using the **Name** and **Value** given in the [AWS Management Console](https://console.aws.amazon.com/acm).

If you want your service to be accessible via HTTPS on *both* the www subdomain and root domain, you will need to add an alternate name to the certificate and determine the **Name** and **Value** to validate the www subdomain manually:

```yaml
CertificateManagerCertificate:
  Type: AWS::CertificateManager::Certificate
  Properties:
    DomainName: !Ref DomainName
    SubjectAlternativeNames:
      - !Sub 'www.${DomainName}'
    ValidationMethod: DNS
```

You will then be able to add the www subdomain to the CloudFront distribution:

```yaml
CloudFrontDistribution:
  Type: AWS::CloudFront::Distribution
  Properties:
    DistributionConfig:
      Aliases:
        - !Ref DomainName
        - !Sub 'www.${DomainName}'
```

**Note**: DNS validation can be done manually via the AWS Management Console: **Certificate Manager** > **Create record in Route 53**.

## Subject Alternative Name

**Subject Alternative Name** (SAN) is an extension to [X.509](https://en.wikipedia.org/wiki/X.509) that allows various values to be associated with a security certificate using a `subjectAltName` field. These values are called *Subject Alternative Names* (SANs). Names include:
 * Email addresses
 * IP addresses
 * URIs
 * DNS names (this is usually also provided as the Common Name RDN within the Subject field of the main certificate.)
 * directory names (alternative Distinguished Names to that given in the Subject)
 * other names, given as a General Name: a registered object identifier followed by a value

## Development

### Installation

#### Serverless

Install Node.js and NPM:

```bash
brew install node
```

Install the Serverless Framework open-source CLI:

```bash
npm install -g serverless
```

#### Python

Create a new virtual environment:

```bash
mkvirtualenv certificate-validator
```

Install requirements:

```bash
pip install -r certificate_validator/requirements_dev.txt
```

## Deployment

Deploy Certificate Validator:

```bash
make deploy
```

**Note**: An optional `STAGE` variable can be used to specify the *stage*. Defaults to `dev`.

**Example**

```bash
make deploy STAGE=prod
```

To remove Certificate Validator, run `make remove`.

**Note**: An optional `STAGE` variable can be used to specify the *stage*. Defaults to `dev`.

**Example**

```bash
make remove STAGE=prod
```

Use [bumpversion](https://pypi.org/project/bumpversion/) to increment the current version:

```bash
cd certificate_validator
bumpversion <major | minor | patch>
```
