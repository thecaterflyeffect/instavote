Insta Voting App
=========

This is something I used to learn Docker and its various components. This is a slightly modified version of a voting app out there in the wild.
I have slightly modified it as I was learning to use it to deploy via: docker-compose, Nomad, ECS etc. 

If you find this and want to try it for your own learning, the docker-compose file is included below.

Getting started
---------------

Download [Docker](https://www.docker.com/products/overview). If you are on Mac or Windows, [Docker Compose](https://docs.docker.com/compose) will be automatically installed. On Linux, make sure you have the latest version of [Compose](https://docs.docker.com/compose/install/).

Run in this directory:
```
docker-compose up
```
The app will be running at [http://localhost:5000](http://localhost:5000), and the results will be at [http://localhost:5001](http://localhost:5001).

Architecture
-----

Instavote is a small app that allows realtime voting on X vs. X (eg. Cats vs Dogs).

It is comprised of:
Vote engine - Python Flask
Result engine - Node.js
Worker engine - .NET
Redis
PostgreSQL

You will need to pull the following images:

thecaterflyeffect/instavote-vote
thecaterflyeffect/instavote-result
thecaterflyeffect/instavote-worker

Redis and PostgreSQL will pull and build when you run the docker-compose.yml

NOTE - Some packages and apps use older versions of Python, Node.js etc. I used what was packaged as a base and did not build on latest versions as this was strictly a learning experience. This is educational in purpose only and not a production example. 

![Architecture diagram](architecture.png)

* A Python webapp which lets you vote between two options
* A Redis queue which collects new votes
* A .NET worker which consumes votes and stores them in…
* A Postgres database backed by a Docker volume
* A Node.js webapp which shows the results of the voting in real time

Note
----

The voting application only accepts one vote per client. It does not register votes if a vote has already been submitted from a client.

Docker Compose - save the text below into a file named docker-compose.yml and place it in your corresponding parent directory of your app.
----
version: "3"

services:
  vote:
    image: 'voting_vote'
    ports:
      - "5000:80"
    networks:
      - front-tier
      - back-tier

  result:
    image: 'voting_result'
    volumes:
      - result-data:/app
    ports:
      - "5001:80"
    networks:
      - front-tier
      - back-tier

  worker:
    image: 'voting_worker'
    networks:
      - back-tier

  redis:
    image: 'redis:3.2-alpine'
    ports: ["6379"]
    networks:
      - back-tier

  db:
    image: postgres:9.4
    volumes:
      - "db-data:/var/lib/postgresql/data"
    networks:
      - back-tier

volumes:
  db-data:
  voting-data:
  result-data:

networks:
  front-tier:
  back-tier:

