# wordpress_template
A wordpress instance running on docker

## QuickFixes: WordPress Redirects to HTTPS
 Option 1: Use a different domain instead of localhost (recommended)

This is the cleanest solution.
1. Add a local domain (like wp.local) to your /etc/hosts:

127.0.0.2 wp.local

2. Update your docker-compose VIRTUAL_HOST:

environment:
  VIRTUAL_HOST: wp.local

3. Visit:

https://wp.local

This way, the domain won’t strip the port in WordPress’ internal redirects.
🔧 Option 2: Force WordPress to use the correct URL manually

If you're stuck with localhost, force WordPress to use the correct URL by editing wp-config.php after the container runs.

Add this inside the file:

define('WP_HOME', 'https://localhost:8081');
define('WP_SITEURL', 'https://localhost:8081');

This tells WordPress to use the proper URL instead of auto-generating it.

You can access the file inside the container like:

docker exec -it wordpress bash
nano /var/www/html/wp-config.php

(Use vi or nano depending on your image.)
✅ Quick Fix Option: Use HTTP Temporarily

If you just want to get this working quickly, switch to HTTP while you debug:

Change Nginx ports:

ports:
  - "8080:80"

    Remove or comment the cert volume mounts.

    Visit: http://localhost:8080

## Run Development Environment

With the `-f` flag, you can specify a different docker-compose file. This is useful for development environments or when 
you want to run multiple instances of the same service.

```bash
docker compose --env-file .env.dev -f docker-compose.dev.yml up -d
```

## Wordpress Production Deployment: Use Basic Auth (Temporary Password Protection)

This is the easiest and cleanest method while still letting you access it from the web.
Step 1: Create a .htpasswd file

Install apache2-utils if needed:

```bash
sudo apt install apache2-utils
```
Then generate a user/password (e.g., admin/password123):

```bash
htpasswd -c ./htpasswd admin
```

Step 2: Mount it to nginx-proxy and define restriction

Update your nginx-proxy service:

```yaml
  nginx-proxy:
    ...
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock:ro
      - ./htpasswd:/etc/nginx/htpasswd:ro  # <-- Add this line
      ...
```

Step 3: Add this label to your WordPress service:

```yaml
    environment:
      ...
      AUTH_BASIC: "Restricted Area"
      AUTH_BASIC_USER_FILE: /etc/nginx/htpasswd
```

Now anyone visiting your WordPress install (including bots) will hit a password prompt until you remove those lines later.

### ACME Companion interference

If you only need Basic Auth temporarily, here’s a simpler path:

Comment out LETSENCRYPT_* lines temporarily

Use Basic Auth for setup

After setup:

    Remove Basic Auth

    Re-add LETSENCRYPT_* variables

    Restart WordPress container

    Acme-companion will issue the cert cleanly

✅ Safe

✅ Easy

⚠️ Slight delay in SSL issuance, but that's usually okay for first setup.

## A Quick Guide on WordPress

User Login URL: `https://<hostname>/wp-login.php`