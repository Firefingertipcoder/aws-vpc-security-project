# Architecture

The AWS VPC security project uses this traffic pattern:

```text
Internet
   |
Internet Gateway
   |
Public-A
   |
Bastion-Secure
   |
SSH
   |
Private-A
   |
Private-Web-Server
   |
NAT Gateway
   |
Internet
```

## Subnet layout

- Public-A: `10.0.1.0/24`
- Public-B: `10.0.2.0/24`
- Private-A: `10.0.11.0/24`
- Private-B: `10.0.12.0/24`

The project spans two Availability Zones. Public-A and Private-A are in one AZ, while Public-B and Private-B are in another.

## Trust boundaries

1. Internet to bastion: restricted by the bastion Security Group.
2. Bastion to private server: restricted by `Web-SG`.
3. Private server to Internet: outbound through NAT Gateway.
4. Private subnet traffic: additionally controlled by `Secure-NACL`.
5. Network activity: monitored through VPC Flow Logs and CloudWatch.
