# Week 3 Runbook: Deploy PropertyLite to EC2

## Goal
Run the same PropertyLite app from Week 0 on a real AWS server and reach it from my laptop over the internet on port 8080.

## Key ideas, in plain words
- **EC2 instance:** a virtual server I rent by the hour.
- **AMI (Amazon Machine Image):** the starting disk image the server boots from. This lab uses Amazon Linux 2023.
- **Instance type:** the server's size. t3.micro is small and Free Tier eligible.
- **Key pair:** the lock and key for SSH. AWS puts the public half on the server, and I keep the private half (`training-key.pem`) on my laptop. AWS cannot re-download it, so losing it means losing SSH access.
- **Security group:** a firewall attached to one instance. It is stateful: if a request is allowed in, the reply is automatically allowed back out. A network ACL (NACL) does a similar job for a whole subnet, but it is stateless, so traffic in each direction needs its own rule.
- **User data:** a script EC2 runs once, as root, the first time the server boots. It lets the server set itself up with no manual steps.
- **EBS volume:** the server's hard drive. It is a separate resource from the server, and a snapshot is a point-in-time backup of it.

## Steps
1. In the EC2 console (region us-east-1, N. Virginia), click **Launch instance** and name it `property-api-01`.
2. Choose the **Amazon Linux 2023** AMI and the **t3.micro** instance type.
3. Under Key pair, create a new key pair named `training-key` (RSA, .pem). Move the download into place and lock its permissions, because SSH refuses a key file that other users could read:
   ```bash
   mkdir -p ~/.ssh
   mv ~/Downloads/training-key.pem ~/.ssh/training-key.pem
   chmod 400 ~/.ssh/training-key.pem
   ```
4. Under Network settings, create a security group with two inbound rules: **SSH, port 22, source My IP** (only my laptop can log in) and **Custom TCP, port 8080, source Anywhere** (anyone can reach the app). Keep Auto-assign public IP enabled.
5. Under Advanced details, paste `bootstrap.sh` (in this folder) into **User data**. On first boot it updates the OS, installs Python, creates a virtual environment with Flask 3.0.3, writes `app.py` and the sample CSV to `/opt/property-api`, and starts the app in the background with its output in `/var/log/property-api.log`.
6. Launch the instance. Wait until it shows **Running** with its status checks passed, then give the bootstrap about three more minutes.
7. Test from the laptop, replacing the placeholder with the instance's Public IPv4 address:
   ```bash
   IP=<public-ip>
   curl http://$IP:8080/health
   curl http://$IP:8080/properties
   curl http://$IP:8080/properties/R100234
   ```
   Success: `/health` returns `{"status":"ok"}` and `/properties/R100234` returns the Sacramento listing. This is the same app tested locally in Week 0, now running on real infrastructure.
8. Connect over SSH to confirm direct access to the server, then type `exit`:
   ```bash
   ssh -i ~/.ssh/training-key.pem ec2-user@$IP
   ```

## Difference from the handbook script
The handbook installs Flask with a system-wide `pip3 install`. `bootstrap.sh` installs it inside a Python virtual environment instead, so pip never tries to replace Python packages that Amazon Linux installed through its own package manager. The app code is unchanged.

## EBS snapshot (Lab 3.2)
In EC2 → Volumes, select the root volume of `property-api-01`, choose Actions → Create snapshot, and wait for the status to reach Completed. Creating a new volume from this snapshot would produce an exact copy of the server's disk at that moment, including the OS, the app, and the CSV, which could be attached to another instance to rebuild the server.

## Troubleshooting
- **SSH says the key's permissions are too open:** run `chmod 400 ~/.ssh/training-key.pem`.
- **SSH times out even though it worked before (Debug Lab 3.3):** my laptop's public IP probably changed. "My IP" in the security group is saved at the moment the rule is created, so it does not follow me to a new network. Edit the SSH inbound rule and set the source to my current IP.
- **curl can't connect on port 8080:** confirm the security group allows port 8080 (not 80), and give the bootstrap a few minutes to finish. Over SSH, `sudo tail -50 /var/log/cloud-init-output.log` shows what the bootstrap did, and `sudo cat /var/log/property-api.log` shows the app's own output.

## Cleanup
Terminate `property-api-01` (Instance state → Terminate), confirm its volume disappears from the Volumes page, and delete the snapshot. Anything left running keeps using credits.
