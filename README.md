# easy-lb

This repository defines a load balancer container for Docker, based on nginx, confd and etcd.

## High-level design

The container runs an [nginx](https://www.nginx.com/resources/wiki/) load balancer process. The configuration is generated by [confd](https://github.com/kelseyhightower/confd) from live data pulled from [etcd](https://github.com/coreos/etcd). Changes made in etcd are automatically reflected in the configuration, making it very convenient for services to add and remove themselves from the system just by interacting with the etcd service.

## Deployment

To deploy a load balancer container, use the following command:

    docker run -p 80:80 -p 443:443 -e ETCD_PEERS=http://172.17.0.2:2379,http://172.17.0.3:2379 miguelgrinberg/easy-lb

In this example, the `ETCD_PEERS` variable is a comma-separated list of URLs for the etcd service, which must be deployed separately.

## Building

To build the container image locally, you can use the included `build.sh` script.

## Configuration

The load balancer is automatically configured from the contents of etcd's `/services` directory. For example, consider the following `etcdctl` commands:

    etcdctl set /services/foo/server1 172.17.0.4:5000
    etcdctl set /services/foo/server2 172.17.0.5:5000
    etcdctl set /services/bar/server3 172.17.0.6:5000

This will cause the load balancer to proxy requests to `/foo/...` URLs to server1 and server2, at the addresses and ports specified, while any requests to `/bar/...` URLs will be sent to server3. Note that the names `server{1,2,3}` are not significant, each service can register itself with any name, as long as it is unique.

Whenever changes are made in etcd to the `/services` subtree, those changes will automatically trigger a configuration update.

## TODO

- SSL support
- Automatic Let's Encrypt certificates
- Select load balancing method for each service