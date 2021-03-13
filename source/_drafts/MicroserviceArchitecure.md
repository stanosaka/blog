---
title: MicroserviceArchitecure
tags: microservices
---
# Definition of Microservices
## Monolith
- Its complexity can be overwhelming
- Difficult to modularize
- Risk for misuseA

# Adding Support for Health
## Liveness and readiness endpoints
![endpoints](https://i.imgur.com/GWY3YPm.png "endpoints")
scenario:
two instances and both of them have endpoints, one **/ready** to determin wheather our application is ready, and the other **/live** used to
determine whether our application is still alive. K8s will periodically make requests of those end points to see whether or not they respond and 
how they respond. If they respond with 200, which is the standard HTTP response for ok, or any other successful code. However, ready start to return
an error code 503, K8s understands this means the application wants to tell K8s is it is no longer ready. And the action it takes when an application
or an instance is no longer ready is to sop sending requests to that application. As only one of our two instances is not ready, the other one contunes to receive
requests. If we also get to the point where live is no longer reporting a 200, but is reporting, say 503, then k8s understands that this is telling k8s
that the application is no onger live and needs to be restarted. This causes K8s to send a **SIGTERM** to the application, and this is a warning that it's about
to be replaced. Shortly afterwards, it sends it a **SIGKILL** , which removes the applicatoin instance. It will then restart the application instance by providing
a new docker container running the application. When his happens, our application will start reporting live as 200 as well and K8s adds it back to traffic. So 
by providing these two endpoints, we can help K8s understand whether our application needs to be restarted or whether it should be taken out of requests.
And we can use that to control the lifecycle of our application.

# Add support for request tracking
## Opentracking and request tracking
Open tracking is designed to take a request that comes into a microservice and track that requests and where any bottleneck for performance problem occurs
inside that flow.

![Online shopping scenario](https://i.imgur.com/2YttfLU.png "Online Shopping Scenario")
![Elapsed Time](https://i.imgur.com/8zx798J.png "Elapsed Time")
### OpenTracing
* Collects data from each enabled service
* Propagates correlation ID using HTTP headers
* Provides sampling, tracing, and debug capabilities

# To enhance apps with cloud native capabilities:
- Self-healing using health checks
- Metrics with Prometheus and Grafana
- Request tracking using OpenTracing, Zipkin, and Jaeger

courses on:
- Configuration and secrets
- Logging
- Circuit breaking
- Service meshes

- Docker
- Kubernetes
- Helm
- Prometheus

