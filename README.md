Ansible Role: wireguard-configuration
=========

Deploy Wireguard network configurtion and and peers stored in AWS SSM Parameter Store. 

It is assumed that:
* Wireguard is installed, but not yet configured
* AWS CLI is installed
* AWS SSM Parameter Store pre-populated with Endpoint and Peer configuration.
* An EC2 tag has been created called wireguard_endpoint

The idea behind this role is to have an off-host location to store a sinle Wireguard EC2 instance's public key. The requirements were:
* Replace OpenVPN
* Single Wireguard VPN endpoint per AWS region
* Make the endpoint instances ephemeral, store useful information off-host

Requirements
------------

A running EC2 instance with an IAM policy assigned which will allow read access to AWS SSM Parameter Store.

This module requires the following:

    collections:
      - name: amazon.aws
        version: 3.3.1


AWS SSM Parameter Store Hierarchy
--------------
Endpoint configuration
```
root@london.example.com:~# for i in address prikey psk; do /usr/local/bin/aws ssm get-parameter --name /wireguard/london.example.com/$i --region eu-west-2 --output json --with-decryption; done
{
    "Parameter": {
        "Name": "/wireguard/london.example.com/address",
        "Type": "String",
        "Value": "172.200.201.2/32",
        "Version": 1,
        "LastModifiedDate": "2022-07-25T13:54:51.649000+00:00",
        "ARN": "arn:aws:ssm:eu-west-2:012345678901:parameter/wireguard/london.example.com/address",
        "DataType": "text"
    }
}
{
    "Parameter": {
        "Name": "/wireguard/london.example.com/prikey",
        "Type": "SecureString",
        "Value": "NoThInGtOsEeHeReNoThInGtOsEeHeReNoThInGtOsE=",
        "Version": 1,
        "LastModifiedDate": "2022-07-25T13:47:49.342000+00:00",
        "ARN": "arn:aws:ssm:eu-west-2:012345678901:parameter/wireguard/london.example.com/prikey",
        "DataType": "text"
    }
}
{
    "Parameter": {
        "Name": "/wireguard/london.example.com/psk",
        "Type": "SecureString",
        "Value": "NoThInGtOsEeHeReNoThInGtOsEeHeReNoThInGtOsE=",
        "Version": 1,
        "LastModifiedDate": "2022-07-21T11:54:20.227000+00:00",
        "ARN": "arn:aws:ssm:eu-west-2:012345678901:parameter/wireguard/london.example.com/psk",
        "DataType": "text"
    }
}
```

Peer configuration
```
root@london.example.com:~# /usr/local/bin/aws ssm get-parameters-by-path --path /wireguard/london.example.com/peers --region eu-west-2 --output json --with-decryption
{
    "Parameters": [
        {
            "Name": "/wireguard/london.example.com/peers/0001",
            "Type": "SecureString",
            "Value": "{ \"pubkey\": \"NoThInGtOsEeHeReNoThInGtOsEeHeReNoThInGtOsE=\", \"host\": \"172.200.201.100/32\"}",
            "Version": 1,
            "LastModifiedDate": "2022-07-21T11:09:28.247000+00:00",
            "ARN": "arn:aws:ssm:eu-west-2:012345678901:parameter/wireguard/london.example.com/peers/0001",
            "DataType": "text"
        }
    ]
}
```

Role Variables
--------------

Customisable variables have been declared in `defaults/main.yml`.

The base directory for Parameter Store:

      ssm_root_path: "/wireguard"

Deploying this on an `us-east-1` instance, will create the following parameter path:

      /wireguard/us-east-1/public_key

Dependencies
------------

You will need a running instance of Wireguard on the EC2 instance and a policy assigned to the EC2 instance's role which will permit read access to SSM Parameter Store, ideally you have limited it to the value of `wireguard_ssm_base_path`.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    hosts:
      localhost:
      roles:
        - role: gavmain.wireguard-configuration
          vars:
            wireguard_ssm_base_path: /wireguard

License
-------

MIT

Author Information
------------------

This role was created in 2021 by [Gav Main](https://github.com/gavmain).
