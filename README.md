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
  * 🟢 *Positivas:* Baixa complexidade de infraestrutura, facilidade de testes e respostas rápidas da API.
  * 🔴 *Negativas:* Exige disciplina para respeitar os limites entre módulos no código.

#### ADR-D02 — Modularização do sistema
* **Status:** `Proposed`
* **Contexto:** Divisão do sistema em componentes para reduzir acoplamento.
* **Decisão:** Divisão por **Domínio de Negócio (Bounded Contexts)** em 5 módulos principais:
  1. **Usuários e Perfis:** Cadastro e autenticação de Alunos e Professores.
  2. **Catálogo e Mídias:** Exercícios, grupos musculares e mídias de demonstração.
  3. **Gestão de Treinos:** Criação e parametrização de fichas pelo professor.
  4. **Execução do Aluno:** Acompanhamento em tempo real, timer e marcação de séries.
  5. **IA e Recomendação:** Sugestão inteligente e busca por linguagem natural.
* **Justificativa:** Mantém alta coesão e permite alterações em um domínio sem impactar os demais.
* **Consequências:**
  * 🟢 *Positivas:* Organização clara do projeto e facilidade na atribuição de tarefas na equipe.
  * 🔴 *Negativas:* Necessidade de especificação de DTOs para comunicação entre módulos.

---

### 🧠 Módulo de Inteligência Artificial

#### ADR-IA01 — Arquitetura para integração de algoritmos de IA
* **Status:** `Proposed`
* **Contexto:** Incorporar algoritmos de sugestão e substituição de exercícios sem acoplar a IA ao núcleo do backend.
* **Decisão:** Pattern **Strategy / Adapter** consumindo um **serviço auxiliar em Python (FastAPI)** com **estratégia de Fallback**.
* **Justificativa:** O ecossistema Python é ideal para IA/ML. Se o serviço de IA falhar ou estiver indisponível, a aplicação aciona o fallback e permite a montagem manual pelo professor sem quebrar a SPA.
* **Consequências:**
  * 🟢 *Positivas:* Total isolamento entre a IA e o backend principal; alta resiliência a falhas.
  * 🔴 *Negativas:* Necessidade de manter dois serviços em execução no servidor.

#### ADR-IA02 — Representação para busca semântica
* **Status:** `Proposed`
* **Contexto:** Permitir buscas por linguagem natural (ex: *"exercício para peito sem halteres para quem tem dor no ombro"*).
* **Decisão:** Geração de **Embeddings** e busca vetorial utilizando a extensão **`pgvector` no PostgreSQL**.
* **Justificativa:** Permite busca semântica e filtros relacionais (SQL) na mesma consulta, sem a necessidade de manter um banco de dados vetorial dedicado.
* **Consequências:**
  * 🟢 *Positivas:* Infraestrutura simplificada e respostas de busca rápidas para a SPA.
  * 🔴 *Negativas:* Requer a inclusão do plugin `pgvector` na imagem Docker do banco de dados.

---

### 🎨 Módulo de Interface (UX)

#### ADR-UX01 — Estratégia de interface multiplataforma
* **Status:** `Proposed`
* **Contexto:** Atender ao professor (Desktop, montagem ágil) e ao aluno (Mobile, uso com uma mão só na academia).
* **Decisão:** **SPA Responsiva Unificada (React PWA)**.
* **Justificativa:** Base de código única que adapta a interface conforme o tamanho de tela, elimina a necessidade de publicação em lojas de aplicativos e roda direto no navegador.
* **Consequências:**
  * 🟢 *Positivas:* Menor custo de desenvolvimento e atualizações instantâneas.
  * 🔴 *Negativas:* Dependência de suporte do navegador para recursos avançados de hardware em background.

