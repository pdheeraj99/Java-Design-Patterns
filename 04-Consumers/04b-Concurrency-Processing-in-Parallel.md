# Spring Kafka: Zero to Hero - 04b: Oke Debba ki Rendu Pittalu! (Concurrency) 🚀

Namaste mawa! Mana listener ippudu prashanthanga, oke thread tho pani chestundi. Kani, real-world lo traffic anedi tsunami 🌊 laaga vachi padutundi. Appudu mana single-threaded listener em avthundi? Slow aipothundi, messages queue lo undipothai.

Ee problem ni solve cheyadaniki, manam multiple threads tho, oke saari multiple messages ni process cheyali. Deenine **Concurrency** antaru.

---

### Concurrency Ela Work Avthundi?

Kafka lo, concurrency anedi **Partitions** meeda aadharapadi untundi.
*   Oka topic ni manam multiple partitions ga divide cheyochu.
*   Oke consumer group lo, prathi consumer oka (or konni) partition(s) ki assign avthundi.
*   **Golden Rule**: Oka partition ni oke saari oke consumer matrame read cheyagaladu (within the same consumer group).

So, manam throughput (message processing speed) penchali ante, rendu panulu cheyali:
1.  Topic ki ekkuva partitions create cheyali.
2.  Mana application lo antha mandi consumers ni (threads ni) create cheyali.

Spring Kafka lo ee pani cheyadam chala easy. Manam just `ConcurrentKafkaListenerContainerFactory` lo `concurrency` property ni set chesthe chalu.

### Configuration Update (`KafkaConsumerConfig.java`) 🛠️

Mana listener container factory lo `setConcurrency()` method ni call cheddam.

```java
// In KafkaConsumerConfig.java

@Configuration
public class KafkaConsumerConfig {

    // ... consumerFactory() bean ...

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());

        // Ikkade magic undi! ✨
        factory.setConcurrency(3); // 3 threads create chey ani cheptunnam

        return factory;
    }
}
```
Ippudu, `my-first-topic` ki 3 (or more) partitions unte, Spring Kafka automatic ga 3 consumer threads ni start chestundi. Prathi thread velli oka partition ni theeskuni, daantlo unna messages ni process cheyadam start chestundi. Anthe, parallel processing achieved!

### Diagram: Concurrency in Action ⚡

```mermaid
graph TD
    subgraph Kafka Topic (3 Partitions)
        P1["Partition 0"]
        P2["Partition 1"]
        P3["Partition 2"]
    end

    subgraph "Spring App (Concurrency = 3)"
        direction LR
        C1["Consumer Thread 1"]
        C2["Consumer Thread 2"]
        C3["Consumer Thread 3"]
    end

    P1 -- "Reads from" --> C1
    P2 -- "Reads from" --> C2
    P3 -- "Reads from" --> C3
```
Ee diagram lo, manam 3 threads create chesam, and prathi thread velli oka separate partition ni process chestundi. Super fast! 🏎️

---

### 📝 Interview Point:

"**How do you achieve parallel message processing in a Spring Kafka consumer?**"
"Concurrency in Spring Kafka is achieved by configuring the listener container.
1.  First, the Kafka topic must have multiple partitions.
2.  Then, in the `ConcurrentKafkaListenerContainerFactory` bean, we set the `concurrency` property to a number greater than one (e.g., `factory.setConcurrency(3)`).
Spring will then create that many consumer threads. Kafka will automatically assign different partitions to each thread within the same consumer group, enabling parallel processing and significantly increasing throughput."

---

### Next Enti? (What's Next?)

Mawa, manam ippudu multiple threads tho messages ni process chestunnam. Kani, oka chinna question undi. Manam message ni process chesaka, "Ee message pani aipoindi, deenini malli naaku pampaku" ani Kafka ki ela cheptham?

Deenine **Offset Committing** or **Acknowledgement** antaru. Spring Kafka lo deeniki chala modes unnai (`AckMode`). Ee important concept gurinchi next section lo lothuga thelusukundam. Don't miss it! ✅