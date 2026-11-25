# Approach

    Physical Machine
    └── Oracle VirtualBox
        └── Ubuntu VM
            ├── Kubernetes
            ├── Envoy Gateway
            ├── NGINX
            └── cloudflared   ← Everywhere

## Step 1: Login to Ubuntu VM

    # Verify
    curl http://192.168.56.11:30080

## Step 2: Install cloudflared

    wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb

    sudo dpkg -i cloudflared-linux-amd64.deb

    cloudflared --version

## Step 3: Start Quick Tunnel

    cloudflared tunnel --url http://192.168.56.11:30080
