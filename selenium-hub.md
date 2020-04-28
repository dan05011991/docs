# Selenium test scalability

Issue:
- Selenium tests take hours to completed
- leads to subset of tests being executed regularly
- delayed knowledge of broken tests until full set are executed
- reluctance to write more tests due to timing issue

Solution will be to:
- scale executors to a feasible number on each node and replicate this across available machines

Known issues with this solution:
- Tests have to be written to run in isolation - should not share variables between tests

## Solution

Docker images to run a selenium hub and as many chrome nodes as possible across multiple machines to allow us to reduce test time. 

### Terminology
- Selenium Node - this is a container running a number of web drivers (can be different applications and/or versions). In this scenario each selenium node runs 2 chrome web drivers.
- Chome web driver - this is an individual instance of a chrome web driver running within the selenium node container. Each instance has its own web driver, there is no shared execution.
- Selenium hub - this is the controller & interface between test runs (executed in jenkins) and the actual web drivers which execute the test instructions (Chrome web drivers). Instructions are sent to the selenium hub and those instructions are passed to the corresponding web driver node (chrome node)

### Configuration
- browser timeout = 10 [10 seconds] (if the instruction executing in the browser hasn't responding in this amount of time then the test is aborted)
- cleanup cycle = 5000 [5 seconds] (time interval between checking for hanging processes)
- timeout = 10 [10 seconds] (if the web driver has received any more instructions in this time, the web driver is free'ed up to execute a waiting test)

## Architecture 

![Selenium hub layout](https://github.com/dan05011991/diagrams/raw/master/Selenium%20Hub.png)

In this example we have 5 separate boxes, 1 running the selenium hub and the others running chrome web drivers.

## Test results

### Setup
- Junit5 tests triggered on host (personal) machine
- Chrome web driver images deployed on AWS Centos 8 EC2 instances, 1 CPU and 1Gb RAM.
- Selenium hub image deployed on a separate EC2 instance.

Total EC2 instances used: 5 (1 hub, 4 x chrome web drivers)

### Results

| # of Tests| Concurrent | Time taken |
|---|---|---|
| 5 | N | 30s |
| 5 | Y | 12s |
| 10 | N | 59s |
| 10 | Y | 13s |
| 20 | N | 2m 5s |
| 20 | Y | 22s |
