# Smart Inventory Assistant

<div align="center">

[![Java](https://img.shields.io/badge/Java-17+-ED8B00?style=flat-square&logo=openjdk)](https://www.oracle.com/java/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x-6DB33F?style=flat-square&logo=springboot)](https://spring.io/projects/spring-boot)
[![Spring AI](https://img.shields.io/badge/Spring%20AI-1.0.0--M6-6DB33F?style=flat-square&logo=spring)](https://spring.io/projects/spring-ai)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15+-316192?style=flat-square&logo=postgresql)](https://www.postgresql.org/)
[![Docker](https://img.shields.io/badge/Docker-Supported-2496ED?style=flat-square&logo=docker)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg?style=flat-square)](LICENSE)

**English** • **[Português Brasileiro](./docs/README.pt-br.md)**

> Conversational AI-powered inventory management system built with Spring AI and Ollama

</div>

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Quick Start](#quick-start)
- [Usage Examples](#usage-examples)
- [Project Structure](#project-structure)
- [Technology Stack](#technology-stack)
- [Development](#development)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

**Smart Inventory Assistant** is a proof-of-concept (POC) demonstrating how to integrate **Spring AI** with traditional Spring Boot applications using **API-first design** and **clean hexagonal architecture**.

Instead of navigating dropdowns and complex forms, users interact with inventory management through **natural language conversations**—asking to find products under a price point, add stock to warehouses, or update inventory counts in plain English.

This project explores the practical integration of LLMs into existing business systems without requiring architectural rewrites, proving that AI-enhanced UIs can coexist seamlessly with clean, maintainable code.

---

## ✨ Features

- 🗣️ **Natural Language Interface** – Manage inventory via conversational prompts
- 🤖 **Spring AI Integration** – Local LLM support with Ollama (no external API calls)
- 🏗️ **Hexagonal Architecture** – Clean separation of concerns with ports & adapters
- 📡 **API-First Design** – OpenAPI specification with auto-generated clients
- 🧪 **Fully Tested** – Unit & integration tests with WireMock
- 🐳 **Docker Ready** – Complete containerized setup with docker-compose
- 🔧 **Extensible** – Easily add new tools and capabilities

---

## 🏛️ Architecture

```
┌──────────────────────────────────────────────────────┐
│         Frontend/Client                              │
│     (Natural Language Chat Interface)                │
└────────────────────┬─────────────────────────────────┘
                     │ HTTP/REST
                     ▼
┌──────────────────────────────────────────────────────┐
│  Spring AI Chat Client (stock-ai-client)             │
│  • Prompt Processing                                 │
│  • Tool Orchestration (@Tool beans)                  │
│  • LLM Integration                                   │
└────────────────┬─────────────────────────────────────┘
                 │ Generated OpenAPI Clients
                 ▼
┌──────────────────────────────────────────────────────┐
│  Stock Server (Hexagonal Architecture)               │
│                                                      │
│  ┌────────────────────────────────────────────┐    │
│  │ HTTP Controllers (Ports)                   │    │
│  └────────────┬─────────────────────────────┬─┘    │
│               │                             │       │
│  ┌────────────▼──────┐        ┌────────────▼─┐   │
│  │ Application Logic  │◄──────►│ Domain Models│   │
│  │ (Use Cases)        │        │ (Entities)   │   │
│  └────────────┬──────┘        └────────────┬─┘   │
│               │ Repositories              │       │
│  ┌────────────▼──────────────────────────▼─┐    │
│  │ Persistence Adapters & Specifications    │    │
│  └────────────┬──────────────────────────┬─┘    │
└───────────────┼──────────────────────────┼──────┘
                │                          │
         ┌──────▼──────┐          ┌────────▼──────┐
         │  PostgreSQL │          │  Ollama (LLM) │
         └─────────────┘          └───────────────┘
```

---

## 🚀 Quick Start

### Prerequisites

- **Java** 17+
- **Maven** 3.8+
- **Docker & Docker Compose** (recommended)

### Option 1: Docker Compose (Recommended)

```bash
git clone https://github.com/yourusername/smart-inventory-assistant.git
cd smart-inventory-assistant

# Start all services
docker-compose up -d

# View logs
docker-compose logs -f stock-ai-client
```

**Services:**
- `stock-ai-client`: Chat endpoint - http://localhost:8080
- `stock-server`: Backend API - http://localhost:8081
- `postgres`: Database - localhost:5432
- `ollama`: LLM Runtime - localhost:11434

### Option 2: Local Development

```bash
# Clone repository
git clone https://github.com/yourusername/smart-inventory-assistant.git
cd smart-inventory-assistant

# Build
mvn clean install -DskipTests

# Terminal 1: Start dependencies
docker run -d --name postgres -e POSTGRES_PASSWORD=postgres -p 5432:5432 postgres:15
docker run -d --name ollama -p 11434:11434 ollama/ollama
docker exec ollama ollama pull llama2

# Terminal 2: Start server
cd stock-server/stock-server-starter
mvn spring-boot:run

# Terminal 3: Start AI client
cd stock-ai-client
mvn spring-boot:run
```

**Chat Endpoint:**
```
http://localhost:8080/api/chat/process
```

---

## 💬 Usage Examples

### 1. Create a Product

**Request:**
```json
POST /api/chat/process
{
  "prompt": "Add a product called Wireless Mouse with SKU MOU-987, category ELECTRONICS, unit UNIT, price 29.99"
}
```

**AI Response:**
```
✓ Created product "Wireless Mouse" (SKU: MOU-987) in ELECTRONICS category - €29.99
```

**Generated API Call:**
```http
POST /api/v1/products
{
  "sku": "MOU-987",
  "name": "Wireless Mouse",
  "category": "ELECTRONICS",
  "unitOfMeasure": "UNIT",
  "price": 29.99
}
```

---

### 2. Search Products

**Request:**
```json
POST /api/chat/process
{
  "prompt": "Find all products in ELECTRONICS under 50 euros"
}
```

**AI Response:**
```
Found 3 products:
- Wireless Mouse (€29.99)
- USB Hub (€34.99)
- Keyboard (€44.99)
```

---

### 3. Update Stock

**Request:**
```json
POST /api/chat/process
{
  "prompt": "Add 150 units of MOU-987 to warehouse WAREHOUSE-001"
}
```

**AI Response:**
```
✓ Added 150 units to warehouse. Current stock: 250 units
```

---

### 4. Manage Inventory

**Request:**
```json
POST /api/chat/process
{
  "prompt": "What's the total stock level for all ELECTRONICS products?"
}
```

**AI Response:**
```
Total ELECTRONICS stock: 1,428 units across 3 warehouses
- WAREHOUSE-001: 450 units
- WAREHOUSE-002: 320 units
- WAREHOUSE-003: 658 units
```

---

### 5. Delete Product

**Request:**
```json
POST /api/chat/process
{
  "prompt": "Delete product with SKU MOU-987"
}
```

**AI Response:**
```
✓ Product deleted successfully
```

---

## 📁 Project Structure

```
smart-inventory-assistant/
│
├── README.md                          # This file
├── docker-compose.yml                 # Multi-container orchestration
├── Dockerfile                         # Stock server image
├── pom.xml                            # Parent Maven POM
│
├── stock-spec/                        # OpenAPI Specification
│   └── v1/stock-api-v1.yaml          # REST API contracts
│
├── stock-server/                      # Backend (Hexagonal)
│   ├── stock-server-api/              # HTTP layer
│   │   ├── controller/                # REST Endpoints
│   │   └── mapper/                    # DTO/Entity mapping
│   │
│   ├── stock-server-domain/           # Business Logic
│   │   ├── feature/                   # Domain features (inventory, product, warehouse)
│   │   └── model/                     # Domain entities
│   │
│   ├── stock-server-persistence/      # Data Access
│   │   ├── adapter/                   # JPA adapters
│   │   ├── entity/                    # JPA entities
│   │   ├── repository/                # Spring Data repositories
│   │   └── specification/             # JPA specifications
│   │
│   └── stock-server-starter/          # Spring Boot App
│       └── StockServerStarterApplication.java
│
├── stock-ai-client/                   # AI Client (Spring AI)
│   ├── config/                        # Spring configuration
│   │   ├── ApiClientConfig.java       # Generated API clients
│   │   └── ChatClientConfig.java      # Spring AI ChatClient
│   │
│   ├── controller/                    # Chat endpoint
│   │   └── ChatController.java
│   │
│   ├── model/                         # DTO classes
│   │   └── PromptRequest.java
│   │
│   ├── tools/                         # Spring AI @Tool beans
│   │   ├── ProductTools.java          # Product operations
│   │   ├── InventoryTools.java        # Inventory operations
│   │   └── WarehousesTools.java       # Warehouse operations
│   │
│   └── test/                          # Tests (WireMock stubs)
│       └── ProductChatControllerTest.java
│
└── docs/
    └── README.pt-br.md                # Portuguese version
```

### Key Modules

| Module | Responsibility |
|--------|-----------------|
| **stock-spec** | Source of truth - OpenAPI contracts |
| **stock-server-api** | HTTP/REST layer, request validation |
| **stock-server-domain** | Business rules, aggregates, use cases |
| **stock-server-persistence** | Database adapters, queries, mappers |
| **stock-server-starter** | Spring Boot bootstrap, configuration |
| **stock-ai-client** | LLM integration, tool orchestration, chat UI |

---

## 🛠️ Technology Stack

### Core Framework
- **Java 17+** – Language
- **Spring Boot 3.x** – Application framework
- **Maven** – Build automation
- **PostgreSQL 15+** – Relational database

### AI & LLM
- **Spring AI 1.0.0-M6** – LLM integration library
- **Ollama** – Local LLM runtime
- **Llama 2** – Default language model

### API & Code Generation
- **OpenAPI 3.0** – API specification
- **OpenAPI Generator** – Client stub generation

### Testing
- **JUnit 5** – Unit testing
- **Mockito** – Mocking
- **WireMock** – HTTP stub/mock server
- **Spring Boot Test** – Integration testing

### Architecture Patterns
- **Hexagonal Architecture** – Ports & adapters
- **Domain-Driven Design** – Entity modeling
- **Repository Pattern** – Data abstraction

---

## 📖 Development Guide

### Adding a New Chat Tool

1. **Define OpenAPI Endpoint** in `stock-spec/v1/stock-api-v1.yaml`

2. **Regenerate Clients:**
   ```bash
   mvn clean install
   ```

3. **Create Tool Class** in `stock-ai-client/src/main/java/.../tools/`:

```java
@Component
@RequiredArgsConstructor
public class CustomTools {
    private final CustomApi api;

    @Tool(
        name = "custom_operation",
        description = "Performs a custom business operation"
    )
    public Result customOperation(
        @Param(description = "Input parameter") String input
    ) {
        return api.execute(input);
    }
}
```

4. **Register Tool** in `ChatClientConfig.java`:

```java
@Bean
public ChatClient chatClient(
    ChatClient.Builder builder,
    ProductTools productTools,
    CustomTools customTools
) {
    return builder
        .defaultTools(productTools, customTools)
        .build();
}
```

### Domain Model Changes

1. Update entity in `stock-server-domain/src/main/java/.../model/`
2. Update persistence entity in `stock-server-persistence/src/main/java/.../entity/`
3. Create/update mapper in `stock-server-api/src/main/java/.../mapper/`
4. Regenerate client: `mvn clean install`
5. Update tool wrappers if API changed

### Database Migrations

1. Create migration in `stock-server-persistence/src/main/resources/db/migration/`
2. Run `mvn flyway:migrate` (if Flyway is configured)
3. Or update `init-db.sh` for initial setup changes

---

## 🧪 Testing

### Run All Tests

```bash
mvn clean test
```

### Run Specific Module Tests

```bash
# AI Client tests
mvn test -pl stock-ai-client

# Server tests
mvn test -pl stock-server
```

### Test Coverage Report

```bash
mvn clean test jacoco:report
# Open: target/site/jacoco/index.html
```

### Example Test

```java
@SpringBootTest
@ActiveProfiles("test")
class ChatControllerTest {

    @Autowired
    private ChatClient chatClient;

    @Autowired
    private WireMockServer wireMock;

    @Test
    void shouldAddProductViaChat() {
        // Arrange
        wireMock.stubFor(
            post("/products")
                .willReturn(okJson("{ \"id\": \"123\" }"))
        );

        // Act
        String response = chatClient.call(
            "Add product X with price 99.99"
        );

        // Assert
        assertThat(response).contains("success");
    }
}
```

---

## ⚙️ Configuration

### Environment Variables

Create `.env` or set in `docker-compose.yml`:

```env
# Database
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=stock_assistant
DB_HOST=postgres
DB_PORT=5432

# Ollama/LLM
OLLAMA_HOST=http://ollama:11434
OLLAMA_MODEL=llama2

# Stock Server
STOCK_SERVER_PORT=8081
STOCK_SERVER_CONTEXT_PATH=/api

# Chat Client
CHAT_CLIENT_PORT=8080
CHAT_CLIENT_TIMEOUT=30000

# Spring Profiles
SPRING_PROFILES_ACTIVE=docker
```

### Spring Profiles

- **`default`** – Production settings
- **`dev`** – Development mode with extra logging
- **`test`** – Test configuration with mocked services
- **`docker`** – Container-optimized settings

### Application Properties

Edit `stock-ai-client/src/main/resources/application.yml`:

```yaml
spring:
  ai:
    ollama:
      base-url: ${OLLAMA_HOST}
      chat:
        options:
          model: ${OLLAMA_MODEL}
          temperature: 0.7

stock:
  api:
    base-url: http://stock-server:${STOCK_SERVER_PORT}
    timeout: ${CHAT_CLIENT_TIMEOUT}
```

---

## 🐛 Troubleshooting

### Port Already in Use

```bash
# Find process using port 8080
lsof -i :8080

# Kill process
kill -9 <PID>
```

### PostgreSQL Connection Failed

```bash
# Check if container is running
docker ps | grep postgres

# Check logs
docker logs postgres

# Restart
docker restart postgres
```

### Ollama Model Not Found

```bash
# Pull model
docker exec ollama ollama pull llama2

# List available models
docker exec ollama ollama list
```

### WireMock Stubs Not Working

```bash
# Run tests with debug output
mvn test -X -pl stock-ai-client

# Check stub definitions in test classes
# Look for: wireMock.stubFor(...)
```

### Maven Build Failures

```bash
# Clean build
mvn clean

# Skip tests if urgent
mvn install -DskipTests

# Check for Java version
java -version
```

| Issue | Solution |
|-------|----------|
| OutOfMemory errors | Increase Docker memory: Docker Desktop → Settings → Resources |
| Connection timeouts | Increase `CHAT_CLIENT_TIMEOUT` value |
| Tests fail randomly | Check for port conflicts or timing issues |
| Slow queries | Run `docker exec postgres psql -U postgres -d stock_assistant -c 'ANALYZE;'` |

---

## 📊 Performance Tips

- **LLM Response Times**: Local Ollama is slower than cloud APIs; expect 5-30 seconds
- **Database Queries**: Use JPA specifications for complex filters
- **Caching**: Consider caching product lookups in ChatClient
- **Batch Operations**: Group multiple inventory updates

---

## 🤝 Contributing

Contributions are welcome! Please follow these guidelines:

### Process

1. **Fork** the repository
2. **Create** a feature branch: `git checkout -b feat/amazing-feature`
3. **Commit** with conventional messages: `git commit -m 'feat: add amazing feature'`
4. **Push**: `git push origin feat/amazing-feature`
5. **Open** a Pull Request

### Code Standards

- Follow Spring framework conventions
- Write meaningful variable/method names
- Add tests for new features
- Use SLF4J for logging
- Add JavaDoc for public APIs

### Commit Messages

Use conventional commits:
- `feat:` – New feature
- `fix:` – Bug fix
- `docs:` – Documentation
- `test:` – Test updates
- `refactor:` – Code refactoring
- `chore:` – Build, dependencies

Example:
```
feat: add warehouse capacity limits
```

---

## 📝 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

## 🔗 Resources

- [Spring AI Documentation](https://spring.io/ai)
- [Ollama Documentation](https://github.com/ollama/ollama)
- [OpenAPI Specification](https://spec.openapis.org/oas/v3.0.3)
- [Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/)
- [Domain-Driven Design](https://www.domaindriven.org/)

---

## 📞 Support

**Found an issue?**

1. Check [existing issues](../../issues)
2. Create a [new issue](../../issues/new) with:
   - Clear description
   - Steps to reproduce
   - Expected vs. actual behavior
   - Logs and error messages
   - Environment details (Java version, OS, Docker version)

**Have a question?**

- Open a [Discussion](../../discussions)
- Check [documentation](./docs/)

---

<div align="center">

Made with ❤️ for efficient inventory management

[⬆ back to top](#smart-inventory-assistant)

</div>
