# Prometheus and Grafana for Spring Boot Application

1. Add below dependencies in pom.xml
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

2. Add below if using Spring Boot Security.
```
.requestMatchers("/actuator/prometheus").permitAll()
```

3. Add below in application.properties

```
# For Prometheus Monitoring
management.endpoints.web.exposure.include=health,info,prometheus
management.endpoint.prometheus.enabled=true
management.server.port=8080
```

4. Access metrics at:
```
http://localhost:8081/actuator/prometheus
```

5. Grafana Dashboard link:
```
https://grafana.com/grafana/dashboards/19004-spring-boot-statistics/
```