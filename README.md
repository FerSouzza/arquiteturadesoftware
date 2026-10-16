# 🏋️‍♂️ Platforma Open-Source de Gestão e Execução de Treinos (PWA / SPA)

Plataforma *open-source* projetada para conectar professores de educação física e alunos. A solução permite que professores montem rotinas de treino personalizadas com agilidade via desktop, enquanto alunos executam e acompanham seus treinos em tempo real na academia através de um Progressive Web App (PWA) responsivo e resiliente a falhas de conexão.

---

## 📑 Registros de Decisões Arquiteturais (ADRs)

Este projeto segue o padrão **Docs-as-Code** para registrar suas decisões arquiteturais de forma transparente e versionada junto ao código-fonte.

---

### 🎯 Módulo de Planejamento e Arquitetura do Produto

#### ADR-P01 — Direcionadores arquiteturais
* **Status:** `Proposed`
* **Contexto:** Garantir que o aluno execute o treino no chão da academia mesmo com oscilações de rede (4G/5G/Wi-Fi), e que o professor monte fichas com rapidez via navegador Web (SPA).
* **Decisão:** Adoção de arquitetura **Offline-First via PWA (SPA)**. Priorização dos atributos de qualidade:
  1. **Disponibilidade Offline**
  2. **Desempenho**
  3. **Usabilidade**
  4. **Modularidade e Extensibilidade**
* **Justificativa:** O aluno precisa acessar treinos e mídias sem internet. O uso de Service Workers e banco de dados local no navegador elimina a dependência contínua de rede móvel.
* **Consequências:**
  * 🟢 *Positivas:* Treino funciona perfeitamente sem internet; desenvolvimento 100% focado no ecossistema Web.
  * 🔴 *Negativas:* Necessidade de gerenciar sincronização delta de dados e cotas de armazenamento local no navegador.

#### ADR-P02 — Estratégia de evolução do produto
* **Status:** `Proposed`
* **Contexto:** O projeto é software livre e deve permitir a adição de novos extratores de exercícios, mídias e modelos de IA sem alterar ou desacelerar o núcleo da aplicação.
* **Decisão:** Arquitetura de **Plugins Core-Extension (Microkernel)** baseada em contratos e interfaces bem definidas.
* **Justificativa:** Protege o núcleo do sistema, permitindo que contribuidores externos adicionem novas funcionalidades de forma isolada e desacoplada.
* **Consequências:**
  * 🟢 *Positivas:* Extensibilidade simples e isolamento de falhas de código de terceiros.
  * 🔴 *Negativas:* Exige rigor no versionamento de interfaces e hooks do sistema.

---

### 💻 Módulo de Tecnologia

#### ADR-T01 — Stack tecnológica
* **Status:** `Proposed`
* **Contexto:** Seleção da stack para a SPA unificada (aluno e professor) e a API backend.
* **Decisão:** **React (SPA/PWA)** no frontend, **Node.js (NestJS)** em TypeScript no backend e **PostgreSQL** como banco de dados relacional.
* **Justificativa:** Unificação da linguagem em TypeScript, alta produtividade, PWA instalável direto do navegador (sem dependência de lojas de aplicativos) e forte suporte a bibliotecas offline.
* **Consequências:**
  * 🟢 *Positivas:* Reuso de código e tipos (DTOs) entre cliente e servidor; desenvolvimento acelerado.
  * 🔴 *Negativas:* Tratar limitações pontuais de PWA em navegadores do ecossistema iOS (Safari).

#### ADR-T02 — Estratégia de armazenamento local e em nuvem
* **Status:** `Proposed`
* **Contexto:** Definir como armazenar dados estruturados, histórico e mídias de exercícios no cliente e no servidor.
* **Decisão:** **Armazenamento Híbrido**: **IndexedDB (Dexie.js)** + **Cache API** no navegador; **PostgreSQL** + **Object Storage (S3/MinIO)** na nuvem.
* **Justificativa:** O IndexedDB guarda as fichas de treino e a Cache API salva GIFs/vídeos curtos no celular, permitindo o funcionamento offline do PWA.
* **Consequências:**
  * 🟢 *Positivas:* Treinos executam instantaneamente sem internet; menor consumo do plano de dados móveis.
  * 🔴 *Negativas:* Exige controle de cota de armazenamento no navegador e tratamento de conflitos ao sincronizar.

---

### 🏗️ Módulo de Desenvolvimento e Arquitetura do Sistema

#### ADR-D01 — Estilo arquitetural
* **Status:** `Proposed`
* **Contexto:** Definir o estilo do backend e deliberar sobre a necessidade de microsserviços.
* **Decisão:** **Monolito Modular baseado em Clean Architecture / Hexagonal**. **Não serão utilizados microsserviços**.
* **Justificativa:** Evita a alta complexidade e os custos operacionais de microsserviços, mantendo a organização por camadas e portas/adaptadores em um deploy único e enxuto.
* **Consequências:**
  * 🟢 *Positivas:* Baixa complexidade de infraestrutura, facilidade de testes e
