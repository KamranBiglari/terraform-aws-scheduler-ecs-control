# terraform-aws-scheduler-event

[![GitHub release (latest by date)](https://img.shields.io/github/v/release/KamranBiglari/terraform-aws-scheduler-event)](https://github.com/KamranBiglari/terraform-aws-scheduler-event/releases/latest)


## How to use 


## Example

```
terraform {
  required_version = ">= 0.13.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region  = "eu-west-2"
}


module "scheduler_ecs_control" {
    source  = "KamranBiglari/scheduler-event/aws"
    name_prefix    = "ecs-servicecontrol"
    group_name = "default"

    create_default_role = true
    default_role_name = "-servicecontrol-iam-role"
    default_role_policy_arns = [
        "arn:aws:iam::aws:policy/CloudWatchFullAccess",
        "arn:aws:iam::aws:policy/AWSStepFunctionsFullAccess",
    ]

    rules   = [
        {
            name = "run-on-2030-01-01-01-00"
            timezone = "America/New_York"
            expression = "2030-01-01T01:00:00Z"
            flexible_time_window = {
                mode = "OFF"
            } 
            target = {
                arn = "arn:aws:scheduler:::aws-sdk:ecs:updateService"
                role_arn = "arn:aws:iam::123456789012:role/ecs-service-role"
                input = {
                    Service = "MyData"
                    Cluster = "MyCluster"
                }
                retry_policy = {
                    maximum_retry_attempts = 1
                    maximum_event_age_in_minutes = 60
                }
            }
        },
        {
            name = "start_at_17_00_every_sunday"
            expression = "cron(1 17 ? * SUN *)"
            timezone = "Asia/Tokyo"
            flexible_time_window = {
                maximum_window_in_minutes = 1
                mode = "FLEXIBLE"
            }
            target = {
                arn = "arn:aws:scheduler:::aws-sdk:ecs:updateService"
                role = {
                    name = "ecs-service-role"
                    assume_role_policy = {
                        "Version" : "2008-10-17",
                        "Statement" : [
                        {
                            "Sid" : "",
                            "Effect" : "Allow",
                            "Principal" : {
                            "Service" : "scheduler.amazonaws.com"
                            },
                            "Action" : "sts:AssumeRole"
                        }
                        ]
                    }
                    managed_policy_arns = [
                        "arn:aws:iam::aws:policy/CloudWatchFullAccess",
                        "arn:aws:iam::aws:policy/AWSStepFunctionsFullAccess",
                    ]
                }
                input = {
                    Service = "MyData"
                    Cluster = "MyCluster"
                }
            }
            state = "DISABLED" # Optional. Default: ENABLED
        },
        {
            name = "new_deployment_every_5_minutes"
            type = "rate"
            action = "new_deployment"
            expression = "rate(5 minutes)"
            target = {
                arn = "arn:aws:scheduler:::aws-sdk:ecs:updateService"
                input = {
                    Service = "MyData"
                    Cluster = "MyCluster"
                }
            }
        }
    ]

}

```
<!-- BEGIN_TF_DOCS -->
## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 1.0.11 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_aws"></a> [aws](#provider\_aws) | 5.29.0 |

## Modules

No modules.

## Resources

| Name | Type |
|------|------|
| [aws_iam_role.default](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role) | resource |
| [aws_iam_role.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role) | resource |
| [aws_scheduler_schedule.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/scheduler_schedule) | resource |
| [aws_scheduler_schedule_group.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/scheduler_schedule_group) | resource |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_create_default_role"></a> [create\_default\_role](#input\_create\_default\_role) | create default role | `bool` | `false` | no |
| <a name="input_create_group_name"></a> [create\_group\_name](#input\_create\_group\_name) | create group | `bool` | `false` | no |
| <a name="input_default_role_name"></a> [default\_role\_name](#input\_default\_role\_name) | default role name | `string` | `""` | no |
| <a name="input_default_role_policy"></a> [default\_role\_policy](#input\_default\_role\_policy) | default role assume policy | `any` | <pre>{<br>  "Statement": [<br>    {<br>      "Action": "sts:AssumeRole",<br>      "Effect": "Allow",<br>      "Principal": {<br>        "Service": "scheduler.amazonaws.com"<br>      },<br>      "Sid": ""<br>    }<br>  ],<br>  "Version": "2008-10-17"<br>}</pre> | no |
| <a name="input_default_role_policy_arns"></a> [default\_role\_policy\_arns](#input\_default\_role\_policy\_arns) | default role policy arns | `list(string)` | `[]` | no |
| <a name="input_group_name"></a> [group\_name](#input\_group\_name) | group name | `string` | `"default"` | no |
| <a name="input_group_tags"></a> [group\_tags](#input\_group\_tags) | group tags | `map(string)` | `{}` | no |
| <a name="input_name_prefix"></a> [name\_prefix](#input\_name\_prefix) | name prefix | `string` | n/a | yes |
| <a name="input_rules"></a> [rules](#input\_rules) | rules | `any` | n/a | yes |

## Outputs

No outputs.
<!-- END_TF_DOCS -->