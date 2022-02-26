# 1. Create Apache Server Container

## Summary

The apache web server is where we will run our containerized app.
The main requirement is to use ubuntu as the base image so we will need to build a new docker image with all the apache dependicies required.

## Folder Structure

- website/config/site.conf
- website/html/index.html
- website/Dockerfile

## Dockerfile

```dockerfile
FROM ubuntu:22.04

# Install apache2
RUN set -eux; \
	apt update; \
	apt install -y --no-install-recommends \
		apache2 \
	;

# Redirect log files to stderr/stdout
RUN ln -sf /proc/self/fd/1 /var/log/apache2/access.log && \
    ln -sf /proc/self/fd/1 /var/log/apache2/error.log

# Copy site configurations into container
COPY config/ /etc/apache2/sites-available/

# Copy site configurations into container
COPY html/ /var/www/html/

EXPOSE 80

CMD apachectl -D FOREGROUND
```

## HAProxy

### Folder Structure

- haproxy/certs/www.mysites.com-full-chain.crt (contains the certificate and the key)
- haproxy/certs/www.mysites.com.crt (certificate generated with openssl with csr and key)
- haproxy/certs/www.mysites.com.csr
- haproxy/certs/www.mysites.com.key
- haproxy/Dockerfile
- haproxy/haproxy.cfg

### Dockerfile

```dockerfile
FROM haproxy:2.3

COPY haproxy.cfg /usr/local/etc/haproxy/haproxy.cfg
COPY certs/*-full-chain.crt /usr/local/etc/haproxy/certs/
```

# 2. Running the application stack

## Apache Server

### Build Image

```sh
~/apache-with-haproxy/website$ docker build -t apache-docker .
```

Output:
```
Sending build context to Docker daemon  7.168kB
Step 1/7 : FROM ubuntu:22.04
 ---> 6795c0065f64
Step 2/7 : RUN set -eux;        apt update && apt install -y --no-install-recommends            apache2         ;
 ---> Using cache
 ---> 9913b74272c1
Step 3/7 : RUN ln -sf /proc/self/fd/1 /var/log/apache2/access.log &&     ln -sf /proc/self/fd/1 /var/log/apache2/error.log
 ---> Using cache
 ---> afbdba8925cf
Step 4/7 : COPY config/ /etc/apache2/sites-available/
 ---> Using cache
 ---> 829f7e96ec74
Step 5/7 : COPY html/ /var/www/html/
 ---> Using cache
 ---> ad052a0e2689
Step 6/7 : EXPOSE 80
 ---> Using cache
 ---> 7e0cfd356b5e
Step 7/7 : ENTRYPOINT ["apachectl", "-D", "FOREGROUND"]
 ---> Using cache
 ---> 197d64876e47
Successfully built 197d64876e47
Successfully tagged apache-docker:latest
```

### Run it (with no port)

```sh
apache-with-haproxy/website$ docker run -it --name my-running-app apache-docker
```

Output:
```
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
[Sat Feb 26 00:23:26.144546 2022] [mpm_event:notice] [pid 15:tid 139909789734784] AH00489: Apache/2.4.52 (Ubuntu) configured -- resuming normal operations
[Sat Feb 26 00:23:26.144716 2022] [core:notice] [pid 15:tid 139909789734784] AH00094: Command line: '/usr/sbin/apache2 -D FOREGROUND'
```

`Note:` If you want to test if the apache webserver is working, then specify a port mapping in the container like so:

```sh
docker run -it --name my-running-app apache-docker -p 8080:80
```

And test if its working in your local browser with the following url: http://localhost:8080/index.html

## HAProxy

### Build Image

```sh
apache-with-haproxy/haproxy$ docker build -t my-haproxy .
```

Output:
```
Sending build context to Docker daemon  14.34kB
Step 1/3 : FROM haproxy:2.5.3-alpine
2.5.3-alpine: Pulling from library/haproxy
59bf1c3509f3: Pull complete 
e9c09ecd6852: Pull complete 
aa37371f0e32: Pull complete 
bd09ebc00124: Pull complete 
Digest: sha256:8e2b6b9367ef478f277a4d58fde13e843a2e7130467f5018f553ff4a3f7cc72c
Status: Downloaded newer image for haproxy:2.5.3-alpine
 ---> 9fd928051ae4
Step 2/3 : COPY haproxy.cfg /usr/local/etc/haproxy/haproxy.cfg
 ---> 979447de75a4
Step 3/3 : COPY certs/*-full-chain.crt /usr/local/etc/haproxy/certs/
 ---> fdb10d70ba98
Successfully built fdb10d70ba98
Successfully tagged my-haproxy:latest
```

### Run it (with port since we want to expose it to the outside world)

```sh
apache-with-haproxy/haproxy$ docker run -d --name my-running-haproxy --sysctl net.ipv4.ip_unprivileged_port_start=0 -p 443:443 my-haproxy
```

Check HAProxy's logs to see if the backend is healthy:
```sh
apache-with-haproxy/haproxy$ docker logs my-running-haproxy 
[NOTICE]   (1) : haproxy version is 2.5.3-abf078b
[WARNING]  (1) : config : log format ignored for frontend 'https-webservers' since it has no log address.
[NOTICE]   (1) : New worker (9) forked
[NOTICE]   (1) : Loading success.
[WARNING]  (9) : Health check for server web_servers/webserver1 succeeded, reason: Layer4 check passed, check duration: 0ms, status: 3/3 UP.
```

### Test the website in the browser

In order to test the `www.mysite.com` site locally we need to edit `/etc/hosts` with the following entry:

```
127.0.0.1       www.mysite.com
```

Try this URL in your browser, it should show "Hello World!": https://www.mysite.com/index.html

# References

- [Create SSL Certificates](https://www.baeldung.com/openssl-self-signed-cert)
- [HAProxy SSL Termination](https://www.haproxy.com/blog/haproxy-ssl-termination)
- [HAProxy with docker](https://www.haproxy.com/blog/how-to-run-haproxy-with-docker)
- [Production Ready Containers](https://dev.to/wimpress/creating-production-ready-containers-the-basics-3k6f)
- [Production Ready Containers(Advanced)](https://dev.to/wimpress/creating-production-ready-containers-advanced-techniques-4jm3)