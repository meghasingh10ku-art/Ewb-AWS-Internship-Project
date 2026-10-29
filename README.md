# AWS VPC Peering Lab Notes

Hey! This repository contains my documentation, configuration notes, and verification logs for setting up a secure VPC Peering connection between two isolated VPCs on AWS. 

The main goal of this lab was to figure out how two private EC2 instances in completely different networks can talk to each other securely without exposing them to the public internet, using a bastion host and custom route tables.

---

## Architecture Setup

- VPC A (`10.10.0.0/16`):
  - Public Subnet (`10.10.1.0/24`) — houses the bastion host with a public IP.
  - Private Subnet A (`10.10.2.0/24`) — houses `ewb-app-a` (no public IP).
- VPC B (`10.20.0.0/16`):
  - Private Subnet B (`10.20.1.0/24`) — houses `ewb-app-b` (no public IP).
- Peering Link: Connected via `ewb-a-b-peer` (`pcx-...`).

---

## How I Did It (Quick Walkthrough)

1. VPCs & Subnets: Created two VPCs with non-overlapping CIDRs so routing doesn't break. Attached an Internet Gateway to VPC A so the bastion can reach the outside world.
2. Security Groups: Locked down SSH (Port 22) on the bastion to my own IP. For the private app instances, I made sure they only accept SSH coming from the bastion and ICMP pings coming from each other.
3. SSH ProxyJump (`~/.ssh/config`): Set up my local SSH config so I can jump straight into the private instances via the bastion without needing to copy my `.pem` private key onto the server. 
4. The Peering Connection: Created the peering request from VPC A to VPC B, accepted it, and then went into the route tables to add the explicit forward (`10.20.0.0/16`) and reverse (`10.10.0.0/16`) routes targeting the peering connection ID (`pcx-...`).

---

## Testing & Troubleshooting Notes

- Baseline Test: Before adding the peering routes, I SSH'd into `ewb-app-a` and tried to ping `ewb-app-b`. Got 100% packet loss (as expected—traffic had nowhere to go). Took a screenshot for the report.
- Post-Peering Test: Once the routes were active, re-ran the ping test and got 0% packet loss immediately.
- The Bastion Timeout Issue: My bastion IP changed midway because of an instance restart. Had to quickly update the HostName in my local `~/.ssh/config` file to get SSH working again.
- Security Group Check: Temporarily deleted the ICMP rule on App-B's security group to prove it blocks traffic, then added it back. Classic stateful firewall behavior.

---

Find the full visual evidence and screenshots attached in the main PDF report!
