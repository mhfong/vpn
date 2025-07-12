# Setup EC2 Instance as a Tailscale Exit Node for VPN

This guide walks you through launching an EC2 instance, configuring it as a Tailscale exit node, and connecting your iPhone to use it as a VPN.

## Prerequisites
- An AWS account with Free Tier access (750 hours/month for t3.micro instances).
- Tailscale account and admin access.
- SSH key pair (e.g., `keypair.pem`) or ability to create one.
- iPhone with the Tailscale app installed.

## Step 1: Launch an EC2 Instance
### Choose Region
- For Philippines proximity: Use `ap-southeast-1` (Singapore).
- For Japan VPN: Use `ap-northeast-1` (Tokyo).
- In the AWS Management Console, set the region (top right corner).

### Configure Instance
- Select a t3.micro instance (Free Tier eligible).
- Network settings:
  - Enable "Auto-assign public IP".
  - Choose a public subnet with an internet gateway.
- Security group:
  - Allow inbound SSH (port 22) from your IP.
  - Allow all outbound traffic.
- Key pair: Create `keypair.pem`.
- Launch the instance

## Step 2: SSH into the EC2 Instance
- 2a. Fix permissions:
```console
chmod 400 yourkeypair.pem
```
- 2b. Command:
```console
ssh -i yourkeypair.pem ubuntu@<ec2-public-ip>
```
- Fix permissions if needed:
chmod 400 keypair.pem


## Step 3: Install Tailscale
- Install Tailscale:
```console
curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/jammy.noarmor.gpg | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null
curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/jammy.tailscale-keyring.list | sudo tee /etc/apt/sources.list.d/tailscale.list
sudo apt-get update
sudo apt-get install tailscale
```
- Start and enable Tailscale:
```console
sudo systemctl enable tailscaled
sudo systemctl start tailscaled
```

## Step 4: Configure as a Tailscale Exit Node
- Authenticate:
- Get an auth key from https://login.tailscale.com/admin/settings/keys.
- Run:
```console
sudo tailscale up --auth-key=<your-auth-key> --advertise-exit-node
```
- Replace `<your-auth-key>` with your key.
- Enable IP forwarding:
```console
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
sudo sysctl -p /etc/sysctl.d/99-tailscale.conf
```
- Verify status:
```console
sudo tailscale status
```
- Note the Tailscale IP (e.g., `100.x.x.x`).

## Step 5: Enable the Exit Node
- Log in to https://login.tailscale.com/admin/machines.
- Find the EC2 instance, click the three-dot menu, select **Edit route settings**, and check **Use as exit node**.
- Save the changes.

## Step 6: Connect Your iPhone to the VPN
- Open the Tailscale app on your iPhone.
- Go to the Exit Node section and select the EC2 instance.
- Verify connection:
- Visit https://whatismyipaddress.com. It should show:
  - Tokyo, Japan (for `ap-northeast-1`).


## Additional Notes
- **Free Tier**: Limits to 750 hours/month across all regions. Request vCPU quota increase for multiple instances.
- **Security**: After setup, disable port 22 and use `sudo tailscale up --ssh`.
- **Cost**: Monitor data transfer to avoid charges beyond Free Tier.
- **Service Management**: Update with `sudo apt-get dist-upgrade` or `sudo yum update`. Restart services with `sudo systemctl restart <service>` if needed.
