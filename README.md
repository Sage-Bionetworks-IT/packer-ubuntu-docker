# packer-ubuntu-bionic-docker
A packer project to build an AMI containing docker

## Development

### Setup
* Install [packer](https://www.packer.io/intro/getting-started/install.html)
* Install [ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

### Validate a template
Choose an ImageName such as "MyImage" and run
```
packer validate -var 'ImageName=MyImage' template.json
```

#### Access
* Request an IAM account in [Imagecentral](https://github.com/Sage-Bionetworks/imagecentral-infra)
* Change password and set up MFA
* Create a Keypair
* Add your access code and secrety key to `~/.aws/credentials`, using a profile such as "imagecentral.jsmith"
* Authenticate with `awsmfa`, for example `awsmfa -i imagecentral.jsmith -t jsmith@imagecentral`
* Finally, get the correct role ARN for the PackerServiceRole then add the following:
```
[profile packer-service-imagecentral]
region = us-east-1
role_arn = *****
source_profile = jsmith@imagecentral
```

Now you will be able to build in Imagecentral.

### Manual AMI Build
If you would like to test building an AMI run:
```
cd src
packer build -var PACKER_LOG=1 -var AwsProfile=packer-service-imagecentral -var AwsRegion=us-east-1 -var ImageName=packer-ubuntu-bionic-docker-DEV template.json
```

Packer will do the following:
* Create a temporary EC2 instance, configure it with shell/ansible/puppet/etc. scripts.
* Create an AMI from the EC2
* Delete the EC2

__Note__: Packer deploys a new AMI to the AWS account specified by the AwsProfile

### Pull Requests and Versions.
To make changes, we create pull requests.
Once these changes are merged to master, create a tag.
When the tag is pushed travis will build an AMI version with that tag number.

### Searching
List the built images by using the AWS CLI:
```
aws ec2 describe-images --owners 867686887310 --filters Name=tag:Name,Values=my-test-image
```

### Removal
Building an AMI will create the AMI and one or more snapshots for the AMI.  When deleting
the AMI remember to also delete its snapshots. Use the provided [bash script](deregister_ami.sh)
to remove the AMI and its snapshots.

## Contributions
Contributions are welcome.

Requirements:
* Install [pre-commit](https://pre-commit.com/#install) app
* Clone this repo
* Run `pre-commit install` to install the git hook.

## Testing
As a pre-deployment step we syntatically validate our packer json
files with [pre-commit](https://pre-commit.com).

Please install pre-commit, once installed the file validations will
automatically run on every commit.  Alternatively you can manually
execute the validations by running `pre-commit run --all-files`.

## Deployments
Travis runs packer which temporarily deploys an EC2 to create an AMI.

## Continuous Integration
We have configured Travis to deploy updates.

## Issues
* https://sagebionetworks.jira.com/projects/IT

## Builds
* https://travis-ci.org/Sage-Bionetworks/packer-ubuntu-bionic-docker

## Secrets
* We use the [AWS SSM](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-paramstore.html)
to store secrets for this project.  Sceptre retrieves the secrets using
a [sceptre ssm resolver](https://github.com/cloudreach/sceptre/tree/v1/contrib/ssm-resolver)
and passes them to the cloudformation stack on deployment.
