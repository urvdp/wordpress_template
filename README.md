# wordpress_template
A wordpress instance running on docker

## QuickFixes: WordPress Redirects to HTTPS
 Option 1: Use a different domain instead of localhost (recommended)

This is the cleanest solution.
1. Add a local domain (like wp.local) to your /etc/hosts:

127.0.0.1 wp.local

2. Update your docker-compose VIRTUAL_HOST:

environment:
  VIRTUAL_HOST: wp.local

3. Visit:

https://wp.local:8081

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
