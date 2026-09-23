aws-vpc-network-infrastructure

Multi-tier AWS VPC with public/private subnets, isolating internet-facing workloads from internal ones.

Architecture
1 VPC (10.0.0.0/16)
2 public subnets (across 2 AZs) — route to an Internet Gateway
2 private subnets (across 2 AZs) — route to a NAT Gateway
Tiered security groups (web → app → db), each tier only reachable from the tier in front of it
EC2 instances deployed to validate routing and connectivity end to end
Status

Work in progress. Scripts are being added incrementally as each piece of the network is built and tested.

Scripts

All provisioning is done with the AWS CLI (no Terraform/CloudFormation), so each stage is a standalone, readable script under scripts/. Run them in order:

Script	Creates
00-variables.sh	Shared config (region, CIDRs, AZs) sourced by the rest
01-create-vpc.sh	The VPC
02-create-subnets.sh	2 public + 2 private subnets
03-create-igw.sh	Internet Gateway, attached to the VPC
04-create-nat.sh	Elastic IP + NAT Gateway in a public subnet
05-create-route-tables.sh	Public route table (→ IGW) and private route table (→ NAT)
06-create-security-groups.sh	web-sg / app-sg / db-sg, tiered ingress rules
07-launch-instances.sh	EC2 instances in public and private subnets
99-cleanup.sh	Tears everything down, in dependency-safe order
Usage
bash
aws configure                 # set your credentials/region first
source scripts/00-variables.sh
./scripts/01-create-vpc.sh
./scripts/02-create-subnets.sh
# ...continue in order

Each script prints the resource IDs it creates — copy them into 00-variables.sh (or export them) before running the next script, since later stages depend on earlier IDs.

Requirements
AWS CLI v2, configured with credentials that can manage VPC/EC2 resources
An existing EC2 key pair for SSH (used in 07-launch-instances.sh)