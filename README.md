# docker_load_balancer
Testing DNS round robin in Docker

# DNS Round Robin: A “poor man’s” load balancing solution.

There are different approaches and techniques for load balancing and DNS round robin is just one of them. With DNS round robin, we rely on the DNS server responses instead of a strictly dedicated appliance, server or container with more sophisticated algorithms and strategies…so we often refer DNS round robin as poor’s man load balancing. How does it work exactly? Simply by getting the DNS servers to respond to DNS requests not only with a single potential IP address, but with one out of a list of potential IP addresses corresponding to several servers or containers that host identical services.


# DNS Round Robin in Docker

So all we need is a DNS server then. Luckily for us, the Docker Engine implements an embedded DNS server for containers in user-defined networks since Docker 1.10. In particular, containers that are run with a network alias ( — net-alias) are resolved by this embedded DNS with the IP address of the container when the alias is used.


## 
```
git clone https://github.com/LyleH/docker_load_balancer
$ cd lb
$ docker-compose -v
docker-compose version 1.7.1, build 0a9ab35
$ docker-compose up -d
Creating lb_client_1
Creating lb_server_1
$ docker-compose scale server=5
Creating and starting lb_server_2 ... done
Creating and starting lb_server_3 ... done
Creating and starting lb_server_4 ... done
Creating and starting lb_server_5 ... done
$ docker-compose scale client=2
Creating and starting lb_client_2 ... done
$ docker inspect — format “{{.Name}} {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}” $(docker ps -aq) | sort -t ‘ ‘ -k2
/lb_server_1 172.20.0.2
/lb_client_1 172.20.0.3
/lb_server_2 172.20.0.4
/lb_server_3 172.20.0.5
/lb_server_4 172.20.0.6
/lb_server_5 172.20.0.7
/lb_client_2 172.20.0.8
$ docker-compose logs client | grep client_1 | grep PING | sort | uniq -c
 19 client_1  | PING server (172.20.0.2)
$ docker-compose logs client | grep client_2 | grep PING | sort | uniq -c
  4 client_2  | PING server (172.20.0.2)
  2 client_2  | PING server (172.20.0.4)
  5 client_2  | PING server (172.20.0.5)
  2 client_2  | PING server (172.20.0.6)
  5 client_2  | PING server (172.20.0.7)
  ```
  
  
