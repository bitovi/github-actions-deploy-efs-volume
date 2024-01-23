# Deploy Amazon (AWS) EFS volume
`bitovi/github-actions-deploy-efs-volume` deploys an EFS volume with multiple inputs to adjust it to your needs.

This action uses our new GitHub Actions Commons repository, a library that contains multiple Terraform modules, allowing us to condense all of our tools in one repo, hence continuous improvements are made to it. 
![alt](https://bitovi-gha-pixel-tracker-deployment-main.bitovi-sandbox.com/pixel/qPIWmgBCJYN8ODODrif5T)
## Action Summary
This action creates an EFS volume and allows you to tailor it to your needs. Will output the needed values to chain this into another action.

If you would like to deploy a backend app/service, check out our other actions:
| Action | Purpose |
| ------ | ------- |
| [Deploy Docker to EC2](https://github.com/marketplace/actions/deploy-docker-to-aws-ec2) | Deploys a repo with a Dockerized application to a virtual machine (EC2) on AWS |
| [Deploy React to GitHub Pages](https://github.com/marketplace/actions/deploy-react-to-github-pages) | Builds and deploys a React application to GitHub Pages. |
| [Deploy static site to AWS (S3/CDN/R53)](https://github.com/marketplace/actions/deploy-static-site-to-aws-s3-cdn-r53) | Hosts a static site in AWS S3 with CloudFront |
<br/>

**And more!**, check our [list of actions in the GitHub marketplace](https://github.com/marketplace?category=&type=actions&verification=&query=bitovi)

# Need help or have questions?
This project is supported by [Bitovi, A DevOps consultancy](https://www.bitovi.com/services/devops-consulting).

You can **get help or ask questions** on our:

- [Discord Community](https://discord.gg/J7ejFsZnJ4Z)

Or, you can hire us for training, consulting, or development. [Set up a free consultation](https://www.bitovi.com/services/devops-consulting).

# Basic Use

For basic usage, create `.github/workflows/deploy.yaml` with the following to build on push.
This will create you a volume with the corresponding security group and a mount-target in your current region. (only one)
```yaml
on:
  push:
    branches:
      - "main" # change to the branch you wish to deploy from

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - id: deploy-efs
      uses: bitovi/github-actions-deploy-efs-volume@v0.1.0
      with:
        aws_access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws_default_region: us-east-1

```

# Advanced usage
If you already have an existing volume, with no mount targets, this will create the mount targets for all of the regions available in the VPC you are creating.

```yaml
on:
  push:
    branches:
      - "main" # change to the branch you wish to deploy from

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - id: deploy-efs
      uses: bitovi/github-actions-deploy-efs-volume@v0.1.0
      with:
        aws_access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws_default_region: us-east-1

        aws_efs_create: false
        aws_efs_fs_id: fs-00000000000
        aws_efs_create_mount_target: true
        aws_efs_create_ha: true

        # AWS VPC Inputs
        aws_vpc_create: true
```

### Inputs
1. [AWS Specific](#aws-specific)
1. [Action default inputs](#action-default-inputs)
1. [EFS Inputs](#efs-inputs)
1. [VPC Inputs](#vpc-inputs)

### Outputs
1. [Outputs](#outputs)

The following inputs can be used as `step.with` keys
<br/>

#### **AWS Specific**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `aws_access_key_id` | String | AWS access key ID |
| `aws_secret_access_key` | String | AWS secret access key |
| `aws_session_token` | String | AWS session token |
| `aws_default_region` | String | AWS default region. Defaults to `us-east-1` |
| `aws_resource_identifier` | String | Set to override the AWS resource identifier for the deployment. Defaults to `${GITHUB_ORG_NAME}-${GITHUB_REPO_NAME}-${GITHUB_BRANCH_NAME}`. |
| `aws_additional_tags` | JSON | Add additional tags to the terraform [default tags](https://www.hashicorp.com/blog/default-tags-in-the-terraform-aws-provider), any tags put here will be added to all provisioned resources.|
<br/>

#### **Action default inputs**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `tf_stack_destroy` | Boolean  | Set to `true` to destroy the stack. |
| `tf_state_file_name` | String | Change this to be anything you want to. Carefull to be consistent here. A missing file could trigger recreation, or stepping over destruction of non-defined objects. Defaults to `tf-state-aws`. |
| `tf_state_file_name_append` | String | Appends a string to the tf-state-file. Setting this to `unique` will generate `tf-state-aws-unique`. (Can co-exist with `tf_state_file_name`) |
| `tf_state_bucket` | String | AWS S3 bucket name to use for Terraform state. See [note](#s3-buckets-naming) | 
| `tf_state_bucket_destroy` | Boolean | Force purge and deletion of S3 bucket defined. Any file contained there will be destroyed. `tf_stack_destroy` must also be `true`. Default is `false`. |
| `bitops_code_only` | Boolean | If `true`, will run only the generation phase of BitOps, where the Terraform and Ansible code is built. |
| `bitops_code_store` | Boolean | Store BitOps generated code as a GitHub artifact. |
<br/>

#### **EFS Inputs**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `aws_efs_create` | Boolean | Toggle to indicate whether to create an EFS volume and mount it to the EC2 instance as a part of the provisioning. Note: The stack will manage the EFS and will be destroyed along with the stack. |
| `aws_efs_fs_id` | String | ID of existing EFS volume if you wish to use an existing one. |
| `aws_efs_create_mount_target` | String | Toggle to indicate whether we should create a mount target for the EFS volume or not. Defaults to `true`.|
| `aws_efs_create_ha` | Boolean | Toggle to indicate whether the EFS resource should be highly available (mount points in all available zones within region). |
| `aws_efs_vol_encrypted` | String | Toggle encryption of the EFS volume. Defaults to `true`.|
| `aws_efs_kms_key_id` | String | The ARN for the KMS encryption key. Will use default if none defined. |
| `aws_efs_performance_mode` | String | Toggle perfomance mode. Options are: `generalPurpose` or `maxIO`.|  
| `aws_efs_throughput_mode` | String | Throughput mode for the file system. Defaults to `bursting`. Valid values: `bursting`, `provisioned`, or `elastic`. When using provisioned, also set `aws_efs_throughput_speed`. |
| `aws_efs_throughput_speed` | String | The throughput, measured in MiB/s, that you want to provision for the file system. Only applicable with throughput_mode set to provisioned. |
| `aws_efs_security_group_name` | String | The name of the EFS security group. Defaults to `SG for ${aws_resource_identifier} - EFS`. |
| `aws_efs_allowed_security_groups` | String | Extra names of the security grou-ps to access the EFS volume. Accepts comma separated list of. |
| `aws_efs_ingress_allow_all` | Boolean | Allow access from 0.0.0.0/0 in the same VPC. Defaults to `true`. |
| `aws_efs_create_replica` | Boolean | Toggle whether a read-only replica should be created for the EFS primary file system. |
| `aws_efs_replication_destination` | String | AWS Region to target for replication. |
| `aws_efs_enable_backup_policy` | Boolean | Toggle whether the EFS should have a backup policy. |
| `aws_efs_transition_to_inactive` | String | Indicates how long it takes to transition files to the IA storage class. Defaults to `AFTER_30_DAYS`. |
| `aws_efs_mount_target` | String | Directory path in efs to mount directory to. Default is `/`. |
| `aws_efs_ec2_mount_point` | String | The `aws_efs_ec2_mount_point` input represents the folder path within the EC2 instance to the data directory. Default is `/user/ubuntu/<application_repo>/data`. Additionally, this value is loaded into the docker-compose `.env` file as `HOST_DIR`. |
| `aws_efs_additional_tags` | JSON | Add additional tags to the terraform [default tags](https://www.hashicorp.com/blog/default-tags-in-the-terraform-aws-provider), any tags put here will be added to efs provisioned resources.|
<hr/>
<br/>

#### **VPC Inputs**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `aws_vpc_create` | Boolean | Define if a VPC should be created. Defaults to `false`. |
| `aws_vpc_name` | String | Define a name for the VPC. Defaults to `VPC for ${aws_resource_identifier}`. |
| `aws_vpc_cidr_block` | String | Define Base CIDR block which is divided into subnet CIDR blocks. Defaults to `10.0.0.0/16`. |
| `aws_vpc_public_subnets` | String | Comma separated list of public subnets. Defaults to `10.10.110.0/24`|
| `aws_vpc_private_subnets` | String | Comma separated list of private subnets. If no input, no private subnet will be created. Defaults to `<none>`. |
| `aws_vpc_availability_zones` | String | Comma separated list of availability zones. Defaults to `aws_default_region+<random>` value. If a list is defined, the first zone will be the one used for the mount target. |
| `aws_vpc_id` | String | **Existing** AWS VPC ID to use. Accepts `vpc-###` values. |
| `aws_vpc_subnet_id` | String | **Existing** AWS VPC Subnet ID. If none provided, will pick one. (Ideal when there's only one). |
| `aws_vpc_enable_nat_gateway` | String | Adds a NAT gateway for each public subnet. Defaults to `false`.|
| `aws_vpc_single_nat_gateway` | String | Toggles only one NAT gateway for all of the public subnets. Defaults to `false`.|
| `aws_vpc_external_nat_ip_ids` | String | **Existing** comma separated list of IP IDs if reusing. (ElasticIPs). |
| `aws_vpc_additional_tags` | JSON | Add additional tags to the terraform [default tags](https://www.hashicorp.com/blog/default-tags-in-the-terraform-aws-provider), any tags put here will be added to vpc provisioned resources.|
<br/>

#### **Outputs**
| Name             | Description                        |
|------------------|------------------------------------|
| `aws_efs_fs_id` | AWS EFS FS ID of the volume. |
| `aws_efs_replica_fs_id` | AWS EFS FS ID of the replica volume. |
| `aws_efs_sg_id` | SG ID for the EFS Volume. |
| `aws_vpc_id` | VPC ID if a VPC was created. |
<br/>

## Contributing
We would love for you to contribute to [`bitovi/github-actions-deploy-efs-volume`](hhttps://github.com/bitovi/github-actions-deploy-efs-volume).   [Issues](https://github.com/bitovi/github-actions-deploy-efs-volume/issues) and [Pull Requests](https://github.com/bitovi/github-actions-deploy-efs-volume/pulls) are welcome!

## License
The scripts and documentation in this project are released under the [MIT License](https://github.com/bitovi/github-actions-deploy-efs-volume/blob/main/LICENSE).

# Provided by Bitovi
[Bitovi](https://www.bitovi.com/) is a proud supporter of Open Source software.

# We want to hear from you.
Come chat with us about open source in our Bitovi community [Discord](https://discord.gg/J7ejFsZnJ4Z)!