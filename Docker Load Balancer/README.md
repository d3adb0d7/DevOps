
# Simple Layer 4 Load Balancer  using Docker



## What is Layer 4 Load Balancer

Layer 4 load balancing uses information defined at the networking transport layer (Layer 4) as the basis for deciding how to distribute client requests across a group of servers. For Internet traffic specifically, a Layer 4 load balancer bases the load-balancing decision on the source and destination IP addresses and ports recorded in the packet header, without considering the contents of the packet.
![App Screenshot](https://www.loadbalancer.org/blog/content/images/2020/05/concept-dsr-1.png)

![App Screenshot](https://www.esds.co.in/blog/wp-content/uploads/2016/07/Layer-4-Load-Balancing.png)

## Whats is Docker
Docker is a software platform that allows you to build, test, and deploy applications quickly. Docker packages software into standardized units called containers that have everything the software needs to run including libraries, system tools, code, and runtime. Using Docker, you can quickly deploy and scale applications into any environment and know your code will run.
Running Docker on AWS provides developers and admins a highly reliable, low-cost way to build, ship, and run distributed applications at any scale.
## Installation

#### Create a directory for project

```bash
mkdir docker_project
```
#### Create a new file nginx.conf

```bash
upstream loadbalancer {
server 172.17.0.1:5001 weight=6;
server 172.17.0.1:5002 weight=4;
}
server {
location / {
proxy_pass http://loadbalancer;
}}
```
#### Create Dockerfile dockerfile
```bash
FROM nginx
RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d/default.conf
```
#### Now we need to build our Docker container.This Image will be used for Load Balancer.We will use nginx as our Load Balancer.
```bash
sudo docker build -t lb .
```
#### Run load Balancer Docker container.
```bash
sudo docker run -p 8080:80 lb
```
```bash
sudo docker ps
```

### Expected Output

```bash
CONTAINER ID   IMAGE     COMMAND                  CREATED        STATUS        PORTS                                   NAMES
9ca7354c3c1d   lb        "/docker-entrypoint.…"   34 hours ago   Up 34 hours   0.0.0.0:8080->80/tcp, :::8080->80/tcp   gifted_cohen
```
#### Now we need to Deploy another container for Webserver.Deploy two nginx container.
#### For first web server :
```bash
sudo docker run -d nginx
```
#### For second web server one:
```bash
sudo docker run -d nginx
```
### Expected Output

```bash
CONTAINER ID   IMAGE     COMMAND                  CREATED        STATUS        PORTS                                   NAMES
9ca7354c3c1d   lb        "/docker-entrypoint.…"   34 hours ago   Up 34 hours   0.0.0.0:8080->80/tcp, :::8080->80/tcp   gifted_cohen
ec99b2d16d8d   nginx     "/docker-entrypoint.…"   34 hours ago   Up 34 hours   80/tcp                                  confident_mayer
c42fb268cadc   nginx     "/docker-entrypoint.…"   37 hours ago   Up 37 hours   80/tcp                                  silly_hellman
```
#### Now we Need to Know the ip of container which is used  by as a Webserver.

```bash
sudo docker inspect c42fb
```
```bash
sudo docker inspect ec99
```
### Expected Output
<a href="https://ibb.co/Cn8ybHM"><img src="https://i.ibb.co/7WNk2wP/download.png" alt="veth-peer" border="0"></a><br />

<a href="https://ibb.co/std4YQ0"><img src="https://i.ibb.co/1btNWKD/download-1.png" alt="veth-peer" border="0"></a><br />
#### Need to Re-configure nginx.conf

```bash
upstream loadbalancer {
server 172.17.0.3:80 weight=6;
server 172.17.0.5:80 weight=4;
}
server {
location / {
proxy_pass http://loadbalancer;
}}
```
#### We have some modification so need to build again.
```bash
sudo docker build -t lb .
```
#### Run load Balancer Docker container.
```bash
sudo docker run -p 8080:80 lb
```
### Curl Web server:
```bash
curl http://localhost:8080
```
### Expected Output
```bash
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx! webserver 2</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
<a href="https://ibb.co/Hd14vkT"><img src="https://i.ibb.co/PQ3jpfG/download-3.png" alt="veth-peer" border="0"></a><br />
### Taking Trace using Tcp Dump. Is it working or not ?
#### Enter to load Balancer Docker container.
```bash
sudo docker exec -it 9ca7354c3c1d /bin/bash
```
#### Install Tcp dump on docker container.
```bash
sudo apt update
sudo apt install tcpdump
tcpdump -i eth0 -w trace_test.pcap
```
#### Uploading Tcpdump file to temp upload server.
```bash
curl bashupload.com -T trace_test.pcap
```
#### Download Tcpdump file from temp upload server.
```bash
wget http://bashupload.com/####/trace_test.pcap
```
### Analysis:
<a href="https://ibb.co/fMcG7x8"><img src="https://i.ibb.co/F6PqtDs/download-2.png" alt="veth-peer" border="0"></a><br />
## Our Simple Layer 4 Loadbalancer using Docker working successfully.

## Authors

- [@d3adb0d7](https://github.com/d3adb0d7)

## Reference
- https://www.loadbalancer.org/blog/layer-4-vs-layer-7-load-balancing-we-still-love-dsr/
- https://aws.amazon.com/docker/
- https://www.esds.co.in/blog/types-of-load-balancing/
