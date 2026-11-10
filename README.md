# BI Copilot: Autonomous Natural Language Data Agent

> **Arquitetura de BI Conversacional e Pipeline ETL com Execução Segura de Consultas e Síntese de Decisão.**

> [!NOTE]
> **Nota de Contexto e Evolução Técnica**:
> Este projeto representa uma solução desenvolvida no início da minha trajetória com sistemas de agentes autônomos e orquestração de dados. Preservo esta versão original no portfólio como registro histórico da resolução eficaz do case de negócio na época. Hoje, minhas implementações contam com padrões arquiteturais significativamente mais maduros — incluindo pipelines de avaliação contínua (LLM-as-a-judge), guardrails semânticos avançados, camadas de observabilidade distribuída (OpenTelemetry/Langfuse) e orquestrações modulares prontas para escala enterprise de alta criticidade.

---

## 1. Contexto e Desafio de Negócio

Em ambientes corporativos com múltiplos painéis de Business Intelligence (Power BI, Tableau, Metabase), lideranças enfrentam o fenômeno de sobrecarga informacional:
- **Atrito de Acesso**: Dificuldade em localizar métricas cruciais dispersas em múltiplos relatórios.
- **Baixa Adoção**: Restrição de tempo para navegação em ferramentas de desktop ou layouts complexos.
- **Demanda por Mobilidade**: Necessidade de obter respostas analíticas confiáveis diretamente nos canais de comunicação corporativos (WhatsApp, Slack, Teams) em menos de 10 segundos.
- **Risco de Informação Não Auditada**: Bases brutas e não padronizadas geram divergências de relatórios.

---

## 2. Solução Proposta

Copilot de BI operando como ponte entre linguagem natural e bancos de dados relacionais auditados.

```
[ Usuário / Canal de Mensageria ]
              │ (Pergunta em Linguagem Natural)
              ▼
   [ n8n: Orquestrador de Agente ]
              │ (Buffer Memory + Contexto)
              ▼
   [ LLM: Claude 3.5 Sonnet / Gemini ]
              │ (Geração de SQL Parametrizado)
              ▼
   [ Sub-Workflow Isolado: SQL Executor ]
              │ (RBAC: SELECT apenas + LIMIT 100)
              ▼
   [ Banco de Dados: Supabase / PostgreSQL ] (Gold Layer Auditada)
              │ (Retorno de Dados)
              ▼
   [ Síntese Analítica + Insight Estratégico ]
              │
              ▼
[ Resposta Formatada em < 10s no Chat ]
```

---

## 3. Escolha de Ferramentas e Alinhamento Técnico

A seleção da stack atendeu diretamente às preferências e aos requisitos operacionais solicitados na avaliação:

| Componente | Tecnologia | Justificativa Técnica |
| :--- | :--- | :--- |
| **Orquestrador** | **n8n** | Automação de fluxos e orquestração de agentes LangChain com baixo custo de manutenção e facilidade de ajuste contínuo. |
| **Banco de Dados** | **Supabase / PostgreSQL** | Fonte única da verdade (*Gold Layer*), combinando performance relacional, segurança por perfil e escalabilidade. |
| **Motor de Raciocínio (LLM)** | **Claude 3.5 Sonnet / Gemini** | Raciocínio lógico estruturado para geração precisa de queries SQL sem alucinação aritmética. |
| **Canal de Entrega** | **Mensageria (WhatsApp / Teams)** | Elimina atrito de login e treinamento, entregando métricas diretamente na rotina do gestor. |

---

## 4. Riscos e Mitigações de Segurança

1. **Prevenção de Injeção SQL**: O executor opera em sub-workflow isolado sob usuário de banco com privilégio estritamente mínimo (`GRANT SELECT`). Bloqueio de queries mutáveis (`DROP`, `DELETE`, `UPDATE`).
2. **Eliminação de Alucinações**: A LLM não processa agregações aritméticas diretas em texto; gera a query exata contra o schema formal (`Semantic Layer / Dicionário de Dados`).
3. **FinOps e Performance**: Paginação forçada (`LIMIT 100`) e estimativa de plano de custo via `EXPLAIN` para prevenir sobrecarga de leitura.
4. **Governança de Dados**: Camada de ETL dedicada para deduplicação, normalização e carga auditada antes de qualquer exposição à camada de IA.

---

## 5. Estrutura de Arquivos

```
bi-copilot-agent/
├── README.md
├── .env.example
├── .gitignore
└── workflows/
    ├── 01_bi_copilot_agent.json     # Orquestrador do Agente (LangChain + Prompt Executivo + Memory)
    ├── 02_data_sync_pipeline.json   # Pipeline ETL (Ingestão, Limpeza, Deduplicação e Carga)
    └── 03_tool_sql_executor.json    # Sub-workflow Tool para execução segura de queries SQL
```

---

## 6. Instruções de Execução

1. Importar os arquivos JSON em uma instância **n8n** (v1.x+).
2. Configurar as credenciais no ambiente:
   - Google Sheets / Fonte de dados bruta.
   - Supabase / PostgreSQL (usuário com privilégios apenas de leitura).
   - Provedor de LLM (Google Gemini / Anthropic Claude).
3. Executar o workflow `02_data_sync_pipeline.json` para processar e carregar a camada auditada `sales_data`.
4. Ativar `03_tool_sql_executor.json` e associar o identificador ao nó de ferramenta em `01_bi_copilot_agent.json`.
5. Iniciar consultas via Chat Trigger ou webhook de mensageria.

---

## 7. Licença

Distribuído sob licença MIT. Autor: [Matheus Santos de Paula](https://github.com/TheMatheusSantos).
