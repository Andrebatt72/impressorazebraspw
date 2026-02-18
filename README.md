# 📘 Documentação Oficial — Arquitetura de Impressão Zebra (Saveinformatica)

**Versão 1.0 — Formato Markdown (para hospedagem no GitHub Pages)**  
**Data:** 18/02/2026  
**Autor:** André Di Battista 

---

## 1. 🎯 Resumo Executivo

A Saveinformatica necessita substituir o aplicativo local “**savecloud**”, que acessa diretamente o banco de dados de produção e cria tabelas temporárias com privilégios elevados, por uma arquitetura **segura**, **multicliente** e **totalmente desacoplada** do banco.

A nova solução utiliza:

- **Aplicativo Cloud (App A)** — responsável por orquestrar jobs, registrar impressoras, e atualizar o ERP.  
- **Agente Local (App B)** — rodando no cliente, responsável por imprimir localmente o ZPL/PRN e confirmar execução.

---

## 2. 🧱 Arquitetura Geral da Solução

### 2.1 Visão Geral — Diagrama (Mermaid)

```mermaid
flowchart LR
    ERP["ERP - App Service"]
    DB["Azure SQL - Tabelas SPOOL_ZEBRA e portalImpressora"]
    AppA["App A - Cloud API - Orquestracao e Multicliente"]
    AppB["App B - Agente Local - Cliente"]
    Zebra["Impressora Zebra - ZPL via TCP 9100"]

    ERP -->|Gera Job P| DB
    DB -->|Consulta via DAO interno| AppA
    AppA -->|Distribui Job P| AppB
    AppB -->|Envia ZPL TCP 9100| Zebra
    AppB -->|ACK OK ou ERRO| AppA
    AppA -->|Atualiza status E ou ERRO| DB
```

---

## 3. 🖨️ Fluxo Completo de Impressão

### 3.1 Diagrama de Fluxo do Job

```mermaid
sequenceDiagram
    participant ERP as ERP
    participant DB as Azure SQL
    participant AppA as App A (Cloud)
    participant AppB as App B (Cliente)
    participant Printer as Zebra (TCP 9100)

    ERP->>DB: Insere job em SPOOL_ZEBRA (status=P)
    AppA->>DB: Le jobs novos (P)
    AppA->>AppB: Envia job para o cliente correto
    AppB->>Printer: Envia ZPL via socket TCP 9100
    Printer-->>AppB: Impressao concluida
    AppB->>AppA: ACK (OK ou ERRO)
    AppA->>DB: Atualiza status (E ou ERRO)
```
