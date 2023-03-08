[![Release][release-image]][release] [![CI][ci-image]][ci] [![License][license-image]][license] [![Registry][registry-image]][registry] [![Source][source-image]][source]

# terraform-aws-s3-loader-kinesis-ec2

A Terraform module which deploys a Snowplow S3 Loader application on AWS running on top of EC2.  If you want to use a custom AMI for this deployment you will need to ensure it is based on top of Amazon Linux 2.

## Telemetry

This module by default collects and forwards telemetry information to Snowplow to understand how our applications are being used.  No identifying information about your sub-account or account fingerprints are ever forwarded to us - it is very simple information about what modules and applications are deployed and active.

If you wish to subscribe to our mailing list for updates to these modules or security advisories please set the `user_provided_id` variable to include a valid email address which we can reach you at.

### How do I disable it?

To disable telemetry simply set variable `telemetry_enabled = false`.

### What are you collecting?

For details on what information is collected please see this module: https://github.com/snowplow-devops/terraform-snowplow-telemetry

## Usage

An S3 Loader requires an input stream, an output stream for corrupted data that cannot be saved to S3 and an S3 bucket to save the data into.

```hcl
module "s3_pipeline_bucket" {
  source  = "snowplow-devops/s3-bucket/aws"
  version = "0.2.0"

  bucket_name = "your-bucket-name"
}

module "enriched_stream" {
  source  = "snowplow-devops/kinesis-stream/aws"
  version = "0.2.0"

  name = "enriched-stream"
}

module "bad_1_stream" {
  source  = "snowplow-devops/kinesis-stream/aws"
  version = "0.2.0"

  name = "bad-1-stream"
}

module "s3_loader_enriched" {
  source = "snowplow-devops/s3-loader-kinesis-ec2/aws"

  name             = "s3-loader-enriched-server"
  vpc_id           = var.vpc_id
  subnet_ids       = var.subnet_ids
  in_stream_name   = module.enriched_stream.name
  bad_stream_name  = module.bad_1_stream.name
  s3_bucket_name   = module.s3_pipeline_bucket.id
  s3_object_prefix = "enriched/"

  ssh_key_name     = "your-key-name"
  ssh_ip_allowlist = ["0.0.0.0/0"]
}
```

## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 1.0.0 |
| <a name="requirement_aws"></a> [aws](#requirement\_aws) | >= 3.72.0 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_aws"></a> [aws](#provider\_aws) | >= 3.72.0 |

## Modules

| Name | Source | Version |
|------|--------|---------|
| <a name="module_instance_type_metrics"></a> [instance\_type\_metrics](#module\_instance\_type\_metrics) | snowplow-devops/ec2-instance-type-metrics/aws | 0.1.2 |
| <a name="module_kcl_autoscaling"></a> [kcl\_autoscaling](#module\_kcl\_autoscaling) | snowplow-devops/dynamodb-autoscaling/aws | 0.2.0 |
| <a name="module_service"></a> [service](#module\_service) | snowplow-devops/service-ec2/aws | 0.1.1 |
| <a name="module_telemetry"></a> [telemetry](#module\_telemetry) | snowplow-devops/telemetry/snowplow | 0.4.0 |

## Resources

| Name | Type |
|------|------|
| [aws_cloudwatch_log_group.log_group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/cloudwatch_log_group) | resource |
| [aws_dynamodb_table.kcl](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/dynamodb_table) | resource |
| [aws_iam_instance_profile.instance_profile](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_instance_profile) | resource |
| [aws_iam_policy.iam_policy](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_policy) | resource |
| [aws_iam_role.iam_role](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role) | resource |
| [aws_iam_role_policy_attachment.policy_attachment](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role_policy_attachment) | resource |
| [aws_security_group.sg](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group) | resource |
| [aws_security_group_rule.egress_tcp_443](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group_rule) | resource |
| [aws_security_group_rule.egress_tcp_80](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group_rule) | resource |
| [aws_security_group_rule.egress_udp_123](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group_rule) | resource |
| [aws_security_group_rule.ingress_tcp_22](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group_rule) | resource |
| [aws_caller_identity.current](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/caller_identity) | data source |
| [aws_region.current](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/region) | data source |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_bad_stream_name"></a> [bad\_stream\_name](#input\_bad\_stream\_name) | The name of the bad kinesis stream that the S3 Loader will insert bad data into | `string` | n/a | yes |
| <a name="input_in_stream_name"></a> [in\_stream\_name](#input\_in\_stream\_name) | The name of the input kinesis stream that the S3 Loader will pull data from | `string` | n/a | yes |
| <a name="input_name"></a> [name](#input\_name) | A name which will be pre-pended to the resources created | `string` | n/a | yes |
| <a name="input_s3_bucket_name"></a> [s3\_bucket\_name](#input\_s3\_bucket\_name) | The name of the S3 Bucket to load data into | `string` | n/a | yes |
| <a name="input_s3_object_prefix"></a> [s3\_object\_prefix](#input\_s3\_object\_prefix) | The prefix to store objects within (e.g. good) (Note: trailing forward slashes are automatically removed) | `string` | n/a | yes |
| <a name="input_ssh_key_name"></a> [ssh\_key\_name](#input\_ssh\_key\_name) | The name of the SSH key-pair to attach to all EC2 nodes deployed | `string` | n/a | yes |
| <a name="input_subnet_ids"></a> [subnet\_ids](#input\_subnet\_ids) | The list of subnets to deploy the S3 Loader across | `list(string)` | n/a | yes |
| <a name="input_vpc_id"></a> [vpc\_id](#input\_vpc\_id) | The VPC to deploy the S3 Loader within | `string` | n/a | yes |
| <a name="input_amazon_linux_2_ami_id"></a> [amazon\_linux\_2\_ami\_id](#input\_amazon\_linux\_2\_ami\_id) | The AMI ID to use which must be based of of Amazon Linux 2; by default the latest community version is used | `string` | `""` | no |
| <a name="input_associate_public_ip_address"></a> [associate\_public\_ip\_address](#input\_associate\_public\_ip\_address) | Whether to assign a public ip address to this instance | `bool` | `true` | no |
| <a name="input_byte_limit"></a> [byte\_limit](#input\_byte\_limit) | The amount of bytes to buffer events before pushing them to S3 | `number` | `25000000` | no |
| <a name="input_cloudwatch_logs_enabled"></a> [cloudwatch\_logs\_enabled](#input\_cloudwatch\_logs\_enabled) | Whether application logs should be reported to CloudWatch | `bool` | `true` | no |
| <a name="input_cloudwatch_logs_retention_days"></a> [cloudwatch\_logs\_retention\_days](#input\_cloudwatch\_logs\_retention\_days) | The length of time in days to retain logs for | `number` | `7` | no |
| <a name="input_enable_auto_scaling"></a> [enable\_auto\_scaling](#input\_enable\_auto\_scaling) | Whether to enable auto-scaling policies for the service | `bool` | `true` | no |
| <a name="input_iam_permissions_boundary"></a> [iam\_permissions\_boundary](#input\_iam\_permissions\_boundary) | The permissions boundary ARN to set on IAM roles created | `string` | `""` | no |
| <a name="input_initial_position"></a> [initial\_position](#input\_initial\_position) | Where to start processing the input Kinesis Stream from (TRIM\_HORIZON or LATEST) | `string` | `"TRIM_HORIZON"` | no |
| <a name="input_instance_type"></a> [instance\_type](#input\_instance\_type) | The instance type to use | `string` | `"t3a.micro"` | no |
| <a name="input_java_opts"></a> [java\_opts](#input\_java\_opts) | Custom JAVA Options | `string` | `"-Dorg.slf4j.simpleLogger.defaultLogLevel=info -XX:MinRAMPercentage=50 -XX:MaxRAMPercentage=75"` | no |
| <a name="input_kcl_read_max_capacity"></a> [kcl\_read\_max\_capacity](#input\_kcl\_read\_max\_capacity) | The maximum READ capacity for the KCL DynamoDB table | `number` | `10` | no |
| <a name="input_kcl_read_min_capacity"></a> [kcl\_read\_min\_capacity](#input\_kcl\_read\_min\_capacity) | The minimum READ capacity for the KCL DynamoDB table | `number` | `1` | no |
| <a name="input_kcl_write_max_capacity"></a> [kcl\_write\_max\_capacity](#input\_kcl\_write\_max\_capacity) | The maximum WRITE capacity for the KCL DynamoDB table | `number` | `10` | no |
| <a name="input_kcl_write_min_capacity"></a> [kcl\_write\_min\_capacity](#input\_kcl\_write\_min\_capacity) | The minimum WRITE capacity for the KCL DynamoDB table | `number` | `1` | no |
| <a name="input_max_size"></a> [max\_size](#input\_max\_size) | The maximum number of servers in this server-group | `number` | `2` | no |
| <a name="input_min_size"></a> [min\_size](#input\_min\_size) | The minimum number of servers in this server-group | `number` | `1` | no |
| <a name="input_partition_format"></a> [partition\_format](#input\_partition\_format) | The pattern to partition data saved to S3 (e.g. https://github.com/snowplow/snowplow-s3-loader/blob/master/config/config.hocon.sample#L28-L31) | `string` | `""` | no |
| <a name="input_purpose"></a> [purpose](#input\_purpose) | Describes the purpose which this S3 loader is being used for (RAW, ENRICHED\_EVENTS or JSON). RAW simply sinks data 1:1, ENRICHED\_EVENTS work with monitoring.statsd to report metrics (identical to RAW otherwise), SELF\_DESCRIBING partitions self-describing data (such as JSON) by its schema | `string` | `"RAW"` | no |
| <a name="input_record_limit"></a> [record\_limit](#input\_record\_limit) | The number of events to buffer before pushing them to S3 | `number` | `100000` | no |
| <a name="input_s3_format"></a> [s3\_format](#input\_s3\_format) | The format of the data to be loaded into S3 ('gzip' or 'lzo') | `string` | `"gzip"` | no |
| <a name="input_scale_down_cooldown_sec"></a> [scale\_down\_cooldown\_sec](#input\_scale\_down\_cooldown\_sec) | Time (in seconds) until another scale-down action can occur | `number` | `600` | no |
| <a name="input_scale_down_cpu_threshold_percentage"></a> [scale\_down\_cpu\_threshold\_percentage](#input\_scale\_down\_cpu\_threshold\_percentage) | The average CPU percentage that we must be below to scale-down | `number` | `20` | no |
| <a name="input_scale_down_eval_minutes"></a> [scale\_down\_eval\_minutes](#input\_scale\_down\_eval\_minutes) | The number of consecutive minutes that we must be below the threshold to scale-down | `number` | `60` | no |
| <a name="input_scale_up_cooldown_sec"></a> [scale\_up\_cooldown\_sec](#input\_scale\_up\_cooldown\_sec) | Time (in seconds) until another scale-up action can occur | `number` | `180` | no |
| <a name="input_scale_up_cpu_threshold_percentage"></a> [scale\_up\_cpu\_threshold\_percentage](#input\_scale\_up\_cpu\_threshold\_percentage) | The average CPU percentage that must be exceeded to scale-up | `number` | `60` | no |
| <a name="input_scale_up_eval_minutes"></a> [scale\_up\_eval\_minutes](#input\_scale\_up\_eval\_minutes) | The number of consecutive minutes that the threshold must be breached to scale-up | `number` | `5` | no |
| <a name="input_ssh_ip_allowlist"></a> [ssh\_ip\_allowlist](#input\_ssh\_ip\_allowlist) | The list of CIDR ranges to allow SSH traffic from | `list(any)` | <pre>[<br>  "0.0.0.0/0"<br>]</pre> | no |
| <a name="input_tags"></a> [tags](#input\_tags) | The tags to append to this resource | `map(string)` | `{}` | no |
| <a name="input_telemetry_enabled"></a> [telemetry\_enabled](#input\_telemetry\_enabled) | Whether or not to send telemetry information back to Snowplow Analytics Ltd | `bool` | `true` | no |
| <a name="input_time_limit_ms"></a> [time\_limit\_ms](#input\_time\_limit\_ms) | The amount of time to buffer events before pushing them to S3 | `number` | `180000` | no |
| <a name="input_user_provided_id"></a> [user\_provided\_id](#input\_user\_provided\_id) | An optional unique identifier to identify the telemetry events emitted by this stack | `string` | `""` | no |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_asg_id"></a> [asg\_id](#output\_asg\_id) | ID of the ASG |
| <a name="output_asg_name"></a> [asg\_name](#output\_asg\_name) | Name of the ASG |
| <a name="output_sg_id"></a> [sg\_id](#output\_sg\_id) | ID of the security group attached to the S3 Loader servers |

# Copyright and license

The Terraform AWS S3 Loader Kinesis on EC2 project is Copyright 2021-2023 Snowplow Analytics Ltd.

Licensed under the [Apache License, Version 2.0][license] (the "License");
you may not use this software except in compliance with the License.

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

[release]: https://github.com/snowplow-devops/terraform-aws-s3-loader-kinesis-ec2/releases/latest
[release-image]: https://img.shields.io/github/v/release/snowplow-devops/terraform-aws-s3-loader-kinesis-ec2

[ci]: https://github.com/snowplow-devops/terraform-aws-s3-loader-kinesis-ec2/actions?query=workflow%3Aci
[ci-image]: https://github.com/snowplow-devops/terraform-aws-s3-loader-kinesis-ec2/workflows/ci/badge.svg

[license]: https://www.apache.org/licenses/LICENSE-2.0
[license-image]: https://img.shields.io/badge/license-Apache--2-blue.svg?style=flat

[registry]: https://registry.terraform.io/modules/snowplow-devops/s3-loader-kinesis-ec2/aws/latest
[registry-image]: https://img.shields.io/static/v1?label=Terraform&message=Registry&color=7B42BC&logo=terraform

[source]: https://github.com/snowplow/snowplow-s3-loader
[source-image]: https://img.shields.io/static/v1?label=Snowplow&message=S3%20Loader&color=0E9BA4&logo=GitHub
