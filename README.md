# 44

To set up an NGINX container as a load balancer to redirect traffic to multiple instances of your Flask application


#### 1. Set Up Multiple Instances of the Flask Application as in Assignment 43

1. **Run Multiple Flask Containers**  
   Start two or more instances of your Flask app container with different container names and mapped ports.

   ```bash
   docker run -d --name flask-app-1 -p 5001:5000 flask-hello-world
   docker run -d --name flask-app-2 -p 5002:5000 flask-hello-world
   ```

   - Here, `flask-app-1` and `flask-app-2` are names for each container instance.
   - `-p 5001:5000` maps port 5001 on the host to port 5000 in `flask-app-1`, and `5002` does the same for `flask-app-2`.

2. **Verify the Containers are Running**  
   Check if both containers are running with:

   ```bash
   docker ps
   ```

#### 2. Create an NGINX Configuration File for Load Balancing in the same directory

1. **Create an NGINX Configuration File**  
   Create a file named `nginx.conf` in a directory (e.g., in the project directory where you have the Flask app).

   ```nginx
   # nginx.conf

   events {}

   http {
       upstream flask_app {
           server flask-app-1:5000;
           server flask-app-2:5000;
       }

       server {
           listen 80;

           location / {
               proxy_pass http://flask_app;
           }
       }
   }
   ```

   - `upstream flask_app` defines a load balancing group named `flask_app` with two backend servers (`flask-app-1` and `flask-app-2`).
   - `proxy_pass` will forward requests from NGINX to this upstream group.

#### 3. Run the NGINX Container with Load Balancing

1. **Run the NGINX Container**  
   Now, start an NGINX container with the custom configuration file:
   
   ```bash
   docker run -d --name nginx-loadbalancer -p 8080:80 --link flask-app-1 --link flask-app-2 -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf:ro nginx (Linux)
   docker run -d --name nginx-loadbalancer -p 8080:80 --link flask-app-1 --link flask-app-2 -v ${PWD}/nginx.conf:/etc/nginx/nginx.conf:ro nginx (Windows)
   ```

   - `-p 8080:80` maps port 8080 on the host to port 80 in the NGINX container.
   - `--link flask-app-1` and `--link flask-app-2` link the NGINX container to the two Flask app containers for network communication.
   - `-v $(pwd)/nginx.conf:/etc/nginx/nginx.conf:ro` mounts the custom `nginx.conf` configuration file in the NGINX container as read-only.

#### 4. Verify Load Balancing

1. **Access the Load Balancer**  
   Open a browser and go to [http://localhost:8080](http://localhost:8080).  
   You should see the "Hello, World!" response from the Flask app, with NGINX balancing requests between `flask-app-1` and `flask-app-2`.

   If you refresh the page several times, the responses should alternate between the two Flask instances, as NGINX distributes the load.

   ```bash
   (Linux)  for i in {1..10}; do curl -s http://localhost:8080; echo; done
   (Windows in Powershell) for ($i=0; $i -lt 1000; $i++) {
    curl http://localhost:8080 | Out-Null }

   ```
   verify the load balancing using the command of doker :
   ```bash
   docker stats flask-app-1 flask-app-2
   ```
