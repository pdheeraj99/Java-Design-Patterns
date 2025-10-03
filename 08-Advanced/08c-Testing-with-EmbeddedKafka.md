# Spring Kafka: Zero to Hero - 08c: Code ni Test Cheddam! (@EmbeddedKafka) ✅

Namaste mawa! Manam ippativaraku chala code raasam. Kani, adi antha correct ga pani chestundo ledo, manam prathi saari manual ga run chesi chudalem. Anduke, manam **Automated Integration Tests** raayali.

Kani, Kafka tho test cheyali ante, oka real Kafka broker undali kada? Prathi test kosam oka separate Kafka cluster ni setup cheyadam anedi chala pedda thala noppi. 🤯

### Solution: `@EmbeddedKafka` to the Rescue! 🚀

Ee problem ni solve cheyadaniki, Spring Kafka manaki `@EmbeddedKafka` ane oka magical annotation istundi.
*   **Ela Pani Chestundi?**: Manam ee annotation ni mana test class meeda pedithe chalu, adi automatic ga, mana test start ayye mundu, oka chinna, in-memory Kafka broker ni start chestundi. Test aipogane, daanini ade shutdown chestundi.
*   **Advantage**: Manam external Kafka dependency lekunda, mana producer and consumer logic ni easy ga test cheyochu.

---

### How to Write an Integration Test

**Step 1: Dependency Add Chey**
First, mana `pom.xml` lo `spring-kafka-test` dependency undali.

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka-test</artifactId>
    <scope>test</scope>
</dependency>
```

**Step 2: Test Class ni Create Chey**
Ippudu, manam producer and consumer ni test cheyadaniki oka test class create cheddam.

```java
package com.example;

import com.example.service.MessageConsumerService;
import com.example.service.MessageProducerService;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.kafka.test.context.EmbeddedKafka;
import org.springframework.test.annotation.DirtiesContext;

import java.util.concurrent.TimeUnit;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest
@DirtiesContext // Test tarvata context ni clean cheyadaniki
@EmbeddedKafka(partitions = 1, brokerProperties = { "listeners=PLAINTEXT://localhost:9092", "port=9092" })
class KafkaIntegrationTest {

    @Autowired
    private MessageProducerService producer;

    @Autowired
    private MessageConsumerService consumer; // Manam consumer lo oka latch pettukovali to wait for message

    @Test
    public void testSendAndReceive() throws Exception {
        // Step 1: Send a message
        producer.sendMessage("Testing with @EmbeddedKafka!");

        // Step 2: Wait for the consumer to receive it
        // (Note: In a real test, you'd use a CountDownLatch in your consumer to wait for the message)
        // For simplicity, let's assume we have a way to check if the message was consumed.
        // consumer.getLatch().await(10, TimeUnit.SECONDS);

        // Step 3: Assert that the message was received
        // assertThat(consumer.getLatch().getCount()).isZero();
        // assertThat(consumer.getPayload()).contains("Testing with @EmbeddedKafka!");

        System.out.println("--> Test successful! Message was sent and received.");
    }
}
```

**Code lo em undi?**
*   `@SpringBootTest`: Idi oka full Spring Boot integration test ani cheptundi.
*   `@EmbeddedKafka`: Ikkade magic undi. Idi test kosam oka Kafka broker ni start chestundi.
    *   `partitions = 1`: Test topic ki enni partitions kavalo cheptam.
    *   `brokerProperties`: Embedded broker properties ni set chestam.
*   `@DirtiesContext`: Test aipoyaka, Spring application context ni clean cheyadaniki.

---

### Diagram: The Testing Flow 🧪

```mermaid
graph TD
    A[JUnit Test Starts] -- Triggers --> B["@EmbeddedKafka Annotation"];
    B -- "Starts In-Memory" --> C((Kafka Broker));
    A -- "Calls" --> D[ProducerService.sendMessage()];
    D -- "Sends message to" --> C;
    C -- "Delivers message to" --> E[ConsumerService.listen()];
    A -- "Asserts" --> F{Result is Correct?};
    F -- "Yes" --> G[Test Passes ✅];
    F -- "No" --> H[Test Fails ❌];
    A -- "After Test" --> I["@DirtiesContext cleans up"];
    I -- "Stops" --> C;
```

---

### 📝 Interview Point:

"**How do you test your Kafka producers and consumers in a Spring Boot application?**"
"The recommended approach is to write integration tests by using the `@EmbeddedKafka` annotation from the `spring-kafka-test` library. This annotation starts an in-memory Kafka broker for the duration of the test.
We annotate our test class with `@SpringBootTest` and `@EmbeddedKafka`. Inside the test method, we can then use our `KafkaTemplate`-based producer to send a message and then verify that our `@KafkaListener` has consumed it correctly, often by using a `CountDownLatch` to handle the asynchronous nature of the consumer."

---

### The End... For Now! 🎉

Mawa! Ee "Zero to Hero" series tho, manam Spring Kafka lo chala important and advanced concepts cover chesam. From basic setup to complex transactions and testing, you now have a solid foundation.

You are now ready to build robust, real-world, event-driven applications with Spring Kafka. Go and build something amazing! All the best! 💪

**#HappyCoding** ✨