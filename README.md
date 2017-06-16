tf_aws_ecs_pganalyze
===========

Terraform module for deploying and managing [Amazon Elasticsearch Service](https://aws.amazon.com/documentation/ecs_pganalyze-service/).

This module creates an Elasticsearch domain and applies an access policy which permits unlimited access based on the intersection of the following two criteria:

* source IP address
* client IAM role

See [this Stack Overflow post](http://stackoverflow.com/questions/32978026/proper-access-policy-for-amazon-elastic-search-cluster) for further discussion of access policies for Elasticsearch.

Several options affect the resilience and scalability of your Elasticsearch domain.  For a production deployment, set `instance_count` to an even number greater than or equal to 10 (the default is 6), choose an `instance_type` that is not in the T2 family, and set `es_zone_awareness` to `true`.  This will result in a cluster with three dedicated master nodes, balanced across two availability zones.

For a production deployment it may also make sense to use EBS volumes rather that instance storage; to do so, set `ebs_volume_size` greater than 0 and optionally specify a value for `ebs_volume_type` (right now the only supported values are `gp2` and `magnetic`).

----------------------
#### Required
None (but `domain_name` and `management_public_ip_addresses` are strongly recommended).

#### Optional
- `domain_name` - unique identifier for the domain.  The module will prefix it with `tf-`. _e.g._ `domain_name = foo` will result in a domain called `tf-foo`.
- `es_version` - Elasticsearch version.
- `instance_type` - Elasticsearch instance type to use for data nodes (and dedicated master nodes unless otherwise specified).
- `instance_count` - Number of instances in the cluster.
- `dedicated_master_type` - Elasticsearch instance type to use for dedicated master nodes.
- `management_iam_roles` - List of ARNs of IAM roles to be granted full access to the domain.
- `management_public_ip_addresses` - List of IP addresses or CIDR blocks from which to permit full access to the domain.  Remember that for hosts inside a VPC, you'll want to specify the Elastic IP(s) of any relevant NAT Gateway(s).
- `es_zone_awareness` - Enable or disable zone awareness (balancing instances across multiple availability zones).  Note that setting this parameter to `true` and then requesting an odd number of nodes will result in an invalid cluster configuration.
- `ebs_volume_size` - Size in GB of EBS volume to attach to each node and use for data storage.  If this parameter is set to 0 (the default), nodes will use instance storage.
- `ebs_volume_type` - Storage class for EBS volumes.  Just use `gp2`.
- `snapshot_start_hour` - Hour of the day (in UTC) at which to begin daily snapshots.

Usage
-----

```hcl

module "es" {
  source                         = "github.com/terraform-community-modules/tf_aws_ecs_pganalyze?ref=0.0.1"
  domain_name                    = "my-ecs_pganalyze-domain"
  management_public_ip_addresses = ["34.203.XXX.YYY"]
  instance_count                 = 16
  instance_type                  = "m4.2xlarge.ecs_pganalyze"
  dedicated_master_type          = "m4.large.ecs_pganalyze"
  es_zone_awareness              = true
  ebs_volume_size                = 100
  ...
}

```

Outputs
=======
- `arn` - ARN of the created Elasticsearch domain.
- `domain_id` - Unique identifier for the domain.
- `endpoint` - Domain-specific endpoint used to submit index, search, and data upload requests.  Kibana is available at `https://${endpoint}/_plugin/kibana/`.

Authors
=======

[Steve Huff](https://github.com/hakamadare)

Changelog
=========

0.0.1 - Initial release.

License
=======

This software is released under the MIT License (see `LICENSE.md`).
