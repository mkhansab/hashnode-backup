---
title: "What Is IMDSv2 in AWS? IMDSv1 vs IMDSv2 Explained with Examples"
seoTitle: "IMDSv1 vs IMDSv2 in AWS EC2: Security and Differences"
seoDescription: "Learn IMDSv1 vs IMDSv2 in AWS EC2, key differences, and how IMDSv2 improves security against SSRF with practical CLI and curl examples."
datePublished: 2026-04-18T06:05:42.303Z
cuid: cmo3xo84d00gq2am0de5cbp7x
slug: what-is-imdsv2-in-aws-imdsv1-vs-imdsv2-explained-with-examples
cover: https://cdn.hashnode.com/uploads/covers/6633c2252c01edc0085a6004/04ec53f9-7231-467c-b335-9f047ca827a5.jpg
tags: ec2, aws, devops, cloud-security, imdsv2, imdsv1, imds, instance-metadata-service

---

## Overview

The Instance Metadata Service (IMDS) simplifies credential management on EC2 instances by providing temporary, automatically rotated credentials. This eliminates the need to hardcode or manually distribute sensitive credentials to applications. IMDS runs locally on every EC2 instance and is accessible via the link-local IP address `169.254.169.254`. It is also accessible over IPv6 for EC2 instances built on the Nitro System.

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">By default, the IPv6 endpoint for the instance metadata service (IMDS) is disabled. You can enable the IPv6 endpoint at instance launch when the following requirements are met: 1) The selected instance type is built on the AWS Nitro System. 2) The selected subnet supports IPv6, where the subnet is either dual stack of IPv6 only. If the above requirements are not met, the field is disabled.</div>
</div>

The EC2 IMDS provides important information about each individual EC2 instance. This includes several categories of information, such as AMI ID, hostname, associated security groups, instance id, local IPv4, etc

Here's an example of what IAM credentials might look like when retrieved from the EC2 instance metadata:

```shell
$ curl http://169.254.169.254/latest/meta-data/identity-credentials/ec2/security-credentials/ec2-instance
{
  "Code" : "Success",
  "LastUpdated" : "2026-04-17T18:20:09Z",
  "Type" : "AWS-HMAC",
  "AccessKeyId" : "ASIAZYX123M4ABCD567E",
  "SecretAccessKey" : "exponentialTHlF2HmgNWsecretRNB2JXexample",
  "Token" : "IQoJb3JpluX2VjEBIaC//////wEQABoMMTzMjUyIgw8uDv8CA7wN9Y0wBwqpwSlpN654xzHp7mGsZ9gz5J8uoo2xJqG3Bc7C/1TAFZyEiSz0cN...",
  "Expiration" : "2026-04-18T00:45:39Z"
}
```

**Note**: *The above* `curl` *request uses IMDSv1, as it does not include a session token in the request header.*

* * *

## Server-side request forgery (SSRF) and the Instance Metadata Service

An attacker exploiting a Server-Side Request Forgery (SSRF) vulnerability can trick an application into making requests to the instance metadata endpoint (`169.254.169.254`). If IMDSv1 is enabled, this can allow the attacker to retrieve IAM credentials associated with the instance.

