# Continuous Monitoring

> Continuous Monitoring is a risk management approach to cybersecurity that maintains an accurate picture of an agency’s security risk posture, provides visibility into assets, and leverages use of automated data feeds to quantify risk, ensure effectiveness of security controls, and implement prioritized remedies. - [https://cio.gov/protect/continuous-monitoring](https://cio.gov/protect/continuous-monitoring/)

## Summary
CivicActions has validated the deployment of the nebula application in a configuration that includes continuous monitoring. 

This configuration includes:
* deploying the application onto 18F's FISMA-READY Ubuntu LTS (See: https://github.com/fisma-ready/ubuntu-lts)
* automated scanning with SCAP
* identification of vulnerability feeds
* monitoring of server status

This document provides evidence of integrating security and continuous monitoring into the agile delivery of IT services. 

## 18F's FISMA-READY Ubuntu LTS 
[18F's FISMA-READY Ubuntu LTS](https://github.com/fisma-ready/ubuntu-lts), provides guidance for a "hardened, FISMA Ready Ubuntu LTS Amazon Machine Instances (AMIs) that are suitable for use in Amazon Web Services (AWS)" that "inherit AWS controls assessed by the FedRAMP program" when instantitated in AWS's  US-East or US-West regions.

We cloned the FISMA-Ready LTS Ubuntu repository and ran the `ami.sh` script to generate the publicly available FISMA-Ready LTS Ubuntu 14.04 AMI (`ami-b7393887`) in AWS region `us-west-2`.

We then validated our automated deployment worked using the FISMA-Ready LTS Ubuntu AMI `ami-b7393887` instead of the default Docker Ubuntu 14.04 AMI (`ami-7f675e4f`) by setting AWS environmental variables during deployment:
```bash
export AWS_DEFAULT_REGION=us-west-2
export AWS_AMI=ami-b7393887
export AWS_ROOT_SIZE=30
```

## Automated Scanning with SCAP
To automate scanning of operating system configuration we used GovReady Ubuntu 14.04 SCAP content (see: https://github.com/GovReady/scap-fisma-ready-ubuntu-lts) and JOVAL's Professional scanner.

The SCAP datastream file used for the scan is `Ubuntu_14.04_LTS_Server_Datastream_v0.0.1.xml`.

*(NOTE: Open source licensed SCAP content for open source software is currently rare. GovReady Ubuntu 14.04 SCAP is the first open source SCAP content for Ubuntu and was developed specifically to test FISMA-Ready Ubuntu LTS. The FISMA-Ready SCAP contains OVAL (Open Vulnerability Assessment Language) schema currently being reviewed by OVAL board and can only be tested with a special version of JOVAL's Professional scanner.) No open source SCAP currently exists for Docker, although a Security Benchmark for Docker 1.6 was recently published in April 2015 by the Center for Internet Security (https://web.nvd.nist.gov/view/ncp/repository/checklistDetail?id=589). CIS's Docker Security Benchmark cannot be included in this repository due to licensing restrictions.)*

### Screenshots
The following screenshots are to provide evidence of continuous monitoring using 18F FISMA-Ready Ubuntu LTS and SCAP.

*`screenshot-deployed-52.11.154.239.png` - screenshot of http://52.11.154.239 showing deployment of application*
![screenshot of http://52.11.154.239 showing deployment of application](screenshot-deployed-52.11.154.239.png)

*`screenshot-ssh-52.11.154.239.png` - screenshot of SSH access to `52.11.154.239` showing Docker HOST OS is 18F FISMA-Ready Ubuntu LTS*
![screenshot of SSH access to `52.11.154.239](screenshot-ssh-52.11.154.239.png)

*`screenshot-scan-benchmark-52.11.154.239.png` - screenshot of using GovReady's FISMA-Ready Unbuntu LTS Benchmark SCAP `Ubuntu_14.04_LTS_Server_Datastream_v0.0.1.xml` to scan host `52.11.154.239` using JOVAL scanner*
![screenshot of using GovReady's FISMA-Ready Unbuntu LTS Benchmark SCAP `Ubuntu_14.04_LTS_Server_Datastream_v0.0.1.xml` to scan host `52.11.154.239` using JOVAL scanner](screenshot-scan-benchmark-52.11.154.239.png)

*`screenshot-scanning-52.11.154.239.png` - screenshot of scan underway*
![screenshot of scan underway](screenshot-scanning-52.11.154.239.png)

*`screenshot-scan-result-52.11.154.239.png` - screenshot of scan result of `52.11.154.239` using GovReady's FISMA-Ready Unbuntu LTS Benchmark SCAP `Ubuntu_14.04_LTS_Server_Datastream_v0.0.1.xml`*
![screenshot of scan result of `52.11.154.239` using GovReady's FISMA-Ready Unbuntu LTS Benchmark SCAP `Ubuntu_14.04_LTS_Server_Datastream_v0.0.1.xml`](screenshot-scan-result-52.11.154.239.png)


## Identification of Vulnerability Feeds
* http://www.ubuntu.com/usn/ (RSS: http://www.ubuntu.com/usn/rss.xml) provides CVE feed for Ubuntu, the operating system
* https://alas.aws.amazon.com (RSS: https://alas.aws.amazon.com/alas.rss) provides a CVE feed for Amazon Linux and is useful for its inclusion of upstream open source libray CVE's that may impact software included in Nebula
* http://www.cvedetails.com/vulnerability-list/vendor_id-13534/product_id-28125/Docker-Docker.html is a way to watch for Docker-related CVEs. 

## Monitoring of Server Status
We installed https://www.runscope.com a service to continously monitor and test our APIs.
