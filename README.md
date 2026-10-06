# Deployment & Setup Guide

1. Buy your domain and point it to your VPS’s IP address.

2. Connect to your VPS:

    ```bash
    ssh root@<your-vps-ip>
    ```

3. Install system dependencies (`node.js`, `nginx`):

    ```bash
    sudo apt update
    sudo apt install -y nodejs npm
    sudo apt install -y nginx
    ```

4. Prepare application directory:

    ```bash
    sudo mkdir -p /var/www/<app-name>

    cd /var/www/<app-name>
    ```

5. Generate an SSH key & clone your repo:

    ```bash
    cd /var/www/
    ssh-keygen -t ed25519 -C "<your_email@example.com>"
    cat ~/.ssh/id_ed25519.pub   # copy this key into your Git repo’s deploy keys
    git clone git@github.com:your_username/your_repo.git
    ```

6. Install project dependencies:

    ```bash
    cd your_repo
    npm install
    ```

7. Configure environment variables:

    ```bash
    # Create .env
    cat > .env

    # Paste your variables, then press this to exit:
    Ctrl+D
    ```

8. Create Nginx site file.

    ```bash
    cat > <site-name>
    ```

9. Configure Nginx:

    ```nginx
    # paste this

    server {
      listen 80;
      server_name <your-url>.com www.<your-url>.com;
  
      root /var/www/<folder-name>/dist;
      index index.html;
  
      # React/Vite SPA routing
      location / {
          try_files $uri $uri/ /index.html;
      }
  
      access_log /var/log/nginx/<folder-name>.access.log;
      error_log  /var/log/nginx/<folder-name>.error.log warn;
    }
    ```

10. Activate your config:

    ```bash
    # Disable the default Nginx site
    rm /etc/nginx/sites-enabled/default

    # Enable your site
    ln -s /etc/nginx/sites-available/<site-config-name> /etc/nginx/sites-enabled/<site-config-name>

    # Test Nginx configuration
    nginx -t

    # Reload Nginx
    systemctl reload nginx
    ```

11. Create `deployment-actions.sh`:

    ```bash
     cat > deployment_actions.sh
    ```

12. Add the code:

    ```bash
    #!/bin/bash
    
    set -e
    
    APP_DIR="/var/www/<file-name>"
    BRANCH="main"
    
    echo "================================"
    echo "Starting deployment"
    echo "================================"
    
    cd "$APP_DIR"
    
    echo ""
    echo "==> Updating repository..."
    git fetch origin "$BRANCH"
    git reset --hard "origin/$BRANCH"
    
    echo ""
    echo "==> Installing dependencies..."
    npm i
    
    echo ""
    echo "==> Building application..."
    npm run build
    
    echo ""
    echo "================================"
    echo "Deployment successful!"
    echo "================================"
    ```

13. Then make the file executable:

    ```bash
    chmod +x deployment-actions.sh
    ```

14. Run the `deployment-actions.sh`:

    ```bash
    ./deployment-actions.sh
    ```

15. Enable SSL with Certbot:

    ```bash
    sudo apt update
    sudo apt install -y certbot python3-certbot-nginx

    sudo certbot --nginx \
        -d <site-url> \
        -d www.<site-url> \
        --agree-tos \
        --redirect \
        --no-eff-email \
        -m <your email>

    # Restart nginx server
    sudo nginx -t
    sudo systemctl reload nginx
    ```