Recently, Mandiant identified a threat actor using a known vulnerability, `CVE-2021-21311`, to steal IAM credentials from EC2 instances using the metadata service post-compromise. [\[1\]](https://cloud.google.com/blog/topics/threat-intelligence/cloud-metadata-abuse-unc2903/)

IMDSv2 mitigates this risk by requiring a session token in request headers, which typical SSRF vulnerabilities cannot easily provide.

* * *

## IMDSv2

With IMDSv2, access to instance metadata is protected using session-based authentication. A session begins when a client requests a temporary token using a PUT request. This token must then be included in the header of all subsequent requests to access metadata.

This approach ensures that only requests originating from within the instance can successfully retrieve metadata, significantly reducing the risk of SSRF-based attacks.

For example, this curl recipe retrieves a session token that’s valid for the full six hours (21600 seconds) and then uses that token to access the EC2 instance’s profile metadata:

````shell
```bash
$ TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`

$ curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/identity-credentials/ec2/security-credentials/ec2-instance
```
Output:
{
  "Code" : "Success",
  "LastUpdated" : "2026-04-17T18:42:35Z",
  "Type" : "AWS-HMAC",
  "AccessKeyId" : "ASIAZYX123M4ABCD567E",
  "SecretAccessKey" : "exponentialC/wmIczuv7CGuOmm8iXPSCexample",
  "Token" : "IQoJb3JpZ2luTEiSDBGAiEA9+W761PO/4SOcqcQ3Ty4PsnK4WsSzEZP6WgCIQD4+koxgcjdEPnL9whM0R0TyZdjhFPY5cfBirTBAjc//////////8rxcf1/OykGpvb46+VEIFL+5...",
  "Expiration" : "2026-04-18T01:06:48Z"
}
````

**Note**: *The above* `curl` *request uses IMDSv2, as it includes the session token in the request header.*

* * *

## IMDSv1 vs IMDSv2

There are two versions of the Instance Metadata Service: IMDSv1 and IMDSv2.

IMDSv1 is a simple request and response protocol. When you make a request to the IMDS from the EC2 instance, you receive the result of your request. There are no additional parameters to be passed. Because of this, IMDSv1 is a perfect candidate for SSRF attacks.

IMDSv2, on the other hand, is session-oriented. This means that, before making a request to the IMDS, you must first create a session token with a PUT request and pass it along in subsequent requests inside a header. This provides an additional security benefit, as an adversary is unlikely to be able to set a request header via SSRF. IMDSv2 can be enforced during instance creation by configuring the Instance Metadata Options to require session tokens.

* * *

## How to enable IMDS during Instance creation

While creating the instance, In the Advanced Details you can configure Instance metadata options

![](https://cdn.hashnode.com/uploads/covers/6633c2252c01edc0085a6004/1ba01c8f-9244-4891-9ba8-075220bd0d07.png align="center")

* * *

## How to Verify IMDS Version in AWS EC2

Two ways:

1.  Console  
    In the EC2 console, navigate to:  
    **EC2 → Instances → Select your instance id**
    
    ![](https://cdn.hashnode.com/uploads/covers/6633c2252c01edc0085a6004/a26a16aa-443a-4cb1-9fc5-8eb2f41ed8e4.png align="left")
    
    ![](https://cdn.hashnode.com/uploads/covers/6633c2252c01edc0085a6004/12cc41e5-e552-4502-9287-63c041aa7ccc.png align="center")
    
    `HttpTokens = required` → IMDSv2 is enforced (IMDSv1 disabled)  
    `HttpTokens = optional` → Both IMDSv1 and IMDSv2 are allowed
    
2.  CLI  
    When IMDS v2 is enabled
    
    ```shell
    $ aws ec2 describe-instances --instance-ids i-0f581c1c90fc521c2 --query "Reservations[].Instances[].MetadataOptions.HttpTokens" --output table
    -------------------
    |DescribeInstances|
    +-----------------+
    |  required       |
    +-----------------+
    ```
    
    When IMDS v1 is enabled
    
    ```shell
    $ aws ec2 describe-instances --instance-ids i-0ef0ce715ba1fc912 --query "Reservations[].Instances[].MetadataOptions.HttpTokens" --output table
    -------------------
    |DescribeInstances|
    +-----------------+
    |  optional       |
    +-----------------+
    ```
    

* * *

## How to Modify Instance Metadata

1.  Console
    
    ![](https://cdn.hashnode.com/uploads/covers/6633c2252c01edc0085a6004/71c7592c-01d0-4b55-8d72-440273394c9c.png align="center")
    
2.  CLI  
    To transition from IMDSv1 to IMDSv2
    
    ```shell
    $ aws ec2 modify-instance-metadata-options --instance-id i-0ef0ce715ba1fc912 --http-endpoint enabled --http-tokens required
    {
        "InstanceId": "i-0ef0ce715ba1fc912",
        "InstanceMetadataOptions": {
            "State": "pending",
            "HttpTokens": "required",
            "HttpPutResponseHopLimit": 2,
            "HttpEndpoint": "enabled",
            "HttpProtocolIpv6": "disabled",
            "InstanceMetadataTags": "disabled"
        }
    }
    ```
    
    To transition from IMDSv2 to IMDSv1
    
    ```shell
    $ aws ec2 modify-instance-metadata-options --instance-id i-0ef0ce715ba1fc912 --http-endpoint enabled --http-tokens optional
    ```
    
    `HttpTokens = required` → IMDSv2 is enforced (IMDSv1 disabled)  
    `HttpTokens = optional` → Both IMDSv1 and IMDSv2 are allowed
    
    To disable IMDS
    
    ```shell
    $ aws ec2 modify-instance-metadata-options --instance-id i-0ef0ce715ba1fc912 --http-endpoint disabled
    ```
    

> Enforcing IMDSv2 is a simple yet effective step to improve the security posture of your EC2 instances, especially in environments where applications may be exposed to untrusted input.

### References

\[1\] Google Cloud – Cloud Metadata Abuse (UNC2903 Case Study)  
https://cloud.google.com/blog/topics/threat-intelligence/cloud-metadata-abuse-unc2903/

\[2\] Amazon Web Services – Configure Instance Metadata Service  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html

\[3\] Amazon Web Services – IMDSv2 on Amazon Linux 2023  
https://docs.aws.amazon.com/linux/al2023/ug/imdsv2.html

\[4\] Amazon Web Services Security Blog – SSRF and IMDS Protection  
https://aws.amazon.com/blogs/security/defense-in-depth-open-firewalls-reverse-proxies-ssrf-vulnerabilities-ec2-instance-metadata-service/

\[5\] Datadog Security Labs – IMDS Misconfiguration Spotlight  
https://securitylabs.datadoghq.com/articles/misconfiguration-spotlight-imds/