# 🚀 Deploying “Hello Ansible” on AWS EC2

---

## 1. Project Overview

- Provision Ubuntu EC2 on AWS  
- Install and configure NGINX with Ansible  
- Serve a custom “Hello from Ansible” webpage  
- Automate infrastructure and deployment  

---

## 2. Prerequisites

- AWS account with EC2 access  
- SSH key pair for instance  
- Ansible installed locally  
- EC2 Ubuntu instance with security group allowing SSH & HTTP  

---

## 3. Infrastructure Setup

### Manual EC2 Creation

- Launch Ubuntu 22.04 instance  
- Setup security group (ports 22, 80 open)  
- Get public IP  

### Optional: Terraform Provisioning

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0c7217cdde317cfec"
  instance_type = "t2.micro"
  key_name      = "your-key-name"
  vpc_security_group_ids = ["sg-xxxxxxxxxxxxxx"]
  tags = { Name = "ansible-web" }
}

# Deploying Infra on AWS and Running Ansible Playbook

## 4. Configure Ansible Inventory

```ini
[web]
<EC2_PUBLIC_IP> ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/your-key.pem
```

## 5. Run Playbook

`ansible-playbook -i inventory.ini playbook.yaml`

Playbook tasks:
Update apt cache

Install NGINX

Deploy custom index.html

Enable and start NGINX

# 6. Verify Deployment
Open browser → http://<EC2_PUBLIC_IP>

You should see:

Hello from Ansible 🚀
This page was deployed automatically.

# 7. Clean Up
Terminate EC2 instance via AWS Console

Or run:
`terraform destroy`

# 8. Summary
Automated EC2 provisioning and app deployment

Simplifies repeatable infra setup

# Demonstrates Ansible & AWS synergy
# Thank you!
# Happy automating with Ansible & AWS! 🎉

### The Ansible Serie will continue 


