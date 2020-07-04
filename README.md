# Description
This project is a sample configuration that represents the usage of ELK stack

## Docker
- Run the elk container using command `docker-compose up`
- For running in detached mode `docker-compose up -d`
- If you want to access the container run `docker exec -it elk-docker_elk_1  /bin/bash`

## Low memmory error
In some case you might get virutal memmory low error like `[1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]`. In that case execute the command in your host machine
`sudo sysctl -w vm.max_map_count=262144`

## Securing Kibana using Nginx

If you are using Nginx as a reverse proxy. Follow the steps below to secure it

1. The following command will create the administrative Kibana user and password, and store them in the htpasswd.users file. You will configure Nginx to require this username and password and read this file momentarily:

```sh
echo "kibanaadmin:`openssl passwd -apr1`" | sudo tee -a /etc/nginx/htpasswd.users
```

Enter and confirm a password at the prompt. Remember or take note of this login, as you will need it to access the Kibana web interface.

2. Next, we will create an Nginx server block file. As an example, we will refer to this file as `example.com`, although you may find it helpful to give yours a more descriptive name. For instance, if you have a FQDN and DNS records set up for this server, you could name this file after your FQDN:

```sh
sudo nano /etc/nginx/sites-available/example.com
```

Add the following code block into the file, being sure to update `example.com` to match your server’s FQDN or public IP address. This code configures Nginx to direct your server’s HTTP traffic to the Kibana application, which is listening on `localhost:5601`. Additionally, it configures Nginx to read the `htpasswd.users` file and require basic authentication. Create the configuration like below inside the file `/etc/nginx/sites-available/example.com`

```sh
server {
    listen 80;

    server_name example.com;

    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/htpasswd.users;

    location / {
        proxy_pass http://localhost:5601;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

When you’re finished, save and close the file.

3. Next, enable the new configuration by creating a symbolic link to the sites-enabled directory. If you already created a server block file with the same name in the Nginx prerequisite, you do not need to run this command:

```sh
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/example.com
```

Then check the configuration for syntax errors:

```sh
sudo nginx -t
```

If any errors are reported in your output, go back and double check that the content you placed in your configuration file was added correctly. Once you see syntax is ok in the output, go ahead and restart the Nginx service:

```sh
sudo systemctl restart nginx
```