# Prometheus and Grafana for Node Application

## 1. Add below in server.js after `const app = express();` and before `app.use(helmet()));` if present.
```
// Prometheus monitoring setup starts here
import client from 'prom-client';

// Create a Registry
const register = new client.Registry();
client.collectDefaultMetrics({ register });

// Custom metrics
const httpRequestDurationMicroseconds = new client.Histogram({
  name: 'http_request_duration_ms',
  help: 'Duration of HTTP requests in ms',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [50, 100, 200, 500, 1000, 2000],
});

const errorCounter = new client.Counter({
  name: 'http_requests_errors_total',
  help: 'Total number of failed HTTP requests',
  labelNames: ['method', 'route', 'status_code'],
});

// Middleware to track request duration and errors
app.use((req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;
    httpRequestDurationMicroseconds
      .labels(req.method, req.path, res.statusCode)
      .observe(duration);

    if (res.statusCode >= 400) {
      errorCounter.labels(req.method, req.path, res.statusCode).inc();
    }
  });

  next();
});

// Expose metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});

//  Prometheus monitoring setup ends here
```

## 2. Install below dependency:
```
npm install prom-client
```

## 3. Access metrics at:
```
http://localhost:5000/metrics
```

## 4. Grafana Dashboard link:
```
https://grafana.com/grafana/dashboards/11159-nodejs-application-dashboard/
```