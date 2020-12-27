---
title: "Project - GoCAN"
date: 2020-12-26T17:10:36-05:00
draft: true
tags: ["project", "golang"]
author: "Daniel Millson"
showToc: true
---

Hey everyone, welcome to the first post here.

One of the projects I've been working on recently is what I called "[GoCAN](https://github.com/portobello-boy/GoCAN)". It's an implementation of a Content Addressable Network, described [here](https://people.eecs.berkeley.edu/~sylvia/papers/cans.pdf), which I've been working on with a friend of mine.

## What is a CAN?
A Content Addressable Network is a network of systems which store a set of content-addressable data. What that means is the data in the network is stored in a distributed hash table, and the data is partitioned between different systems in the network. This distrubuted hash table is built to be scalable, allowing many different members to join the network and host data.

### Managing Data
Data in a CAN is hashed to an _n_-dimensional Cartesian coordinate space, normalized in the range [0, 1) in any dimension. Our rudimentary implementation involves key-value pairs mapping strings to strings. The key string is hashed to an _n_-dimensional point, so that if someone were to search in the CAN using that key, they would retrieve the value string from the original insert.

Any operations done to data, whether insert, updating, or deleting, require a key string.

### Routing and CAN Hosts
When a CAN is initialized, the host creating the CAN owns the entire coordinate space in which data is stored. Once a separate process (on the same system or otherwise) decides to join the CAN, then it provides a key string which is hashed to a point in the coordinate space. The original CAN host will split the coordinate space and hand over half the region to the joining host, as well as any data that was hashed to that region.

Because a CAN is a distributed network, there must be some sort of routing data and requests throughout. If each CAN host were omniscient, then each one would have knowledge of all other hosts in the network. While this is a practical solution for a small network, if this were scaled up to hundreds of systems, then ensuring new systems are acknowledged by all existing hosts would be a difficult task.

To address this, each host only knows the information (IP address, listening port, and Cartesian space) about its immediate neighbors. This makes the joining process easier as only neighbors of the splitting CAN host must be updated.

If a client requests data from a CAN host that does not own the requested data, then it attempts to route the request to a neighbor whose regional midpoint is closed to point generated from the hashed key of the requested data.

## Our Implementation
Our implementation of a CAN is done using Golang, or Go for short. Go is a feature-rich language which makes it easy to create a REST API using some community-written libraries. This was a good excuse for us to explore a new language and see its strengths and weaknesses. 

I had previously implemented a CAN for a project in a network theory class, using C/C++ and defining my own packet types to write over sockets. For this implementation, we decided it would be easier to use HTTP over TCP/IP, since we could leverage existing libraries without worrying about learning how to write over sockets in Go.