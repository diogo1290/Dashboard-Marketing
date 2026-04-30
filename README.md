# 📊 Dashboard de Performance de Canais Digitais

<div align="center">

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![Databricks](https://img.shields.io/badge/Azure%20Databricks-FF3621?style=for-the-badge&logo=databricks&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Delta Lake](https://img.shields.io/badge/Delta%20Lake-003366?style=for-the-badge&logo=apachespark&logoColor=white)

[![Ver Dashboard ao Vivo](https://img.shields.io/badge/▶%20Ver%20Dashboard%20ao%20Vivo-0D1B2A?style=for-the-badge&logo=powerbi&logoColor=00B4D8)](https://app.powerbi.com/view?r=eyJrIjoiYmFkZWEzZGYtYmE4NC00NzI4LWFkNGEtZDlhNzA2ZTU1MjJmIiwidCI6IjY1OWNlMmI4LTA3MTQtNDE5OC04YzM4LWRjOWI2MGFhYmI1NyJ9)
[![Download PBIX](https://img.shields.io/badge/⬇%20Download%20PBIX-0078D4?style=for-the-badge&logo=microsoftonedrive&logoColor=white)](https://escolatrabalhador4-my.sharepoint.com/personal/diogoferreira1290_escoladotrabalhador40_com_br/Documents/Gest%C3%A3o%20de%20Performance%20de%20Marketing.pbix)

</div>

---

## 🎯 Sobre o Projeto

Pipeline end-to-end analisando o desempenho de **9 canais digitais** — do problema de negócio ao dashboard interativo.

> *"Qual canal gera o melhor retorno sobre o investimento?"*

| Camada | Tecnologia |
|---|---|
| Geração de dados | Python · NumPy · Pandas |
| Armazenamento | Azure Databricks · Unity Catalog |
| Transformação | SQL · Delta Lake |
| Modelagem | Star Schema (1 fato · 5 dims · 500K linhas) |
| Análise | DAX · Power BI (30+ medidas) |

---

## 🗂️ Modelo de Dados

```
   dim_data          dim_canal        dim_campanha
       └──────────────────┐──────────────────┘
                  fato_engajamento
       ┌──────────────────┘──────────────────┐
   dim_publico                          dim_conteudo
```

| Tabela | Linhas | Descrição |
|---|---|---|
| `fato_engajamento` | 500.000 | Impressões, cliques, conversões, custo, receita, CTR, ROAS, CPA |
| `dim_canal` | 9 | Instagram, TikTok, LinkedIn, YouTube, Google Ads, Email, WhatsApp... |
| `dim_campanha` | 60 | 8 marcas · orçamento · objetivo · status |
| `dim_publico` | 120 | Faixa etária · gênero · região · renda |
| `dim_data` | 1.096 | Calendário 2022–2024 com feriados BR |
| `dim_conteudo` | 50 | Formato · tema · CTA · idioma |

---

## 📐 Medidas DAX

```dax
ROAS = DIVIDE([Receita Total], [Custo Total], 0)

Crescimento YoY % =
VAR Atual    = [Receita Total]
VAR Anterior = CALCULATE([Receita Total], SAMEPERIODLASTYEAR(dim_data[data]))
RETURN DIVIDE(Atual - Anterior, Anterior, BLANK())

Rank Canal =
RANKX(ALLSELECTED(dim_canal[canal]), [Receita Total], , DESC, DENSE)

Score Canal =
DIVIDE([ROAS],              MAXX(ALL(dim_canal), [ROAS]),              0) * 50 +
DIVIDE([CTR %],             MAXX(ALL(dim_canal), [CTR %]),             0) * 30 +
DIVIDE([Engagement Rate %], MAXX(ALL(dim_canal), [Engagement Rate %]), 0) * 20
```

---

## 📊 Páginas do Dashboard

**Página 1 — Visão Executiva**
KPIs globais · Receita vs Custo mensal · Impressões por Canal · Top/Bottom Campanhas

**Página 2 — Análise de Performance**
Funil de Conversão · ROAS vs CTR por Canal · Engajamento por Conteúdo · Mapa Regional

---

## 💡 Principais Insights

- **Email** tem o maior ROAS médio (7,0x) com o menor custo por clique
- **TikTok** lidera em impressões mas apresenta o menor CVR
- Maior gargalo do funil: **Alcance → Engajamento** (queda de 92% para 6%)
- **Reels e Carrossel** são os formatos com maior Engagement Rate

---

## 🐍 Geração de Dados — Python

```python
import pandas as pd
import numpy as np

np.random.seed(42)
N = 500_000

# Parâmetros reais por canal (CTR, Engagement Rate, CPM, ROAS médio)
canal_params = {
    1: dict(imp_mu=40000, ctr=0.025, eng=0.060, cvr=0.018, cpm=8.50,  rm=4.5),  # Instagram
    2: dict(imp_mu=55000, ctr=0.018, eng=0.040, cvr=0.015, cpm=6.20,  rm=3.8),  # Facebook
    3: dict(imp_mu=80000, ctr=0.012, eng=0.090, cvr=0.010, cpm=4.30,  rm=3.2),  # TikTok
    4: dict(imp_mu=10000, ctr=0.050, eng=0.050, cvr=0.035, cpm=18.90, rm=6.0),  # LinkedIn
    5: dict(imp_mu=35000, ctr=0.020, eng=0.070, cvr=0.022, cpm=7.10,  rm=4.0),  # YouTube
    6: dict(imp_mu=20000, ctr=0.065, eng=0.030, cvr=0.045, cpm=12.40, rm=5.5),  # Google Ads
    7: dict(imp_mu=15000, ctr=0.040, eng=0.025, cvr=0.060, cpm=0.80,  rm=7.0),  # Email
    8: dict(imp_mu=8000,  ctr=0.055, eng=0.080, cvr=0.050, cpm=1.20,  rm=5.0),  # WhatsApp
    9: dict(imp_mu=25000, ctr=0.015, eng=0.045, cvr=0.012, cpm=5.60,  rm=3.5),  # Pinterest
}

# Geração vetorizada por canal com distribuições log-normais
for cid, p in canal_params.items():
    mask = id_canal_v == cid
    n    = mask.sum()
    imp  = np.clip(np.random.lognormal(np.log(p['imp_mu']), 0.7, n).astype(int), 100, 2_000_000)
    alc  = (imp * np.random.uniform(0.55, 0.90, n)).astype(int)
    clk  = np.clip((imp * np.random.normal(p['ctr'], p['ctr']*0.3, n)).astype(int), 0, imp)
    cst  = ((imp / 1000.0) * p['cpm'] * np.random.uniform(0.85, 1.15, n)).round(2)
    rev  = np.clip((cst * np.random.normal(p['rm'], 0.5, n)).round(2), 0, None)

# Métricas derivadas
ctr  = np.where(imp_a > 0, np.round(clk_a / imp_a * 100, 4), 0.0)
roas = np.where(cst_a > 0, np.round(rev_a / cst_a,       4), 0.0)
cpa  = np.where(con_a > 0, np.round(cst_a / con_a,       4), 0.0)

# Exporta para CSV
fato.to_csv("fato_engajamento.csv", index=False, encoding="utf-8-sig")
```

---

## 🔄 ETL — SQL no Databricks

```sql
-- 1. Ingestão a partir do Volume
CREATE TABLE default.fato_engajamento AS
SELECT * FROM read_files(
    '/Volumes/workspace/default/engajamento/fato_engajamento.csv',
    format => 'csv', header => true, inferSchema => true
);

-- 2. Tipagem, recálculo de métricas e zeros → NULL
CREATE OR REPLACE TABLE default.fato_engajamento AS
SELECT
    CAST(id_engajamento AS INT),
    CAST(id_data        AS INT),
    CAST(id_canal       AS INT),
    CAST(id_campanha    AS INT),
    CAST(id_publico     AS INT),
    CAST(id_conteudo    AS INT),
    CAST(impressoes     AS INT),
    CAST(alcance        AS INT),
    CAST(cliques        AS INT),
    CAST(conversoes     AS INT),

    (curtidas + comentarios + compartilhamentos) AS total_engajamentos,

    ROUND(CAST(custo_brl   AS DOUBLE), 2) AS custo_brl,
    ROUND(CAST(receita_brl AS DOUBLE), 2) AS receita_brl,

    CASE WHEN impressoes > 0
        THEN ROUND(cliques / impressoes * 100, 4) ELSE NULL END AS ctr_pct,

    CASE WHEN alcance > 0
        THEN ROUND((curtidas + comentarios + compartilhamentos) / alcance * 100, 4)
        ELSE NULL END AS eng_rate_pct,

    CASE WHEN cliques > 0
        THEN ROUND(custo_brl / cliques, 2) ELSE NULL END AS cpc_brl,

    CASE WHEN custo_brl > 0
        THEN ROUND(receita_brl / custo_brl, 4) ELSE NULL END AS roas,

    CASE WHEN conversoes > 0
        THEN ROUND(custo_brl / conversoes, 2) ELSE NULL END AS cpa_brl

FROM default.fato_engajamento
WHERE id_engajamento IS NOT NULL
  AND impressoes      >= 0
  AND alcance         <= impressoes;

-- 3. Validação pós-ETL
SELECT
    COUNT(*)                                     AS total_linhas,
    COUNT(CASE WHEN roas    IS NULL THEN 1 END)  AS nulos_roas,
    COUNT(CASE WHEN cpc_brl IS NULL THEN 1 END)  AS nulos_cpc,
    COUNT(CASE WHEN cpa_brl IS NULL THEN 1 END)  AS nulos_cpa
FROM default.fato_engajamento;
```

---

## 👤 Autor

**Diogo Campos Ferreira** · Customer Intelligence Analyst

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://linkedin.com/in/diogo1290)
[![Portfolio](https://img.shields.io/badge/Portfolio-0D1B2A?style=flat&logo=vercel&logoColor=00B4D8)](https://diogoportfolio.lovable.app)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat&logo=github&logoColor=white)](https://github.com/diogo1290)
