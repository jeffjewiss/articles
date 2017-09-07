# Running Multiple Ghost Blogs Alongside Dokku

* take advantage of running databases in Docker containers using Dokku by simply exposing the database container
* continue to run other apps on one server
* take advantage of already running nginx
* works great with Digital Ocean
* transition from a Dokku hosted Ghost blog using a git repo to using `ghost-cli`
* follow the typical ghost-cli instructions
* comment out any conflicting nginx configuration options (check by running `systemctl status nginx.service`
* create a dokku mysql instance: `dokku mysql:create <db name>`
* expose instance with `dokku mysql:expose <db name>`
* find the database IP to connect to (as the host) using `dokku mysql:info <db name>`
* restart nginx since `ghost-cli` will fail to restart it with the conflicting params (like ssl cache) using: `service nginx restart`
