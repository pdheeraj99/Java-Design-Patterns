# Spring Kafka: Zero to Hero - 05b: "Chanipoyina" Messages (Handling Tombstones) 🪦

Namaste mawa! Manam ippudu oka special type of topic gurinchi matladukundam: **Compacted Topic**.

Normal topics lo, manam pampina anni messages untai (retention period varaku). Kani, compacted topic lo, Kafka kevalam prathi key ki **latest value** ni matrame unchutundi. Paatha values ni delete chestundi. Idi state management ki and look-up tables ki chala useful.

**Kani, oka key ni delete cheyali ante ela?**
Simple ga aa key tho, `null` value pampistam. Appudu Kafka, "Okay, ee key ni delete cheyali" ani artham cheskuni, aa key ki sambandinchina anni messages ni theesestundi.

Ee `null` payload unna special message ne **Tombstone Record** antaru. 🪦

---

### The Problem: Null Payload Exception! 💥

Mana `@KafkaListener` method `public void listen(User user)` lanti signature tho unte, adi eppudu oka `User` object ni expect chestundi. Kani, tombstone record vachinappudu, payload `null` ga untundi. Appudu mana listener error isthundi!

### Solution: `@Payload(required = false)` ✅

Ee problem ni solve cheyadaniki, manam mana listener method ki cheppali, "Hey, konni sarlu payload null ravochu, பரவாயில்லை (paravaaledu), handle chesko" ani.

Idi manam `@Payload(required = false)` ane annotation parameter tho cheptam. Appudu manam `null` payloads ni gracefully handle cheyochu.

---

### How to Handle Tombstones

Tombstone vachinappudu, manaki "ఏ key delete ayyindi?" ani theliyali. Anduke, manam payload tho paatu, key ni kuda theeskovali.

**Listener Update (`MessageConsumerService.java`):**

```java
// In MessageConsumerService.java
import org.springframework.kafka.support.KafkaHeaders;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.messaging.handler.annotation.Payload;

@Service
public class MessageConsumerService {

    @KafkaListener(topics = "user-profile-topic") // Assuming this is a compacted topic
    public void listenToUserUpdates(
            @Payload(required = false) User user,
            @Header(KafkaHeaders.RECEIVED_KEY) String key) {

        if (user == null) {
            // This is a Tombstone record!
            System.out.println("#### -> Received Tombstone for key: " + key + ". Deleting user data...");
            // Ekkada manam mana local cache or database nunchi aa user ni delete chese logic raayali.
        } else {
            // This is a regular update.
            System.out.println("#### -> Received update for user: " + user);
            // Ekkada manam user ni create or update chese logic raayali.
        }
    }
}
```

### Diagram: The Tombstone Flow 🪦

```mermaid
graph TD
    subgraph Producer
        A[send("user123", new User(...))]
        B[send("user123", null)]
    end

    subgraph Consumer
        D{Listener Receives Record}
        E{Payload is NULL?}
        F[Process User Update]
        G[Handle Deletion (Tombstone)]
    end

    A --> C((Compacted Topic));
    B -- "sends tombstone for user123" --> C;

    C --> D;
    D --> E;
    E -- "No" --> F;
    E -- "Yes" --> G;
```

---

### 📝 Interview Point:

"**What is a tombstone record in Kafka and how do you handle it in a Spring Kafka listener?**"
"A tombstone record is a message with a `null` payload, used in log-compacted topics to signal the deletion of a specific key.
To handle it in a `@KafkaListener`, the payload parameter must be annotated with `@Payload(required = false)`. This prevents an exception when a null payload is received. Inside the listener, we can check if the payload is `null`. If it is, we know it's a tombstone, and we can then use the message key (retrieved via `@Header(KafkaHeaders.RECEIVED_KEY)`) to perform the corresponding delete operation in our application's state or database."

---

### Next Enti? (What's Next?)

Super mawa! Manam ippudu serialization lo oka important and advanced concept nerchukunnam. Ippudu manam malli **Error Handling** loki jump cheddam, kani inka advanced level lo.

Manam `DefaultErrorHandler` gurinchi thelusukunnam, kani Spring 2.7+ lo inko super powerful, modern approach vachindi: **Non-Blocking Retries with `@RetryableTopic`**. Ee concept tho manam retries ni inka better ga, performance impact lekunda ela handle cheyalo next section lo chuddam. It's a game-changer! 🔥