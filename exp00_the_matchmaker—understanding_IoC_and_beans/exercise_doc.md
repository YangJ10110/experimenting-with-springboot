# Exercise 00: The Matchmaker — Understanding IoC and Beans

**Goal:** Build a "Greeting System" where Spring, not the developer, manages object lifecycles.

**Duration:** 20–30 minutes

**Spring Boot Version:** 3.4.x | **Java Version:** 21 (LTS) | **Build Tool:** Maven

---

## Learning Objectives

* **Understand Inversion of Control:** Moving from "I create my objects" to "Spring manages my objects."
* **Identify a Spring Bean:** Define and register a Java object within the Spring Application Context.
* **Perform Constructor Injection:** Implement the recommended pattern for mandatory dependencies using `@Service` and `@Component`.

---

## The "Business Case"

**User Story 1:** As a system, I need to provide localized greetings (e.g., "Hello" or "Hola") without hardcoding the greeting logic inside the main application class.

**User Story 2:** As a developer, I want to swap the "Greeting Provider" implementation without changing the code that consumes it.

---

## The Project Blueprint (Step-by-Step)

### Step 1: The POM/Build Setup

Use `spring-boot-starter` (the bare minimum) for a non-web Spring Boot CLI application.

**AI Prompt for clean pom.xml:**
```
Generate a minimal Maven pom.xml for Spring Boot 3.4.x with Java 21 that includes only spring-boot-starter (no web dependencies) for a console application focused on IoC container learning.
```

**Expected pom.xml structure:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.4.0</version>
        <relativePath/>
    </parent>
    
    <groupId>com.example</groupId>
    <artifactId>matchmaker-exercise</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>matchmaker-exercise</name>
    
    <properties>
        <java.version>21</java.version>
    </properties>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### Step 2: The Domain/Contract

Define a `GreetingService` interface that establishes the contract for greeting providers.

```java
package com.example.service;

public interface GreetingService {
    String getGreeting();
}
```

### Step 3: The Core Configuration

Create an implementation class marked as a Spring-managed Bean using `@Service`.

```java
package com.example.service;

import org.springframework.stereotype.Service;

@Service
public class EnglishGreetingService implements GreetingService {
    
    @Override
    public String getGreeting() {
        return "Hello from Spring Bean!";
    }
}
```

**Key Point:** The `@Service` annotation tells Spring's component scanner to register this class as a Bean in the ApplicationContext.

### Step 4: The Logic (Controller/Service)

Create a `GreetingManager` class that uses Constructor Injection to receive its dependency.

```java
package com.example.manager;

import com.example.service.GreetingService;
import org.springframework.stereotype.Component;

@Component
public class GreetingManager {
    
    private final GreetingService greetingService;
    
    // Constructor Injection - Spring's recommended pattern
    public GreetingManager(GreetingService greetingService) {
        this.greetingService = greetingService;
    }
    
    public void printGreeting() {
        String greeting = greetingService.getGreeting();
        System.out.println("Manager says: " + greeting);
    }
}
```

### Step 5: The "Verification Stub"

Use a `CommandLineRunner` bean in the main application class to trigger the greeting and verify Spring's wiring.

```java
package com.example;

import com.example.manager.GreetingManager;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class MatchmakerApplication {

    public static void main(String[] args) {
        SpringApplication.run(MatchmakerApplication.class, args);
    }
    
    @Bean
    public CommandLineRunner demo(GreetingManager greetingManager) {
        return args -> {
            System.out.println("=== Spring IoC Demonstration ===");
            greetingManager.printGreeting();
            System.out.println("=== Injection Successful! ===");
        };
    }
}
```

**Run Command:** `mvn spring-boot:run`

---

## Success Checks

When you run `mvn spring-boot:run`, you should see the following console output:

```
=== Spring IoC Demonstration ===
Manager says: Hello from Spring Bean!
=== Injection Successful! ===
```

**Additional Success Indicators:**
- No `NoSuchBeanDefinitionException` errors
- Log message: `Started MatchmakerApplication in X.XXX seconds`
- Clean application shutdown without exceptions

---

## "Under the Hood" Section

### The Component Scan Process

1. **@SpringBootApplication** triggers component scanning starting from the package containing the main class
2. **@Service** and **@Component** annotations mark classes for Bean registration
3. **ApplicationContext** acts as the "Matchmaker," creating instances and resolving dependencies
4. **Constructor Injection** allows Spring to provide the required `GreetingService` implementation to `GreetingManager`

### Spring's ApplicationContext as "The Matchmaker"

The ApplicationContext maintains a registry of all Beans and their dependencies. When `GreetingManager` is instantiated, Spring:
1. Identifies the constructor parameter type (`GreetingService`)
2. Searches the context for a matching Bean (finds `EnglishGreetingService`)
3. Injects the instance automatically
4. Manages the complete object lifecycle

This is **Inversion of Control** in action: instead of `GreetingManager` creating its own `GreetingService`, Spring provides it.

---

## Troubleshooting

### NoSuchBeanDefinitionException
**Symptom:** `No qualifying bean of type 'com.example.service.GreetingService' available`

**Cause:** Missing `@Service` annotation on `EnglishGreetingService`

**Solution:** Ensure the implementation class has `@Service` and is in a package scanned by Spring

### NoUniqueBeanDefinitionException
**Symptom:** `No qualifying bean of type 'com.example.service.GreetingService' available: expected single matching bean but found 2`

**Cause:** Multiple classes implement `GreetingService` (e.g., `EnglishGreetingService` and `SpanishGreetingService`)

**Solution:** Use `@Primary` on the preferred implementation or `@Qualifier` in the injection point

### NullPointerException in GreetingManager
**Symptom:** NPE when calling `greetingService.getGreeting()`

**Cause:** Manual instantiation (`new GreetingManager()`) bypasses Spring's dependency injection

**Solution:** Always let Spring create and manage your Beans—never use `new` for Spring-managed components

---

**Remember:** In Spring's world, you define the contracts and implementations. Spring handles the instantiation, wiring, and lifecycle management. You've just experienced the power of **Inversion of Control**!