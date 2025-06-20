---
parent: Decisions
nav_order: 14
title: 0014 Messaging
tags:
  - messaging
  - cloudevents
  - amqp
  - mqtt
  - http
  - event-driven
  - distributed-systems
  - python
---
# Messaging

Date: 2025-06-20

## Status

Accepted

## Context

Modern applications often require communication between distributed components or services. Effective messaging is critical for building resilient, loosely-coupled systems. We need to establish standards for message formats and transport protocols to ensure consistent, reliable, and interoperable messaging across our applications.

The primary challenges that our messaging strategy addresses include:

1. **Consistency**: Using a standardized message format across different systems
2. **Interoperability**: Enabling communication between diverse services and platforms
3. **Flexibility**: Supporting multiple transport protocols without changing message structure
4. **Reliability**: Ensuring messages are delivered even during network disruptions
5. **Scalability**: Handling increasing message volumes as applications grow

## Decision

We will adopt CloudEvents as our standard message format specification and support multiple transport protocols, with HTTP, AMQP, and MQTT as our primary choices.

### Message Format: CloudEvents

We will use [CloudEvents](https://github.com/cloudevents/spec/blob/main/cloudevents/spec.md) (currently v1.0) as our standard message format. CloudEvents is a CNCF project that provides a specification for describing event data in a common way, making it easier to route, filter, and deliver events across environments and between services.

#### Key Benefits of CloudEvents

- **Standardized Structure**: Common format regardless of source or destination
- **Transport Independence**: Works across different protocols (HTTP, AMQP, MQTT, Kafka, etc.)
- **Language Agnostic**: Implementations available in multiple languages
- **Extensible**: Supports custom attributes and data formats
- **Well-documented**: Clear specification with growing industry adoption

#### CloudEvents Structure

Each message will follow the CloudEvents specification with the following required attributes:

```json
{
  "specversion": "1.0",
  "id": "unique-event-id",
  "source": "service-name/component",
  "type": "com.example.domain.event.type",
  "time": "2025-06-20T12:00:00Z",
  "datacontenttype": "application/json",
  "data": {
    "key": "event-specific-data"
  }
}
```

For Python applications, we will use the official CloudEvents SDK:

```python
from cloudevents.http import CloudEvent
from cloudevents.conversion import to_structured
import json
import uuid
from datetime import datetime, timezone

def create_cloud_event(event_type, source, data):
    attributes = {
        "specversion": "1.0",
        "id": str(uuid.uuid4()),
        "source": source,
        "type": event_type,
        "time": datetime.now(timezone.utc).isoformat(),
        "datacontenttype": "application/json"
    }
    
    event = CloudEvent(attributes, data)
    return event

# Create a CloudEvent
event = create_cloud_event(
    event_type="com.example.user.created",
    source="/services/user-service",
    data={"user_id": "12345", "email": "user@example.com"}
)

# Convert to structured mode (JSON) for HTTP transport
headers, body = to_structured(event)
print(json.dumps(json.loads(body), indent=2))
```

#### Event Type Naming Convention

We will use reverse DNS notation for event types, following this structure:
`com.{company}.{domain}.{action}.{state}`

Examples:
- `com.example.user.profile.updated`
- `com.example.order.payment.completed`
- `com.example.inventory.item.depleted`

### Transport Protocols

We will support multiple transport protocols for different use cases, with the following primary choices:

#### 1. HTTP

HTTP transport is used for synchronous communication, webhooks, and simple integrations:

```python
import requests
from cloudevents.http import CloudEvent
from cloudevents.conversion import to_structured

def send_cloudevent_http(event, endpoint):
    headers, body = to_structured(event)
    response = requests.post(endpoint, headers=headers, data=body)
    return response

# Using httpx for async operations
import httpx
async def send_cloudevent_http_async(event, endpoint):
    headers, body = to_structured(event)
    async with httpx.AsyncClient() as client:
        response = await client.post(endpoint, headers=headers, data=body)
    return response
```

#### 2. AMQP (RabbitMQ)

AMQP is our preferred protocol for reliable message queuing with RabbitMQ as our primary implementation:

```python
import pika
import json
from cloudevents.conversion import to_structured

def send_cloudevent_rabbitmq(event, exchange, routing_key):
    _, body = to_structured(event)
    
    # CloudEvents attributes as message headers
    headers = {k: v for k, v in event.items() if k != "data"}
    
    connection = pika.BlockingConnection(
        pika.ConnectionParameters(host='localhost')
    )
    channel = connection.channel()
    
    # Ensure exchange exists
    channel.exchange_declare(exchange=exchange, exchange_type='topic', durable=True)
    
    # Publish message
    channel.basic_publish(
        exchange=exchange,
        routing_key=routing_key,
        body=body,
        properties=pika.BasicProperties(
            delivery_mode=2,  # make message persistent
            headers=headers,
            content_type='application/cloudevents+json'
        )
    )
    
    connection.close()
```

#### 3. MQTT

MQTT is used for IoT devices and lightweight messaging scenarios:

```python
import paho.mqtt.client as mqtt
import json
from cloudevents.conversion import to_structured

def send_cloudevent_mqtt(event, topic, broker="localhost", port=1883):
    _, body = to_structured(event)
    
    client = mqtt.Client()
    client.connect(broker, port, 60)
    
    # Publish message
    result = client.publish(
        topic=topic,
        payload=body,
        qos=1,
        retain=False
    )
    
    client.disconnect()
    return result
```

### Message Exchange Patterns

We will support the following message exchange patterns:

#### 1. Request-Response (Synchronous)

For synchronous operations requiring immediate feedback:
- Primary transport: HTTP
- Use case: API calls, immediate data retrieval

#### 2. Publish-Subscribe (Asynchronous)

For broadcasting events to multiple consumers:
- Primary transport: AMQP (topic exchanges), MQTT
- Use case: Notifications, event broadcasts

#### 3. Queue-Based (Asynchronous)

For reliable work distribution:
- Primary transport: AMQP (queues)
- Use case: Task processing, work distribution

#### 4. Command/Event Sourcing

For complex business processes:
- Primary transport: AMQP with Dead Letter Exchanges
- Use case: Business workflows, transaction processing

### Message Reliability Considerations

For ensuring message reliability, we implement:

1. **At-least-once delivery**: Using acknowledgments in AMQP and QoS in MQTT
2. **Dead letter queues**: For handling failed message processing
3. **Message persistence**: Storing messages to disk before processing
4. **Message idempotency**: Ensuring operations can be repeated safely
5. **Retries**: Implementing retry logic for transient failures
6. **Publish confirmation**: Using publisher confirms in AMQP to ensure messages are sent successfully
7. **Monitoring and alerting**: Using tools like Prometheus and Grafana to monitor message queues and transport health

### Authentication and Security

All message transport will be secured:

1. **TLS/SSL**: For transport-level encryption
2. **Authentication**: Using mutual TLS, API keys, or tokens depending on transport
3. **Authorization**: Using AMQP virtual hosts, MQTT ACLs, or API gateways

### Deployment Considerations

In containerized environments:

1. **Message brokers**: Deployed as separate services in Kubernetes
2. **High availability**: Multi-node clusters for RabbitMQ
3. **Service discovery**: Using Kubernetes service discovery for broker endpoints
4. **Managed services**: Prefer GCP Pub/Sub, AWS SNS/SQS, or Cloud IoT Core for managed options

## Implementation Examples

### Example 1: User Service Emitting Events

```python
# services/user_service.py
from cloudevents.http import CloudEvent
import uuid
from datetime import datetime, timezone

class UserService:
    def __init__(self, message_publisher):
        self.message_publisher = message_publisher
        self.service_name = "/services/user-service"
    
    def create_user(self, user_data):
        # Create user in database
        user_id = self._persist_user(user_data)
        
        # Emit CloudEvent
        attributes = {
            "specversion": "1.0",
            "id": str(uuid.uuid4()),
            "source": self.service_name,
            "type": "com.example.user.created",
            "time": datetime.now(timezone.utc).isoformat(),
            "datacontenttype": "application/json"
        }
        
        data = {
            "user_id": user_id,
            "email": user_data.email,
            "username": user_data.username,
            "created_at": datetime.now(timezone.utc).isoformat()
        }
        
        event = CloudEvent(attributes, data)
        self.message_publisher.publish(event, "user-events", "user.created")
        
        return user_id
    
    def _persist_user(self, user_data):
        # Database operation to save user
        return str(uuid.uuid4())  # Simplified example
```

### Example 2: Message Consumer

```python
# workers/notification_worker.py
from cloudevents.http import from_http
import pika
import json

class NotificationWorker:
    def __init__(self, email_service):
        self.email_service = email_service
    
    def start_consuming(self):
        connection = pika.BlockingConnection(
            pika.ConnectionParameters(host='localhost')
        )
        channel = connection.channel()
        
        # Declare exchange and queue
        channel.exchange_declare(exchange='user-events', exchange_type='topic', durable=True)
        result = channel.queue_declare(queue='', exclusive=True)
        queue_name = result.method.queue
        
        # Bind to specific events
        channel.queue_bind(exchange='user-events', queue=queue_name, routing_key='user.created')
        
        # Set up consumer
        channel.basic_consume(
            queue=queue_name,
            on_message_callback=self._process_message,
            auto_ack=False
        )
        
        print("Starting to consume messages...")
        channel.start_consuming()
    
    def _process_message(self, ch, method, properties, body):
        try:
            # Parse CloudEvent
            headers = properties.headers or {}
            headers['content-type'] = properties.content_type
            
            # Manually reconstruct CloudEvent
            event_data = json.loads(body)
            event_attrs = {}
            
            for key in headers:
                if key != 'data':
                    event_attrs[key] = headers[key]
            
            # Process based on event type
            if headers.get('type') == 'com.example.user.created':
                self._handle_user_created(event_data)
            
            # Acknowledge message
            ch.basic_ack(delivery_tag=method.delivery_tag)
        
        except Exception as e:
            print(f"Error processing message: {e}")
            # Negative acknowledgment - will be requeued
            ch.basic_nack(delivery_tag=method.delivery_tag, requeue=True)
    
    def _handle_user_created(self, event_data):
        # Send welcome email
        self.email_service.send_welcome_email(
            email=event_data.get('email'),
            username=event_data.get('username')
        )
```

## Consequences

### Positive

- **Standardized Messages**: CloudEvents provides a consistent format across all services
- **Protocol Flexibility**: Applications can change transport protocols without changing message structure
- **Loose Coupling**: Services communicate without direct dependencies
- **Scalability**: Asynchronous communication patterns enable better scaling
- **Interoperability**: CloudEvents enables integration with third-party systems

### Negative

- **Complexity**: Multiple transport protocols increase operational complexity
- **Learning Curve**: Developers need to understand CloudEvents specification
- **Debugging Challenges**: Asynchronous messaging can make troubleshooting more difficult
- **Infrastructure Requirements**: Requires maintaining message brokers

## Related Decisions

* [0005 Application Structure](./0005-application-structure.md)
* [0006 Application Lifecycle](./0006-application-lifecycle.md)
* [0013 Caching](./0013-caching.md)
* [0015 Async Processing](./0015-async-processing.md)
* [0017 API Design](./0017-api-design.md)
