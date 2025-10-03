# Spring Kafka Tutorial - 03: Messages Ela Vinaali? (Consumer & @KafkaListener) 🎧

Hey mawa, welcome back! Producer tho messages pampadam nerchukunnam. Ippudu aa pampina messages ni ela pattukovali (consume cheyali) anedi chuddam. Ikkade manaki `@KafkaListener` ane super-power astra 🦸‍♀️ parichayam avthundi.

---

### Consumer Configuration (Java Based) ☕

Producer laage, consumer ki kuda konni configurations cheyali. Manam `ConsumerFactory` and `ConcurrentKafkaListenerContainerFactory` ane beans ni create cheyali.

Ee code ni copy chesi, mee project lo `config` package lo `KafkaConsumerConfig.java` ane file lo pettuko.

```java
package com.example.config;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;

import java.util.HashMap;
import java.util.Map;

@Configuration
public class KafkaConsumerConfig {

    @Bean
    public ConsumerFactory<String, String> consumerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "my-group-id"); // Chala important!
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest"); // Idi kuda important!
        return new DefaultKafkaConsumerFactory<>(props);
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        return factory;
    }
}
```

**Code lo em undi?**
*   **`consumerFactory()`**: Producer factory laage, idi consumer ni ela create cheyalo cheptundi.
    *   `BOOTSTRAP_SERVERS_CONFIG`: Kafka server address.
    *   `KEY_DESERIALIZER_CLASS_CONFIG` & `VALUE_DESERIALIZER_CLASS_CONFIG`: Producer lo `Serializer` vaadam, ikkada `Deserializer` vaadatam. Ante, network nunchi vachina data ni manaki artham ayye format (String) loki marchadam.
*   **`kafkaListenerContainerFactory()`**: Idi manam create chese `@KafkaListener`s ni manage chestundi.

---

### Deep Dive: rendu mukhyamaina properties 🧐

#### 1. `group.id`
Idi chala chala important mawa. Oka topic ni multiple consumers vintu unte, avi anni oke `group.id` share cheskunte, Kafka aa messages ni aa group lo unna consumers ki panchi pedutundi (load balancing). Ante, oke message group lo unna consumers andariki velladu, okariki matrame velthundi. Ide Kafka power!

```mermaid
graph TD
    subgraph Kafka Topic
        P1["Partition 0"]
        P2["Partition 1"]
        P3["Partition 2"]
    end

    subgraph Consumer Group (group.id: 'my-group-id')
        C1["Consumer 1"]
        C2["Consumer 2"]
        C3["Consumer 3"]
    end

    P1 -- "Message" --> C1
    P2 -- "Message" --> C2
    P3 -- "Message" --> C3
```
*Multiple consumers in the same group read from different partitions of the same topic.*

#### 2. `auto.offset.reset`
Oka vela mana consumer start ayye time ki, topic lo already unna messages ni em cheyali? Ee property ade cheptundi.
*   **`earliest`**: "Naaku topic lo unna anni messages, first nunchi ivvu" ani adagadam.
*   **`latest`**: "Nenu start ayyaka vache kotha messages matrame ivvu, paathavi oddu" ani adagadam. (Default idi).

---

### Message Ela Vinaali? (Let's Listen!) 🎧

Ippudu manam `@KafkaListener` use chesi message ni receive cheskune service create cheddam.

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
Anthe mawa, chala simple! Ee `listen` method meeda `@KafkaListener` petti, topic peru and group id isthe chalu. `my-first-topic` lo eppudu message vachina, ee method automatic ga call avthundi.

---

### 📝 Interview Point:

"100% adige question: '**What is the significance of `group.id` in Kafka?**'. Meeru cheppalsindi: 'A consumer group ID allows multiple consumer instances to work together to consume a topic. Kafka distributes the topic's partitions among the consumers in the same group, ensuring that each message is processed by only one consumer within that group. This enables horizontal scaling and parallel processing.' ani chepthe interviewer flat aipothadu!"

"Inko question: '**Explain `auto.offset.reset` policies.**' Appudu `earliest` vs `latest` gurinchi paina cheppina points explain cheyali."

---

### Next Enti? (What's Next?)

Messages ni pampadam, receive cheskovadam vachesindi. Super! 👍 Kani real-world lo manam simple strings pampam. Complex Java objects (POJOs) pampali. Appudu serialization and deserialization ela handle cheyali?

Next section lo, manam JSON format lo custom objects ni ela pampali and receive cheskovalo chuddam. Adi inka interesting ga untadi! See you there! 😉