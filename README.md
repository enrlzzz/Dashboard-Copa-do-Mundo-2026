# 🏆 Dashboard Copa do Mundo 2026 — Power BI

> Relatório executivo de BI sobre a Copa do Mundo de 2026, construído em **Power BI Desktop** no formato **PBIP** (relatório em PBIR + modelo em TMDL), com **Star Schema**, **DAX documentada** e **componentes visuais em HTML** renderizados dentro do Power BI.

`Power BI Desktop` · `PBIP / PBIR` · `TMDL` · `DAX` · `Star Schema` · `Power Query (M)` · `pt-BR` · `1920×1080`

---

## 📌 Sobre o projeto

Projeto **pessoal, de treino**, com um objetivo bem específico: praticar a transformação de **dado bruto em leitura executiva** — o tipo de relatório que um diretor, um gestor ou um CFO consegue entender em cinco minutos, sem precisar navegar tela por tela.

A aposta central do projeto não está no dado em si, mas na **forma de apresentá-lo**: uma capa que diz o que é e para que serve, um sumário que situa quem chega, e cards de KPI que resumem o relatório inteiro logo na abertura.

O tema (Copa 2026) foi escolhido por ser recente e de fácil leitura — o que importa aqui é a **engenharia por trás**: modelagem, DAX e camada visual.

---

## 🔎 Dados e proveniência

**Transparência primeiro**, porque o relatório trata de um evento real:

- Os dados são **públicos** e foram **coletados, cruzados e consolidados com apoio de IA (Claude)** a partir do site oficial da **FIFA** e de mais de 15 portais esportivos e de notícias.
- Fontes documentadas dentro do próprio arquivo, na aba **`Fontes`** da planilha: FIFA.com (resumos oficiais da fase final), fixturedownload.com (tabela e placares da fase de grupos até as oitavas), CNN Brasil, Exame, Terra, Lance! e Forbes (premiações, artilharia e resultados detalhados do mata-mata).
- **Limitação conhecida e assumida:** a fase de grupos, os "oitavos de 32" e as oitavas de final entraram **apenas com o placar final** — as fontes públicas consultadas não traziam detalhamento de gol por jogador/minuto nessas fases. O detalhamento por marcador existe a partir das **quartas de final**, onde a cobertura jornalística ficou mais completa, e está isolado na tabela `Fato_FaseFinal`.

> ⚠️ **Aviso:** este repositório **não é fonte oficial**. É um exercício de BI montado sobre uma compilação de dados públicos feita com auxílio de IA. Confira nas fontes originais antes de usar qualquer número como referência.

---

## 🗺️ O relatório

Cinco páginas, **39 visuais**, navegação por botões a partir da capa:

```
┌──────────┐   ┌───────────┐   ┌──────────────┐   ┌─────────────────┐   ┌──────────────┐
│  1. Capa │ → │ 2. Sumário│ → │3. Artilheiros│ → │4. Todos os jogos│ → │ 5. Premiações│
└──────────┘   └───────────┘   └──────────────┘   └─────────────────┘   └──────────────┘
    home           índice          ranking          linha do tempo          troféus
```

| # | Página | O que entrega |
|---|--------|---------------|
| 1 | **Capa** | Abertura + 5 cards de KPI em HTML + hub de navegação |
| 2 | **Sumário** | Índice dinâmico em HTML, estilo editorial |
| 3 | **Artilheiros** | Ranking de goleadores (tabela + barras + KPIs) |
| 4 | **Todos os jogos** | Panorama das partidas (cards + linha do tempo) |
| 5 | **Premiações** | Prêmios e reconhecimentos (tabela + cards) |

---

## 🏗️ Como foi feito

### 1. Modelagem — Star Schema

Nomenclatura padronizada (`Fato_` / `Dim_` / `Aux_`), fatos ligados às dimensões por relacionamentos simples, sem tabelas órfãs:

```
                    ┌────────────────┐
                    │ Dim_Calendario │
                    └───────┬────────┘
                     Data   │   Data
              ┌─────────────┴─────────────┐
              ▼                           ▼
     ┌────────────────┐          ┌─────────────────┐
     │   Fato_Jogos   │          │ Fato_FaseFinal  │
     └───────┬────────┘          └─────────────────┘
        Fase │
             ▼
      ┌─────────────┐            ┌──────────────────┐
      │  Dim_Fase   │            │ Fato_Artilheiros │
      └─────────────┘            └────────┬─────────┘
                                  Seleção │
                                          ▼
      ┌──────────────────┐        ┌───────────────┐
      │ Fato_Premiacoes  │───────▶│  Dim_Selecao  │
      └──────────────────┘ Seleção└───────────────┘
```

| Tabela | Tipo | Conteúdo |
|--------|------|----------|
| `Fato_Jogos` | Fato | Todas as partidas, placares e sedes |
| `Fato_Artilheiros` | Fato | Gols por jogador |
| `Fato_Premiacoes` | Fato | Prêmios individuais e coletivos |
| `Fato_FaseFinal` | Fato | Detalhamento de gols (quartas em diante) |
| `Dim_Calendario` | Dimensão | Calendário do torneio |
| `Dim_Selecao` | Dimensão | Seleções participantes |
| `Dim_Fase` | Dimensão | Fase da competição + agrupamento Grupos × Mata-mata |
| `Aux_Fontes` | Auxiliar | Fontes de dados expostas no próprio relatório |
| `_KPIs Copa` | Medidas | Tabela dedicada só a medidas DAX |

