# OpenSSH Server

The remote host OS is assumed to be Ubuntu in this post.

## Install OpenSSH Server

In an isolated intranet environment (e.g., a network behind a router with no port forwarding), [install the OpenSSH Server](https://ubuntu.com/server/docs/openssh-server) on the remote host as follows:

```sh
sudo apt-get update && sudo apt-get install -y openssh-server
```

Verify the setup by attempting to connect from your local host:

```sh
ssh <user>@<private_ip>
```

Replace `<user>` with your remote username and `<private_ip>` with the private IP address of the remote server.

If password-based login is sufficient, you can simply stop here and continue using SSH with your password credentials.

## Generate SSH Key

Reuse an existing SSH key or [generate a new SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=linux) on your local host. To generate a new key, run the following command on your local host in a terminal (for Ubuntu) or PowerShell (for Windows), then press enter to accept the default settings:

```sh
ls ~/.ssh/id_ed25519.pub
ssh-keygen -t ed25519
```

## Copy SSH Public Key

Run the following command on your local host to copy your SSH key from your local machine to the remote host:

```sh
ssh-copy-id <user>@<private_ip>
```

This will automatically add your public key to the remote host's `~/.ssh/authorized_keys` file, enabling passwordless authentication.

For Windows users, this command is available through `Git Bash` after installing [Git for Windows](https://git-scm.com/downloads/win). Otherwise, manually append the contents of your public key `~/.ssh/id_ed25519.pub` on the local host to `~/.ssh/authorized_keys` on the remote host.

After copying the public key, confirm that the setup is working by attempting to connect to the remote host:

```sh
ssh <user>@<private_ip>
```

You should not be prompted for a password.

## Set up SSH Config

Create or edit the `~/.ssh/config` file on your local host and add:

```
Host <hostname>
  HostName <private_ip>
  User <user>
```

Replace `<hostname>` with an arbitrary name for your remote host, `<private_ip>` with the private IP address of the remote server, and `<user>` with your remote username.

Verify the setup by:

```sh
ssh <hostname>
```

## (Optional) Expose to the Internet

Before exposing your SSH server to the internet, it's crucial to apply additional security hardening.

### Disable Password Login

Disable password authentication, allowing only SSH key-based authentication. Run the following command on the remote host to disable password login:

```sh
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
```

Restart the SSH service to apply the changes:

```sh
sudo systemctl restart ssh
```

Confirm the change by [not using SSH key authentication](https://serverfault.com/a/130351):

```sh
ssh -o PubkeyAuthentication=no <user>@<private_ip>
```

If configured correctly, the connection should fail, confirming that only key-based authentication is allowed.

### Change Default Port

> If the remote host is behind a router and you plan to change the default port through port forwarding in the next section, you can skip this step and move directly to the following section.

To improve security, change the default SSH port from `22` to a non-standard port (e.g., `2222`). To do this, run:

```sh
sudo sed -i 's/#Port 22/Port 2222/' /etc/ssh/sshd_config
```

Then, restart the SSH service:

```sh
sudo systemctl restart ssh
```

Confirm the change by:

```sh
ssh -p 2222 <user>@<private_ip>
```

Optionally update the `~/.ssh/config` file on your local host to use the new port:

```
Host <hostname>
  HostName <private_ip>
  User <user>
  Post 2222
```

### Configure Static IP and Port Forwarding

If the remote host is behind a router, ensure it has a static IP address (you may also need to perform [DHCP Lease Renewal](./dhcp-lease-renewal.md)). Then, set up port forwarding on your router to forward a non-default SSH port (e.g., `2222`) from the public network to port `22` (or to port `2222`, if you followed the previous section) on the local static IP of your remote machine.

Verify the setting by

```sh
ssh -p 2222 <user>@<public_ip>
```

Replace `<user>` with your remote username and `<public_ip>` with the public IP address, which you can be found by searching "What is my IP" online.

Note: If the connection fails, your network setup may include multiple routers or other configurations that require further investigation for port forwarding to work correctly.

Optionally update the `~/.ssh/config` file on your local host to use the public IP:

```
Host <hostname>
  HostName <public_ip>
  User <user>
  Post 2222
```
