[![Build Status](https://travis-ci.org/garystafford/voter-service.svg?branch=gke)](https://travis-ci.org/garystafford/voter-service) [![Dependencies](https://app.updateimpact.com/badge/817200262778327040/voter-service.svg?config=compile)](https://app.updateimpact.com/latest/817200262778327040/voter-service) [![Layers](https://images.microbadger.com/badges/image/garystafford/voter-service.svg)](https://microbadger.com/images/garystafford/voter-service "Get your own image badge on microbadger.com") [![Version](https://images.microbadger.com/badges/version/garystafford/voter-service.svg)](https://microbadger.com/images/garystafford/voter-service "Get your own version badge on microbadger.com")

# Voter Service

## Introduction

The Voter [Spring Boot](https://projects.spring.io/spring-boot/) Service is a RESTful Web Service, backed by MongoDB, using Atlas on GCP, and RabbitMQ, using CloudAMQP on GCP. It is part of the Voter API (see diagram below). The Voter service exposes several HTTP API endpoints, listed below. API users can review a list candidates, submit a vote, view voting results, and inspect technical information about the running service.

![Architecture](GKE_AMPQ_v1.png)

## Voter Service Endpoints

The service uses a context path of `/voter`. All endpoints must be are prefixed with this sub-path.

Purpose                                                                                                                  | Method  | Endpoint
------------------------------------------------------------------------------------------------------------------------ | :------ | :-----------------------------------------------------
List All Service Endpoints                                                                                               | GET     | [/voter/mappings](http://localhost:8099/voter/mappings)
Create Vote                                                                                                              | POST    | [/voter/voters](http://localhost:8099/voter/voters)
Read Vote                                                                                                                | GET     | [/voter/voters/{id}](http://localhost:8099/voter/voters/{id})
Read Votes                                                                                                               | GET     | [/voter/voters](http://localhost:8099/voter/voters)
Update Vote                                                                                                              | PUT     | [/voter/voters/{id}](http://localhost:8099/voter/voters/{id})
Delete Vote                                                                                                              | DELETE  | [/voter/voters/{id}](http://localhost:8099/voter/voters/{id})
List Candidates                                                                                                          | GET     | [/voter/candidates/{election}](http://localhost:8099/voter/candidates/db/{election})
View Voting Results                                                                                                      | GET     | [/voter/results/{election}](http://localhost:8099/voter/results/{election})
View Total Votes                                                                                                         | GET     | [/voter/results/{election}/votes](http://localhost:8099/voter/results/{election}/votes)
View Winner(s)                                                                                                           | GET     | [/voter/winners/{election}](http://localhost:8099/voter/winners/{election})
View Winning Vote Count                                                                                                  | GET     | [/voter/winners/{election}/votes](http://localhost:8099/voter/winners/{election}/votes)
Drop All Candidates                                                                                                      | POST    | [/voter/drop/candidates](http://localhost:8099/voter/drop/candidates)
Drop All Votes                                                                                                           | POST    | [/voter/drop/votes](http://localhost:8099/voter/drop/votes)
Service Info                                                                                                             | GET     | [/voter/info](http://localhost:8099/voter/info)
Service Health                                                                                                           | GET     | [/voter/health](http://localhost:8099/voter/health)
Other [Spring Actuator](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready) endpoints | GET     | `voter/actuator`, `voter/metrics`, `voter/env`, `voter/configprops`, etc.
Other [HATEOAS](https://spring.io/guides/gs/rest-hateoas) endpoints for `/voter/votes`                                   | Various | page sort, size, etc.

The [HAL Browser](https://github.com/mikekelly/hal-browser) API browser for the `hal+json` media type is installed alongside the service. It can be accessed at `http://localhost:8099/voter/actuator/`.

## Voting

Submitting a new candidate requires an HTTP `POST` request to the `/voter/votes` endpoint, as follows:

HTTPie

```bash
http POST http://localhost:8099/voter/votes \
  candidate="Jill Stein" \
  election="2016 Presidential Election"
```

cURL

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{ "candidate": "Jill Stein", "election": "2016 Presidential Election" }' \
  "http://localhost:8099/voter/votes"
```

wget

```bash
wget --method POST \
  --header 'content-type: application/json' \
  --body-data '{ "candidate": "Jill Stein", "election": "2016 Presidential Election" }' \
  --no-verbose \
  --output-document - http://localhost:8099/voter/votes
```

## Sample Output

API users can also create random voting data by calling the `/voter/simulation` endpoint. Using [HTTPie](https://httpie.org/) command line HTTP client.

```bash
http http://localhost:8099/voter/simulation/2016%20Presidential%20Election
```

```json
{
    "message": "Simulation data created!"
}
```

```bash
http http://localhost:8099/voter/candidates/db/2016%20Presidential%20Election
```

```json
{
    "candidates": [
        {
            "election": "2016 Presidential Election",
            "fullName": "Darrell Castle",
            "politicalParty": "Constitution Party"
        },
        {
            "election": "2016 Presidential Election",
            "fullName": "Hillary Clinton",
            "politicalParty": "Democratic Party"
        },
        {
            "election": "2016 Presidential Election",
            "fullName": "Gary Johnson",
            "politicalParty": "Libertarian Party"
        }
    ]
}
```

```bash
http http://localhost:8099/voter/results/2016%20Presidential%20Election
```

```json
{
    "results": [
        {
            "candidate": "Darrell Castle",
            "votes": 19
        },
        {
            "candidate": "Donald Trump",
            "votes": 15
        },
        {
            "candidate": "Gary Johnson",
            "votes": 15
        },
        {
            "candidate": "Jill Stein",
            "votes": 13
        }
    ]
}
```

```bash
http http://localhost:8099/voter/results/2016%20Presidential%20Election/votes
```

```json
{
    "votes": 80
}
```

```bash
http http://localhost:8099/voter/winners/2016%20Presidential%20Election
```

```json
{
    "results": [
        {
            "candidate": "Darrell Castle",
            "votes": 19
        }
    ]
}
```

```bash
http http://localhost:8099/voter/winners/2016%20Presidential%20Election/votes
```

```json
{
    "votes": 19
}
```

```bash
http POST http://localhost:8099/voter/votes \
    candidate="Jill Stein" \
    election="2016 Presidential Election"
```

```json
{
    "_links": {
        "self": {
            "href": "http://localhost:8099/voter/votes/590548541b8ebf700f9c2a62"
        },
        "candidate": {
            "href": "http://localhost:8099/voter/votes/590548541b8ebf700f9c2a62"
        }
    },
    "candidate": "Jill Stein",
    "election": "2016 Presidential Election"
}
```
