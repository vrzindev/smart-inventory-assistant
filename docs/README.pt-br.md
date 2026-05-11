# Smart Inventory Assistant

<div align="center">

[![Java](https://img.shields.io/badge/Java-17+-ED8B00?style=flat-square&logo=openjdk)](https://www.oracle.com/java/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x-6DB33F?style=flat-square&logo=springboot)](https://spring.io/projects/spring-boot)
[![Spring AI](https://img.shields.io/badge/Spring%20AI-1.0.0--M6-6DB33F?style=flat-square&logo=spring)](https://spring.io/projects/spring-ai)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15+-316192?style=flat-square&logo=postgresql)](https://www.postgresql.org/)
[![Docker](https://img.shields.io/badge/Docker-Supported-2496ED?style=flat-square&logo=docker)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg?style=flat-square)](../LICENSE)

**[English](../README.md)** • **Português Brasileiro**

> Sistema de gerenciamento de inventário com IA conversacional construído com Spring AI e Ollama

</div>

---

## 📋 Índice

- [Visão Geral](#visão-geral)
- [Características](#características)
- [Arquitetura](#arquitetura)
- [Início Rápido](#início-rápido)
- [Exemplos de Uso](#exemplos-de-uso)
- [Estrutura do Projeto](#estrutura-do-projeto)
- [Stack de Tecnologias](#stack-de-tecnologias)
- [Desenvolvimento](#desenvolvimento)
- [Testes](#testes)
- [Solução de Problemas](#solução-de-problemas)
- [Contribuindo](#contribuindo)
- [Licença](#licença)

---

## Visão Geral

**Smart Inventory Assistant** é um proof-of-concept (POC) que demonstra como integrar **Spring AI** em aplicações Spring Boot tradicionais usando **design API-first** e **arquitetura hexagonal limpa**.

Em vez de navegar por dropdowns e formulários complexos, usuários interagem com o gerenciamento de inventário através de **conversas em linguagem natural**—pedindo para encontrar produtos com preço abaixo de um limite, adicionar estoque em armazéns, ou atualizar contagens de inventário em português simples.

Este projeto explora a integração prática de LLMs em sistemas de negócio existentes sem exigir rewrites arquiteturais, provando que UIs aprimoradas por IA podem coexistir perfeitamente com código limpo e manutenível.

---

## ✨ Características

- 🗣️ **Interface em Linguagem Natural** – Gerencie inventário via prompts conversacionais
- 🤖 **Integração Spring AI** – Suporte a LLM local com Ollama (sem chamadas para APIs externas)
- 🏗️ **Arquitetura Hexagonal** – Separação limpa de responsabilidades com ports & adapters
- 📡 **Design API-First** – Especificação OpenAPI com clientes auto-gerados
- 🧪 **Totalmente Testado** – Testes unitários e de integração com WireMock
- 🐳 **Pronto para Docker** – Setup containerizado completo com docker-compose
- 🔧 **Extensível** – Adicione facilmente novas ferramentas e capacidades

---

## 🏛️ Arquitetura

```
┌──────────────────────────────────────────────────────┐
│         Frontend/Cliente                             │
│     (Interface de Chat em Linguagem Natural)         │
└────────────────┬──────────────────────────────────────┘
                 │ HTTP/REST
                 ▼
┌──────────────────────────────────────────────────────┐
│  Cliente Spring AI Chat (stock-ai-client)            │
│  • Processamento de Prompts                          │
│  • Orquestração de Ferramentas (beans @Tool)         │
│  • Integração com LLM                                │
└────────────────┬─────────────────────────────────────┘
                 │ Clientes OpenAPI Gerados
                 ▼
┌──────────────────────────────────────────────────────┐
│  Servidor Stock (Arquitetura Hexagonal)              │
│                                                      │
│  ┌────────────────────────────────────────────┐    │
│  │ Controladores HTTP (Portas)                │    │
│  └────────────┬─────────────────────────────┬─┘    │
│               │                             │       │
│  ┌────────────▼──────┐        ┌────────────▼─┐   │
│  │ Lógica de Aplicação│◄──────►│ Modelos de   │   │
│  │ (Casos de Uso)     │        │ Domínio(Entid│   │
│  └────────────┬──────┘        └────────────┬─┘   │
│               │ Repositórios              │       │
│  ┌────────────▼──────────────────────────▼─┐    │
│  │ Adaptadores de Persistência & Especs    │    │
│  └────────────┬──────────────────────────┬─┘    │
└───────────────┼──────────────────────────┼──────┘
                │                          │
         ┌──────▼──────┐          ┌────────▼──────┐
         │ PostgreSQL  │          │ Ollama (LLM)  │
         └─────────────┘          └───────────────┘
```

---

## 🚀 Início Rápido

### Pré-requisitos

- **Java** 17+
- **Maven** 3.8+
- **Docker & Docker Compose** (recomendado)

### Opção 1: Docker Compose (Recomendado)

```bash
git clone https://github.com/seu-usuario/smart-inventory-assistant.git
cd smart-inventory-assistant

# Inicie todos os serviços
docker-compose up -d

# Visualize os logs
docker-compose logs -f stock-ai-client
```

**Serviços:**
- `stock-ai-client`: Endpoint de chat - http://localhost:8080
- `stock-server`: API Backend - http://localhost:8081
- `postgres`: Banco de dados - localhost:5432
- `ollama`: Runtime LLM - localhost:11434

### Opção 2: Desenvolvimento Local

```bash
# Clone o repositório
git clone https://github.com/seu-usuario/smart-inventory-assistant.git
cd smart-inventory-assistant

# Build
mvn clean install -DskipTests

# Terminal 1: Inicie as dependências
docker run -d --name postgres -e POSTGRES_PASSWORD=postgres -p 5432:5432 postgres:15
docker run -d --name ollama -p 11434:11434 ollama/ollama
docker exec ollama ollama pull llama2

# Terminal 2: Inicie o servidor
cd stock-server/stock-server-starter
mvn spring-boot:run

# Terminal 3: Inicie o cliente de IA
cd stock-ai-client
mvn spring-boot:run
```

**Endpoint de Chat:**
```
http://localhost:8080/api/chat/process
```

---

## 💬 Exemplos de Uso

### 1. Criar um Produto

**Requisição:**
```json
POST /api/chat/process
{
  "prompt": "Adicione um produto chamado Mouse Wireless com SKU MOU-987, categoria ELETRONICOS, unidade UNIT, preço 29.99"
}
```

**Resposta da IA:**
```
✓ Produto "Mouse Wireless" (SKU: MOU-987) criado na categoria ELETRONICOS - €29,99
```

**Chamada de API Gerada:**
```http
POST /api/v1/products
{
  "sku": "MOU-987",
  "name": "Mouse Wireless",
  "category": "ELETRONICOS",
  "unitOfMeasure": "UNIT",
  "price": 29.99
}
```

---

### 2. Buscar Produtos

**Requisição:**
```json
POST /api/chat/process
{
  "prompt": "Encontre todos os produtos na categoria ELETRONICOS com preço abaixo de 50 euros"
}
```

**Resposta da IA:**
```
Encontrados 3 produtos:
- Mouse Wireless (€29,99)
- Hub USB (€34,99)
- Teclado (€44,99)
```

---

### 3. Atualizar Estoque

**Requisição:**
```json
POST /api/chat/process
{
  "prompt": "Adicione 150 unidades de MOU-987 ao armazém WAREHOUSE-001"
}
```

**Resposta da IA:**
```
✓ Adicionadas 150 unidades ao armazém. Estoque atual: 250 unidades
```

---

### 4. Gerenciar Inventário

**Requisição:**
```json
POST /api/chat/process
{
  "prompt": "Qual é o nível total de estoque para todos os produtos da categoria ELETRONICOS?"
}
```

**Resposta da IA:**
```
Estoque total ELETRONICOS: 1.428 unidades em 3 armazéns
- WAREHOUSE-001: 450 unidades
- WAREHOUSE-002: 320 unidades
- WAREHOUSE-003: 658 unidades
```

---

### 5. Excluir Produto

**Requisição:**
```json
POST /api/chat/process
{
  "prompt": "Delete o produto com SKU MOU-987"
}
```

**Resposta da IA:**
```
✓ Produto deletado com sucesso
```

---

## 📁 Estrutura do Projeto

```
smart-inventory-assistant/
│
├── README.md                          # Arquivo principal (English)
├── docker-compose.yml                 # Orquestração de múltiplos contêineres
├── Dockerfile                         # Imagem do servidor stock
├── pom.xml                            # POM pai do Maven
│
├── stock-spec/                        # Especificação OpenAPI
│   └── v1/stock-api-v1.yaml          # Contratos da API REST
│
├── stock-server/                      # Backend (Hexagonal)
│   ├── stock-server-api/              # Camada HTTP
│   │   ├── controller/                # Endpoints REST
│   │   └── mapper/                    # Mapeamento DTO/Entidade
│   │
│   ├── stock-server-domain/           # Lógica de Negócio
│   │   ├── feature/                   # Features de domínio (inventário, produto, armazém)
│   │   └── model/                     # Entidades de domínio
│   │
│   ├── stock-server-persistence/      # Acesso a Dados
│   │   ├── adapter/                   # Adaptadores JPA
│   │   ├── entity/                    # Entidades JPA
│   │   ├── repository/                # Repositórios Spring Data
│   │   └── specification/             # Especificações JPA
│   │
│   └── stock-server-starter/          # Aplicação Spring Boot
│       └── StockServerStarterApplication.java
│
├── stock-ai-client/                   # Cliente de IA (Spring AI)
│   ├── config/                        # Configuração Spring
│   │   ├── ApiClientConfig.java       # Clientes de API gerados
│   │   └── ChatClientConfig.java      # ChatClient do Spring AI
│   │
│   ├── controller/                    # Endpoint de chat
│   │   └── ChatController.java
│   │
│   ├── model/                         # Classes DTO
│   │   └── PromptRequest.java
│   │
│   ├── tools/                         # Beans Spring AI @Tool
│   │   ├── ProductTools.java          # Operações de produtos
│   │   ├── InventoryTools.java        # Operações de inventário
│   │   └── WarehousesTools.java       # Operações de armazéns
│   │
│   └── test/                          # Testes (stubs WireMock)
│       └── ProductChatControllerTest.java
│
└── docs/
    └── README.pt-br.md                # Versão em português
```

### Módulos Principais

| Módulo | Responsabilidade |
|--------|-----------------|
| **stock-spec** | Fonte da verdade - contratos OpenAPI |
| **stock-server-api** | Camada HTTP/REST, validação de requisição |
| **stock-server-domain** | Regras de negócio, agregados, casos de uso |
| **stock-server-persistence** | Adaptadores de banco de dados, queries, mapeadores |
| **stock-server-starter** | Bootstrap do Spring Boot, configuração |
| **stock-ai-client** | Integração com LLM, orquestração de ferramentas, UI de chat |

---

## 🛠️ Stack de Tecnologias

### Framework Principal
- **Java 17+** – Linguagem
- **Spring Boot 3.x** – Framework de aplicação
- **Maven** – Automatização de build
- **PostgreSQL 15+** – Banco de dados relacional

### IA e LLM
- **Spring AI 1.0.0-M6** – Biblioteca de integração com LLM
- **Ollama** – Runtime LLM local
- **Llama 2** – Modelo de linguagem padrão

### API e Geração de Código
- **OpenAPI 3.0** – Especificação de API
- **OpenAPI Generator** – Geração de stubs de cliente

### Testes
- **JUnit 5** – Testes unitários
- **Mockito** – Mock de dependências
- **WireMock** – Servidor stub/mock HTTP
- **Spring Boot Test** – Testes de integração

### Padrões de Arquitetura
- **Arquitetura Hexagonal** – Ports & adapters
- **Design Dirigido por Domínio** – Modelagem de entidades
- **Padrão Repository** – Abstração de dados

---

## 📖 Guia de Desenvolvimento

### Adicionando uma Nova Ferramenta de Chat

1. **Defina o Endpoint OpenAPI** em `stock-spec/v1/stock-api-v1.yaml`

2. **Regenere os Clientes:**
   ```bash
   mvn clean install
   ```

3. **Crie a Classe de Ferramenta** em `stock-ai-client/src/main/java/.../tools/`:

```java
@Component
@RequiredArgsConstructor
public class CustomTools {
    private final CustomApi api;

    @Tool(
        name = "custom_operation",
        description = "Realiza uma operação de negócio customizada"
    )
    public Result customOperation(
        @Param(description = "Parâmetro de entrada") String input
    ) {
        return api.execute(input);
    }
}
```

4. **Registre a Ferramenta** em `ChatClientConfig.java`:

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

### Alterações em Modelos de Domínio

1. Atualize a entidade em `stock-server-domain/src/main/java/.../model/`
2. Atualize a entidade de persistência em `stock-server-persistence/src/main/java/.../entity/`
3. Crie/atualize o mapeador em `stock-server-api/src/main/java/.../mapper/`
4. Regenere o cliente: `mvn clean install`
5. Atualize os wrappers de ferramentas se a API mudou

### Migrações de Banco de Dados

1. Crie migração em `stock-server-persistence/src/main/resources/db/migration/`
2. Execute `mvn flyway:migrate` (se Flyway estiver configurado)
3. Ou atualize `init-db.sh` para mudanças de setup inicial

---

## 🧪 Testes

### Executar Todos os Testes

```bash
mvn clean test
```

### Executar Testes de Módulo Específico

```bash
# Testes do cliente de IA
mvn test -pl stock-ai-client

# Testes do servidor
mvn test -pl stock-server
```

### Relatório de Cobertura de Testes

```bash
mvn clean test jacoco:report
# Abra: target/site/jacoco/index.html
```

### Exemplo de Teste

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
        // Arranjar
        wireMock.stubFor(
            post("/products")
                .willReturn(okJson("{ \"id\": \"123\" }"))
        );

        // Agir
        String response = chatClient.call(
            "Adicione produto X com preço 99.99"
        );

        // Afirmar
        assertThat(response).contains("sucesso");
    }
}
```

---

## ⚙️ Configuração

### Variáveis de Ambiente

Crie `.env` ou configure em `docker-compose.yml`:

```env
# Banco de Dados
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=stock_assistant
DB_HOST=postgres
DB_PORT=5432

# Ollama/LLM
OLLAMA_HOST=http://ollama:11434
OLLAMA_MODEL=llama2

# Servidor Stock
STOCK_SERVER_PORT=8081
STOCK_SERVER_CONTEXT_PATH=/api

# Cliente de Chat
CHAT_CLIENT_PORT=8080
CHAT_CLIENT_TIMEOUT=30000

# Perfis Spring
SPRING_PROFILES_ACTIVE=docker
```

### Perfis Spring

- **`default`** – Configurações de produção
- **`dev`** – Modo desenvolvimento com logging extra
- **`test`** – Configuração de teste com serviços mockados
- **`docker`** – Configurações otimizadas para contêiner

### Propriedades da Aplicação

Edite `stock-ai-client/src/main/resources/application.yml`:

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

## 🐛 Solução de Problemas

### Porta Já em Uso

```bash
# Encontre o processo usando a porta 8080
lsof -i :8080

# Mate o processo
kill -9 <PID>
```

### Falha de Conexão PostgreSQL

```bash
# Verifique se o contêiner está em execução
docker ps | grep postgres

# Verifique os logs
docker logs postgres

# Reinicie
docker restart postgres
```

### Modelo Ollama Não Encontrado

```bash
# Baixe o modelo
docker exec ollama ollama pull llama2

# Liste modelos disponíveis
docker exec ollama ollama list
```

### Stubs WireMock Não Funcionando

```bash
# Execute testes com saída de debug
mvn test -X -pl stock-ai-client

# Verifique definições de stub em testes
# Procure por: wireMock.stubFor(...)
```

### Falhas de Build Maven

```bash
# Clean build
mvn clean

# Pule testes se urgente
mvn install -DskipTests

# Verifique versão do Java
java -version
```

| Problema | Solução |
|----------|---------|
| Erros OutOfMemory | Aumente memória Docker: Docker Desktop → Settings → Resources |
| Timeouts na conexão | Aumente o valor de `CHAT_CLIENT_TIMEOUT` |
| Testes falhando aleatoriamente | Verifique conflitos de portas ou problemas de timing |
| Queries lentas | Execute `docker exec postgres psql -U postgres -d stock_assistant -c 'ANALYZE;'` |

---

## 📊 Dicas de Performance

- **Tempos de Resposta do LLM**: Ollama local é mais lento que APIs em cloud; espere 5-30 segundos
- **Queries de Banco de Dados**: Use especificações JPA para filtros complexos
- **Caching**: Considere cache para lookups de produtos no ChatClient
- **Operações em Lote**: Agrupe múltiplas atualizações de inventário

---

## 🤝 Contribuindo

Contribuições são bem-vindas! Por favor, siga estas diretrizes:

### Processo

1. **Fork** o repositório
2. **Crie** uma branch de feature: `git checkout -b feat/recurso-incrivel`
3. **Commit** com mensagens convencionais: `git commit -m 'feat: adicionar recurso incrível'`
4. **Push**: `git push origin feat/recurso-incrivel`
5. **Abra** um Pull Request

### Padrões de Código

- Siga as convenções do framework Spring
- Escreva nomes significativos de variáveis/métodos
- Adicione testes para novas funcionalidades
- Use SLF4J para logging
- Adicione JavaDoc para APIs públicas

### Mensagens de Commit

Use conventional commits:
- `feat:` – Nova funcionalidade
- `fix:` – Correção de bug
- `docs:` – Documentação
- `test:` – Atualizações de testes
- `refactor:` – Refatoração de código
- `chore:` – Build, dependências

Exemplo:
```
feat: adicionar limites de capacidade de armazém
```

---

## 📝 Licença

Este projeto é licenciado sob a Licença MIT. Veja o arquivo [LICENSE](../LICENSE) para detalhes.

---

## 🔗 Recursos

- [Documentação Spring AI](https://spring.io/ai)
- [Documentação Ollama](https://github.com/ollama/ollama)
- [Especificação OpenAPI](https://spec.openapis.org/oas/v3.0.3)
- [Arquitetura Hexagonal](https://alistair.cockburn.us/hexagonal-architecture/)
- [Design Dirigido por Domínio](https://www.domaindriven.org/)

---

## 📞 Suporte

**Encontrou um problema?**

1. Verifique [issues existentes](../../issues)
2. Crie uma [nova issue](../../issues/new) com:
   - Descrição clara
   - Passos para reproduzir
   - Comportamento esperado vs. atual
   - Logs e mensagens de erro
   - Detalhes do ambiente (versão Java, SO, versão Docker)

**Tem uma pergunta?**

- Abra uma [Discussão](../../discussions)
- Verifique [documentação](../docs/)

---

<div align="center">

Feito com ❤️ para gerenciamento eficiente de inventário

[⬆ voltar ao topo](#smart-inventory-assistant)

</div>
