# Goal

Confirm the ability to route to a different upstream provided based on a dynamic criteria. For this exercise all 
components will be setup locally.

## Setup

### Eureka

Zuul 2.0 recommends Eureka as the service discovery component and that is what we will use for the sake of this exercise.

Checkout the [Netflix/eureka/MAG-world](https://github.com/sandy-adi/eureka/tree/MAG-world) branch. Run Eureka following
the instructions on the eureka wiki

- [Building Eureka Server war](https://github.com/Netflix/eureka/wiki/Building-Eureka-Client-and-Server)
- [Deploying Eureka War](https://github.com/Netflix/eureka/wiki/Running-the-Demo-Application)

### Setting up sample services

I wrote a simple node js app [pass-through-proxy](https://github.com/sandy-adi/pass-thru-proxy) to be able to route traffic
to a service like mockbin and httpbin. For our example we run two instances as below:

```
$ my_port=3000 bin_url=http://httpbin.org npm run start

$ my_port=3001 bin_url=http://mockbin.org npm run start
``` 

#### Register the mock apps with Eureka

Replace `myHostName` with the actual hostname and `my_ip_address` with the actual ip address of your machine


Register Mockbin Service
```
curl -X POST \
  http://myHostName.local:8080/eureka/v2/apps/app_mockbin_local \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
    "instance": {
        "instanceId": "node-mock-bin",
        "hostName": "myHostName.local",
        "app": "app_mockbin_local",
        "ipAddr": "<my_ip_address>",
        "vipAddress": "vip_mockbin_local",
        "status": "UP",
        "overriddenStatus": "UNKNOWN",
        "port": {
            "$": 3001,
            "@enabled": "true"
        },
        "securePort": {
            "$": 443,
            "@enabled": "false"
        },
        "countryId": 1,
        "dataCenterInfo": {
            "@class": "com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo",
            "name": "MyOwn"
        },
        "leaseInfo": {
            "renewalIntervalInSecs": 30,
            "durationInSecs": 90,
            "evictionTimestamp": 0
        },
        "metadata": {
            "@class": "java.util.Collections$EmptyMap"
        },
        "appGroupName": "api-gateway",
        "homePageUrl": "http://myHostName.local:3001/status/200",
        "statusPageUrl": "http://myHostName.local:3001/status/200",
        "healthCheckUrl": "http://myHostName.local:3001/status/200",
        "isCoordinatingDiscoveryServer": "false"
    }
}'
```

Register Httpbin service
```
curl -X POST \
  http://myHostName.local:8080/eureka/v2/apps/app_httpbin_local \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
    "instance": {
        "instanceId": "node-http-bin",
        "hostName": "myHostName.local",
        "app": "app_httpbin_local",
        "ipAddr": "<my_ip_address>",
        "vipAddress": "vip_httpbin_local",
        "status": "UP",
        "overriddenStatus": "UNKNOWN",
        "port": {
            "$": 3000,
            "@enabled": "true"
        },
        "securePort": {
            "$": 443,
            "@enabled": "false"
        },
        "countryId": 1,
        "dataCenterInfo": {
            "@class": "com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo",
            "name": "MyOwn"
        },
        "leaseInfo": {
            "renewalIntervalInSecs": 30,
            "durationInSecs": 90,
            "evictionTimestamp": 0
        },
        "metadata": {
            "@class": "java.util.Collections$EmptyMap"
        },
        "appGroupName": "api-gateway",
        "homePageUrl": "http://myHostName.local:3000/get",
        "statusPageUrl": "http://myHostName.local:3000/status/200",
        "healthCheckUrl": "http://myHostName.local:3000/status/200",
        "isCoordinatingDiscoveryServer": "false"
    }
}'
```

### Zuul Sample App

- Checkout [Netflix/zuul/chitragupta](https://github.com/sandy-adi/zuul/tree/chitragupta) branch.
- `application.properties` changes to notice in the sample app
   - ```
        eureka.shouldUseDns=false
        eureka.eurekaServer.context=eureka/v2
        eureka.eurekaServer.domainName=myHostName.local
        eureka.eurekaServer.gzipContent=true
        eureka.serviceUrl.default=http://${eureka.eurekaServer.domainName}:8080/${eureka.eurekaServer.context}
     ```
     
   - ```
        mockbin.ribbon.NIWSServerListClassName=com.netflix.niws.loadbalancer.DiscoveryEnabledNIWSServerList
        mockbin.ribbon.DeploymentContextBasedVipAddresses=vip_mockbin_local
        httpbin.ribbon.NIWSServerListClassName=com.netflix.niws.loadbalancer.DiscoveryEnabledNIWSServerList
        httpbin.ribbon.DeploymentContextBasedVipAddresses=vip_httpbin_local
     ```
        
   - ```
        myMockBin=mockbin
        myHttpBin=httpbin
     ```

The last two properties this is just a way to manage a mapping between headers and the actual vip. `myMockBin` and 
`myHttpBin` will be the header values in the request which will then be mapped to the vip for routing.