### Nginx is web server also :

	1. Reverse proxy
	2. Load balancer
	3. HTTP cache
	4. Security ssl/tsl

## Terminal commends:
      - cd /etc/nginx => here is where nginx configure file is located 
      - sudo vim/nvim/nano nginx.conf => to write or read the file 
      - sudo systemctl start/stop/status/reload/restart nginx => to run the server or stop the server or status which is running/ not or reload the active server
      - sudo nginx -t => check the syntex is okvor not and test is successful it is very use full when you edit the file 
  
## Example of nginx:

Create a folder with any name, for example, nginx_test. Inside it, create an index.html file, type anything in it, and save it.

    ```
    mkdir nginx_test
    cd nginx_test/
    nvim index.html
    ```
events and http are both top-level contexts in nginx.conf, so they should be placed side by side under the main context, not inside each other. The events block is for connection handling, and the http block is for HTTP configuration. 

    ```
    events{}
    http{}
    ```
Inside the http context, there is a server context. In that server context, the listen directive sets the port. The default port is 80, but you can change it to another port, such as 6969. Inside the server context, there is also a location context, where the root directive points to the folder containing the previously created file.  
  
    ```
    events {}
    http {
            server{
                    listen 80;        
            location / { 
                    root /home/umar/Desktop/nginx_test;
            }
            }
    }
    ```
 
In the directive ```location /```, the '/' represents the root directory. When you point to this folder, the server automatically finds the index.html file and displays the output. If there are two HTML files, such as index.html and server2.html, navigating to http://localhost will display index.html by default. If you want to access server2.html, navigating to http://localhost/server2.html will show the data for that specific HTML file.

    ```
    events {}
    http {
            server{
                    listen 80;        
            location / { 
                    root /home/umar/Desktop/nginx_test;
            }
            location /server2/ { 
                    root /home/umar/Desktop/nginx_test;
            }
    }
    ```
you can add as much as you want the location in the nginx.conf file. To see the result of the server2 than navigate to http://localhost/server2.html 

## Real example:

### Reverse proxy:

I have both a frontend and a backend, and they are currently running separately on different ports. The frontend runs on port 3000, while the backend runs on port 5000. If you want both the frontend and backend, go to the Git repository and download them from this link: https://github.com/umar9622/nginx  I have a frontend running on port 3000 and a backend running on port 5000. Now I want Nginx to expose only port 80 and forward requests to the correct service. it also called **reverce proxy** 
```
events {}

http {

	server{
		listen 80;

	location / {
		
		proxy_pass http://127.0.0.1:3000;
	}
	location /status {

                proxy_pass http://127.0.0.1:5000;
        }
	location /calculator {

                proxy_pass http://127.0.0.1:5000;
        }

	}
}	
```

In this configuration,**location /** is for the frontend, and the other two location blocks are for the backend. I created a FastAPI app for those specific routes, and Nginx connects those routes to that backend. Now, when we open http://127.0.0.1:80, we access Nginx, and it serves both the frontend and backend from one place. Previously, we used **root** because it is used to serve files. Here, we are using **proxy_pass** because we want to send requests to the URLs of both the frontend and the backend.

### security ssl/tsl:

first download openssl using this commend **sudo apt install openssl** if already installed check by using **openssl --version** and then commend 
```
	openssl genrsa -out nginx.key 2048
```

it will generate private key and than commend

```
	openssl req -x509 -new -nodes \
	  -key nginx.key \
	  -sha256 \
	  -days 365 \
	  -out nginx.crt
	  
  ```
it will generate public key we going to use this both key in nginx.conf file to secure our both frontend and backend by encripted
  
  ```
  events {}

http {

        server{
                listen 443 ssl;
                server_name localhost;
                ssl_certificate /home/umar/nginxTutorial/ssl/nginx.crt;
                ssl_certificate_key /home/umar/nginxTutorial/ssl/nginx.key;

        location / {

                proxy_pass http://127.0.0.1:3000;
        }
        location /api/status {

                proxy_pass http://127.0.0.1:5000;
        }
        location /calculator {

                proxy_pass http://127.0.0.1:5000;
        }

        }

        server {
                listen 80;
                server_name localhost;
                return 301 https://$host$request_uri;
        }
}

  ```
In the nginx.conf file, create two keys: **ssl_certificate and ssl_certificate_key**, and paste the file paths of the generated certificate and key. Then change listen to **443 ssl**, which is the default HTTPS port. After that, create another server block inside http to redirect HTTP requests to HTTPS, using **return 301** to perform the redirect. "$host$request_uri" means the current host and the requested path, such as /api/status or /calculator. Then access the site using https://127.0.0.1 instead of http.
  
### Load balancing:

  Load balancing in Nginx means that when many clients send requests to a server, a single server may not be able to handle them all, so the traffic needs to be distributed across multiple servers. There are several methods, such as **round-robin**. In a simple setup, in the http block we create an **upstream block** with any name we want, connect multiple backend servers inside it, and then replace http://127.0.0.1:5000 in the backend with http://upstream_name.
  ```
http {
	
	upstream backend_server {
			server localhost:5000;
			server localhost:5001;
			server localhost:5002;
		}

}	
  ```
  In the **upstream block**, you can add as many backend servers as you want.
  ```
  location /api/status {

                proxy_pass http://backend_server;
        }
	location /calculator {

                proxy_pass http://backend_server;
        }

	}

  ```
  After creating the upstream, change the backend server URL in the location block to the upstream name. This will distribute all requests among the servers.
  
### HTTP cache:

Cache in Nginx is very important and saves time. Whenever a client sends a request to the server, Nginx does not always need to hit the server again. Instead, it can store the response in cache and send that cached response to the client. The cached data may change sometimes, so the cache is reset or expires after a certain period of time. Cache is mainly used for **GET** requests, not for **POST, PUT, or DELETE** requests.

in the http context 
```
proxy_cache_path /data/nginx/cache levels=1:2 keys_zone=mycache:10m max_size=20m inactive=60m use_temp_path=off;
```

this part help to store the cache moemory in nginx.

* **proxy_cache_path** => tells Nginx to create a cache storage area.
* **/data/nginx/cache** => is the folder where cached files will be stored.
* **levels=1:2** => creates subfolders inside the cache folder to organize files better.
* **keys_zone=mycache:10m** => creates a shared memory zone named mycache with 10 MB for storing cache keys and metadata.
* **max_size=20m** => means the cache can grow up to 20 MB.
* **inactive=60m** => means cached files that are not used for 60 minutes will be removed.
* **use_temp_path=off** => tells Nginx to write cache files directly into the cache folder instead of using a temporary location.

