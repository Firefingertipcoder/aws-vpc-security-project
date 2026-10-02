# Deployment and Verification Commands

> Replace addresses and key names if you recreate the project.

## Connect to Bastion

From Windows PowerShell:

```powershell
ssh -i ".\secure-vpc-key.pem" ec2-user@3.27.141.73
```

## Copy the project key to the bastion for this demonstration

From Windows PowerShell:

```powershell
scp -i ".\secure-vpc-key.pem" ".\secure-vpc-key.pem" ec2-user@3.27.141.73:/home/ec2-user/
```

On the bastion:

```bash
chmod 400 secure-vpc-key.pem
```

> For production environments, avoid copying private keys to a bastion. AWS Systems Manager Session Manager is generally preferable.

## Connect to the private server

From the bastion:

```bash
ssh -i secure-vpc-key.pem ec2-user@10.0.11.92
```

## Install Apache

```bash
sudo dnf update -y
sudo dnf install httpd -y
sudo systemctl enable --now httpd
```

## Deploy the page

```bash
echo '<h1>Secure AWS Private Web Server</h1><p>My VPC security project is working!</p>' | sudo tee /var/www/html/index.html
```

## Verify Apache

```bash
systemctl is-active httpd
```

Expected:

```text
active
```

## Test locally

```bash
curl http://localhost
```

## Test from the bastion

Exit the private server:

```bash
exit
```

Then run:

```bash
curl http://10.0.11.92
```

Expected:

```html
<h1>Secure AWS Private Web Server</h1><p>My VPC security project is working!</p>
```

## Verify private outbound Internet access

From the private server:

```bash
curl -4 -I --connect-timeout 10 https://amazonlinux.com
```

A successful HTTP response demonstrates outbound Internet connectivity through the NAT path.

## Useful checks

```bash
ip addr
ip route
systemctl status httpd
```

The private server should have:

- private IP `10.0.11.92`
- no public IPv4 address
- default route via the private subnet gateway
