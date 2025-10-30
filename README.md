# 🚀 PIPELINE/ DATAFACTORY-->SNOWFLAKE — PL_LOAD_BLOB_TO_SNOWFLAKE

Pipeline do **Azure Data Factory (ADF)** para carregar arquivos CSV do **Blob Storage** para o **Snowflake**. 💾➡️❄️

---

## 🧩 Visão Geral

* **Pipeline:** `PL_LOAD_BLOB_TO_SNOWFLAKE`
* **Fluxo:** Lookup ➡️ ForEach ➡️ Copy
* **Tabelas:** `PRODUTOS`, `CLIENTES`, `VENDAS`
* **Destino:** Snowflake (`ADF_TRAINING_DB.RAW`)
* **Staging:** Blob `adf-staging`

---

## ⚙️ Recursos Criados (Azure)

| Tipo            | Nome Exemplo                 | Descrição                   |
| --------------- | ---------------------------- | --------------------------- |
| Resource Group  | `rs-adf-snowflake-lab`       | Agrupa os recursos do lab   |
| Storage Account | `stgkeplerlab`               | Armazena dados e staging    |
| Containers      | `source data`, `adf-staging` | Fonte e staging do pipeline |
| Data Factory    | `adf-kepler-lab`             | Orquestra o fluxo de dados  |

---

## 🔗 Conexões (Linked Services)

* 🟦 **LS_BLOB_SOURCE** → Blob com os arquivos CSV
* 🟩 **LS_BLOB_STAGING_SASURI** → Staging (SAS URI)
* ❄️ **LS_SNOWFLAKE_DEST** → Conexão com o Snowflake

> 💡 Use **Azure Key Vault** para armazenar credenciais de forma segura.

---

## 📊 Datasets

* **DS_ABLB_CONFIG** → JSON com lista de arquivos/tabelas
* **DS_ABLB_SOURCE** → Fonte CSV parametrizada (`fileName`, `folderPath`)
* **DS_SF_SINK** → Tabela destino no Snowflake (`target_table`)

Exemplo `config.json`:

```json
[
  { "table_name": "PRODUTOS", "file": "produtos.csv" },
  { "table_name": "CLIENTES", "file": "clientes.csv" },
  { "table_name": "VENDAS", "file": "vendas.csv" }
]
```

---

## 🔄 Pipeline — Etapas

1. 🔍 **Lookup_Config** → Lê `config.json`
2. 🔁 **ForEach** → Itera pelas tabelas
3. 📥 **Copy_To_Snowflake** → Copia CSV → staging → Snowflake

   * `enableStaging: true`
   * `stagingSettings`: `LS_BLOB_STAGING_SASURI`

---

## ❄️ Provisionamento Snowflake

Execute o script abaixo no **Snowflake Worksheet** (ou via `snowsql`) para preparar o ambiente que receberá os dados:

```sql
CREATE DATABASE IF NOT EXISTS ADF_TRAINING_DB;
CREATE SCHEMA IF NOT EXISTS RAW;
CREATE WAREHOUSE IF NOT EXISTS COMPUTE_WH 
  WITH WAREHOUSE_SIZE='XSMALL' AUTO_SUSPEND=60 AUTO_RESUME=TRUE;

USE DATABASE ADF_TRAINING_DB;
USE SCHEMA RAW;

CREATE OR REPLACE TABLE PRODUTOS (
    id_produto INT,
    nome_produto VARCHAR(100),
    preco DECIMAL(10,2)
);

CREATE OR REPLACE TABLE CLIENTES (
    id_cliente INT,
    nome VARCHAR(100),
    cidade VARCHAR(50)
);

CREATE OR REPLACE TABLE VENDAS (
    id_venda INT,
    id_cliente INT,
    id_produto INT,
    quantidade INT,
    valor_total DECIMAL(10,2)
);

-- Verificação das tabelas
SHOW TABLES;

-- Contagem de registros (após ingestão)
SELECT COUNT(*) AS TOTAL_CLIENTES FROM CLIENTES;
SELECT COUNT(*) AS TOTAL_PRODUTOS FROM PRODUTOS;
SELECT COUNT(*) AS TOTAL_VENDAS FROM VENDAS;

-- Validação dos dados
SELECT * FROM VENDAS;
```

💡 **Dica:** Após o pipeline executar com sucesso, as consultas acima devem retornar registros nas três tabelas criadas.

---

## 🧠 Dicas Finais

✅ Utilize **Key Vault** para senhas e tokens
✅ Mantenha seus CSVs em pastas organizadas (`source data/`)
✅ Use o **Monitor** do ADF para acompanhar execuções
✅ Configure alertas e logs para falhas

✨ Ambiente pronto para ingestão e análise no Snowflake!

