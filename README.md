# HTTP Event Processor

A lightweight TypeScript application for capturing and processing HTTP request/response data forwarded from HAProxy SPOE.

## Overview

HTTP Event Processor is designed to be the downstream collector for HTTP traffic intercepted by HAProxy's Stream Processing Offload Engine (SPOE). It provides a simple, scalable solution for:

- Capturing complete HTTP requests and responses
- Correlating requests with their corresponding responses
- Storing HTTP traffic for analysis, debugging, or audit purposes
- Providing API access to the captured events

## Flow

![HTTP Event Processor Flow](https://s3.us-east-1.amazonaws.com/assets.magj.dev/http-event-processor-flow.svg)

1. **The primary HTTP traffic flow**:

   - Client sends HTTP requests to the HAProxy sidecar
   - HAProxy forwards requests to the application
   - Application sends responses back through HAProxy to the client

2. **The event capture process**:

   - HAProxy sends request and response events to the SPOE agent
   - SPOE agent forwards copies to the HTTP Event Processor

3. **Inside the HTTP Event Processor**:
   - The `/http-events` API endpoint receives event data
   - The event correlation component matches requests with responses
   - The storage component persists the complete HTTP interactions
   - A cleanup process manages memory and storage
   - The `/events` API provides access to the stored data

This diagram effectively illustrates the non-blocking nature of the system, where the main HTTP traffic flows normally while copies are processed separately by the HTTP Event Processor.

## Features

- **Event Collection**: Receives HTTP events via a REST API endpoint
- **Event Correlation**: Matches request and response events using unique IDs
- **Persistent Storage**: Stores complete request/response pairs as JSON files
- **Memory Management**: Implements cleanup to prevent memory leaks
- **Monitoring**: Provides a status endpoint to check application health
- **API Access**: Offers endpoints to retrieve stored events
- **Kubernetes-Ready**: Includes Helm chart for easy deployment

## Requirements

- Kubernetes 1.16+
- Helm 3.0+
- Node.js 16+ (for local development)
- HAProxy 2.4+ with SPOE module enabled
- SPOE agent configured to forward events

## Installation

### Using Helm (Recommended)

1. Add the HTTP Event Processor Helm repository:

   ```bash
   helm repo add http-event-processor https://kayaman.github.io/http-event-processor/
   helm repo update
   ```

2. Install the chart:

   ```bash
   helm install event-processor http-event-processor/http-event-processor
   ```

3. Customize installation with your own values:
   ```bash
   helm install event-processor http-event-processor/http-event-processor -f my-values.yaml
   ```

### Configuration Options

The following table lists the configurable parameters of the HTTP Event Processor chart and their default values:

| Parameter                                    | Description                   | Default                |
| -------------------------------------------- | ----------------------------- | ---------------------- |
| `replicaCount`                               | Number of replicas            | `1`                    |
| `image.repository`                           | Image repository              | `http-event-processor` |
| `image.tag`                                  | Image tag                     | `latest`               |
| `image.pullPolicy`                           | Image pull policy             | `IfNotPresent`         |
| `app.port`                                   | Application port              | `8080`                 |
| `app.logLevel`                               | Logging level                 | `info`                 |
| `app.eventExpiryMinutes`                     | Time to keep events in memory | `30`                   |
| `app.cleanupIntervalMinutes`                 | Interval for memory cleanup   | `5`                    |
| `persistence.enabled`                        | Enable persistent storage     | `false`                |
| `persistence.size`                           | Size of persistent volume     | `1Gi`                  |
| `persistence.storageClass`                   | Storage class for PV          | `""`                   |
| `persistence.accessMode`                     | Access mode for PV            | `ReadWriteOnce`        |
| `autoscaling.enabled`                        | Enable autoscaling            | `false`                |
| `autoscaling.minReplicas`                    | Minimum number of replicas    | `1`                    |
| `autoscaling.maxReplicas`                    | Maximum number of replicas    | `10`                   |
| `autoscaling.targetCPUUtilizationPercentage` | Target CPU utilization        | `80`                   |

### Local Development

1. Clone the repository:

   ```bash
   git clone https://github.com/kayaman/http-event-processor.git
   cd http-event-processor
   ```

2. Install dependencies:

   ```bash
   npm install
   ```

3. Run in development mode:

   ```bash
   npm run dev
   ```

4. Build for production:
   ```bash
   npm run build
   npm start
   ```

## Integration with HAProxy SPOE

After deployment, the HTTP Event Processor will be available at:

```
http://<service-name>.<namespace>.svc.cluster.local:8080
```

Configure your SPOE agent to send events to the `/http-events` endpoint.

Example configuration in the SPOE agent:

```python
DOWNSTREAM_URL = "http://event-processor.default.svc.cluster.local:8080/http-events"

def send_to_downstream(data):
    requests.post(
        DOWNSTREAM_URL,
        json=data,
        headers={"Content-Type": "application/json"},
        timeout=5
    )
```

## API Endpoints

The following API endpoints are available:

- `POST /http-events` - Receives events from SPOE agent
- `GET /status` - Returns application status and metrics
- `GET /events` - Returns the latest captured events

## Accessing Event Data

Once events are captured, you can access them in several ways:

1. Through the `/events` API endpoint
2. By accessing log files directly (if using a persistent volume)
3. By implementing additional exporters (see Customization)

## Customization

The HTTP Event Processor can be extended in several ways:

1. Add custom processing logic in the `/http-events` endpoint
2. Implement additional exporters (to databases, log aggregators, etc.)
3. Add authentication mechanisms for API security
4. Enhance the event correlation logic for complex scenarios

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
