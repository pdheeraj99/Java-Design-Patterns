# Spring Kafka: Zero to Hero - 06a: Bad Message ni Handle Cheyadam Ela? (Deserialization Errors) 🤢

Namaste mawa! Error handling lo idi first and most common problem. Oka vela producer oka `User` object pampi, mana consumer `Order` object expect chesthe? Leda, message corrupt ayi unte?

Appudu mana `JsonDeserializer` aa message ni manam anukunna object ga marchaleka thala pattukuntundi. Appudu adi oka `DeserializationException` ni visuruthundi.

By default, ee exception vasthe, mana consumer aagipothundi. Inka messages process cheyadu. Idi pedda production issue! 😱

---

### Solution: `ErrorHandlingDeserializer` to the Rescue! 🛡️

Ee problem ni solve cheyadaniki, Spring Kafka manaki `ErrorHandlingDeserializer` ane oka powerful weapon ichindi.
*   **Ela Pani Chestundi?**: Idi mana actual deserializer (`JsonDeserializer`) ki oka cover laaga pani chestundi.
*   Okavela `JsonDeserializer` fail aithe, ee `ErrorHandlingDeserializer` aa exception ni pattukuni, consumer aagipokunda chuskuntundi.
*   Fail ayina record ni `null` ga pampi, exception details ni oka special header (`SerializationUtils.VALUE_DESERIALIZER_EXCEPTION_HEADER`) lo pedutundi.

### Configuration Update (`KafkaConsumerConfig.java`) 🛠️

`ErrorHandlingDeserializer` ni vaadali ante, mana consumer configuration ni konchem marchali.

```java
// In KafkaConsumerConfig.java

//... add these imports
import org.springframework.kafka.support.serializer.ErrorHandlingDeserializer;
import org.springframework.kafka.support.serializer.JsonDeserializer;

@Configuration
public class KafkaConsumerConfig {

    @Bean
    public ConsumerFactory<String, Object> consumerFactory() { // Value type ni Object ga marchu
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "my-group-id");

        // Key Deserializer
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);

        // **Main Change Ikkade**
        // 1. Value Deserializer ni ErrorHandlingDeserializer ga set chey
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, ErrorHandlingDeserializer.class);

        // 2. ErrorHandlingDeserializer ki asalu deserializer evaro cheppu
        props.put(ErrorHandlingDeserializer.VALUE_DESERIALIZER_CLASS, JsonDeserializer.class.getName());

        // 3. JsonDeserializer ki properties pass chey
        props.put(JsonDeserializer.TRUSTED_PACKAGES, "*");

        return new DefaultKafkaConsumerFactory<>(props);
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, Object> kafkaListenerContainerFactory() { // Type ni Object ki marchu
        ConcurrentKafkaListenerContainerFactory<String, Object> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        return factory;
    }
}
```

### Listener Update (`MessageConsumerService.java`) 🎧

Ippudu mana listener `Object` ni accept cheyali, endukante error vasthe `null` payload vastundi, and manam exception ni headers nunchi theeskovali.

```java
// In MessageConsumerService.java
import org.springframework.kafka.support.serializer.DeserializationException;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.kafka.support.KafkaHeaders;

@Service
public class MessageConsumerService {

    @KafkaListener(topics = "my-first-topic", groupId = "my-group-id")
    public void listen(Object payload, @Header(KafkaHeaders.DESERIALIZATION_EXCEPTION) DeserializationException de) {
        if (de != null) {
            // Error vachindi!
            System.err.println("!!!! -> Deserialization Error !!!!");
            System.err.println("Failed to deserialize message: " + new String(de.getData()));
            // Ikkada manam ee error ni log cheyochu or metrics pampochu
        } else {
            // Anni baagunnayi, payload manaki కావలసిన object
            User user = (User) payload;
            System.out.println("#### -> Consumed User: " + user);
        }
    }
}
```
**Note**: Ee approach work avvali ante, manam `DefaultErrorHandler` ni kuda configure cheyali, adi next section lo chuddam. For now, understand the concept.

---

### 📝 Interview Point:

"**How do you prevent a consumer from stopping when it receives a poison pill (a message it cannot deserialize)?**"
"The standard approach is to use Spring Kafka's `ErrorHandlingDeserializer`. It wraps the primary deserializer (e.g., `JsonDeserializer`). If the primary deserializer fails, the `ErrorHandlingDeserializer` catches the exception, returns a `null` payload, and places the `DeserializationException` in a message header. This prevents the listener container from entering an error loop and stopping, allowing it to continue processing subsequent valid messages."

---

### Next Enti? (What's Next?)

Deserialization error ni pattukovadam vachesindi. Kani, message correct ga vachina, daanni process cheseటప్పుడు mana code lo ne exception vachithe? For example, database down undochu.

Next section lo, manam alanti processing-time errors ni **Retries and Backoff Policies** tho ela handle cheyalo nerchukundam. 💪