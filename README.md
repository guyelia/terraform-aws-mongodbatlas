# terraform-aws-mongodbatlas
Terraform module which creates a MongoDB Atlas cluster on AWS

These types of resources are supported:
* [Project](https://www.terraform.io/docs/providers/mongodbatlas/r/project.html)
* [Cluster](https://www.terraform.io/docs/providers/mongodbatlas/r/cluster.html)
* [Teams](https://www.terraform.io/docs/providers/mongodbatlas/r/team.html)
* [Whitelists](https://www.terraform.io/docs/providers/mongodbatlas/r/project_ip_whitelist.html)
* [Network Peering](https://www.terraform.io/docs/providers/mongodbatlas/r/network_peering.html)

## Terraform versions

Terraform versions >=0.12 are supported
   
## Usage

```hcl
module "atlas_cluster" {
  source = ".//terraform-aws-mongodbatlas"

  project_name = "my-project"
  org_id = "5edf67f9b9b528996228111"

  teams = {
    Devops: {
      users: ["example@mail.io", "user@mail.io"]
      role: "GROUP_OWNER"
    },
    DevTeam: {
      users: ["developer@mail.io",]
      role: "GROUP_READ_ONLY"
    }
  }

  white_lists = {
    "example comment": "10.0.0.1/32",
    "second example": "10.10.10.8/32"
  }

  region = "EU_WEST_1"

  cluster_name = "MyCluster"

  instance_type = "M30"
  mongodb_major_ver = 4.2
  cluster_type = "REPLICASET"
  num_shards = 1
  replication_factor = 3
  provider_backup = true
  pit_enabled = false

}
```
## Prerequisites
* [MongoDB Cloud account](https://www.mongodb.com/cloud)
* [MongoDB Atlas Organization](https://cloud.mongodb.com/v2#/preferences/organizations/create)
* [MongoDB Atlas API key](https://www.terraform.io/docs/providers/mongodbatlas/index.html)

#### Manual tasks:

* Configure the API key before applying the module
* Get your MongoDB Atlas Organization ID from the UI

#### Create Teams:
In case you want to create Teams and associate them with the project, users must be added in advance and making sure they have accepted the invitation.
Users who have not accepted an invitation to join the organization cannot be added as team members. 

## VPC Peering
In case vpc peering is required, AWS provider will be used, the following information is required:
* [Access to AWS](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html)
* VPC ID
* [AWS account ID](https://docs.aws.amazon.com/general/latest/gr/acct-identifiers.html)
* Region
* [Routable cidr block](https://www.terraform.io/docs/providers/mongodbatlas/r/network_peering.html#route_table_cidr_block)

You can add them manually, or through other Terraform resources, and pass it to the module via ```vpc_peer``` variable:
```hcl
  vpc_peer = {
    vpc_peer1 : {
      aws_account_id : "020201234877"
      region : "eu-west-1"
      vpc_id : "vpc-0240c8a47312svc3e"
      route_table_cidr_block : "172.16.0.0/16"
    },
    vpc_peer2 : {
      aws_account_id : "0205432147877"
      region : "eu-central-1"
      vpc_id : "vpc-0f0dd82430bhv0e1a"
      route_table_cidr_block : "172.17.0.0/16"
    }
  }
```

You can see this [example](https://github.com/guyelia/terraform-aws-mongodbatlas/blob/master/examples/basic/main.tf) in the examples folder.

## Requirements

| Name | Version |
|------|---------|
| terraform | \>= 0.12 |


## Providers

| Name | Version |
|------|---------|
|mongodbatlas|




## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| auto_scaling_disk_gb_enabled | Indicating if disk auto-scaling is enabled. | `bool` | `true` | no |
| cluster_name | The cluster name. | `string` | `""` | yes |
| cluster_type | The MongoDB Atlas cluster type - SHARDED/REPLICASET/GEOSHARDED. | `string` | `""` | yes |
| disk_size_gb | Capacity,in gigabytes,of the host’s root volume. | `number` |  | no |
| instance_type | The Atlas instance-type name. | `string` | `""` | yes |
| mongodb_major_ver | The MongoDB cluster major version. | `number` | `""` | no |
| num_shards | number of shards. | `number` | `""` | no |
| org_id | The ID of the Atlas organization you want to create the project within. | `string` | `""` | yes |
| pit_enabled | Indicating if the cluster uses Continuous Cloud Backup. | `bool` | `false` | no |
| project_name | The name of the project you want to create. | `string` | `""` | yes |
| provider_backup | Indicating if the cluster uses Cloud Backup for backups. | `bool` | `true` | no |
| provider_disk_iops | The maximum IOPS the system can perform. | `number` | | no |
| provider_encrypt_ebs_volume | Indicating if the AWS EBS encryption feature encrypts the server’s root volume. | `bool` | `false` | no |
| region | The AWS region-name that the cluster will be deployed on. | `string` | `""` | yes |
| replication_factor | The Number of replica set members, possible values are 3/5/7. | `number` |  | no |
| teams | An object that contains all the groups that should be created in the project. | `map(any)` | `{}` | no |
| volume_type | STANDARD or PROVISIONED for IOPS higher than the default instance IOPS. | `string` | `STANDARD` | no |
| vpc_peer | An object that contains all VPC peering requests from the cluster to AWS VPC's | `map(any)` | {} | no
| white_lists | An object that contains all the network white-lists that should be created in the project. | `map(any)` | `{}` | no |


## Outputs

| Name | Description |
|------|-------------|
| cluster_id | The cluster ID. |
| connection_strings | Set of connection strings that your applications use to connect to this cluster. |
| container_id | he Network Peering Container ID. |
| mongo_db_version | Version of MongoDB the cluster runs, in major-version.minor-version format. |
| mongo_uri | Base connection string for the cluster. |
| mongo_uri_updated | Lists when the connection string was last updated. |
| mongo_uri_with_options | connection string for connecting to the Atlas cluster. Includes the replicaSet, ssl, and authSource query parameters in the connection string with values appropriate for the cluster. |
| paused | Flag that indicates whether the cluster is paused or not. |
| srv_address | Connection string for connecting to the Atlas cluster, +srv modifier forces the connection to use TLS/SSL  |
| state_name |  Current state of the cluster. |
