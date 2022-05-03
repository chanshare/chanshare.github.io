# Chanshare V2

## Overview

```

┌─────────────────────┐                                                                                                                                                ┌─────────────────────────────────────────────────────┐
│                     │                                                                                                                                                │ Content Service                                     │
│                     │                                         CDN Requests                                                                                           │ ┌─────────────┐             ┌─────────────────────┐ │       ┌─────────┐
│                     ├────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────► │             │  MISS       │                     │ │       │         │
│                     │                                                                                                                                                │ │             ├─────────────►    Fetch Service    ├─┼───────► 4ch API │
│                     │                                                                                                                                                │ │             │             │                     │ │       │         │
│                     │                                                                                                                                                │ │             │             └─────────────────────┘ │       └─────────┘
│                     │                                                                                                                                   Get Playlist │ │  Cache API  │                                     │
│                     │                                                                                                                           ┌────────────────────┼─►             │                                     │
│                     │                                                                                                                           │                    │ │             │             ┌─────────────────────┐ │
│                     │                                                                                                                           │                    │ │             │  HIT        │                     │ │
│                     │                             ┌───────────────────────────────┐                                                             │                    │ │             ├─────────────►      Minio S3       │ │
│                     │        Create Room        ┌─┴──────────────────────────────┐│                                                             │                    │ │             │             │                     │ │
│                     ├───────────────────────────►                                ◄├─────────────────────────────────────────────────────────────┘                    │ │          ┌──┼────────┐    └─────────────────────┘ │
│                     │                           │                                ││ Create New Room with Specified Parameters and Playlist                           │ └──────────┼──┘        │                            │
│                     │        Room Name          │       Room Provisioner         │┼────────────────────────────────────────────────────────┐                         │            │   Redis   │                            │
│                     ◄───────────────────────────┤                                ││                                                        │                         │            │           │                            │
│                     │                           │                              ┌─┼┴─────────┐                                              │                         │            └───────────┘                            │
│                     │                           └──────────────────────────────┼─┘          │                                              │                         │                                                     │
│                     │                                                          │   Redis    │                                              │                         └─────────────────────────────────────────────────────┘
│                     │                                                          │            │                                              │
│                     │                                                          └────────────┘                                              │
│                     │                                                                                                                      │
│                     │                                                                                                                      │
│                     │                                                                                                                      │
│                     │                                                                                                                      │
│                     │                                                                                                                      │
│                     │               ┌───────────────────────────────────────────────────────────────┐       ┌──────────────────────────────│─────────────────────────────────┐
│                     │             ┌─┴──────────────────────────────────────────────────────────────┐│     ┌─┴──────────────────────────────▼────────────────────────────────┐│
│                     │             │ Websocket Handlers                                             ││     │ Room Manager                                                    ││
│    API GATEWAY      │  Websocket  │                                                                │In/Out│ ┌──────────────────┐ ┌───────────────────┐ ┌─────────────────┐  ││
│      Traefik        │  Messaging  │  ┌─────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐  │ NATS │ │                  │ │                   │ │                 │  ││
│                     │             │  │             ◄─┤            ├─┤            ├─┤            ├──┼┬─────┼─►                  │ │                   │ │                 │  ││
│                     ├─────────────►  │             │ │            │ │            │ │            │  ││     │ │                  │ │                   │ │                 │  ││
│                     │             │  │   Room 1    │ │   Room 1   ◄─┤   Room 1   ├─┤  Room 1    ├──┼┼─────┼─►      Room 1      │ │      Room 2       │ │      Room 3     │  ││
│                     ◄─────────────┤  │   User 1    │ │   User 2   │ │   User 3   │ │  User 4    │  ││     │ │                  │ │                   │ │                 │  ││
│                     │             │  │             │ │            │ │            ◄─┤            ├──┼┼─────┼─►                  │ │                   │ │                 │  ││
│                     ├─────────────►  │             │ │            │ │            │ │            │  ││     │ │                  │ │                   │ │                 │  ││
│                     │             │  │             │ │            │ │            │ │            ◄──┼┼─────┼─►                  │ │                   │ │                 │  ││
│                     ◄─────────────┤  └─────────────┘ └────────────┘ └────────────┘ └────────────┘  ││     │ └──────────────────┘ └───────────────────┘ └─────────────────┘  ││
│                     │             │                                                                ││     │                                                                 ││
│                     ├─────────────►  ┌─────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐  ││     │ ┌──────────────────┐  ┌──────────────────┐  ┌────────────────┐  ││
│                     │             │  │             │ │            │ │            │ │            │  ││     │ │                  │  │                  │  │                │  ││
│                     ◄─────────────┤  │             │ │            │ │            │ │            │  ││     │ │                  │  │                  │  │                │  ││
│                     │             │  │   Room 2    │ │   Room 2   │ │   Room 2   │ │   Room 2   │  ││     │ │                  │  │                  │  │                │  ││
│                     ├─────────────►  │   User 1    │ │   User 2   │ │   User 3   │ │   User 4   │  ││     │ │       Room 4     │  │      Room 5      │  │     Room 6     │  ││
│                     │             │  │             │ │            │ │            │ │            │  ││     │ │                  │  │                  │  │                │  ││
│                     ◄─────────────┤  │             │ │            │ │            │ │            │  ├┘     │ │                  │  │                  │  │                │  ││
│                     │             │  └─────────────┘ └────────────┘ └────────────┘ └────────────┘  │      │ │                  │  │                  │  │                │  ├┘
│                     │             │                                                                │      │ └──────────────────┘  └──────────────────┘  └────────────────┘  │
│                     │             └────────────────────────────────────────────────────────────────┘      │                                                                 │
└─────────────────────┘                                                                                     └─────────────────────────────────────────────────────────────────┘
``` 

## Components

[OpenAPI Spec](/redoc-static.html)

* API Gateway:
  * Traefik
  * Handles routing requests to the correct service
  * Scaling maybe
  * HTTP & Websocket
* Room Provisioner 
  * Go 
  * Orchestrates the building of a room
  * The decision still stands as to build a playlist then build the room or vice versa
  * Has an attached redis instance for keeping track of open rooms
  * Horizontally scaleable
  * HTTP in, HTTP out (maybe NATS out)
* Content Service 
  * Go & Minio
  * Serves as a cache layer instead of sending every client to the 4cdn
  * Has attached redis to actually serve as the cache, could potentially use some other in memory option
  * Horizontally scaleable
  * HTTP
  * Cache API
    * Go
    * Serves as an HTTP API to the fetch service and minio storage
  * Fetch service
    * Will fetch the media for a given threadid and board
* Room Manager
  * Go maybe rust 
  * Handles the backend logic of a room e.g. keeping clients in sync etc.
  * Connects back to the websockets via NATS
  * May need to connect back to the provisioner's Redis
  * HTTP and NATS in, NATS out
* Websocket Handlers
  * Rust 
  * Reads in from the ws and sends it to the room via NATS
  * Read from the room via NATS and send back to the client

The intention is that all services will export logs and tracing to Prometheus and Jaeger respectively

## Infrastructure

* Kubernetes
* Grafana & Prometheus
* Consul
* Redis
* NATS
* Jaeger
* Minio
