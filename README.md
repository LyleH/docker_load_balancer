# docker_load_balancer
Testing DNS round robin in Docker

# Benefits of load balancing

Scalability - The amount of traffic a website receives has a substantial effect on its performance, and load balancing provides the capability to handle most sudden spikes in traffic by spreading the traffic across multiple servers. Adding more load balanced servers to handle increased traffic is much easier and faster to implement than moving a site to an entirely new, more powerful server.

Flexibility - Using multiple load balanced servers to handle a site’s traffic allows administrators the flexibility to perform maintenance on a server without impacting site uptime. This can be done by pointing all traffic to one server and placing the load balancer in active/passive mode.

Stability - Perhaps the biggest win, instead of your website / service being a single point of failure, load balancing allows the site / service to continue operation when some instraces of the service has died due to unforeseen circumstances like environmental issues etc.. 

# DNS Round Robin: A “poor man’s” load balancing solution.

There are different approaches and techniques for load balancing and DNS round robin is just one of them. With DNS round robin, we rely on the DNS server responses instead of a strictly dedicated appliance, server or container with more sophisticated algorithms and strategies…so we often refer DNS round robin as poor’s man load balancing. How does it work exactly? Simply by getting the DNS servers to respond to DNS requests not only with a single potential IP address, but with one out of a list of potential IP addresses corresponding to several servers or containers that host identical services.


# DNS Round Robin in Docker

So all we need is a DNS server then. Luckily for us, the Docker Engine implements an embedded DNS server for containers in user-defined networks since Docker 1.10. In particular, containers that are run with a network alias ( — net-alias) are resolved by this embedded DNS with the IP address of the container when the alias is used. Let´s see it in practice:

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
  
  # Things to note
  When doing source compose.env, it defaults to
  ```
  export COMPOSE_FILE=docker-compose.yml:dev.yml
  ```
  this needs to be changed by running the following command after source compose.env, run
  ```
  export COMPOSE_FILE=docker-compose.yml
  ```
  
# Setting up a particular load balancer
In the docker-compose.yml:dev.yml of you lb, add the services you would like to load balance.
Ensure that you do not clash ports with the actual services, this can be achived by creating a env file in your lb with port mappings as below:
```
WMS_API_HOST=10.4.1.51
WMS_API_PORT=8021
ADAPTRIS_SERVICE_HOST=10.4.1.51
ADAPTRIS_SERVICE_PORT=8022
```

# Now update your dev.yml something like
```
version: '2'
services:
  wmsapi:
    env_file: env
    extends:
      file: ../../../services/wmsapi.yml
      service: wmsapi
  service_wms:
    env_file: env
    extends:
      file: ../../../services/service_wms.yml
      service: service_wms
    command: bash -c "/opt/protobuf/bin/protoc --proto_path=wms/interfaces/protobuffs --python_out=wms/interfaces/protobuffs wms/interfaces/protobuffs/*.proto && pip install --trusted-host pypi01.stagealot.com --index-url http://pypi01.stagealot.com/pypi -e . && /usr/local/bin/s3f_service.py --service-type wms -in 0.0.0.0:6333 -ll INFO"
```
# Start the load balancer and also your service and they will run in parallel as below.
![screen shot 2016-11-09 at 13 24 44](https://cloud.githubusercontent.com/assets/16188304/20139069/eef7f526-a68c-11e6-8cef-3f73f4c24e85.png)
