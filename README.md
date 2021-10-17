# Go Project Overview

## Backend Arcitecture

I usually apply [Clean Architecture](http://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) for any project including my Go projects.
The main reason why use this architecture is to achieve [the separation of concerns](https://deviq.com/principles/separation-of-concerns). The general idea is that we should separate different concerns (e.g., Domain, UI, Application and so on) into its corresponding layer to establish well-organized systems. It has the following benefits:

- __testability__: easy to test each component/concern
- __independence__: easy to replace a component with a new one including external/internal dependencies (e.g., DB, Web Framework, any external API)
- __decouping__: reduce regression errors if one of the component/concern need to be changed. 

There is important rule to accomplish the above benefits, which is called the Depednency Rule. The rule is pretty simple. The components (e.g., classes/structs) in the higher layer can use or have dependencies of interfaces in the lower layer, not vice versa. For instance, a component in the Infrastructure layer can have dependencies of components in the Application layer, but components in the Domain layer cannot have dependencies of the Application layer. 

I think that this rule strongly contributes to the independence of components/layers. For example, your team decided to use RDBMS such as MySQL for your project initially. After while you release the project, you need to scale up the system for some reason. Then, you team decided to use NoSQL to take an advantage of the scalability. If you apply the Clean Architecture, you can easily replace RDBMS with NoSQL.     

## Infra Architecture

## Main Dependencies

- [__Go Module__](https://go.dev/blog/using-go-modules): the main package management system. 

- [__wire__](https://github.com/google/wire): (v0.4.0) the dependency management system.

- [__gin__](https://github.com/gin-gonic/gin): (v1.6.3) a web framework for Go.

- [__gocron__](https://github.com/go-co-op/gocron): (v0.5.0) a scheduled task manager.

- [__gorm__](https://github.com/go-gorm/gorm): (v1.9.12) an ORM.

- [__godotenv__](https://github.com/joho/godotenv): (v1.3.0) to read env files. 

- [__gin-jwt__](https://github.com/dgrijalva/jwt-go): (v2.6.4) an extension libary to handle jwt with the gin framework. 

- [__cors__](https://github.com/rs/cors): (v1.7.0) to allow to communicate SPA and API esp different origins.

- [__aws-sdk-go__](https://github.com/aws/aws-sdk-go): (v1.31.0) to communicate with AWS services
 
- [__testify__](https://github.com/stretchr/testify): (v1.5.1) unit/integration testing library

## Features

### AWS API Integration

### RESTFul API 

### Common Flow (How to handle request at an endpoint)

## Security

### JWT & Double Cookie Submission

## Code Reviews

I usually follow [the coding guideline](https://github.com/golang/go/wiki/CodeReviewComments) 

1. use gofmt (go formatter package).

