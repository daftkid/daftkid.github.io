---
title:   "My Contribution to Open Source"
tags:    opensource
excerpt: ""
permalink: /prs/
---

On this page you can find a list of all my `Pull Requests` into Opensource projects.

Legend:
- ✅ - merged
- 👀 - waiting for review

## [terraform-provider-aws][tf-aws]

The Terraform AWS provider is a plugin for `Terraform` that allows for the full lifecycle management of AWS resources.
This provider is maintained internally by the HashiCorp AWS Provider team.

- ✅ [r/aws_cloudfront_response_headers_policy added server_timing_headers_config attribute](/prs/24913)
- ✅ [r/aws_cloudsearch_domain added source_fields attribute for index](/prs/24915)
- ✅ [r/aws_dms_endpoint: Secrets manager secret for aws_dms_endpoint with engine_name = redshift](/prs/25080)
- ✅ [r/aws_neptune_cluster: added allow_major_version_upgrade attribute](/prs/25140)
- ✅ [r/aws_emr_cluster: Add sc1 as EBS allowed volume type for AWS EMR cluster resource](/prs/25255)
- ✅ [r/aws_lambda_function: Added Lambda function name validation](/prs/25259)
- 👀 [r/aws_fms_policy: Added policy_option config block for FMS policy](/prs/25362)

## [prowler]

`Prowler` is an Open Source security tool to perform AWS and Azure security best practices assessments, audits, incident response, continuous monitoring, hardening and forensics readiness.

- ✅ [fix(metadata) fixed typo in title for awslambda_function_not_publicly_accessible_check](/prs/1826)


[prowler]: https://github.com/prowler-cloud/prowler
[tf-aws]: https://github.com/hashicorp/terraform-provider-aws
