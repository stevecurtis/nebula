# Nebula


Try our [application](https://www.sideeffect.io/) and give us feedback by opening an issue!

And our [api](https://api.sideeffect.io/) will someday be useful, but is not
ready now.

This is our application to 18F's Agile BPA RFQ.  However, it is all freely sharable, and we aim to make a product 
truly valuable. Feel free to fork this code.

## Licence

For licence details, including licences of third-party free and open source software incorporated into this repository please see the LICENSE.md file.

## Development environment setup

The development environment is fully self-contained, and is based on Docker and Docker Compose.

### Requirements
1. [Docker](https://www.docker.com/)
1. [Docker Compose](https://docs.docker.com/compose/)
1. An [FDA API Key](https://open.fda.gov/api/reference/#your-api-key)

### Instructions

Clone the repository, and change to the project directory:
```bash
git clone git@github.com:CivicActions/nebula.git
cd nebula
```

If you are using boot2docker, make sure it is started up and it's shell environment variable are available before continuing.

Export your FDA API Key, adding it to the end of this command:
```bash
export FDA_API_KEY=
```

To start docker containers, initiate database schemas and import:
```bash
./bin/build
```
The first run will take a while. Use this command to restart containers if you reboot your workstation.

* The frontend component will be available on port 2086 http://localhost:2086 (if on boot2docker it will be on http://[boot2docker IP]:2086).
* The backend component will be available on port 2095 http://localhost:2095 (if on boot2docker it will be on http://[boot2docker IP]:2095).
* If on boot2docker, determine ip address of boot2docker virtual machine with command `boot2docker ip`.
* To view logs run `docker-compose logs`.
* To stop the containers run `docker-compose stop`.
* If the backendphp container is modified, rebuild the containers using `docker-compose rm` followed by `./bin/build`.


## Testing

To execute all the tests (frontend and backend), build your environment as normal, then run:
```bash
./bin/run-tests
```
A zero exit code (`echo $?`) indicates a successful run, a non-zero result indicates a test failure.

### Frontend Tests

The frontend tests use the (Selenium Builder)[https://saucelabs.com/builder] Selenium 2 JSON test format, which is easy to version control and edit with the open source Selenium Builder tool (which can run/debug the tests manually). The tests are in the ("frontend/tests/selenium2-core-tests.json")[https://github.com/CivicActions/nebula/tree/master/frontend/tests] file.

Tests are automated using the (se-interpreter)[https://github.com/Zarkonnen/se-interpreter] which is made available as a Docker container, together with the (SeleniumHQ Chrome and Firefox Docker images)[https://github.com/SeleniumHQ/docker-selenium]. Note that these are fully automated using the "run-tests" script - there is no need to install any additional software to run the tests.

Test results are recorded in chrome.log and firefox.log files in the "frontend/tests" directory.

### Backend Tests

There is a PHPUnit test framework available in the "backend" directory, with tests in the ("backend/tests")[https://github.com/CivicActions/nebula/tree/master/backend/tests] directory. Guzzle is available for convenient testing of http endpoints.

A JUnit format report.xml is available in the "testing" directory for the backend tests. You can include additional PHPUnit arguments after the above command, as needed.

### Automated Testing

A sample Jenkins configuration is available in the (devops/jenkins/testing/)[https://github.com/CivicActions/nebula/blob/master/devops/jenkins/testing/] directory. This will, upon each git push to github:
* Bootstrap docker-compose and start the containers.
* Run the tests and record the results.
* Report the result to a Slack channel.

## Deployment

This deploys an instance of the frontend and an instance of the backend on Amazon Web Services instances, then configures DNS (with CDN and SSl) for each IP using Cloudflare. Docker Machine is used to provision and bootstrap the instances. The AWS CLI is used to manage network configuration and a Cloudflare CLI is used to configure Cloudflare (the AWS CLI and Cloudflare tools are run via Docker, so dependencies are minimized).

### Requirements
1. [Docker](https://www.docker.com/) to manage containers
1. [Docker Compose](https://docs.docker.com/compose/) to automate defining and runnning multi-container applications with Docker
1. [Docker Machine](https://docs.docker.com/machine/) to automate creating Docker hosts locally, on cloud providers, or in a data center
1. An [FDA API Key](https://open.fda.gov/api/reference/#your-api-key)
1. Amazon Web Services account and API keys to automate interactions with AWS
	* VPC configured (default OK)
	* Subnet within VPC
1. AWS ec2 commandline tools
1. Cloudflare account and API keys to automate interactions with Cloudflare

### Instructions

Clone the repository, and change to the project directory:
```bash
git clone git@github.com:CivicActions/nebula.git
cd nebula
```

If you are using boot2docker, make sure it is started up and it's shell environment variable are available before continuing.
```bash
# Below steps if using boot2docker only
boot2docker init
boot2docker up

# Export returned docker environmental values (example below)
export DOCKER_HOST=tcp://192.168.59.103:2376
export DOCKER_CERT_PATH=/Users/greg/.boot2docker/certs/boot2docker-vm
export DOCKER_TLS_VERIFY=1
```

Create the following required environment variables, containing your AWS, Cloudflare and FDA API access details:
```bash
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=
export AWS_VPC_ID=
export AWS_DEFAULT_REGION=
export CLOUDFLARE_EMAIL=
export CLOUDFLARE_TOKEN=
export FDA_API_KEY=
```

NOTE: For **continuous monitoring**, set to below environmental variables to use [18F's hardened, FISMA-Ready Ubuntu-LTS](https://github.com/fisma-ready/ubuntu-lts) for Docker Host instead of Docker's default Ubuntu-LTS. 
```bash
export AWS_DEFAULT_REGION="us-west-2"
export AWS_AMI=ami-b7393887
export AWS_ROOT_SIZE=30
```

Depending on your region and VPC selected, you may also need to set the following environment variables:
```bash
export AWS_ZONE=
export AWS_SUBNET_ID=
```

NOTE: Subnets must be available with in the AWS Region and Zone you are using. Ensure that your VPC is contained within the Region you select, and that the zone/subnet (if selected) are available on that VPC. The AWS environment variables and default values are detailed in the [Docker Machine](https://docs.docker.com/machine/#amazon-web-services) documentation. Use the below command to check availability of subnets:

```bash
aws ec2 describe-subnets
```

Run the ./bin/deploy script to deploy the frontend and backend respectively, where the second parameter is the subdomain to deploy to, and the third is a Cloudflare DNS hosted domain name. For example:
```bash
./bin/deploy frontend www sideeffect.io
./bin/deploy backend api sideeffect.io
```
Will deploy to https://www.sideeffect.io/ and https://api.sideeffect.io/

### Monitoring

There is a simple Jenkins availability monitor that you can install. See [configuration](https://github.com/CivicActions/nebula/blob/master/devops/jenkins/sideeffect.io-ping/config.xml).

For full availability, performance and API correctness multi-location monitoring and alert notifications, we use [Runscope](https://www.runscope.com/), see [devops/monitoring](https://github.com/CivicActions/nebula/tree/master/devops/monitoring) directory for configuration details.
