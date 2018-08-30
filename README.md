#  Basic Vault Setup

- create an EC2 instance
- mount an efs filesystem /apps to the EC2
- copy these files to the /usr/local/bin
- cd /usr/local/bin
- ./vault.install
- systemctl stop vault (no more vault process running)
- umount /apps
- built a new image based on this EC2 instance
- update your terraform to use the new image with the following entry in the userdata
- cd /usr/local/bin
- ./vault.install
