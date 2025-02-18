# Big Brother
This project defines a service to effectively communicate observability events to application stakeholders.  

# How does it work

Basically, it collects the necessary metrics from a client provided `bb-promster` cluster endpoint.

More specifically, the **Big Brother** app federates it's own monitoring cluster to the provided endpoint, and starts collecting **Big Brother** specific metrics, with the help of some useful programming libraries.

These metrics are treated as the fundamental protocol behind **Big Brother's** capabilities.

A valid **Big Brother** library should expose the following metrics: 

```
request_seconds_bucket{type, status, method, addr, le}
request_seconds_count{type, status, method, addr}
request_seconds_sum{type, status, method, addr}
response_size_bytes{type, status, method, addr}
dependency_up{name}
```

Where, for a specific request, `type` tells which request protocol was used (e.g. `grpc` or `http`), `status` registers the response status code, `method` registers the request method and `addr` registers the requested endpoint address.

In detail:

1. The `request_seconds_bucket` metric defines the histogram of how many requests are falling into the well defined buckets represented by the label `le`;
2. The `request_seconds_count` is a counter that counts the overall number of requests with those exact label occurrences;
3. The `request_seconds_sum` is a counter that counts the overall sum of how long the requests with those exact label occurrences are taking;
4. The `response_size_bytes` is a counter that computes how much data is being sent back to the user for a given request type. It captures the response size from the `content-length` response header. If there is no such header, the value exposed as metric will be zero;
5. Finally, `dependency_up` is a metric to register weather a specific dependency is up (1) or down (0). The label `name` registers the dependency name;

The following libraries make part of **Big Brother** official libraries:

1. [`express-monitor`](https://github.com/labbsr0x/express-monitor) for the Node JS Express framework.
1. [TODO] `iris-monitor` for the Katara's Iris web framework;
2. [TODO] `mux-monitor` for the Gorilla's Mux web framework;
3. [TODO] `spring-monitor` for the Pivotal's Spring web framework;
4. [TODO] `ejb-monitor` for Enterprise Java Bean web frameworks;

Without these, you would have to expose the metrics by yourself, possibly leading to inconsistencies and other errors when setting up your app's observability infrastructure with **Big Brother**.   

# Components

The **Big Brother** app is composed by an **ETCD cluster**, a **Dialogflow Bot**, a **Prometheus Alertmanager**, a **Grafana** and a **Promster** cluster, all with their own configuration needs.

## ETCD

The ETCD cluster serves 3 purposes:

1. Register client `bb-promster` clusters;
2. Register **Big Brother's** own monitoring cluster, to allow for sharding via federation;
3. [TODO] Register **Big Brother's** alertmanager cluster, for high availability;

Gets configured by:

1. `ETCD_LISTEN_CLIENT_URLS`: the addresses ETCD daemon listens to client traffic;
2. `ETCD_ADVERTISE_CLIENT_URLS`: list of an ETCD client URLs to advertise to the rest of the cluster; 

## Dialogflow Bot

A bot to communicate with the interested stakeholders. It's purposes are to:

1. Enable CRUD of client apps to be observed by **Big Brother**; and
2. Alert on possible problems; 

## Prometheus Alertmanager

A service to host alerting configuration on top of the alerts being dispatched by the **Promster Cluster**.

Gets configured by:

1. `WEBHOOK_URL`: the bot address

## Promster Cluster (monitor)

The service that federate's on the client's `bb-promster` cluster, hosts and evaluates alerting rules and dispatches alerts accordingly.

Gets configured by:

1. `BB_PROMSTER_LEVEL`: integer greater than 0 that defines which level this promster sits on it's own federation cluster; 
2. `ETCD_URLS`: defines the ETCD cluster urls, separated by comma;
3. `ALERT_MANAGER_URLS`: defines the alertmanager cluster urls;

# How to Run

Type the following commands in your terminal:

```bash
TELEGRAM_ADMIN=<yourid> TELEGRAM_TOKEN=<XXXXX:YYYYYY> docker-compose up -d --build
```

This will run an example app with its own `bb-promster` cluster and the **Big Brother** app with its components.

By running:
```bash
docker-compose exec --env ETCDCTL_API=3 etcd etcdctl put "/clients/Example/example-bb-promster:9090" -- ""
```

The client app `bb-promster` cluster will get registered to the **Big Brother's** ETCD and **Big Brother** will then start collecting metrics by federating it.

# Trivia

The name is inspired by George Orwell's 1984 **Big Brother** character. 

In this book, **Big Brother** is an entity that is *omniconscious*, being able to watch everyone, everywhere. 

This is exactly what we aim to achieve with this project: a way for you to easily and effective watch every project you have without any prior knowledge of observability concepts and Prometheus best practices. 
