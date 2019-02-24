# terraform-aws-wireguard

A Terraform module to deploy a WireGuard VPN server on AWS.

Beware! Work in progress! Use at your own peril! Here be dragons!

## Prerequisites
Before using this module, you'll need to generate a key pair for your server and client, and store the server's private key and client's public key in AWS SSM, which cloud-init will source and add to WireGuard's configuration.

- Install the WireGuard tools for your OS: https://www.wireguard.com/install/
- Generate a key pair for the client
  - `wg genkey | tee client-privatekey | wg pubkey > client-publickey`
- Generate a key pair for the server
  - `wg genkey | tee server-privatekey | wg pubkey > server-publickey`

- Add the client public key to the AWS SSM parameter: `/wireguard/wg-laptop-public-key`
  - `aws ssm put-parameter --name /wireguard/wg-laptop-public-key --type SecureString --value $ClientPublicKeyValue`
- Add the server private key to the AWS SSM parameter: `/wireguard/wg-server-private-key`
  - `aws ssm put-parameter --name /wireguard/wg-server-private-key --type SecureString --value $ServerPrivateKeyValue`

## Variables
| Variable Name | Type | Required |Description |
|---------------|-------------|-------------|-------------|
|`public_subnet_ids`|`list`|Yes|A list of subnets for the Autoscaling Group to use for launching instances. May be a single subnet, but it must be an element in a list.|
|`ssh_key_id`|`string`|Yes|A SSH public key ID to add to the VPN instance.|
|`vpc_id`|`string`|Yes|The VPC ID in which Terraform will launch the resources.|
|`ingress_security_group_id`|`string`|Yes|The ID of the Security Group to allow SSH access from.|
|`ami_id`|`string`|No. Defaults to Ubuntu 16.04 AMI in us-east-1|The AMI ID to use.|

## Usage
```
module "wireguard" {
  source            = "git@github.com:jmhale/terraform-wireguard.git"
  ssh_key_id        = "ssh-key-id-0987654"
  vpc_id            = "vpc-01234567"
  public_subnet_ids = ["subnet-01234567"]
}
```

## Outputs
| Output Name | Description |
|---------------|-------------|
|`vpn_ip`|The public IPv4 address of the AWS Elastic IP assigned to the instance.|
|`vpn_sg_id`|ID of the internal Security Group to associate with other resources needing to be accessed on VPN|

## Caveats

- I would strongly recommend forking this repo or cloning it locally and change the `source` definition to be something that you control. You really don't want your infra to be at the mercy of my changes.


## To-do

- Add the mechanism to attach the EIP to the instance, via cloud-init.
