## GitHub Action SSH Tunnel via ngrok

A GitHub Action for connecting to the runner via SSH.

### Why?

Debugging GitHub Actions remotely can be difficult. Maybe you want to connect to the runner environment live to troubleshoot.

### Requirements

1. An [ngrok](https://ngrok.com/) account (free)
2. An SSH public key (e.g. `/.ssh/id_rsa.pub`)

### Compatibility

This Action was only tested on the **Ubuntu 20.04** runner, but it may work on other Linux based runners.

### Setup

Create a YAML workflow (e.g. `ssh.yml`) in `.github/workflows` following this example:

```yaml
name: SSH Tunnel
on: push

jobs:
  deploy:
    name: Set up tunnel
    runs-on: ubuntu-20.04
    steps:
    - name: Checkout
      uses: actions/checkout@v2
      
    - name: Setup tunnel
      uses: joshlarsen/ssh-tunnel-action@main
      with:
        timeout: 1h
        ssh_public_key: ${{ secrets.SSH_PUBLIC_KEY }}
        ngrok_token: ${{ secrets.NGROK_TOKEN }}
```

### Required Secrets

Create two repository secrets (Settings -> Secrets -> New repository secret)

`SSH_PUBLIC_KEY`: your local SSH public key (e.g. `~/.ssh/id_rsa.pub`)

`NGROK_TOKEN`: your ngrok auth token

### Deploy

On the next push, GitHub Actions will download the ngrok binary and set up a TLS tunnel on a random port. Check the [ngrok dashboard](https://dashboard.ngrok.com/status/tunnels) to get the hostname and port the tunnel is listening on.

![ngrok tunnels](https://user-images.githubusercontent.com/2565382/108560004-179c1f80-72ca-11eb-8fb1-92436b1e9024.png)

### Connect via SSH

The runner username is `runner`. Connect to the ngrok tunnel port using SSH:

```
$ ssh -p 11785 runner@0.tcp.ngrok.io 

The authenticity of host '[0.tcp.ngrok.io]:11785 ([3.134.39.220]:11785)' can't be established.
ECDSA key fingerprint is SHA256:f27aouAtzHOx7rzEnrGUfKy9xhpFK5auzq6+ZY.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[0.tcp.ngrok.io]:11785,[3.134.39.220]:11785' (ECDSA) to the list of known hosts.

Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-1039-azure x86_64)

  System load:  0.0                Processes:                153
  Usage of /:   75.5% of 83.18GB   Users logged in:          0
  Memory usage: 10%                IPv4 address for docker0: 172.17.0.1
  Swap usage:   0%                 IPv4 address for eth0:    10.1.0.4

runner@fv-az214-809:~$ 
runner@fv-az214-809:~$ curl ipinfo.io
{
  "ip": "52.173.149.212",
  "city": "Des Moines",
  "region": "Iowa",
  "country": "US",
  "loc": "41.5878,-93.6274",
  "org": "AS8075 Microsoft Corporation",
  "postal": "50392",
  "timezone": "America/Chicago",
  "readme": "https://ipinfo.io/missingauth"
}
runner@fv-az214-809:~$ 
```
