# Spring Kafka: Zero to Hero - 04a: The Power of @KafkaListener 🦸‍♀️

Namaste mawa! Producer side antha chusam. Ippudu asalu magic chuddam. Piles of code rayakunda, just okke okka annotation tho messages ni ela receive cheskovacho chuddam.

Aa super-power annotation pere **`@KafkaListener`**.

---

### `@KafkaListener`: The 24/7 Watcher 🕵️‍♀️

Ee annotation oka method meeda pedithe, Spring Kafka aa method ni oka dedicated "watcher" ga convert chestundi. Aa watcher anukshanam (24/7) manam cheppina topic ni gamanistu untundi. Topic loki kotha message రాగానే, ee watcher daanini pattukuni, manam రాసిన method ki istundi.

**Advantages enti?**
*   **Zero Boilerplate**: Manam `consumer.poll()` lanti low-level code rayalsina avasaram ledu.
*   **Automatic**: Antha automatic ga jaruguthundi.
*   **Simple**: Just oka annotation, anthe!

### Basic Usage: A Simple Listener 🎧

Ee code ni copy chesi, mee project lo `service` package lo `MessageConsumerService.java` ane file lo pettuko.

```java
package com.example.service;

import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;

@Service
public class MessageConsumerService {

    @KafkaListener(topics = "my-first-topic", groupId = "my-group-id")
    public void listen(String message) {
        System.out.println("#### -> Consumed message -> " + message);
    }
}
```

**Code lo em undi?**
*   `@KafkaListener`: Ee annotation tho ne asalu magic start avthundi.
*   `topics = "my-first-topic"`: Ee listener ఏ topic ni vinali anedi ikkada cheptam. Multiple topics ni kuda ivvochu, like `topics = {"topic1", "topic2"}`.
*   `groupId = "my-group-id"`: Ee listener ఏ consumer group ki chendinaది anedi ikkada cheptam. Idi chala important, manam deeni gurinchi mundu nerchukunnam.
*   `public void listen(String message)`: Topic lo message రాగానే, aa message ee method ki parameter ga vastundi, and mana code execute avthundi.

Anthe mawa! Intha simple ga manam Kafka consumer ni create chesam.

---

### Diagram: The Listener Flow 👂

```mermaid
graph TD
    A((Kafka Topic <br/> 'my-first-topic')) -- "New Message Arrives" --> B{Message Listener Container};
    B -- "Invokes the method" --> C[listen(message)];

    subgraph Spring Application
        B
        C
    end

    style B fill:#ccf,stroke:#333,stroke-width:2px
    style C fill:#cfc,stroke:#333,stroke-width:2px
```

---

### 📝 Interview Point:

"**How do you consume messages from a Kafka topic in a Spring application?**"
"The simplest and most common way is by using the `@KafkaListener` annotation on a method. This annotation designates the method as a Kafka message listener. We configure it with the `topics` to subscribe to and a `groupId`. Spring Kafka's listener container infrastructure handles all the low-level details of polling the consumer and invoking the listener method when a message arrives."

---

### Next Enti? (What's Next?)

Okay, manam ippudu oke saari oka message ni process chestunnam. Kani, manaki high traffic unte? Oke saari vanda messages vasthe? Appudu mana application slow avthundi.

Ee problem ni solve cheyadaniki, manam **Concurrency** ni use chestam. Ante, oke saari multiple threads tho messages ni parallel ga process cheyadam. Ee concept gurinchi next section lo lothuga thelusukundam! 🚀