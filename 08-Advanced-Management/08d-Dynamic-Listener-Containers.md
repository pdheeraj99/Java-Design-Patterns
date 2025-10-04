# Spring Kafka: Zero to Hero - 08d: Runtime Magic! (Dynamic Listeners) ✨🎩

Namaste mawa! Manam ippativaraku listeners ni `@KafkaListener` annotation tho, compile-time lo ne define chesam. Ante, application start ayye mundu, anni listeners, vaati topics, anni fix aipothai.

Kani, konni advanced scenarios lo, manaki ee flexibility saripodu. Manaki runtime lo, on-the-fly, kotha listeners ni create chesi, start chesi, or unna vaatini stop cheyalsina avasaram ravochu.

**Scenario**: Oka admin UI undi. Admin aa UI nunchi oka kotha topic peru isthe, ventane aa topic ni vinadaniki oka kotha consumer start avvali. Leda, oka topic meeda load ekkuva ayindi, daaniki inko consumer ni add cheyali.

Ee lanti dynamic behavior kosam, manam **Dynamic Listener Containers** ni create cheyali.

---

### How to Create Containers Programmatically?

Deeniki manam `@KafkaListener` annotation ni vaadamu. Instead, manam direct ga `ConcurrentMessageListenerContainer` objects ni create chestam.

**The Key Players:**
*   **`ConcurrentKafkaListenerContainerFactory`**: Manam deenini already chusam. Idi mana "factory" anamata, kotha containers ni create cheyadaniki.
*   **`KafkaListenerEndpointRegistry`**: Idi mana "control panel", create chesina containers ni manage cheyadaniki.
*   **`MessageListener`**: Idi asalu logic. Vachina message tho em cheyalo ee class lo rastam.

### Let's Build a Dynamic Container Manager

Manam oka `DynamicContainerManager` service create cheddam. Ee service lo, manam kotha listeners ni create chesi, register chesi, start cheyochu.

```java
package com.example.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.listener.ConcurrentMessageListenerContainer;
import org.springframework.kafka.listener.MessageListener;
import org.springframework.stereotype.Service;

@Service
public class DynamicContainerManager {

    @Autowired
    private ConcurrentKafkaListenerContainerFactory<String, String> factory;

    // Note: In a real app, you would manage these containers in a Map
    // to stop or get their status later.

    public void createAndStartListener(String topic, String groupId) {
        System.out.println("#### -> Creating a dynamic listener for topic: " + topic);

        // 1. Create a container using the factory
        ConcurrentMessageListenerContainer<String, String> container =
            factory.createContainer(topic);

        // 2. Set properties for this specific container
        container.getContainerProperties().setGroupId(groupId);

        // 3. Set the listener logic (can be a lambda or a separate class)
        container.getContainerProperties().setMessageListener(
            (MessageListener<String, String>) record -> {
                System.out.println(String.format("Dynamic listener received: %s from topic %s",
                    record.value(), record.topic()));
            });

        // 4. Set a unique bean name for the container
        container.setBeanName("dynamicListener-" + topic + "-" + groupId);

        // 5. Start the container!
        container.start();

        System.out.println("#### -> Listener for " + topic + " started!");
    }
}
```
Ippudu, manam `dynamicContainerManager.createAndStartListener("new-topic", "new-group")` ani call chesthe, on-the-fly oka kotha consumer aa topic kosam start avthundi!

### Diagram: The Dynamic Flow 🪄

```mermaid
graph TD
    A[Admin Action <br/> (e.g., REST call)] --> B{DynamicContainerManager};
    B -- "Uses" --> C[ListenerContainerFactory];
    C -- "Creates" --> D(New MessageListenerContainer);
    D -- "Is given a" --> E[MessageListener Logic];
    D -- "Starts" --> F((Kafka Consumer));
    F -- "Starts polling" --> G((Kafka Topic));
```

---

### 📝 Interview Point:

"**Is it possible to create Kafka listeners dynamically at runtime in Spring Kafka?**"
"Yes, absolutely. Instead of using the `@KafkaListener` annotation, which is declarative and processed at compile time, we can create listener containers programmatically.
1.  We `@Autowired` the `ConcurrentKafkaListenerContainerFactory` into our service.
2.  We call `factory.createContainer("topic-name")` to create a new `ConcurrentMessageListenerContainer` instance.
3.  We then programmatically set its properties, such as the `groupId` and the `MessageListener` implementation (which contains the business logic).
4.  Finally, we call `.start()` on the container instance to begin consumption. This approach is powerful for use cases that require dynamic topic subscriptions."

---

### Next Enti? (What's Next?)

Mawa, manam ippudu container management lo kuda chala nerchukunnam. Kani, manam `KafkaTemplate` gurinchi matladinappudu, adi thread-safe eh na? Multiple threads oke saari `KafkaTemplate` ni use cheste emavthundi?

Next section lo, manam ee **Thread Safety Considerations** gurinchi thelusukundam. Idi chala important topic for building multi-threaded applications. Let's go! 🧵🕸️