# br/acc open graph

[![WTG Header](../brand/bracc-header.jpg)](../brand/bracc-header.jpg)

[English](../../README.md) | [Portugues](README.md)

**Infraestrutura open-source em grafo que cruza bases públicas brasileiras para gerar inteligência acionável para melhoria cívica.**

[![CI](https://github.com/World-Open-Graph/br-acc/actions/workflows/ci.yml/badge.svg)](https://github.com/World-Open-Graph/br-acc/actions/workflows/ci.yml)
[![License: AGPL v3](https://img.shields.io/badge/License-AGPL_v3-blue.svg)](https://www.gnu.org/licenses/agpl-3.0)
[![Last Commit](https://img.shields.io/github/last-commit/World-Open-Graph/br-acc)](https://github.com/World-Open-Graph/br-acc/commits)
[![Issues](https://img.shields.io/github/issues/World-Open-Graph/br-acc)](https://github.com/World-Open-Graph/br-acc/issues)
[![Stars](https://img.shields.io/github/stars/World-Open-Graph/br-acc?style=social)](https://github.com/World-Open-Graph/br-acc/stargazers)
[![Forks](https://img.shields.io/github/forks/World-Open-Graph/br-acc?style=social)](https://github.com/World-Open-Graph/br-acc/network/members)
[![Twitter Follow](https://img.shields.io/twitter/follow/brunoclz?style=social)](https://x.com/brunoclz)
[![Discord](https://img.shields.io/badge/Discord-Entre%20no%20servidor-5865F2?logo=discord&logoColor=white)](https://discord.gg/YyvGGgNGVD)

[Discord](https://discord.gg/YyvGGgNGVD) | [Twitter](https://x.com/brunoclz) | [Website](https://bracc.org) | [Contribuir](#contribuindo)

---

## O que é br/acc?

br/acc é um movimento descentralizado de builders brasileiros usando tecnologia e dados abertos para tornar informação pública mais acessível. Este repositório é um de seus projetos: uma infraestrutura open-source em grafo que ingere bases de dados públicas brasileiras oficiais, registros de empresas, saúde, educação, emprego, finanças públicas, licitações, meio ambiente e normaliza tudo em um único grafo consultável.

Ele torna dados públicos que já são abertos, mas espalhados em dezenas de portais, acessíveis em um só lugar. Não interpreta, pontua ou classifica resultados — apenas exibe conexões e deixa os usuários tirarem suas próprias conclusões.

[Saiba mais em bracc.org](https://bracc.org)

---

## Funcionalidades

- **45 modulos ETL implementados** — status rastreado em `docs/source_registry_br_v1.csv` (loaded/partial/stale/blocked/not_built)
- **Infraestrutura de grafo Neo4j** — schema, loaders e superficie de consulta para entidades e relacionamentos
- **Frontend React** — busque, explore redes empresariais e analise conexoes de entidades
- **API publica** — acesso programatico aos dados do grafo via FastAPI
- **Ferramentas de reprodutibilidade** — bootstrap local em um comando e fluxo BYO-data para ETL
- **Privacy-first** — compativel com LGPD, defaults publicos seguros, sem exposicao de dados pessoais

---

## Início Rápido

```bash
cp .env.example .env
make bootstrap-demo
```

Esse comando inicia os serviços Docker, espera Neo4j/API ficarem saudáveis e carrega seed determinístico de desenvolvimento.

Verifique em:

- API: http://localhost:8000/health
- Frontend: http://localhost:3000
- Neo4j Browser: http://localhost:7474

### Subir com Docker

Você pode subir a stack (Neo4j, API, frontend) com Docker Compose sem rodar o bootstrap completo:

```bash
cp .env.example .env
docker compose up -d
```

Opcional: incluir o serviço ETL (para rodar pipelines no container):

```bash
docker compose --profile etl up -d
```

As mesmas URLs de verificação valem. Para um grafo demo pronto com dados de seed, use `make bootstrap-demo`.

---

## Fluxo em um Comando

```bash
# Fluxo demo local (recomendado para primeira execução)
make bootstrap-demo

# Orquestração pesada de ingestão completa (Docker + todos pipelines implementados)
make bootstrap-all

# Execução pesada não interativa (automação)
make bootstrap-all-noninteractive

# Exibir último relatório do bootstrap-all
make bootstrap-all-report
```

`make bootstrap-all` e propositalmente pesado:
- alvo padrão de ingestão historica completa
- pode levar horas (ou mais), dependendo das fontes externas
- exige disco, memória e banda de rede significativos
- continua em caso de erro e grava resumo auditável por fonte em `audit-results/bootstrap-all/`

Guia detalhado: [`docs/bootstrap_all.md`](../bootstrap_all.md)

---

## O que está incluído neste repositório público

- Codigo de API, frontend, framework ETL e infraestrutura.
- Registro de fontes e documentação de status de pipelines.
- Dataset demo sintético e caminho deterministico de seed local.
- Gates públicos de segurança/compliance e documentação de release.

## O que não está incluído por padrão

- Dump Neo4j de produção pré-populado.
- Garantia de estabilidade/disponibilidade de todos os portais externos.
- Módulos institucionais/privados e runbooks operacionais internos.

## O que é reproduzivel localmente hoje

- Subida completa local (`make bootstrap-demo`) com grafo demo.
- Fluxo BYO-data via pipelines `bracc-etl`.
- Orquestração pesada em um comando (`make bootstrap-all`) com relatório explicito de fontes bloqueadas/falhas.
- Comportamento da API pública em modo seguro de privacidade.

Contadores de escala de produção são publicados como **snapshot de referência de produção** em [`docs/reference_metrics.md`](../reference_metrics.md), não como resultado esperado do bootstrap local.

---

## Arquitetura

| Camada | Tecnologia |
|---|---|
| Banco de Grafo | Neo4j 5 Community |
| Backend | FastAPI (Python 3.12+, async) |
| Frontend | Vite + React 19 + TypeScript |
| ETL | Python (pandas, httpx) |
| Infra | Docker Compose |

```mermaid
graph LR
    A[Fontes de Dados Publicos] --> B[Pipelines ETL]
    B --> C[(Neo4j)]
    C --> D[FastAPI]
    D --> E[Frontend React]
    D --> F[API Publica]
```

---

## Mapa do repositorio

```
api/          Backend FastAPI (rotas, servicos, modelos)
etl/          Pipelines ETL e scripts de download
frontend/     App React (Vite + TypeScript)
infra/        Docker, schema Neo4j, scripts de seed
scripts/      Scripts utilitarios e de automacao
docs/         Documentacao, assets de marca, indice legal
data/         Datasets baixados (ignorado pelo git)
```

---

## Referencia da API

| Metodo | Rota | Descricao |
|---|---|---|
| GET | `/health` | Health check |
| GET | `/api/v1/public/meta` | Metricas agregadas e saude das fontes |
| GET | `/api/v1/public/graph/company/{cnpj_or_id}` | Subgrafo publico de empresa |
| GET | `/api/v1/public/patterns/company/{cnpj_or_id}` | Analise de padroes (quando habilitado) |

Documentacao interativa completa em `http://localhost:8000/docs` apos iniciar a API.

---

## Contribuindo

Contribuiçes de todos os tipos s~~ao bem-vindas — código, pipelines de dados, documentação e relatos de bugs. Veja as issues abertas para primeiras tarefas, ou abra uma nova para discutir sua ideia.

Se você achou o projeto útil, **dê uma estrela no repo** — ajuda outras pessoas a descobri-lo.

---

## Apoie o Projeto

[![Sponsor](https://img.shields.io/badge/GitHub_Sponsors-Apoiar-ea4aaa?logo=githubsponsors&logoColor=white)](https://github.com/sponsors/brunoclz)

Se quiser apoiar o desenvolvimento diretamente:

| Rede | Endereço |
|---|---|
| Solana | `HFceUyei1ndQypNKoiYSsHLHrVcaMZeNBeRhs8LmmkLn` |
| Ethereum | `0xbB3538D3e1B1Dd7c916BE7DfAC9ac7e322f592c7` |

---

## Comunidade

- **Discord**: [discord.gg/YyvGGgNGVD](https://discord.gg/YyvGGgNGVD)
- **Twitter**: [@brunoclz](https://x.com/brunoclz)
- **Website**: [bracc.org](https://bracc.org)
- **Comunidade Brazilian Accelerationism** no X

---

## Legal e Ética

Todos os dados processados por este projeto são públicos por lei. Cada fonte é publicada por um portal do governo brasileiro ou iniciativa internacional de dados abertos, disponibilizada sob um ou mais dos seguintes instrumentos legais:

| Lei | Escopo |
|---|---|
| **CF/88 Art. 5 XXXIII, Art. 37** | Direito constitucional de acesso a informação pública |
| **Lei 12.527/2011 (LAI)** | Lei de Acesso a Informação — regula o acesso a dados governamentais |
| **LC 131/2009 (Lei da Transparência)** | Obriga publicação em tempo real de dados fiscais e orcamentários |
| **Lei 13.709/2018 (LGPD)** | Proteção de dados — Art. 7 IV/VII permitem tratamento de dados públicos para interesse público |
| **Lei 14.129/2021 (Governo Digital)** | Obriga dados abertos por padrão para orgãos governamentais |

<details>
<summary><b>Matriz de Datasets Brasil (Base Legal)</b></summary>

| # | Fonte | Portal | Base Legal |
|---|-------|--------|------------|
| 1 | CNPJ (Cadastro de Empresas) | Receita Federal | LAI, CF Art. 37 |
| 2 | TSE (Eleições e Doações) | dadosabertos.tse.jus.br | Lei 9.504/1997 (Lei Eleitoral), LAI |
| 3 | Portal da Transparência | portaldatransparencia.gov.br | LC 131/2009, LAI |
| 4 | CEIS/CNEP (Sanções) | Portal da Transparência | LAI, Lei 12.846/2013 (Lei Anticorrupção) |
| 5 | BNDES (Empréstimos) | bndes.gov.br | LAI, LC 131/2009 |
| 6 | PGFN (Dívida Ativa) | portaldatransparencia.gov.br | LAI, Lei 6.830/1980 |
| 7 | ComprasNet (Licitações) | comprasnet.gov.br | Lei 14.133/2021 (Licitações), LAI |
| 8 | TCU (Sanções de Auditoria) | portal.tcu.gov.br | LAI, CF Art. 71 |
| 9 | TransfereGov | transferegov.sistema.gov.br | LC 131/2009, LAI |
| 10 | RAIS (Estatisticas Trabalhistas) | PDET/MTE | LAI (agregado, sem dados pessoais) |
| 11 | INEP (Censo Educacional) | dados.gov.br | LAI, Lei 14.129/2021 |
| 12 | DataSUS/CNES (Saúde) | datasus.saude.gov.br | LAI, Lei 8.080/1990 (SUS) |
| 13 | IBAMA (Embargos) | dados.gov.br | LAI, Lei 9.605/1998 (Crimes Ambientais) |
| 14 | DOU (Diário Oficial) | in.gov.br | CF Art. 37 (publicidade) |
| 15 | Camara (Despesas de Deputados) | dadosabertos.camara.leg.br | LAI, CF Art. 37 |
| 16 | Senado (Despesas de Senadores) | dadosabertos.senado.leg.br | LAI, CF Art. 37 |
| 17 | ICIJ (Offshore Leaks) | offshoreleaks.icij.org | Base de dados jornalística de interesse público |
| 18 | OpenSanctions (PEPs Globais) | opensanctions.org | Agregador open-data (licenca CC) |
| 19 | CVM (Processos de Valores Mobiliarios) | dados.cvm.gov.br | LAI, Lei 6.385/1976 |
| 20 | CVM Fundos | dados.cvm.gov.br | LAI, Lei 6.385/1976 |
| 21 | Servidores Públicos | Portal da Transparência | LC 131/2009, LAI |
| 22 | CEAF (Servidores Expulsos) | portaldatransparencia.gov.br | LAI, Lei 8.112/1990 |
| 23 | CEPIM (ONGs Impedidas) | portaldatransparencia.gov.br | LAI |
| 24 | CPGF (Cartoes Corporativos) | portaldatransparencia.gov.br | LC 131/2009, LAI |
| 25 | Viagens a Serviço | portaldatransparencia.gov.br | LC 131/2009, LAI |
| 26 | Renúncias Fiscais | portaldatransparencia.gov.br | LC 131/2009, LAI |
| 27 | Acordos de Leniencia | portaldatransparencia.gov.br | Lei 12.846/2013, LAI |
| 28 | BCB Penalidades | dados.bcb.gov.br | LAI, Lei 4.595/1964 |
| 29 | STF (Supremo Tribunal Federal) | portal.stf.jus.br | CF Art. 93 IX (publicidade judiciária) |
| 30 | PEP CGU | portaldatransparencia.gov.br | LAI, Decreto 9.687/2019 |
| 31 | TSE Bens (Patrimônio de Candidatos) | dadosabertos.tse.jus.br | Lei 9.504/1997 |
| 32 | TSE Filiados (Filiação Partidaria) | dadosabertos.tse.jus.br | Lei 9.096/1995 (Lei dos Partidos) |
| 33 | OFAC SDN | treasury.gov | Lista pública de sanções dos EUA |
| 34 | EU Sanctions | data.europa.eu | Lista pública de sanções da UE |
| 35 | UN Sanctions | un.org | Lista pública do Conselho de Segurança da ONU |
| 36 | World Bank Debarment | worldbank.org | Lista pública de impedimentos |
| 37 | Holdings (derivado) | — | Derivado dos dados CNPJ |
| 38 | SIOP (Emendas Orcamentárias) | siop.planejamento.gov.br | LC 131/2009, LAI |
| 39 | Senado CPIs | dadosabertos.senado.leg.br | LAI, CF Art. 58 §3 |

</details>

Todos os achados são apresentados como conexões de dados atribuidas a fontes, nunca como acusações. A plataforma aplica defaults públicos seguros que impedem exposição de informações pessoais em deployments públicos.

<details>
<summary><b>Defaults publicos seguros</b></summary>

```
PRODUCT_TIER=community
PUBLIC_MODE=true
PUBLIC_ALLOW_PERSON=false
PUBLIC_ALLOW_ENTITY_LOOKUP=false
PUBLIC_ALLOW_INVESTIGATIONS=false
PATTERNS_ENABLED=false
VITE_PUBLIC_MODE=true
VITE_PATTERNS_ENABLED=false
```
</details>

- [ETHICS.md](../../ETHICS.md)
- [LGPD.md](../../LGPD.md)
- [PRIVACY.md](../../PRIVACY.md)
- [TERMS.md](../../TERMS.md)
- [DISCLAIMER.md](../../DISCLAIMER.md)
- [SECURITY.md](../../SECURITY.md)
- [ABUSE_RESPONSE.md](../../ABUSE_RESPONSE.md)
- [Indice Legal](../legal/legal-index.md)

---

## Releases

- [Historico de releases](https://github.com/World-Open-Graph/br-acc/releases)
- [Politica de releases](../release/release_policy.md)
- [Runbook do mantenedor](../release/release_runbook.md)

---

## Licenca

[GNU Affero General Public License v3.0](../../LICENSE)
