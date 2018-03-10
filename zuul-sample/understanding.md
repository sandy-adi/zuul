# understanding the sample zuul app

## Entry Point

### Bootstrap

- Loads the default configuration from `application.properties`
- Environment specific properties can be overriden by `application-<env>.properties`


## Eureka

- When registering the ip address is mandatory, this might be a problem