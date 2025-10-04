# Spring Kafka: Zero to Hero - 04f: Naaku Ee Message Oddu! (Filtering Messages) 🗑️

Namaste mawa! Manam ippudu chala messages theeskuntunnam. Kani, anni messages manaki avasaram undakapovachu. Konni messages ni manam "Ee message naaku oddu, ignore chey" ani cheppali anukuntam.

**Scenario**: Oka topic lo chala rakala events vastunnayi, kani mana specific listener kevalam konni events ni matrame process cheyali. For example, message lo "IMPORTANT" ane word unte ne process cheyali, lekapothe vadileyali.

Ee filtering ni manam listener method lo `if` condition petti cheyochu, kani adi clean approach kaadu. Spring Kafka manaki inko "gatekeeper" lanti mechanism istundi: **`RecordFilterStrategy`**.

---

### `RecordFilterStrategy`: The Bouncer at the Club Door 💂

Ee strategy oka bouncer laaga pani chestundi. Prathi message, mana listener method loki velle mundu, ee bouncer daggara aagi, check cheskuntundi.
*   **How it works?**: Manam oka custom filter logic rastam. Aa logic `true` return chesthe, bouncer aa message ni "Nuvvu lopaliki vellodu" ani cheppi, discard chestadu. `false` return chesthe, "Okay, nuvvu vellu" ani lopaliki pampistadu.
*   **Advantage**: Mana listener method chala clean ga untundi. Adi kevalam process cheyalsina messages gurinche alochistundi. Filtering logic antha separate ga untundi.

---

### How to Implement a Filter Strategy?

**Step 1: Create a Custom Filter Strategy Bean**
Manam `RecordFilterStrategy` interface ni implement chesi, mana custom logic `filter()` method lo rastam.

```java
package com.example.filter;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.springframework.kafka.listener.RecordFilterStrategy;
import org.springframework.stereotype.Component;

@Component
public class MyFilterStrategy implements RecordFilterStrategy<String, String> {

    @Override
    public boolean filter(ConsumerRecord<String, String> consumerRecord) {
        // Message value lo "ignore" ane word unte, filter chey (discard)
        boolean shouldDiscard = consumerRecord.value().contains("ignore");
        if (shouldDiscard) {
            System.out.println("#### -> Filtering out message: " + consumerRecord.value());
        }
        return shouldDiscard;
    }
}
```

**Step 2: Configure the Container Factory**
Ippudu, ee filter ni mana listener container factory ki cheppali.

```java
// In KafkaConsumerConfig.java

@Configuration
public class KafkaConsumerConfig {

    // ... consumerFactory() bean ...

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory(
            ConsumerFactory<String, String> consumerFactory,
            RecordFilterStrategy<String, String> myFilterStrategy) { // Mana filter ni inject chesko

        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory);

        // Factory ki filter strategy ni set chey
        factory.setRecordFilterStrategy(myFilterStrategy);

        // Optional but important: Discard chesina messages ni kuda acknowledge chey
        // Lekapothe, rebalance ayinappudu avi malli vastai
        factory.setAckDiscarded(true);

        return factory;
    }
}
```
Anthe! Ippudu mana listener ki "ignore" ane word unna message raadu.

### Diagram: The Filtering Flow 🛡️

```mermaid
graph TD
    A((Kafka Topic)) -- "Message" --> B{Listener Container};
    B -- "Passes to" --> C{RecordFilterStrategy};
    C -- "filter() returns true" --> D[Message Discarded 🗑️];
    C -- "filter() returns false" --> E[@KafkaListener Method ✅];
```

---

### 📝 Interview Point:

"**How can you prevent certain messages from being processed by a `@KafkaListener` without adding filtering logic inside the listener method itself?**"
"Spring Kafka provides a `RecordFilterStrategy` for this purpose. We can create a bean that implements this strategy and define our custom filtering logic in the `filter` method. This strategy is then set on the `ConcurrentKafkaListenerContainerFactory`. The container uses this strategy to evaluate each message after it's polled but before it's passed to the listener. If the strategy returns `true`, the message is discarded. It's also a good practice to set `ackDiscarded` to `true` on the factory to ensure the offsets for these discarded messages are still committed."

---

### Next Enti? (What's Next?)

Mawa, ippudu manam messages ni filter cheyadam nerchukunnam. Kani, oka vela manam oka message ni process chesi, daani result ni inko topic ki pampali anukunte? Ee "Request-Reply" pattern ni Spring Kafka lo ela implement cheyalo chala mandiki theliyadu.

Next section lo, manam `@SendTo` annotation tho ee pattern ni ela easy ga achieve cheyalo chuddam. Get ready for some cool stuff! 🚀➡️💌