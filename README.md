# 🎫 API de Helpdesk Corporativo V2 & Bug Bounty

Esta API foi desenvolvida para simular um sistema interno de gerenciamento de chamados de TI, evoluindo para um ecossistema com **Programa de Recompensas (Bug Bounty)**. O foco arquitetural do projeto engloba **Bancos de Dados Relacionais**, evolução de *schemas* e proteção de transações financeiras contra falhas de rede/SGBD.

---

## 🛠️ Tecnologias e Padrões Utilizados

* **Java 21+** | **Spring Boot 3.x**
* **PostgreSQL** | **Docker** | **Flyway** (Migrations V1 a V3)
* **Lombok** (Clean Code nas Entidades e DTOs)
* **Transações ACID:** Uso rigoroso de `@Transactional` para garantir o Rollback em operações financeiras.
* **Design Patterns & Domínio Rico:** Uso de interfaces polimórficas para isolar regras de negócio (`AnalisadorDeRecompensa`) e manipulação segura de moeda via `BigDecimal`.

---

## 🏗️ Arquitetura e Evolução do Banco de Dados

* **Módulo de Helpdesk:** Relacionamento `1:N` entre Funcionários e Chamados. Tráfego de identificadores via Cabeçalhos HTTP (`@RequestHeader`) simulando tokens JWT.
* **Módulo de Recompensas:** Evolução do banco em tempo de execução via Flyway (`ALTER TABLE` com `DEFAULT` e restrições `NOT NULL`).
* **Proteção de Duplo Gasto (Double-Spending):** Lógica no Service para impedir pagamentos duplicados, protegida pela bolha transacional do Spring. Se a atualização do saldo falhar, o status do chamado sofre rollback automático.

---

## 🚀 Endpoints da API

### 👤 Funcionários & 🎫 Chamados
* `POST /funcionarios` - Cadastra funcionário (Valida e-mail único).
* `GET /funcionarios` - Lista funcionários e seus saldos.
* `POST /chamados` - Abre chamado (Requer cabeçalho `X-Funcionario-Id`).
* `PUT /chamados/{id}/fechar` - Fecha o chamado.

### 💰 Bug Bounty (Recompensas)
* `POST /recompensa/{id}/recompensar` - Endpoint de processamento financeiro. 
  * Analisa o título/descrição do chamado.
  * Se elegível (ex: "CRITICO" ou "LEVE"), adiciona o valor em `BigDecimal` ao `saldo_bonus` do funcionário.
  * Atualiza o status do chamado para "PAGO".
  * **Proteção:** Reverte (Rollback) toda a operação em caso de falhas.
