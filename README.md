
[![Build Status](https://travis-ci.org/imrenagi/microservice-skeleton.svg?branch=master)](https://travis-ci.org/imrenagi/microservice-skeleton)  [![codecov](https://codecov.io/gh/imrenagi/microservice-skeleton/branch/master/graph/badge.svg)](https://codecov.io/gh/imrenagi/microservice-skeleton) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT) 

# BHYM: Mission Management System Based on a Service Oriented Architecture

BHYM is a mission management system that follows the Service Oriented Architecture (SOA) integration pattern using technologies like Mule ESB, Spring Boot, Spring Web Services, and Spring for GraphQL among many others. The system is intended to manage missions, mission requests, and mission reimbursement requests in a modular, decoupled, and efficient fashion.

The system is currently comprised of 9 services, each of which can be tested, built, and deployed independently. The system supports two kinds of communications between the existing services; synchronous via Mule's ESB, and asynchronous via Rabbit's message broker.

![Infrastructure plan](https://i.ibb.co/7vPFrWG/Copy-of-Mission-Management-System-Page-1-drawio-1.png)

