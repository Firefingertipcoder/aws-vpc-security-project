# AWS VPC Security Project

A hands-on AWS networking and security project demonstrating a secure VPC architecture with public/private subnets, a bastion host, NAT Gateway, Security Groups, Network ACLs, VPC Flow Logs, CloudWatch monitoring, and a private Apache web server.

## Project objective

Build and demonstrate a secure AWS VPC where:

- Public resources are placed in public subnets.
- The web server is placed in a private subnet.
- Administrative access to the private server is controlled through a bastion host.
- The private server has no public IPv4 address.
- The private subnet can make outbound Internet connections through a NAT Gateway.
- Security Groups and Network ACLs restrict network traffic.
- VPC Flow Logs and CloudWatch monitor rejected network traffic.

## Architecture

```text
                           INTERNET
                              |
                              v
                    +-------------------+
                    | Internet Gateway  |
                    +---------+---------+
                              |
                  +-----------+-----------+
                  |       Secure-VPC      |
                  |      10.0.0.0/16      |
                  |                       |
                  | Public-A              |
                  | 10.0.1.0/24           |
                  |       |                |
                  |       v                |
                  |  Bastion-Secure        |
                  | 3.27.141.73            |
                  | 10.0.1.137             |
                  |       | SSH             |
                  |       v                |
                  | Private-A              |
                  | 10.0.11.0/24           |
                  |       |                |
                  |       v                |
                  | Private-Web-Server     |
                  | 10.0.11.92             |
                  | Apache :80             |
                  |       |                |
                  |       v                |
                  |  NAT Gateway           |
                  +-----------+------------+
                              |
                              v
                           INTERNET
```

## Network configuration

| Component | CIDR / Address | Purpose |
|---|---|---|
| Secure-VPC | `10.0.0.0/16` | Main VPC |
| Public-A | `10.0.1.0/24` | Bastion subnet |
| Public-B | `10.0.2.0/24` | Second public subnet |
| Private-A | `10.0.11.0/24` | Private web server |
| Private-B | `10.0.12.0/24` | Second private subnet |
| Bastion-Secure | `10.0.1.137` / `3.27.141.73` | Bastion host |
| Private-Web-Server | `10.0.11.92` | Private Apache server |

## AWS components

- Custom VPC
- Two public subnets
- Two private subnets
- Internet Gateway
- NAT Gateway
- Public and private route tables
- Bastion host
- Private EC2 web server
- `Bastion-SG`
- `Web-SG`
- Custom `Secure-NACL`
- VPC Flow Logs
- CloudWatch metric filter
- CloudWatch alarm
- SNS notification
- Apache HTTP server

## Web application

The project deploys a simple static web application using Apache.

The page displays:

> Secure AWS Private Web Server  
> My VPC security project is working!

The source is in [`web-app/index.html`](web-app/index.html).

The application is hosted on the private EC2 instance and is reachable from the bastion using its private IP:

```bash
curl http://10.0.11.92
```

## Security design

### Bastion host

The bastion is located in `Public-A` and has a public IPv4 address so that controlled SSH administration can enter the VPC.

### Private web server

The web server is located in `Private-A` and has no public IPv4 address.

### Security Groups

`Web-SG` permits:

- TCP 22 from `Bastion-SG`
- TCP 80 from `Bastion-SG`

Administrative access is therefore routed through the bastion instead of exposing the private server directly to the Internet.

### NAT Gateway

The private route table sends:

```text
0.0.0.0/0 -> NAT Gateway
```

This allows the private server to initiate outbound Internet connections without giving it a public IPv4 address.

### Network ACL

`Secure-NACL` is associated with `Private-A` and `Private-B`. Because NACLs are stateless, return traffic and ephemeral ports must be handled appropriately.

### Monitoring

VPC Flow Logs are sent to:

```text
/aws/vpc/secure-vpc
```

A CloudWatch metric filter named `RejectedTrafficFilter` produces the `RejectedNetworkTraffic` metric in the `SecureVPC` namespace.

The alarm is:

```text
SecureVPC-RejectedTraffic
```

with a 5-minute period and a threshold of 10 rejected events.

## Demonstration

### 1. SSH from your computer to the bastion

```powershell
ssh -i ".\secure-vpc-key.pem" ec2-user@3.27.141.73
```

### 2. SSH from the bastion to the private server

```bash
ssh -i secure-vpc-key.pem ec2-user@10.0.11.92
```

### 3. Verify Apache

```bash
systemctl is-active httpd
```

Expected:

```text
active
```

### 4. Test the web application

From the private server:

```bash
curl http://localhost
```

From the bastion:

```bash
curl http://10.0.11.92
```

Expected:

```html
<h1>Secure AWS Private Web Server</h1>
<p>My VPC security project is working!</p>
```

## Verification checklist

- [x] Custom VPC created
- [x] Public-A and Public-B created
- [x] Private-A and Private-B created
- [x] Internet Gateway attached
- [x] Public route table configured
- [x] NAT Gateway available
- [x] Private route table configured
- [x] Bastion host deployed in Public-A
- [x] Private web server deployed in Private-A
- [x] Private web server has no public IPv4
- [x] Bastion-to-private SSH tested
- [x] Apache installed and running
- [x] Private web page tested through the bastion
- [x] Security Groups configured
- [x] Custom NACL configured
- [x] VPC Flow Logs enabled
- [x] CloudWatch metric filter created
- [x] CloudWatch alarm created
- [x] Old incorrect bastion terminated

## Repository structure

```text
aws-vpc-security-project/
├── .github/
│   └── workflows/
├── architecture/
│   └── README.md
├── commands/
│   └── deployment-commands.md
├── documentation/
│   └── README.md
├── screenshots/
│   └── README.md
├── web-app/
│   └── index.html
├── .gitignore
├── LICENSE
└── README.md
```

## Important security warning

Never commit:

- `.pem` or `.key` private keys
- AWS access keys
- AWS secret keys
- passwords
- session tokens
- `.env` files containing credentials

The repository intentionally does **not** contain the project's `secure-vpc-key.pem`.

## Future improvements

For a production-style implementation, this project could be extended with:

- Terraform or CloudFormation for Infrastructure as Code
- AWS Systems Manager Session Manager instead of copying an SSH key to the bastion
- Application Load Balancer for public application delivery
- HTTPS/TLS with ACM
- AWS WAF
- CloudTrail
- GuardDuty
- centralized logging
- automated CI/CD
