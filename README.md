# terraform aws quicksight destination ![](https://img.shields.io/github/actions/workflow/status/wearetechnative/terraform-aws-quicksight-destination/tflint.yaml?style=plastic)

This Terraform configuration is designed to set up a receiving S3 bucket and optionally enable AWS Cost and Usage Reports (CUR) by configuring the necessary IAM roles and policies. The setup ensures secure storage, replication, and access to your CUR data for analysis in a destination account.

## How does it work

### First use after you clone this repository or when .pre-commit-config.yaml is updated

Run `pre-commit install` to install any guardrails implemented using pre-commit.

See [pre-commit installation](https://pre-commit.com/#install) on how to install pre-commit

## Usage

Below is an example configuration for using this module with the necessary providers:

```hcl
provider "aws" {
  profile = "data_collection"
  region  = "eu-central-1"
  alias   = "data_collection"
}

provider "aws" {
  region = "us-east-1"
  alias  = "useast1"
}

module "cur_data_collection_account" {
  source = "./destination/"

  source_account_ids = ["123456789012"] # Change to sending accounts 
  create_cur         = false # Set to true to create an additional CUR in the aggregation account

  providers = {
    aws.useast1 = aws.useast1
  }
}
```
### Example `terraform.tfvars`

```hcl
resource_prefix = "TechNative"
kms_key_id = "arn:aws:kms:us-east-1:123456789012:key/your-kms-key-id"
tags = {
  Environment = "Production"
  Owner       = "Finance"
}

s3_access_logging = {
  enabled = true
  bucket  = "my-logging-bucket"
  prefix  = "logs/"
}

source_account_ids = ["123456789012"] # Change to sending accounts
create_cur         = false # Set to true to create an additional CUR in the aggregation account
```
## Troubleshooting

- **Access Denied Errors**: Ensure that your AWS credentials have sufficient permissions to create and manage the resources defined in this Terraform configuration.
- **KMS Key Issues**: If using KMS encryption, verify that the key exists and that your IAM roles have the correct permissions to use the key.

<!-- BEGIN_TF_DOCS -->
## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 1.0 |
| <a name="requirement_aws"></a> [aws](#requirement\_aws) | >= 3.0 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_aws"></a> [aws](#provider\_aws) | >= 3.0 |
| <a name="provider_aws.useast1"></a> [aws.useast1](#provider\_aws.useast1) | >= 3.0 |

## Modules

No modules.

## Resources

| Name | Type |
|------|------|
| [aws_cur_report_definition.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/cur_report_definition) | resource |
| [aws_s3_bucket.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket) | resource |
| [aws_s3_bucket_lifecycle_configuration.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_lifecycle_configuration) | resource |
| [aws_s3_bucket_logging.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_logging) | resource |
| [aws_s3_bucket_ownership_controls.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_ownership_controls) | resource |
| [aws_s3_bucket_policy.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_policy) | resource |
| [aws_s3_bucket_public_access_block.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_public_access_block) | resource |
| [aws_s3_bucket_server_side_encryption_configuration.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_server_side_encryption_configuration) | resource |
| [aws_s3_bucket_versioning.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_versioning) | resource |
| [aws_caller_identity.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/caller_identity) | data source |
| [aws_iam_policy_document.bucket_policy](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/iam_policy_document) | data source |
| [aws_partition.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/partition) | data source |
| [aws_region.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/region) | data source |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_create_cur"></a> [create\_cur](#input\_create\_cur) | Whether to create a local CUR in the destination account or not. Set this to true if the destination account is NOT covered in the CUR of the source accounts | `bool` | n/a | yes |
| <a name="input_cur_name_suffix"></a> [cur\_name\_suffix](#input\_cur\_name\_suffix) | Suffix used to name the local CUR report if create\_cur is `true` | `string` | `"cur"` | no |
| <a name="input_enable_split_cost_allocation_data"></a> [enable\_split\_cost\_allocation\_data](#input\_enable\_split\_cost\_allocation\_data) | Enable split cost allocation data for ECS and EKS for this CUR report | `bool` | `false` | no |
| <a name="input_kms_key_id"></a> [kms\_key\_id](#input\_kms\_key\_id) | !!!WARNING!!! EXPERIMENTAL - Do not use unless you know what you are doing. The correct key policies and IAM permissions<br>on the S3 replication role must be configured external to this module.<br>  - If create\_cur is true, the "billingreports.amazonaws.com" service must have access to encrypt S3 objects with the key ID provided<br>  - See https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication-config-for-kms-objects.html for information<br>    on permissions required for replicating KMS-encrypted objects | `string` | `null` | no |
| <a name="input_resource_prefix"></a> [resource\_prefix](#input\_resource\_prefix) | Prefix used for all named resources, including S3 Bucket | `string` | `"cid"` | no |
| <a name="input_s3_access_logging"></a> [s3\_access\_logging](#input\_s3\_access\_logging) | S3 Access Logging configuration for the CUR bucket | <pre>object({<br>    enabled = bool<br>    bucket  = string<br>    prefix  = string<br>  })</pre> | <pre>{<br>  "bucket": null,<br>  "enabled": false,<br>  "prefix": null<br>}</pre> | no |
| <a name="input_source_account_ids"></a> [source\_account\_ids](#input\_source\_account\_ids) | List of all source accounts that will replicate CUR Data. Ex:  [12345678912,98745612312,...] (fill only on Destination Account) | `list(string)` | n/a | yes |
| <a name="input_tags"></a> [tags](#input\_tags) | Map of tags to apply to module resources | `map(string)` | `{}` | no |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_cur_bucket_arn"></a> [cur\_bucket\_arn](#output\_cur\_bucket\_arn) | ARN of the S3 Bucket where the Cost and Usage Report is delivered |
| <a name="output_cur_bucket_name"></a> [cur\_bucket\_name](#output\_cur\_bucket\_name) | Name of the S3 Bucket where the Cost and Usage Report is delivered |
| <a name="output_cur_report_arn"></a> [cur\_report\_arn](#output\_cur\_report\_arn) | ARN of the Cost and Usage Report |
<!-- END_TF_DOCS -->