#### ADR-UX02 — Estratégia de interação para exploração da biblioteca e montagem
* **Status:** `Proposed`
* **Contexto:** Facilitar a criação de treinos pelo professor e a execução intuitiva pelo aluno.
* **Decisão:** **Drag-and-Drop em colunas (Desktop/Professor)** + **Cards em fluxo único com Timer automático (Mobile/Aluno)**.
* **Justificativa:** O professor arrasta exercícios para a ficha rapidamente, enquanto o aluno visualiza um exercício por vez com foco em marcar séries e descansar.
* **Consequências:**
  * 🟢 *Positivas:* Montagem de treinos ultrarrápida e interface sem distrações no celular.
  * 🔴 *Negativas:* Exige rigor no desenvolvimento de componentes touch/drag-and-drop responsivos.

---

### 🧪 Módulo de Qualidade e Testes (QA)

#### ADR-QA01 — Estratégia de testabilidade
* **Status:** `Proposed`
* **Contexto:** Garantir testes automatizados rápidos e isolados.
* **Decisão:** **Pirâmide de Testes**: Foco principal em **Testes Unitários de Regra de Negócio** + **Testes de Componentes (SPA)** + **Testes de API**.
* **Justificativa:** A Injeção de Dependências da Clean Architecture permite testar regras sem subir banco de dados ou navegador, garantindo execuções em poucos segundos na esteira de CI.
* **Consequências:**
  * 🟢 *Positivas:* Suíte de testes extremamente rápida e confiável.
  * 🔴 *Negativas:* Necessidade de criar e manter mocks de dependências nos testes.

#### ADR-QA02 — Estratégia de testes para componentes externos
* **Status:** `Proposed`
* **Contexto:** Testar integrações (S3, IA, IndexedDB) sem dependência de serviços pagos ou internet.
* **Decisão:** **Emulação via LocalStack** (containers de S3) + **Mocks de APIs do Navegador (`fake-indexeddb`)**.
* **Justificativa:** Permite rodar toda a suíte de testes de forma determinística, offline e com custo zero.
* **Consequências:**
  * 🟢 *Positivas:* Testes independentes de conexão e reprodutíveis em qualquer máquina.
  * 🔴 *Negativas:* Manutenção dos stubs de APIs do navegador no ambiente de testes.

---

### 📖 Módulo de Documentação e Governança

#### ADR-DOC01 — Estratégia de documentação arquitetural
* **Status:** `Proposed`
* **Contexto:** Registrar a arquitetura de forma transparente e atualizada.
* **Decisão:** **Docs-as-Code**: Arquivos **Markdown**, diagramas **Mermaid.js** no Git (`/docs`) e especificação **OpenAPI/Swagger** gerada pela API.
* **Justificativa:** A documentação é versionada junto com o código nos Pull Requests, permanecendo sempre atualizada e acessível no repositório.
* **Consequências:**
  * 🟢 *Positivas:* Documentação mantida junto do código, rastreável e sem custo de ferramentas terceiras.
  * 🔴 *Negativas:* Exige rigor nos Code Reviews para garantir atualização dos arquivos `.md`.

#### ADR-DOC02 — Estratégia de documentação para extensões
* **Status:** `Proposed`
* **Contexto:** Permitir que desenvolvedores externos criem novos plugins para a plataforma open-source.
* **Decisão:** **Guia `CONTRIBUTING-PLUGINS.md`** + **Interfaces Tipadas públicas** + **Projetos de Exemplo em `/examples`**.
* **Justificativa:** Disponibilizar exemplos funcionais prontos (boilerplates) permite que novos contribuidores criem extensões por clonagem e adaptação sem precisar entender todo o código-fonte.
* **Consequências:**
  * 🟢 *Positivas:* Expansão segura do ecossistema por contribuidores da comunidade.
  * 🔴 *Negativas:* Exige atualização periódica dos projetos de exemplo pela equipe principal.

---

## 🛠️ Como Contribuir

Consulte o arquivo [`CONTRIBUTING-PLUGINS.md`](./docs/CONTRIBUTING-PLUGINS.md) para entender como criar novos extratores de mídias ou algoritmos de recomendação para a plataforma.
