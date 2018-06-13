# aws_manage_natgws

## Description:

Manages AWS Virtual Private Clouds (VPCs) NAT Gateways as follows:

- Running the role will create the NAT Gateways defined in `aws_natgws` when the variable `arg_action` is not defined or is defined and set to `create`
- Running the role will delete the NAT Gateways defined in `aws_natgws` and any associated elastic IP addresses and Route Table routes when the variable `arg_action` is defined and is set to `delete`
- Running the [get_natgw_facts.yml](./tasks/get_natgw_facts.yml) play will return information about the specified NAT Gateway.

## Dependencies

This role uses the AWS cli.  It therefore requires AWS credentials as environment variables or in a configuration file (see [here](https://docs.aws.amazon.com/cli/latest/topic/config-vars.html#cli-aws-help-config-vars)).  The role `aws_credentials` will create a config file for use with the AWS cli.

## Behaviour:

**Feature:** Create AWS NAT Gateways
- **Given** valid AWS credentials
- **Given** one or more existing AWS VPCs
- **Given** Subnets exist in each VPC
- **When** the script is executed when `arg_action` is not defined or is defined and not set to `delete`
- **Then** the NAT Gateway is created in the specified subnet
- **When** the script is executed when `arg_action` is defined and is set to `delete`
- **Then** the NAT Gateway and its associated elastic IPs and Route Table routes are deleted 

## Configuration:

Common variables used by this role:

| Variable | Description | Example |
|---|---|---|
| **aws_region** | AWS region | See [AWS regions](http://docs.aws.amazon.com/general/latest/gr/rande.html#ec2_region) |
| **aws_access_key** | AWS access key | AKIAIOSFODNN7EXAMPLE |
| **aws_secret_key** | AWS secret key | wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY |
| **resource_tags** | List of project tags | see usage |

Variables specific to this role:

| Variable | Description | Example |
|---|---|---|
| **aws_natgws** | List of NAT Gateways to create | see usage |
| **natgw_tag** | The `Name:` tag of the NAT Gateway to get information about | see usage |

## Usage:

How to invoke the role from a playbook (create NAT Gateways):

```yaml
- name: Create AWS NAT Gateway
  include_role:
    name: "aws_manage_natgws"
  vars:
    aws_access_key: AKIAIOSFODNN7EXAMPLE
    aws_secret_key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
    aws_region: eu-west-2  # London
    resource_tags:
      Name: "{{ resource_name }}"   # Set to upper case aws_natgw_tag
      Owner: "Acme Co."

    aws_natgws:
      - aws_natgw_subnet: "Test-Public-A"
        aws_natgw_region: "{{ aws_region }}"
        aws_natgw_tag: "TEST-NAT-GW-A"
```

How to invoke the role from a playbook (delete NAT Gateways):

```yaml
- name: Create AWS NAT Gateway
  include_role:
    name: "aws_manage_natgws"
  vars:
    arg_action: "delete"
    aws_access_key: AKIAIOSFODNN7EXAMPLE
    aws_secret_key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
    aws_region: eu-west-2  # London

    aws_natgws:
      - aws_natgw_subnet: "Test-Public-A"
        aws_natgw_region: "{{ aws_region }}"
        aws_natgw_tag: "TEST-NAT-GW-A"
```

How to get NAT Gateway information:

```yaml
- name: Get NAT Gateway facts
  include_role:
    name: "aws_manage_natgws"
    tasks_from: "get_natgw_facts"
  vars:
    natgw_tag: "TEST-NAT-GW-A"
```

Facts are registered in `natgw_facts` as shown below:

```yaml
"natgw_facts": {
        "changed": false, 
        "failed": false, 
        "result": [
            {
                "create_time": "2017-10-04T14:36:24+00:00", 
                "nat_gateway_addresses": [
                    {
                        "allocation_id": "eipalloc-04faf59d", 
                        "network_interface_id": "eni-739fab62", 
                        "private_ip": "10.12.1.137", 
                        "public_ip": "52.56.40.208"
                    }
                ], 
                "nat_gateway_id": "nat-0b7df79d0d3763787", 
                "state": "available", 
                "subnet_id": "subnet-f6ad385d", 
                "tags": [
                    {
                        "key": "Name", 
                        "value": "TEST-CTRLNAT-GW-A"
                    }
                ], 
                "vpc_id": "vpc-a60fl4de"
            }
        ]
    }
```