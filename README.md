# Nginx As A LoadBalancer Using Docker
#### In this Guide we will discuss how to setup multible servers on containers and make a Nginx container acts as a load balancer
What's LoadBalancer : \
![image](https://github.com/user-attachments/assets/bbf939cc-f49e-4bd3-ae44-8cae6390ceaa)
\
\
\
\
\
\
Load balancer is just like a router, it efficiently distributing incoming network traffic and also the method to solve high concurrency problem. In this Guide, I will use Nginx and docker to build load balancer, hope you guys will like it.

### How to Write Nginx conf file 
we need to define upstream directive and choose the load balancing method.
```bash
http {
    upstream loadbalancer{
        ${method you want};
        server serverA;
        server serverB;
        server serverC;
    }
    
    server{
        location/ {
            proxy_pass http://loadbalancer;
        }
    }
}
```
1. upstream `${name you wanted}` In this section you need to define a group name and consist of N servers.
2. server `{ip of the server}` Normally you need to type the ip of the server, but we are going to use docker to build our server. Therfore, we can use the name in docker-compose file to substitute the ip.
3. `loaction/ proxy_pass http://{group name}` To pass requests to a server group, the name of the group is specified in the proxy_pass directive

#### methods of load balancing:
-   Round Robin(default) : Requests are distributed evenly across the servers, with server weights taken into consideration.
-   Least Connections : A request is sent to the server with the least number of active connections.
-   Ip Hash : Request is determined from the client IP address, which will guarantees that same address get same server.
-   Generic Hash : Request is determined from the user-define key.


#### Server weight
Nginx using Round Robin as method, the weight of each server is set as 1 in default. However you can set the weight whatever you want.
```
upstream loadbalancer{
    server serverA weight 5;   // A will handle 50% requests
    server serverB weight 3;   // B will handle 30% requests
    server serverC weight 2;   // C will handle 20% requests
}
```

### Example
In this example i will use FastAPI as backend framework. I will build three server and a load balance with Nginx.
* Structure
```
Example
├── docker-compose.yml
├── nginx
│   ├── Dockerfile
│   └── nginx.conf
├── servera
│   ├── Dockerfile
│   ├── main.py
│   └── requirements.txt
├── serverb
│   ├── Dockerfile
│   ├── main.py
│   └── requirements.txt
└── serverc
    ├── Dockerfile
    ├── main.py
    └── requirements.txt
```    
* nginx.conf
```
upstream loadbalancer {
    server serverA:8000 weight=1;
    server serverB:8000 weight=1;
    server serverC:8000 weight=1;
}

server {
    location / {
        proxy_pass http://loadbalancer;
    }
}

```
* nginx Dockerfile
```
FROM nginx
RUN rm /etc/nginx/conf.d/default.conf    // remove origin config file
COPY ./nginx.conf /etc/nginx/conf.d/default.conf
```

* docker-compose.yml
```
version: '3'
services:
  servera:
    build: ./servera
    ports:
      - "8001:8000" 

  serverb:
    build: ./serverb
    ports:
      - "8002:8000"   

  serverc:
    build: ./serverc
    ports:
      - "8003:8000" 

  nginx:
    build: ./nginx
    ports:
      - 8080:80
    depends_on:
      - servera
      - serverb
      - serverc
```
