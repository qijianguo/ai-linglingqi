# ai-agent-linglingqi

Composable Spring Boot services for experimenting with AI-powered agents and related authentication helpers.

---

## Repository Layout

| Path | Description |
| --- | --- |
| `pom.xml` | Maven root that manages shared plugins/dependencies and aggregates the agent module. |
| `ai-linglingqi-agent/` | HTTP-facing Agent service that will orchestrate Spring AI models. |
| `ai-linglingqi-auth/` | Stand-alone Auth service scaffold (not yet wired into the root aggregator). |

Both modules currently expose only their Spring Boot entry points, so this document focuses on those public components and how to extend them safely.

---

## Build & Run

Prerequisites:

- JDK 17+
- Maven 3.9+

Common commands (run from repo root):

```bash
# Compile every module
mvn clean verify

# Start only the agent service on port 60001
mvn -pl ai-linglingqi-agent spring-boot:run

# Run the auth service (stand-alone)
cd ai-linglingqi-auth && mvn spring-boot:run
```

To run all tests for every module:

```bash
mvn test
```

---

## Module: `ai-linglingqi-agent`

### Purpose

A Spring Boot Web service that will eventually host AI agent workflows. Dependencies include `spring-boot-starter-web` and `spring-ai-starter-model-openai`, so the module is already wired for REST APIs plus OpenAI model access.

### Public Components

| Component | Visibility | Description | Notes |
| --- | --- | --- | --- |
| `AiLinglingqiAgentApplication` (`com.qijianguo.ai.agent`) | `public` | Main Spring Boot application class annotated with `@SpringBootApplication`. | Boots the embedded server and auto-configured Spring AI beans. |
| `main(String[] args)` | `public static` | Hands control to `SpringApplication.run(...)`. | Entry point for all deployment targets (CLI, container, IDE). |

Application configuration is in `ai-linglingqi-agent/src/main/resources/application.properties`:

- `spring.application.name=ai-linglingqi-agent`
- `server.port=60001`

### Usage Example

1. Start the service:
   ```bash
   mvn -pl ai-linglingqi-agent spring-boot:run
   ```
2. Watch the console for the `Started AiLinglingqiAgentApplication` log line to confirm the server is listening on `http://localhost:60001`.
3. Add controllers (see below) to serve domain APIs; until then the server responds with `404 Not Found` because no HTTP endpoints are defined.

### Adding REST APIs

Use Spring Web MVC to expose new endpoints while reusing the existing context:

```java
package com.qijianguo.ai.agent.api;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Map;

@RestController
@RequestMapping("/v1/agent")
public class AgentController {

    @GetMapping("/ping")
    public Map<String, String> ping() {
        return Map.of("status", "ok");
    }
}
```

Place the controller under `com.qijianguo.ai.agent` so it is discovered by component scanning. With this in place you can call:

```bash
curl http://localhost:60001/v1/agent/ping
```

### Integrating OpenAI via Spring AI

The dependency on `spring-ai-starter-model-openai` means all OpenAI clients are auto-configured. Supply credentials via environment variables or `application.properties`:

```properties
spring.ai.openai.api-key=${OPENAI_API_KEY}
spring.ai.openai.chat.options.model=gpt-4o-mini
spring.ai.openai.chat.options.temperature=0.2
```

Then inject the `ChatClient` bean wherever you need LLM calls.

---

## Module: `ai-linglingqi-auth`

### Purpose

Minimal Spring Boot starter for future authentication/authorization flows. It currently depends only on `spring-boot-starter`, so the context loads but no HTTP endpoints or security filters exist yet.

### Public Components

| Component | Visibility | Description |
| --- | --- | --- |
| `AiLinglingqiAuthApplication` (`com.qijianguo.ai.auth`) | `public` | Main class annotated with `@SpringBootApplication`. |
| `main(String[] args)` | `public static` | Standard Spring Boot entry point. |

`application.properties` configures only the service name, so the default server port (8080) is used when the web starter is added later.

### Usage Example

```bash
cd ai-linglingqi-auth
mvn spring-boot:run
```

Because no controllers exist, add one under `com.qijianguo.ai.auth` to publish auth-related endpoints. Share DTOs/interfaces with other modules via a new common module or Maven dependency once those contracts stabilize.

---

## Testing

Each module ships with a Spring Boot context smoke test (`contextLoads`) that verifies the application starts with the current dependency graph. Extend these tests with slice or integration tests once real controllers/services are introduced.

Run tests:

```bash
mvn -pl ai-linglingqi-agent test
mvn -pl ai-linglingqi-auth test
```

---

## Next Steps

- Flesh out REST controllers in `ai-linglingqi-agent` to expose chatbot, tool-calling, or retrieval pipelines.
- Add authentication filters/controllers to `ai-linglingqi-auth`, then declare it as a Maven module in the root `pom.xml` if you want it built alongside the agent.
- When public contracts stabilize, capture them in OpenAPI/Swagger and link them back to this document.
