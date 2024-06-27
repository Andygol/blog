---
layout: "post"
title: "Deploy a container from the ghcr.io private registry to Amazon ECS"
date: "2024-06-26 10:30:00 +0200"
categories: uk
tags:
  - React
  - GitHub
  - Actions
  - CI/CD
comments: true
lang: en
ref: 2024-06-26-deploy-container-from-ghcr-private-registry-to-ecs
---

Everyone reaches a point where you need to deploy a container from a private repository to ECS. Amazon's documentation isn't the epitome of clarity, so I've prepared a guide for you (and primarily for myself) on what needs to be done.

## Prerequisites

### Container in ghcr.io

First, you need to upload your container to the [GitHub Container Registry](https://ghcr.io/). This can be a public or private repository.

Log in to GitHub and navigate to the repository page where your container is located. In the **Packages** section, select the artifact that is your previously created container.

![GitHub Container Registry](/images/2024/06/ghcr-to-ecs-container.png)

To access your registry, you will need a Private Access Token (PAT). To create a PAT, go to your account settings.

![GitHub Profile Settings](/images/2024/06/ghcr-to-ecs-profile-settings.png)

At the bottom of the page, select **Developer settings**.

![GitHub Developer Settings](/images/2024/06/ghcr-to-ecs-profile-dev-settings.png)

Next, select **Personal access tokens**/**Tokens (classic)**/**Generate new token (classic)**.

![GitHub Personal Access Tokens](/images/2024/06/ghcr-to-ecs-profile-token-classic.png)

Alternatively, follow this link to create a PAT — <https://github.com/settings/tokens/new>.

Add a note if needed, choose the expiration date for the token, and specify the required permissions — in this case, we only need permissions to fetch packages (**read:packages** — Download packages from GitHub Package Registry). At the bottom of the page, click **Generate token**.

![GitHub New Personal Access Token](/images/2024/06/ghcr-to-ecs-profile-new-token.png)

After that, you will see a page with your new token. Copy it and save it in a secure place. We will use it to access the registry.

![GitHub Personal Access Token](/images/2024/06/ghcr-to-ecs-profile-created-token.png)

### ECS

You should have an AWS account set up, optionally with [AWS CLI](https://aws.amazon.com/cli/) and [Amazon ECS CLI](https://github.com/aws/amazon-ecs-cli) installed locally. If you don't want to install them locally, you can use [AWS CloudShell](https://aws.amazon.com/cloudshell/).

## Creating a CMK in AWS KMS

First, we need to create a [CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#kms_keys) (Customer Master Key) and an alias for it in [AWS KMS](https://aws.amazon.com/kms/). The CMK is used by [AWS Secret Manager](https://docs.aws.amazon.com/kms/latest/developerguide/services-secrets-manager.html) for [envelope encryption](https://en.wikipedia.org/wiki/Hybrid_cryptosystem#Envelope_encryption) of data containing sensitive information. An alias acts as a name for your CMK, making it easier to remember and use than the key ID itself. You can also use the alias in your code. You can change the key in the future (to another one) while keeping the alias unchanged.

```sh
aws kms create-key --query KeyMetadata.Arn --output text
```

In response, you will receive the key ID in the form of an Amazon Resource Name ([ARN](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html)):

```none
arn:aws:kms:eu-central-1:123456789012:key/abc123de-4567-89fa-0bcd-efgh12345678
```

Now we will create an alias for our key:

```sh
aws kms create-alias \
  --alias-name alias/ecs-ghcr \
  --target-key-id arn:aws:kms:eu-central-1:123456789012:key/abc123de-4567-89fa-0bcd-efgh12345678
```

If you haven't set up AWS CLI locally, you can use CloudShell and run all the commands in the command line there.

![AWS CloudShell](/images/2024/06/ghcr-to-ecs-cloudshel.png)

You can create the CMK in the AWS Console by going to [AWS KMS](https://eu-central-1.console.aws.amazon.com/kms/home?region=eu-central-1#/kms/keys) and clicking the **Create key** button.

![AWS KMS Create Key](/images/2024/06/ghcr-to-ecs-kms-create-key.png)

You will need the ARN of the CMK when creating the trust policy document in the next step.

## Creating a Secret in AWS Secrets Manager

At this stage, we need to create a Secret that will store your login and password (access code) encrypted with the CMK for pulling your container image from a private registry.

```sh
aws secretsmanager create-secret \
  --name ghcr_io_pat \
  --description "Secret to get packages from ghcr.io" \
  --kms-key-id alias/ecs-ghcr \
  --secret-string '{"username":"your_nickname", "password":"ghp_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"}'
```

You should receive the following in response:

```json
{
    "ARN": "arn:aws:secretsmanager:eu-central-1:123456789012:secret:ghcr_io_pat-abcdEF",
    "Name": "ghcr_io_pat",
    "VersionId": "4b43b832-df4c-48b3-b59a-bb18287e6c15"
}
```

The ARN of the secret should be in the output ☝️ of the previous command — in the ARN field. You will need to refer to this ARN when creating the trust policy document in the next step.

## Creating an IAM Role for Task Execution

If you already have the `ecsTaskExecutionRole`, you can skip this step.

First, you need to create a trust policy document to specify the principal that will assume the role, which in this case is ECS task:

```sh
cat << EOF > ecs-trust-policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "ecs-tasks.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
EOF
```

Create the role using the AWS CLI and the `ecs-trust-policy.json` file with the role description.

```sh
aws iam create-role \
  --role-name ecsTaskExecutionRole \
  --assume-role-policy-document file://ecs-trust-policy.json
```

To add basic permissions to other AWS service resources needed to run Amazon ECS tasks, attach the AWS ECS task execution role policy to the newly created role:

```sh
aws iam attach-role-policy \
  --role-name ecsTaskExecutionRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy
```

Now create a permissions policy document that allows the ECS task to decrypt and retrieve the secret created in AWS Secrets Manager.

```sh
cat << EOF > ecs-secret-permission.json 
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "kms:Decrypt",
        "secretsmanager:GetSecretValue"
      ],
      "Resource": [
        "arn:aws:secretsmanager:eu-central-1:123456789012:secret:ghcr_io_pat-abcdEF",
        "arn:aws:kms:eu-central-1:123456789012:key/abc123de-4567-89fa-0bcd-efgh12345678"
      ]
    }
  ]
}
EOF
```

☝️ Specify your Secret and CMK in the `"Resource": [ <Secret>, <CMK> ]` values.

Finally, add the inline permissions policy that allows your task to fetch ghcr.io username and password from AWS Secrets Manager. Note that you refer to the permissions policy document created in the previous step. Change the file path as needed to specify the correct file location:

```sh
aws iam put-role-policy \
  --role-name ecsTaskExecutionRole \
  --policy-name ECS-SecretsManager-Permission \
  --policy-document file://ecs-secret-permission.json
```

## Configuring ECS CLI (Optional)

The Amazon ECS Command Line Interface (ECS CLI) provides commands to simplify the creation of an Amazon ECS cluster and the AWS resources required to set it up. After installing the ECS CLI, you can further configure your AWS credentials in a named ECS profile. Profiles are stored in the `~/.ecs/credentials` file.

```sh
ecs-cli configure profile \
  --access-key <AWS_ACCESS_KEY_ID> \
  --secret-key <AWS_SECRET_ACCESS_KEY> \
  --profile-name <PROFILE_NAME>
```

You can also specify a default profile to use for all ECS CLI commands:

```sh
ecs-cli configure profile default --profile-name <PROFILE_NAME>
```

If you do not configure an ECS profile or set environment variables, the default AWS profile will be used. The access key and secret access key values can be viewed in the [AWS Management Console](https://console.aws.amazon.com/iam/home?#security_credential).

You can further configure the ECS cluster name, launch type, and AWS region to use with the ECS CLI using the `ecs-cli configure` command. The `<LAUNCH_TYPE>` variable can be set to `FARGATE` or `EC2`.

```sh
ecs-cli configure \
  --cluster <CLUSTER_NAME> \
  --default-launch-type <LAUNCH_TYPE> \
  --config-name <CONFIG_NAME> \
  --region <AWS_REGION>
```

These values can also be specified or overridden using command flags in the subsequent steps.

## Creating an Amazon ECS Cluster

Create an Amazon ECS cluster using the `ecs-cli up` command, specifying the cluster name, AWS region (e.g., `eu-central-1`), and `FARGATE` as the launch type:

```sh
ecs-cli up \
  --cluster <CLUSTER_NAME> \
  --region eu-central-1 \
  --launch-type FARGATE
```

By using the FARGATE launch type, AWS Fargate manages the compute resources on your behalf, so you do not need to specify your own EC2 container instances. By default, the ECS CLI will also create an AWS CloudFormation stack to create a new VPC with an attached internet gateway, 2 public subnets, and a security group. You can also specify your own resources using flags in the command above.

## Configuring Security Group

After successfully creating the ECS cluster, you should see the VPC and subnet IDs displayed in the terminal. Next, get the JSON description of the newly created security group and note the security group ID or `GroupId`. Replace the `<VPC_ID>` variable with the ID of the newly created VPC.

```sh
aws ec2 describe-security-groups \
  --filters Name=vpc-id,Values=<VPC_ID> \
  --region eu-central-1
```

In response, you should get an output similar to this one:

```json
{
    "SecurityGroups": [
        {
            "Description": "default VPC security group",
            "GroupName": "default",
            "IpPermissions": [
                {
                    "IpProtocol": "-1",
                    "IpRanges": [],
                    "Ipv6Ranges": [],
                    "PrefixListIds": [],
                    "UserIdGroupPairs": [
                        {
                            "GroupId": "sg-04512c8a7bff9b34e",
                            "UserId": "123456789012"
                        }
                    ]
                },
                …
            ],
            …
        }
    ]
}
```

Add an inbound rule to the security group to allow HTTP traffic from any IPv4 address. Replace the `<SG_ID>` variable with the `GroupId` obtained in the previous step. This inbound rule will allow you to verify that your server is running in your task and that the image from the private registry was successfully pulled from GHCR.

```sh
aws ec2 authorize-security-group-ingress \
  --group-id <SG_ID> \
  --protocol tcp \
  --port 8080 \
  --cidr 0.0.0.0/0 \
  --region eu-central-1
```

## Creating an Amazon ECS Service

Amazon ECS service allows you to run and maintain a specified number of instances of a task definition simultaneously. The ECS CLI allows you to create a service using a Docker compose file. Create the following `docker-compose.yml` file that defines a container that serves port 8080 for incoming traffic to the server. To reference an image stored in your private registry in GHCR, replace the `<USER_NAME>` variable with your GitHub username, the `<REPO_NAME>` variable with your private repository name, and the `<TAG_NAME>` variable with the tag you used.

```sh
cat << EOF > docker-compose.yml
version: "3"
services:
    web:
        image: ghcr.io/<USER_NAME>/<REPO_NAME>:<TAG_NAME>
        ports:
            - 8080:8080
EOF
```

Thus, if you try to deploy the image from the example at the beginning, the value of the `image` key will be as follows: `ghcr.io/andygol/switch2osm-mkdocs:main`.

You will also need to create the following `ecs-params.yml` file to specify additional parameters for your service specific to Amazon ECS. Note that the `services` field below corresponds to the `services` field in the Docker Compose file above, corresponding to the name of the container to run. When the ECS CLI creates a task definition from the `docker-compose.yml` file, the `web` fields will be merged into the ECS container definition, including the container image it will use and the GHCR repository credentials it will need to access it. Replace the `<SECRET_ARN>` variable with the ARN of the AWS Secrets Manager secret you created earlier. Replace the `<SUB_1_ID>`, `<SUB_2_ID>`, and `<SG_ID>` variables with the IDs of the 2 public subnets and the security group that were created along with the ECS cluster.

```sh
cat << EOF > ecs-params.yml
version: 1
task_definition:
  task_execution_role: ecsTaskExecutionRole
  ecs_network_mode: awsvpc
  task_size:
    mem_limit: 0.5GB
    cpu_limit: 256
  services:
    web:
        repository_credentials: 
            credentials_parameter: "<SECRET_ARN>"
run_params:
  network_configuration:
    awsvpc_configuration:
      subnets:
        - "<SUB_1_ID>"
        - "<SUB_2_ID>"
      security_groups:
        - "<SG_ID>"
      assign_public_ip: ENABLED
EOF
```

Next, create the ECS service from your compose file using the `ecs-cli compose service up` command. This command will look for your `docker-compose.yml` and `ecs-params.yml` files in the current directory. Replace the `<CLUSTER_NAME>` variable with the name of your ECS cluster and the `<PROJECT_NAME>` variable with the desired name of your ECS service.

```sh
ecs-cli compose \
  --project-name <PROJECT_NAME> \
  --cluster <CLUSTER_NAME> \
  service up \
  --launch-type FARGATE
```

Allow some time for your ECS service to deploy. You can now check the status of your ECS service using the `ecs-cli ps` command.

```sh
ecs-cli compose \
  --project-name <PROJECT_NAME> \
  --cluster <CLUSTER_NAME> \
  service ps
```

By navigating to the IP address specified on port 8080, you will be able to view the homepage of our project, confirming that your task was able to successfully pull the container image from the GHCR registry using your credentials for authentication.

## Cleanup

Stop your ECS service using the `ecs-cli compose service down` command.

```sh
ecs-cli compose \
  --project-name <PROJECT_NAME> \
  --cluster <CLUSTER_NAME> \
  service down
```

Delete the AWS CloudFormation stack that was created by `ecs-cli up` and the associated resources using the `ecs-cli down` command:

```sh
ecs-cli down --cluster <CLUSTER_NAME>
```
