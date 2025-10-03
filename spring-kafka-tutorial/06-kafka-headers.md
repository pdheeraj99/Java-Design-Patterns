# Spring Kafka Tutorial - 06: Message ki Metadata Add Cheddam (Kafka Headers) 🏷️

Hey mawa, manam ippudu message payload (ante asalu content) tho ela pani cheyalo chusam. Kani konni sarlu, manaki message tho paatu konchem extra information pampali anipistundi. Ee extra information ni **Headers** lo pedatam.

Headers anevi key-value pairs anamata, kani ivi message content lo kalavavu. Separate ga untai.

**Enduku vaadali ee headers? 🤔**
*   **Tracing**: Oka message ఏ service nunchi vachindi, daani trace ID enti lanti information pettukovachu.
*   **Routing**: Header lo unna value batti, aa message ni ela process cheyalo decide avvochu.
*   **Versioning**: Message format version ento header lo pampochu.

---

### Headers Ela Pampaali? (Sending Headers) 📤

`KafkaTemplate` tho message pampetappudu headers add cheyadam chala easy. Manam `ProducerRecord` object ni create chesi, daaniki headers ni add cheyali.

**Producer Service Update (`MessageProducerService.java`):**

```java
// In MessageProducerService.java
import org.apache.kafka.clients.producer.ProducerRecord;
import org.springframework.kafka.core.KafkaTemplate;
// ... other imports

@Service
public class MessageProducerService {

    private static final String TOPIC = "my-first-topic";

    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;

    public void sendUserWithHeaders(User user, String traceId) {
        System.out.println(String.format("#### -> Producing user with traceId %s -> %s", traceId, user.toString()));

        // 1. Create a ProducerRecord
        ProducerRecord<String, Object> record = new ProducerRecord<>(TOPIC, user);

        // 2. Add headers
        record.headers().add("X-Trace-Id", traceId.getBytes());
        record.headers().add("X-Source-System", "OrderService".getBytes());

        // 3. Send the record
        this.kafkaTemplate.send(record);
    }
}
```
Ikkada `traceId` and `sourceSystem` anevi manam pampina extra information (metadata).

---

### Headers Ela Chadavaali? (Reading Headers) 📥

Listener side lo headers ni chadavaali ante, manam method signature lo `@Header` annotation ni use cheyali. Spring Kafka manaki chala built-in header names `KafkaHeaders` aney class lo istundi.

**Consumer Service Update (`MessageConsumerService.java`):**

```java
// In MessageConsumerService.java
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.kafka.support.KafkaHeaders;
// ... other imports

@Service
public class MessageConsumerService {

    @KafkaListener(topics = "my-first-topic", groupId = "my-group-id", containerFactory = "kafkaListenerContainerFactory")
    public void listenWithHeaders(User user,
                                  @Header(KafkaHeaders.RECEIVED_TOPIC) String topic,
                                  @Header("X-Trace-Id") String traceId) {

        System.out.println("#### -> Consumed user from topic: " + topic);
        System.out.println("#### -> Trace ID: " + traceId);
        System.out.println("#### -> User: " + user);
    }
}
```

**Code lo em undi?**
*   `@Header(KafkaHeaders.RECEIVED_TOPIC)`: Ee message ఏ topic nunchi vachindo theluskovadaniki.
*   `@Header("X-Trace-Id")`: Manam pampina custom header `X-Trace-Id` ni ikkada receive cheskuntunnam.

### Diagram: Headers Flow 🏷️➡️📬

```mermaid
graph TD
    subgraph Producer App
        A[Create User Object] --> B{Create ProducerRecord};
        B --> C[Add 'X-Trace-Id' Header];
        C --> D[kafkaTemplate.send(record)];
    end

    subgraph Consumer App
        G[Receive Message] --> H{Extract Headers};
        H --> I[Process Payload (User)];
        H --> J[Use Header value (traceId)];
    end

    D -- "Message with Headers" --> Kafka[Kafka Topic];
    Kafka -- " " --> G;
```

---

### 📝 Interview Point:

"**How can you pass metadata along with a Kafka message without altering the payload?**" ani adigithe, straight ga "Using Kafka Headers" ani cheppandi.
"We can add custom headers to the `ProducerRecord` before sending it with `KafkaTemplate`. On the consumer side, we can access these headers in our `@KafkaListener` method by using the `@Header` annotation. This is useful for passing cross-cutting concerns like trace IDs or source system information."

---

### The End... For Now! 🎉

Mawa! Ee series lo manam Spring Kafka gurinchi chala nerchukunnam.
*   Basics 🚀
*   Producer & Consumer setup 🛠️
*   JSON tho objects pampadam 📦
*   Error Handling & DLT 🤕
*   Headers tho metadata pampadam 🏷️

Ee concepts tho meeru oka solid foundation build cheskunnaru. You are now ready to tackle most of the real-world problems with Spring Kafka.

Of course, inka chala advanced topics unnai (transactions, streams, etc.), kani vaati gurinchi malli kaluddam. For now, go and build something amazing! All the best! 💪

**#HappyCoding** ✨