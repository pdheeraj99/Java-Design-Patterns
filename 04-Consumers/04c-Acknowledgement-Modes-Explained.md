# Spring Kafka: Zero to Hero - 04c: Pani Aipoindi ani Cheppadam Ela? (Acknowledgement Modes) ✅

Namaste mawa! Last section lo manam parallel ga messages ela process cheyalo nerchukunnam. Ippudu inko chala important vishayam: **Acknowledgement** (or Offset Committing).

Simple ga cheppali ante, idi manam Kafka ki "Mawa, ee message ni nenu successful ga process chesanu. Inka deeni gurinchi marchipo, next pampu" ani cheppadam. Ee process sarigga cheyakapothe, manam messages ni miss avvochu, or oke message ni malli malli process chese chance undi.

Spring Kafka manaki ee pani cheyadaniki `AckMode` ane oka property istundi. Deenilo chala options unnai, kani manam most important వాటి gurinchi matladukundam.

---

### Key `AckMode` Options

#### 1. `BATCH` (The Default One 👍)
*   **Ela Pani Chestundi?**: Idi default behavior. Listener container `poll()` method tho konni messages (oka batch) theeskuntundi. Aa batch lo unna anni messages ni listener method process chesaka, appudu offset commit avthundi.
*   **Pros**: Chala efficient. Prathi message ki commit cheyakunda, oke saari batch antha commit chestundi.
*   **Cons**: Batch madhyalo edaina message fail aithe, aa batch antha malli re-process avthundi.
*   **Config**: `factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.BATCH);` (or just leave it, as it's the default).

#### 2. `RECORD`
*   **Ela Pani Chestundi?**: `poll()` method batch theeskuntundi, kani prathi message ni listener ki ichi, adi process cheyagane ventane offset commit chestundi.
*   **Pros**: Oke message ni malli malli process chese chance thakkuva.
*   **Cons**: Performance konchem thaggutundi, endukante prathi message ki commit chestunnam.
*   **Config**: `factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.RECORD);`

#### 3. `MANUAL_IMMEDIATE`: "Ventane Chey!" ⚡
*   **Ela Pani Chestundi?**: Ikkada antha control mana chethilo untundi. Spring automatic ga commit cheyadu. Maname, mana listener method lo, `ack.acknowledge()` ani call cheyagane, ventane, aa consumer thread meeda ne offset commit aipothundi.
*   **Pros**: Full control and immediate confirmation. Database write successful aithe, ventane commit cheyochu.
*   **Cons**: Prathi message ki oka separate commit request broker ki velthundi. Performance meeda impact undochu.
*   **Config**: `factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL_IMMEDIATE);`

#### 4. `MANUAL`: "Aagu, Anni Oke Saari Cheddam" 🐢
*   **Ela Pani Chestundi?**: Ee mode lo kuda maname `ack.acknowledge()` call cheyali, kani offset ventane commit **avvadu**. Adi just "Ee message pani aipoindi" ani mark cheskuntundi. Asalu commit eppudu avthundi? Batch antha process aipoyaka, appudu mark chesina anni offsets ni oke saari commit chestundi.
*   **Pros**: `MANUAL_IMMEDIATE` kanna better performance, kani manaki individual message acknowledgement control untundi.
*   **Cons**: `BATCH` mode laage, batch madhyalo fail aithe, already acknowledged messages kuda malli re-process avvochu.
*   **Config**: `factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL);`

---

### Code Example: `MANUAL_IMMEDIATE` vs `MANUAL`

`MANUAL_IMMEDIATE` or `MANUAL` vaadinappudu, mana listener method ki `Acknowledgment` object ni parameter ga theeskovali.

```java
// In MessageConsumerService.java
import org.springframework.kafka.support.Acknowledgment;
import org.springframework.stereotype.Service;

@Service
public class ManualAckConsumerService {

    // Ee listener MANUAL or MANUAL_IMMEDIATE tho pani chestundi
    @KafkaListener(topics = "my-manual-ack-topic", groupId = "manual-ack-group")
    public void listen(String message, Acknowledgment ack) {
        System.out.println("#### -> Consumed message -> " + message);

        // ... Ekkada mana critical logic untundi, for example, database write ...

        System.out.println("#### -> Acknowledging the message");
        ack.acknowledge(); // Ippudu commit chey/mark chey ani cheptunnam!
    }
}
```

### Diagram: AckMode Decision Flow 🧠

```mermaid
graph TD
    A[Message Batch Received] --> B{Process Each Message};
    B --> C{AckMode?};

    C -- BATCH --> D[Process ALL messages in batch];
    D --> E[Commit offsets for entire batch];

    C -- RECORD --> F[Process ONE message];
    F --> G[Commit offset for that message];
    G --> B;

    C -- MANUAL / MANUAL_IMMEDIATE --> H[Process ONE message];
    H --> I[Call ack.acknowledge()];
    I -- "If MANUAL_IMMEDIATE" --> G;
    I -- "If MANUAL" --> J[Queue offset for later commit];
    J --> B;
```

---

### 📝 Interview Point:

"**What are the different `AckMode`s in Spring Kafka and when would you use them?**"
"Spring Kafka provides several `AckMode`s to control offset commits:
*   **`BATCH` (default):** Commits offsets after the entire batch of records from a poll is processed. It's efficient but can lead to reprocessing.
*   **`RECORD`:** Commits the offset for each record as soon as it's processed. Use this for better message-level control at the cost of some performance.
*   **`MANUAL` & `MANUAL_IMMEDIATE`:** Give the developer full control via the `Acknowledgment` object. `MANUAL_IMMEDIATE` commits synchronously when `acknowledge()` is called, which is ideal for critical operations. `MANUAL` queues the acknowledgment and commits them in a batch later, offering a balance between control and performance."

---

### Next Enti? (What's Next?)

Mawa, manam ippudu offset committing lo masters aipoyam! Kani, konni sarlu manaki oka topic lo unna anni messages avasaram undadu. Konni specific messages ni matrame process cheyali anukuntam.

Next section lo, manam **Filtering Messages** gurinchi thelusukundam. Vachina messages ni listener ki ivvakunda, madyalo ne ela filter cheyalo chuddam! 🗑️➡️✅