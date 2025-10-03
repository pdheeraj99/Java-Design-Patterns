# Spring Kafka: Zero to Hero - 03c: Message ki Metadata Add Cheddam (Kafka Headers) 🏷️

Namaste mawa! Manam ippudu message payload (ante asalu content) tho ela pani cheyalo chusam. Kani konni sarlu, manaki message tho paatu konchem extra information pampali anipistundi. Ee extra information ni **Headers** lo pedatam.

Headers anevi key-value pairs anamata, kani ivi message content lo kalavavu. Separate ga untai.

### Enduku vaadali ee headers? 🤔
*   **Tracing**: Oka message ఏ service nunchi vachindi, daani trace ID enti lanti information pettukovachu.
*   **Routing**: Header lo unna value batti, aa message ni ela process cheyalo decide avvochu.
*   **Versioning**: Message format version ento header lo pampochu.

---

### Headers Ela Pampaali? (Sending Headers) 📤

`KafkaTemplate` tho message pampetappudu headers add cheyadam chala easy. Manam `ProducerRecord` object ni create chesi, daaniki headers ni add cheyali.

**Producer Service Update (`MessageProducerService.java`):**

```java
package com.example.service;

import org.apache.kafka.clients.producer.ProducerRecord;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;

@Service
public class MessageProducerService {

    private static final String TOPIC = "my-first-topic";

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    public void sendMessageWithHeaders(String message, String traceId) {
        System.out.println(String.format("#### -> Producing message with traceId %s -> %s", traceId, message));

        // 1. Create a ProducerRecord
        ProducerRecord<String, String> record = new ProducerRecord<>(TOPIC, message);

        // 2. Add headers
        record.headers().add("X-Trace-Id", traceId.getBytes());
        record.headers().add("X-Source-System", "OrderService".getBytes());

        // 3. Send the record
        this.kafkaTemplate.send(record);
    }
}
```
Ikkada `X-Trace-Id` and `X-Source-System` anevi manam pampina extra information (metadata). Note chesko, header values `byte[]` format lo pampaali.

---

### Diagram: Headers Flow 🏷️➡️📬

```mermaid
graph TD
    subgraph Producer App
        A[Create Message] --> B{Create ProducerRecord};
        B --> C[Add 'X-Trace-Id' Header];
        C --> D[kafkaTemplate.send(record)];
    end

    subgraph Consumer App
        G[Receive Message] --> H{Extract Headers};
        H --> I[Process Payload];
        H --> J[Use Header value (traceId)];
    end

    D -- "Message with Headers" --> Kafka[Kafka Topic];
    Kafka -- " " --> G;
```

---

### 📝 Interview Point:

"**How can you pass metadata along with a Kafka message without altering the payload?**"
"The best way is by using **Kafka Headers**. We can add custom headers to the `ProducerRecord` before sending it with `KafkaTemplate`. On the consumer side, we can access these headers in our `@KafkaListener` method by using the `@Header` annotation. This is a clean way to pass cross-cutting concerns like trace IDs or source system information."

---

### Next Enti? (What's Next?)

Producer side antha cover chesam mawa! 💪 Manam messages pampadam lo pro aipoyam.

Ippudu coin ki maro vaipu chuddam. Asalu ee pampina messages ni ela receive cheskovali? `@KafkaListener` antha powerful enduku? Ee vishayalu anni next section lo, **The Art of Consuming Messages**, lothuga thelusukundam. See you there! 🎧