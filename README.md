[![Experimental Project header](https://github.com/newrelic/opensource-website/raw/master/src/images/categories/Experimental.png)](https://opensource.newrelic.com/oss-category/#experimental)

# AWS QuickStart for New Relic EC2 Infrastructure integration

>The [AWS QuickStart](https://aws.amazon.com/quickstart/) for [New Relic EC2 Infrastructure integration](https://newrelic.com/integrations/aws-ec2-integration)  comprises of an [AWS CloudFormation](https://aws.amazon.com/cloudformation/) template that you deploy to your AWS account(s) using [StackSets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/what-is-cfnstacksets.html). After you deploy the StackSet, it automatically triggers the automation workflow for the installation of [New Relic Infrastructure agent](https://docs.newrelic.com/docs/infrastructure/install-infrastructure-agent/get-started/install-infrastructure-agent) in your [AWS EC2](https://aws.amazon.com/ec2/) instances based on a tag key-value pair match, as soon as the instances are launched. You can also run this automation worflow on-demand using [AWS CLI](https://aws.amazon.com/cli/) or [AWS Management Console](https://aws.amazon.com/console/). 

>The QuickStart relies on [AWS Systems Manager](https://aws.amazon.com/systems-manager/) (SSM) [Automation](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-automation.html) and requires that EC2 instances have the [SSM Agent](https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent.html) installed.

## Getting Started
>The solution requires that:
>1. You have [Administrator](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_job-functions.html#jf_administrator) access to your AWS account so that you can deploy, manage and test the solution. If you are using the [AWS Control Tower](https://aws.amazon.com/controltower/), it is recommended you launch the StackSet from the AWS Control Tower [master account](https://docs.aws.amazon.com/controltower/latest/userguide/how-control-tower-works.html#what-is-master) in your home region, where the AWS Control Tower [Landing Zone](https://aws.amazon.com/controltower/features/#Landing_Zone) was set up.
>2. You have [setup permissions for StackSets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-prereqs.html) in your AWS account. It is recommended that you use `self-managed` permissions for a better security posture and control.  In case you are using the AWS Control Tower, you should use the pre-configured IAM Roles for StackSet operations, viz. `AWSControlTowerStackSetRole` and `AWSControlTowerExecution`.
>3. SSM Agent is installed on your EC2 instances. SSM Agent is preinstalled, by default, on many Windows and Linux Amazon Machine Images (AMIs). In case you are using an AMI that doesn't have the agent installed, you must manually install the SSM Agent for [Linux](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-manual-agent-install.html) and [Windows](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-install-win.html). You can also install AWS Systems Manager Agent (SSM Agent) on an Amazon EC2 Linux instance [at launch.](https://aws.amazon.com/premiumsupport/knowledge-center/install-ssm-agent-ec2-linux/)
>4. Configure an AWS Identity and Access Management (IAM) [instance profile](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html) that enables Systems Manager to securely run commands on your EC2 instances on your behalf. The managed IAM policy, `AmazonSSMManagedInstanceCore`, enables an instance to use AWS Systems Manager service core functionality. You can also use [Systems Manager Quick Setup](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-quick-setup.html) to quickly configure an instance profile on all instances in your AWS account. For more details, refer to, [create an IAM instance profile for Systems Manager.](https://docs.aws.amazon.com/systems-manager/latest/userguide/setup-instance-profile.html)
>5. An active [New Relic account](https://one.newrelic.com/) using the [New Relic One pricing plan](https://docs.newrelic.com/docs/accounts/accounts-billing/new-relic-one-pricing-users/pricing-billing) with Standard or higher pricing tier. If you don't already have an account, you may [signup](https://newrelic.com/signup) for a new account, for free.

## Usage
>This solution includes a CloudFormation template, viz. [NR-EC2InfraAgentSSMAutomation.yml](templates/NR-EC2InfraAgentSSMAutomation.yml) that you deploy in your AWS account as StackSet. You should deploy the StackSet instances in your AWS accounts across all regions where you plan to have New Relic Infrastructure agent installed on your EC2 instances.

>First, create a StackSet, viz. `NR-EC2InfraSSMAutomation` in your AWS account. It is recommended that you create it in your StackSet administration account or your Control Tower master account, but you can use any other account if necessary. Use your [AWS account ID](https://docs.aws.amazon.com/IAM/latest/UserGuide/console_account-alias.html) in place of `AWS_ACCOUNT_ID` placeholder. The template accepts your [New Relic license key](https://docs.newrelic.com/docs/accounts/accounts-billing/account-setup/new-relic-license-key) as a parameter. Use it in place of `NEW_RELIC_LICENSE_KEY` placeholder. Use the actual IAM Role names for your StackSet administration and execution roles, for the placeholders, viz. `STACK_SET_ADMINISTRATION_ROLE_NAME` and `STACK_SET_EXECUTION_ROLE_NAME`, respectively.

```
aws cloudformation create-stack-set \
    --stack-set-name NR-EC2InfraSSMAutomation \
    --template-body file:./templates/NR-EC2InfraAgentSSMAutomation.yml \
    --description "SSM Automation workflow for installing New Relic Infrastructure Agent on EC2 instances" \
    --parameters ParameterKey=NewRelicLicenseKey,ParameterValue=<NEW_RELIC_LICENSE_KEY> \
    --capabilities CAPABILITY_NAMED_IAM \
    --administration-role-arn arn:aws:iam::<AWS_ACCOUNT_ID>:role/<STACK_SET_ADMINISTRATION_ROLE_NAME> \
    --execution-role-name <STACK_SET_EXECUTION_ROLE_NAME> \
    --permission-model SELF_MANAGED
```
>After the command returns successfully, the StackSet creation starts. You can check the status of the StackSet creation after a few seconds. To verify the StackSet was successfully created, run the below command, and make sure the StackSet name, viz. `NR-EC2InfraSSMAutomation` is returned.
```
aws cloudformation describe-stack-set --stack-set-name NR-EC2InfraSSMAutomation --query "StackSet.StackSetName"
```

>Next, create instances of the `NR-EC2InfraSSMAutomation` StackSet across various AWS accounts and regions where you plan to launch EC2 instances. The following `create-stack-instances` example creates instances in two accounts and in four US regions.

```
aws cloudformation create-stack-instances \
    --stack-set-name NR-EC2InfraSSMAutomation \
    --accounts 123456789012 223456789012 \
    --regions us-east-1 us-east-2 us-west-1 us-west-2
```
>After the command returns successfully, the Stack instances creation operation starts and an `OperationId` is returned. You can check the status of the operation after a few seconds. To verify the operation was successfully completed, run the below command, and make sure the status, viz. `SUCCEEDED` is returned. Use the returned `OperationId` value in place of `OPERATION_ID` placeholder.
```
aws cloudformation describe-stack-set-operation \
    --stack-set-name NR-EC2InfraSSMAutomation \
    --operation-id <OPERATION_ID> \
    --query "StackSetOperation.Status"
```
>To test the solution, log into your AWS account and make sure you choose a region where StackSet instance is deployed, and launch an EC2 instance. Make sure that:
>1. AMI is supported by New Relic Infrastructure agent after reviewing the [requirements for the infrastructure agent](https://docs.newrelic.com/docs/infrastructure/install-infrastructure-agent/get-started/requirements-infrastructure-agent). 
>2. The SSM Agent is installed by default for the chosen AMI, or install it yourself, either at launch or afterwards. 
>3. The instance profile has permission to run SSM commands on the EC2 instance. 
>4. Add the tag with name `NR-Infrastructure` and value `Install`. This tag key-value pair allows the automation workflow to automatically install the New Relic Infrastructure agent on the EC2 instance, as soon as it is launched and in running state.

>Once the EC2 instance is launched, connect to it using [Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html), and once you are connected, run this command in the terminal.
```
newrelic-infra --version
```
>You will see the New Relic Infrastructure agent version info. This confirms that the agent was successfully installed automatically on the EC2 instance as soon as it was launched, and you should now be able to monitor the instance from your New Relic account.

>Log into your [New Relic account](https://one.newrelic.com/) and navigate to [Entity explorer](https://one.newrelic.com/launcher/nr1-core.explorer) and look for the newly launched EC2 instance in the `Hosts` page. You can launch more instances using a variety of different AMIs supported by New Relic Infrastructure agent and you will see that they all automatically show up in your New Relic account.

## Support

New Relic hosts and moderates an online forum where customers can interact with New Relic employees as well as other customers to get help and share best practices. Like all official New Relic open source projects, there's a related Community topic in the New Relic Explorers Hub. You can find this project's topic/threads here:

>Add the url for the support thread here

## Contributing
We encourage your contributions to improve AWS QuickStart for New Relic EC2 Infrastructure integration! Keep in mind when you submit your pull request, you'll need to sign the CLA via the click-through using CLA-Assistant. You only have to sign the CLA one time per project.
If you have any questions, or to execute our corporate CLA, required if your contribution is on behalf of a company,  please drop us an email at opensource@newrelic.com.

## License
AWS QuickStart for New Relic EC2 Infrastructure integration is licensed under the [Apache 2.0](http://apache.org/licenses/LICENSE-2.0.txt) License.
