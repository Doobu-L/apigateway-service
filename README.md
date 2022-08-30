# apigateway-service

### use Filter
1. yml 설정파일에 추가
spring:
    cloud:
      gateway:
        routes:
          filters:
            - AddRequestHeader=first-request, first-requests-header2
            - AddResponseHeader=first-response, first-response-header2
2. FilterConfig.java 에 RouteLocator를 반환하는 빈을 등록해서 사용.
```
@Configuration
public class FilterConfig {
  @Bean
  public RouteLocator gatewayRoutes(RouteLocatorBuilder builder){
    return builder.routes()
        .route(r -> r.path("/first-service/**")
                    .filters(f -> f.addRequestHeader("first-request","first-request-header")
                                    .addResponseHeader("first-response","first-response-header"))
                    .uri("http://localhost:8001"))
        .route(r -> r.path("/second-service/**")
            .filters(f -> f.addRequestHeader("second-request","second-request-header")
                .addResponseHeader("second-response","second-response-header"))
            .uri("http://localhost:8002"))
        .build();
  }
}
```
3. CustomFilter 를 등록해서 사용. (yml에 등록)
```
@Component
@Slf4j
public class CustomFilter extends AbstractGatewayFilterFactory<CustomFilter.Config> {

  public CustomFilter(){
    super(Config.class);
  }

  @Override
  public GatewayFilter apply(Config config) {

    return (exchange, chain) -> {
      ServerHttpRequest request = exchange.getRequest();
      ServerHttpResponse response = exchange.getResponse();

      log.info("Custom PRE filter: request id ->{}",request.getId());

      // Custom Post Filter
      return chain.filter(exchange).then(Mono.fromRunnable(() -> {
        log.info("Custom POST filter: response id ->{}",response.getStatusCode());
      }));
    };
  }

  public static class Config{
    // Put the configuration properties
  }
}
```

4. GlobalFilter.java 생성하고 
  - yml에서 아래와 같은 방식으로 default-filter 등록. 
  spring:
    cloud:
      gateway:
        default-filter:
          -name: GlobalFilter
           args:
            baseMessage: Spring ~~~~~
            preLogger: true
            postLogger: true
            

**Eureka Server**

DiscoveryServer

**Eureka Client**

SpringCloudGateway , first-service , second-service

**LoadBalance**

SpringCloudGateway 에 aplication.yml - eureka client에 

url : lb://{service-name}

를 통해 {service-name} 으로 로드밸런싱을 한다.




## 주문서비스 구성  ##
![MSA_ORDER_SERVICE](https://user-images.githubusercontent.com/60733417/187461359-3fa41f77-257c-44dd-8083-afb8a09ea863.PNG)
 
