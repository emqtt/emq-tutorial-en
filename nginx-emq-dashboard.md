# Nginx EMQ Dashboard

[TOC]

## Nginx EMQ Dashboard

### 1. Overview

#### 1.1 Architecture of EMQ Dashboard

The EMQ Dashboard frontend page is designed using a single-page application (SPA), the back end uses the MochiWeb server with [RESTful API] (http://emqtt.io/docs/v2/rest.html) and the backend server proxies all The front-end routing requests are forwarded to the front end. This architecture enables a good front-to-back end-of-line separation in deployment with less reliance on Dashboard, greatly simplifying the Dashboard startup process.

#### 1.2 EMQ API description

> EMQ provides two types of API interfaces: the 8080 port API is provided for external applications, and the 18083 port API is provided to EMQ Dashboard itself.　

- API for external application access

  ```bash
  http(s)://host:8080/api/v2/
  ```

- Dahboard's own API

  ```bash
  http(s)://host:18083/api/v2/
  ```

####1.3 Dashboard static resource file directory structure

> Note: The Dashboard homepage always loads apis and other resources in absolute paths.

```bash
www
    ├── favicon.ico
    ├── index.html
    └── static
        ├── css
        │   ├── *
        ├── fonts
        │   ├── *
        └── js
            ├── *
```


### 2. Deployment instructions

#### 2.1 Nginx server configuration specification
> Nginx configuration file is commonly ` / etc/nginx/nginx.conf ` or ` install_path/conf/nginx.conf `, It's depending on your installation steps. After modifying the configuration file, you need to restart the server to apply the changes.

- Basic nginx virtual host configuration

```bash
http {
        server {
                listen  80;
                # Multiple domain names are separated by spaces
                server_name  example.com dashboard.example.com;

                # Opening gzip can greatly increase the speed of Dashboard loading
                gzip  on;
                gzip_types  text/plain application/x-javascript text/css application/javascript text/javascript;

                # Support for static files
                location ~* ^.+.(jpg|jpeg|gif|css|png|js|ico|html)$ {
                    access_log  off;
                    expires  30d;
                }
        }
}
```

#### 2.2 Simple reverse proxy configuration

> This configuration simply forwards the browser-initiated request back to the Dashboard backend server with no disparity, and the Dashboard backend server still handles heavy, static resource distribution tasks.

```bash
server {
        listen  80;
        server_name  example.com;
        
        location / {
            proxy_pass  http://127.0.0.1:18083;
        }
}
```

Through the above configuration, access ` example.com ` can use Dashboard.

#### 2.3 Use nginx to handle static resources and proxy forward API data

- Processing static resource files through nginx

  > front dashboard priv/WWW directory, see [https://github.com/emqtt/emq-dashboard/tree/master/priv/www] (https://github.com/emqtt/emq-dashboard/tree/master/priv/www)  

- Proxy requests for `/api/v2` to http://127.0.0.1:18083/api/v2

- Proxy requests for `/external_api` to http://127.0.0.1:8080/api/v2

```bash
server {
        listen  80;
        server_name  example.com;
        
        # static resources
        location / {
            # Copy a static resource to root path
            root  /www;
            # You can use static resources that are already installed in the native EMQ
            # /emqttd/lib/emq_dashboard-2.3.0/priv/www/
            index  index.html;
        }
        
        # The API used by Dashboard must be set to /api/v2
        location /api/v2/ {
            proxy_pass   http://127.0.0.1:18083/api/v2;
        }

        # The API for external application calls, /external_api can be customized to other paths
        location /external_api {
            proxy_pass   http://127.0.0.1:8080/api/v2;
        }
}
```

#### 2.4 Bind Dashboard to a path
-  To access Dashboard using a URL like `http://example.com/dashboard`, refer to the following configuration

```bash
server {
        listen  80;
        server_name  example.com;

        location /dashboard {
            proxy_pass  http://127.0.0.1:18083/;
        }
        
        # Ensure that static resource dependencies and Dashboard API are accessible on absolute path
        location /static {
            proxy_pass  http://127.0.0.1:18083/static/;
        }
        
        location /api/v2 {
            proxy_pass  http://127.0.0.1:18083/api/v2/;
        }
}
```


#### 2.5 Other configurations
- It is recommended that you set the proxy request header during `proxy_pass` configuration to carry information such as client IP address
```bash
server {
        listen  80;
        server_name  example.com;

        location /dashboard {
            proxy_pass  http://127.0.0.1:18083/;
            # Set proxy request header
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Real-PORT $remote_port;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
}
```

### 3. Reference material

- [nginx Chinese document](http://www.nginx.cn/doc/)

- [nginx English document](http://nginx.org/en/docs/)