### 2. DAX — organizada e documentada

Todas as medidas vivem em uma **tabela dedicada** (`_KPIs Copa`), separadas em **pastas de exibição** e com descrição individual:

| Pasta | Exemplos |
|-------|----------|
| `_Bases` | `# Jogos`, `# Gols`, `# Seleções`, `# Sedes`, `# Prêmios`, `# Artilheiros` |
| `_Razões` | `AVG Gols por Jogo`, `Maior Goleada`, `Maior Nº de Gols em 1 Jogo`, `% Jogos Mata-mata` |
| `_Rankings` | `Artilheiro Líder`, `Gols do Líder`, `Seleção do Líder`, `Campeão`, `Seleção + Goleadora` |

Os rankings usam padrões de `TOPN` + `SUMMARIZE` + `CONCATENATEX` para resolver empates de forma previsível (retornando os nomes empatados em vez de um valor arbitrário).

### 3. Os componentes HTML — o destaque técnico

Dois componentes renderizados pelo custom visual **HTML Content**, escritos **inteiramente em DAX** que devolve HTML + CSS:

**`HTMLC_Capa_KPIs`** — **5 cards de KPI em um único visual.** Grid CSS de 5 colunas, estética *glassmorphism* (fundo translúcido, `backdrop-filter: blur`, bordas com gradiente via máscara, brilho radial). Valores 100% dinâmicos, vindos das medidas do modelo e respeitando o contexto de filtro.

**`HTML_Sumario_Dinamico`** — índice de navegação em estilo editorial, com numeração romana, contagem automática de tópicos preenchidos e *gap* do grid ajustado por `SWITCH` conforme o volume de itens. Esconder um item = deixar o título vazio.

Ambos seguem o mesmo padrão de arquitetura:

```
🔧 BLOCO CONFIG  → o que se edita: design tokens em HEX + conteúdo dos cards/tópicos
⚙️  MOTOR         → o que não se mexe: montagem do HTML, grid e estilos
```

Trocar a marca do componente inteiro = trocar **4 valores HEX**. Na prática, um mini design system reaproveitável dentro do Power BI.

**Por que 1 visual em vez de 5:** menos objetos na tela, alinhamento perfeito garantido pelo grid e um único ponto de manutenção.

### 4. Camada visual

- **Arte como plano de fundo de página** (`background.image`, `scaling: Fill`) em vez de imagem solta sobre a tela — os visuais "flutuam" sobre um layout fixo e não desalinham.
- Cada página interna usa uma **variação da mesma arte-base**, mantendo consistência entre telas.
- **Botões de navegação com 3 estados** (padrão / focalizar / pressionar), dando feedback tátil de produto, não de protótipo.
- Todos os recursos ficam empacotados em `StaticResources/RegisteredResources` — o relatório é **autocontido**.

---

## 📂 Estrutura do repositório

```
├── Dashboard Copa do Mundo 2026.pbip          # ponto de entrada do projeto
├── Dashboard Copa do Mundo 2026.Report/       # relatório em PBIR (páginas e visuais em JSON)
│   ├── definition/                            # pages.json, page.json, visual.json
│   └── StaticResources/                       # fundos, botões e tema
├── Dashboard Copa do Mundo 2026.SemanticModel/# modelo em TMDL
│   └── definition/                            # tabelas, relacionamentos, medidas, cultura
├── Copa do Mundo 2026.xlsx                    # fonte de dados (inclui a aba `Fontes`)
└── Pacote-Botoes-Copa-2026/                   # kit de botões (4 estados) usado no relatório
```

O formato **PBIP** guarda relatório e modelo como **texto** (JSON e TMDL) em vez de um binário `.pbix` — por isso o projeto é versionável no Git, com diff e histórico legíveis a cada alteração.

---

## ▶️ Como abrir

1. **Power BI Desktop** atualizado, com a *preview* **Power BI Project (.pbip)** habilitada em `Arquivo → Opções → Recursos de visualização`.
2. Clone o repositório e abra o arquivo `Dashboard Copa do Mundo 2026.pbip`.
3. **Reaponte a fonte de dados:** as consultas M referenciam o caminho local do `Copa do Mundo 2026.xlsx`. Vá em `Transformar dados → Configurações da fonte de dados → Alterar origem` e selecione o `.xlsx` na pasta onde você clonou o repositório.
4. Atualize o modelo.
5. Os cards em HTML dependem do custom visual **HTML Content** (AppSource). Se as páginas Capa e Sumário aparecerem em branco, instale o visual e recarregue.

---

## ✍️ Autoria e créditos

Dashboard concebido e construído por **Enrico** — modelagem, DAX, arquitetura dos componentes HTML e direção visual.

A **coleta e consolidação dos dados** foi feita com apoio de **IA (Claude)**, a partir de fontes públicas (FIFA.com e mais de 15 portais esportivos e de notícias), com as origens registradas na aba `Fontes` da planilha e expostas dentro do próprio relatório.

Marcas, nomes de seleções e referências à FIFA pertencem aos seus respectivos detentores. Este é um projeto **educacional, sem fins comerciais** e sem qualquer vínculo com a FIFA.
