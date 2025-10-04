# Spring Kafka: Zero to Hero - 02c: Topics ni Code tho Create Cheddam! 🏗️

Namaste mawa! Manam ippativaraku topics unnayi ani anukuni messages pampam, receive cheskunnam. Kani, oka kotha environment lo mana application deploy chesinappudu, aa topic mundu nunchi create chesi undakapothe? Appudu mana application start avvagane fail avthundi. 🤯

Ee problem ni solve cheyadaniki, maname mana application start ayyetappudu కావలసిన topics ni create cheyochu. "Topic unda leda ani check chey, lekapothe create chey" ani manam Spring ki cheppochu.

---

### Key Players: Ee Aata lo Iddaroo Pedda Thalaakaayalu! 👑

1.  **`KafkaAdmin`**: Veedu oka manager lantodu. Mana application context lo ee bean unte, adi velli Kafka broker tho matladi, topics create cheyadam, modify cheyadam lanti panulu chuskuntundi. Spring Boot vaaduthunte, ee `KafkaAdmin` bean automatic ga create avthundi, manam em cheyanakkarledu!

2.  **`NewTopic`**: Idi oka blueprint anamata. Manam create cheyali anukuntunna topic ki ela undali—ante topic peru enti, enni partitions undali, enni replicas undali—lanti details anni ee bean lo define chestam.

Application start ayinappudu, `KafkaAdmin` mana context lo unna `NewTopic` beans anni theeskuni, Kafka broker lo aa topics ni create chestundi.

---

### Configuration: Topic ni Ela Create Cheyali? 🔨

Manam `config` package lo `KafkaTopicConfig.java` ane kotha file create chesi, daantlo `NewTopic` bean ni define cheddam.

```java
package com.example.config;

import org.apache.kafka.clients.admin.NewTopic;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.config.TopicBuilder;

@Configuration
public class KafkaTopicConfig {

    @Bean
    public NewTopic myFirstTopic() {
        return TopicBuilder.name("my-first-topic")
                .partitions(3)    // 3 partitions create chey
                .replicas(1)      // 1 replica undali (dev environment lo 1 saalu)
                .build();
    }

    @Bean
    public NewTopic myDlt() {
        // Manam DLT gurinchi tarvata matladukundam, kani ippude create cheddam
        return TopicBuilder.name("my-first-topic.DLT")
                .partitions(1)
                .replicas(1)
                .build();
    }
}
```

**Code lo em undi?**
*   Manam `my-first-topic` and `my-first-topic.DLT` ane rendu topics ni create cheyadaniki `NewTopic` beans ni define chesam.
*   **`partitions(3)`**: Topic ni 3 bhagaluga (partitions) divide chestundi. Idi parallel processing ki help chestundi.
*   **`replicas(1)`**: Data loss avvakunda undataniki, topic data ni enni sarlu copy cheyali anedi cheptundi. Production lo usually `3` pedatharu for fault tolerance.

Spring Boot vaaduthunte, inthakante em cheyanakkarledu. Application start avvagane, ee topics automatic ga create aipothai! ✅

### Diagram: The Auto-Creation Flow ✨

```mermaid
graph TD
    A[Application Starts] --> B{Spring Context Initializes};
    B --> C[Finds NewTopic Beans <br/>(myFirstTopic, myDlt)];
    B --> D[Auto-configures KafkaAdmin];
    C --> E[Passes Topic Info];
    D --> E;
    E[KafkaAdmin] -- "Checks/Creates Topics" --> F((Kafka Broker));
```

---

### 📝 Interview Point:

"**How do you ensure Kafka topics are created before your Spring application starts consuming or producing messages?**"
Ee question ki answer simple: "In a Spring Boot application, we can define one or more `@Bean`s of type `NewTopic`. Spring Boot's auto-configuration provides a `KafkaAdmin` bean, which automatically discovers these `NewTopic` beans on startup. The `KafkaAdmin` then connects to the broker and creates the topics if they don't already exist. This is an idempotent operation, so it won't fail if the topic is already there."

---

### Next Enti? (What's Next?)

Super mawa! Configuration section antha cover chesam. Producer ready, Consumer ready, and Topic kuda ready. Ippudu asalu pani modaledadam.

Next section lo, manam mana `KafkaTemplate` ni use chesi, mana modati message ni ela pampalo chuddam. Let's start sending some data! 📤