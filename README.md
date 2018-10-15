# Hooklift Platform Containers

Lightweight Hooklift platform containers.





### TODO: automate release

0. Monitor minor version releases upstream
1. Update version number in Dockerfile
2. docker build -t haproxy/v1.8.9 .
3. docker build -t hooklift/haproxy:latest .
4. docker push hooklift/haproxy:v1.8.9
5. docker push hooklift/haproxy:latest
