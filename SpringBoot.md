# Prometheus and Grafana for Spring Boot Application

## 1. Add below dependencies in pom.xml
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

 - Spring Boot Actuator: Provides endpoints for monitoring and management.

 - Micrometer: Acts as a metrics collection facade.

 - Micrometer Registry Prometheus: Exposes metrics in a format Prometheus can scrape.

## 2. Add below if using Spring Boot Security.
```
.requestMatchers("/actuator/prometheus").permitAll()
```

## 3. Add below in application.properties

Allow public access to the Prometheus endpoint:
```
# For Prometheus Monitoring
# Expose necessary actuator endpoints
management.endpoints.web.exposure.include=health,info,prometheus,loggers,metrics,env

# Enable Prometheus endpoint
management.endpoint.prometheus.enabled=true

# Set server port
management.server.port=8080

# Show detailed health info
management.endpoint.health.show-details=always
```
## 4. Configure Prometheus `scarap_configs`
Example:
```
scrape_configs:
  - job_name: 'spring-boot-app'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['host.docker.internal:8080']
```

## 5. Access metrics at:
```
http://localhost:8081/actuator/prometheus
```

## 6. Grafana Dashboard link:
```
https://grafana.com/grafana/dashboards/19004-spring-boot-statistics/
```

## 7. Workflow
```
Spring Boot App (Actuator + Micrometer)
              ↓
         /actuator/prometheus
              ↓
          Prometheus
              ↓
           Grafana
```