and in location where we get data block:

```
location /api/status {
            proxy_pass http://backend_server;
            proxy_cache mycache;
            proxy_cache_valid 200 10m;
            add_header X-Cache-Status $upstream_cache_status;
```

* **proxy_cache mycache;** means we are telling Nginx to store the data using this cache zone.
* **proxy_cache_valid 200 10m;** means Nginx will cache the 200 OK response from the server for 10 minutes. After 10 minutes, that cached response expires, and Nginx fetches fresh data from the backend and stores it again in the cache.
* **add_header X-Cache-Status $upstream_cache_status;** means the cache status will be shown in the browser, so we can understand whether it is working or not. You can check it in the browser by pressing **F12 → Network → reload the page → click /api/status → Headers → Response Headers**. There you will see X-Cache-Status: MISS or X-Cache-Status: HIT.
	* X-Cache-Status: MISS means that, for the first request, the client request was sent to the backend server.
	* X-Cache-Status: HIT means that, on the second request, the response came from Nginx instead of the backend server.

For easier checking, you can use the terminal itself. **Run: curl -k -i https://localhost/api/status**. In the response headers, you will see X-Cache-Status: MISS or X-Cache-Status: HIT.

Other cache statuses include:

1. **MISS** => Not in cache → Nginx contacted FastAPI
2. **HIT**	=> Found in cache → Nginx served it without contacting FastAPI
3. **EXPIRED** => Cached response existed but expired → Nginx gets a fresh one
4. **BYPASS** => Nginx was instructed not to use the cache
	
* ### Note:
	- inactive=60m means that if the client does not request that URL for 60 minutes, the cached data will be deleted.
	- In proxy_cache_valid 200 10m; 10m means the stored data expires after 10 minutes, and Nginx gets a new response from the backend and stores it again in the cache. 


### Full code:
	```
	events {}
	
	http {
	    proxy_cache_path /data/nginx/cache levels=1:2 keys_zone=mycache:10m max_size=20m inactive=60m use_temp_path=off;

	    upstream backend_server {
	        server localhost:5000;
	        server localhost:5001;
	        server localhost:5002;
	    }
	
	    server {
	        listen 443 ssl;
	        server_name localhost;
	        
	        ssl_certificate /home/umar/nginxTutorial/ssl/nginx.crt;
	        ssl_certificate_key /home/umar/nginxTutorial/ssl/nginx.key;
	
	        location / {
	            proxy_pass http://127.0.0.1:3000;
	        }
	
	        location /api/status {
	            proxy_pass http://backend_server;
	            proxy_cache mycache;
	            proxy_cache_valid 200 10m;
	            add_header X-Cache-Status $upstream_cache_status;
	        }
	
	        location /calculator {
	            proxy_pass http://backend_server;
	        }
	    }
	
	    server {
	        listen 80;
	        server_name localhost;
	        return 301 https://$host$request_uri;
	    }
	}
	```
	