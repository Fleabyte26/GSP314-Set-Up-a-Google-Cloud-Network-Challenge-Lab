# Google Cloud Challenge Lab: GSP314 Reference Guide

This guide provides the exact commands and variable template for the updated version of **GSP314**.

> **Note for Students:** Qwiklabs randomizes resource names, regions, and CIDR blocks for every student. Do **not** run these commands without first updating the variables in Step 1 to match your assigned instructions.


Step 1: Copy & Update Your Assigned Variables
Check your lab instruction pane, update the quoted values below to match your assignment, and paste the entire block into your Cloud Shell:
# ==============================================================================
# 1. TASK 1: VPC & SUBNET NAMES (Find in Task 1 table)
# ==============================================================================
export VPC_NAME="vpc-network-n12c"          # Replace with your VPC network name

# Subnet A
export SUBNET_A_NAME="subnet-a-886o"        # Subnet A name
export SUBNET_A_REGION="us-east4"           # Subnet A region
export SUBNET_A_RANGE="10.10.10.0/24"       # Subnet A CIDR range

# Subnet B
export SUBNET_B_NAME="subnet-b-31fn"        # Subnet B name
export SUBNET_B_REGION="europe-west1"       # Subnet B region
export SUBNET_B_RANGE="10.10.20.0/24"       # Subnet B CIDR range

# ==============================================================================
# 2. TASK 2: FIREWALL RULES (Find in Task 2 bullets)
# ==============================================================================
export FW_SSH="izol-firewall-ssh"           # SSH rule name
export FW_RDP="vhyr-firewall-rdp"           # RDP rule name
export FW_RDP_RANGE="0.0.0.0/24"            # Check your prompt! (Often 0.0.0.0/24, NOT /0)
export FW_ICMP="zhgf-firewall-icmp"         # ICMP rule name

# ==============================================================================
# 3. TASK 3: COMPUTE INSTANCES (Find in Task 3 bullets)
# ==============================================================================
export VM_A_NAME="us-test-01"               # First VM instance name
export VM_A_ZONE="us-east4-c"               # Zone located inside Subnet A region

export VM_B_NAME="us-test-02"               # Second VM instance name
export VM_B_ZONE="europe-west1-c"           # Zone located inside Subnet B region

export MACHINE_TYPE="e2-standard-2"         # Machine type specified in Task 3


Step 2: Task 1 — Create VPC and Subnets
Run this block in Cloud Shell to set up the custom VPC network and subnets:
# Create custom VPC with regional dynamic routing
gcloud compute networks create $VPC_NAME \
    --subnet-mode=custom \
    --bgp-routing-mode=regional

# Create Subnet A
gcloud compute networks subnets create $SUBNET_A_NAME \
    --network=$VPC_NAME \
    --region=$SUBNET_A_REGION \
    --range=$SUBNET_A_RANGE \
    --stack-type=IPV4_ONLY

# Create Subnet B
gcloud compute networks subnets create $SUBNET_B_NAME \
    --network=$VPC_NAME \
    --region=$SUBNET_B_REGION \
    --range=$SUBNET_B_RANGE \
    --stack-type=IPV4_ONLY


Step 3: Task 2 — Create Firewall Rules
Common Pitfall: If you only receive 20/50 on the first checkpoint, your VPC created correctly but the firewall rules failed validation. Ensure:
RDP rule source range strictly matches your prompt (0.0.0.0/24).
The ICMP rule allows traffic from both Subnet A and Subnet B CIDR ranges without whitespace between them.
Rules apply to all instances on the network (do not pass target tags).

# 1. SSH Rule (TCP: 22, Priority: 1000, Source: 0.0.0.0/0)
gcloud compute firewall-rules create $FW_SSH \
    --network=$VPC_NAME \
    --direction=INGRESS \
    --priority=1000 \
    --action=ALLOW \
    --rules=tcp:22 \
    --source-ranges=0.0.0.0/0

# 2. RDP Rule (TCP: 3389, Priority: 65535, Source: specific CIDR, e.g., 0.0.0.0/24)
gcloud compute firewall-rules create $FW_RDP \
    --network=$VPC_NAME \
    --direction=INGRESS \
    --priority=65535 \
    --action=ALLOW \
    --rules=tcp:3389 \
    --source-ranges=$FW_RDP_RANGE

# 3. ICMP Rule (ICMP, Priority: 1000, Sources: Subnet A & Subnet B ranges)
gcloud compute firewall-rules create $FW_ICMP \
    --network=$VPC_NAME \
    --direction=INGRESS \
    --priority=1000 \
    --action=ALLOW \
    --rules=icmp \
    --source-ranges="\({SUBNET_A_RANGE},\){SUBNET_B_RANGE}"

Check Progress: Click Check my progress on "Create network, subnetworks and firewalls". You should now have 50 / 50.
Step 4: Task 3 — Create VMs & Test Connectivity
Deploy the two virtual machines across their respective zones and subnets:
# Create Instance 1 in Subnet A
gcloud compute instances create $VM_A_NAME \
    --zone=$VM_A_ZONE \
    --machine-type=$MACHINE_TYPE \
    --subnet=$SUBNET_A_NAME

# Create Instance 2 in Subnet B
gcloud compute instances create $VM_B_NAME \
    --zone=$VM_B_ZONE \
    --machine-type=$MACHINE_TYPE \
    --subnet=$SUBNET_B_NAME


Ping / Latency Test
Wait ~30 seconds for the VMs to initialize, then SSH from VM A to ping VM B using internal DNS:
Bash

gcloud compute ssh \(VM_A_NAME --zone=\)VM_A_ZONE --quiet \
    --command="ping -c 3 \({VM_B_NAME}.\){VM_B_ZONE}"


Check Progress: Click Check my progress on "Create two instances in specified zones for Traceroute and performance testing". Your total score should now reach 100 / 100. 
