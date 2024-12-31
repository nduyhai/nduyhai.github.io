+++
date = '2024-12-30T21:23:48+07:00'
draft = false
title = 'Keycloak multiple realms authentication'
+++

## Introduction

The shared-authentication repository provides a set of tools and configurations to facilitate seamless integration of multiple OAuth2 authentication providers into your applications. This is particularly useful in microservices architectures where services may need to interact with various authentication mechanisms.

## Key Features

* Auto-Configuration: Automatically sets up multiple OAuth2 issuers, reducing manual configuration efforts.

* Spring Boot Integration: Includes Spring Boot starter modules for easy incorporation into Spring-based applications.

* Extensibility: Designed to be flexible, allowing customization to fit specific project requirements.

## Getting Started

To utilize the shared-authentication package in your project, follow these steps:


To utilize the shared-authentication package in your project, follow these steps:

1. Clone the Repository
   Begin by cloning the repository to your local machine:

```bash
git clone https://github.com/nduyhai/shared-authentication.git
```
2. Set Up Docker Environment
   The project includes a docker-compose.yml file to set up the necessary services. Ensure Docker is installed on your system, then navigate to the project directory and execute:
```bash
docker-compose up
```
This command will initialize the required services, including Keycloak, an open-source identity and access management solution.

3. Configure Keycloak
   After the Docker services are running, access the Keycloak admin console at http://localhost:8080/. Use the credentials specified in the docker-compose.yml file to log in.

Create Users: For each realm, create new users as needed. For example, you might create a user with the username user-2 and password 2 in the merchant-auth realm.

Obtain Access Tokens: Retrieve access tokens for the created users using the following curl command, replacing the placeholder values with actual data:

```bash
curl --location 'http://localhost:8080/realms/merchant-auth/protocol/openid-connect/token' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'client_id=admin-cli' \
--data-urlencode 'username=user-2' \
--data-urlencode 'password=2' \
--data-urlencode 'grant_type=password'
```
4. Integrate with Your Application
   Incorporate the shared-authentication modules into your Spring Boot application by including the relevant dependencies in your pom.xml file. Configure your application to recognize and utilize the multiple OAuth2 issuers as per your project requirements.

## Conclusion
The shared-authentication project simplifies the integration of multiple OAuth2 authentication providers in your applications, enhancing security and scalability. By following the setup instructions and leveraging the provided configurations, you can streamline your authentication processes and focus on developing core features.

For more detailed information and to access the source code, visit the shared-authentication GitHub repository.