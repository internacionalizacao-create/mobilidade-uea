# Dashboard Mobilidade Acadêmica Internacional — UEA

**Acesse online:**
- https://willpine1992.github.io/mobilidade-uea/
- https://internacionalizacao-create.github.io/mobilidade-uea/

Painel estático (HTML/CSS/JS, sem build step) com a identidade visual
PROPESP/UEA, mostrando os estudantes estrangeiros recebidos na
pós-graduação da UEA.

## Fonte dos dados

`Modelo_PowerBI_Mobilidades_UEA_Estrutura_Completa_ERASMUS_Atualizada.xlsm`
(Google Drive, pasta da UEA/PROPESP — internacionalizacao@uea.edu.br). A
planilha já vem estruturada como modelo de Power BI (fato + dimensões),
com um dicionário de dados e medidas DAX próprios.

- 66 registros consolidados: 55 GCUB-MOB + 11 ERASMUS+.
- Modalidades "Move La América" e "PROAFRI" já estão no modelo, mas ainda
  sem dados (`Aguardando dados`).
- Atualizado pela última vez em 28/08/2026: `Nome_ABNT` passou a trazer
  todas as iniciais do nome (SOBRENOME, I. I. I., em vez de só a
  primeira — ver seção Privacidade) e `Continente_Origem` passou a
  distinguir "América Central" de "América do Norte" (Haiti e Honduras
  foram reclassificados).

## O modelo de dados

O modelo segue o padrão estrela típico de Power BI: uma tabela fato no
centro e várias dimensões que descrevem cada linha da fato.

### Tabela fato

**`Fato_Mobilidades`** — uma linha por mobilidade (não por pessoa: quem
tem duas mobilidades aparece duas vezes). Principais colunas:

| Coluna | O que é |
|---|---|
| `ID_Mobilidade` / `ID_Participante` | chaves — identificam a mobilidade e a pessoa, sem usar CPF/passaporte/e-mail |
| `Programa_Mobilidade` | modalidade: GCUB-MOB, ERASMUS+, Move La América, PROAFRI |
| `Edicao` | edição/ano de ingresso da fonte original (`2022-2023` … `2025-2026`; `2026` para o lote Erasmus+) |
| `Pais_Origem` + `Codigo_Pais_ISO2/ISO3` + `Continente_Origem` | país de origem do estudante, já com código ISO para mapas |
| `Codigo_PPG` + `Programa_Pos_Graduacao` | programa de pós-graduação da UEA que recebeu o estudante |
| `Nivel_Academico` | Mestrado, Doutorado ou Graduação |
| `Situacao_Participacao` | Recebido, A confirmar, Chegada prevista, Desistente, Em conferência – possível desligamento |
| `Fonte_Financiamento` | CAPES, FAPEAM, CNPq, ERASMUS+ ou Não informado |
| `Fluxo_Mobilidade` | direção do fluxo (`IN` — todos os 66 registros atuais; o cartão "Fluxo" sempre mostra IN e OUT lado a lado, OUT com 0 até a modalidade ERASMUS+ passar a registrar saídas) |
| `Tipo_Mobilidade` | Mobilidade acadêmica (GCUB-MOB) ou Mobilidade breve (ERASMUS+) |
| `Programa_Original` | texto exatamente como veio na planilha de origem, antes da padronização por PPG — usado só para auditoria interna (`js/data.js` traz o campo, mas não tem painel próprio na página pública; o painel "PPG — Siglas" usa `Codigo_PPG` no lugar) |
| `Status_Qualidade_Dado` | Validado, Parcial ou Pendente — resultado da conferência do registro (usado nos indicadores, não tem painel próprio no dashboard) |
| `Incluir_Indicadores_Oficiais` / `Incluir_Recebidos` | flags booleanas (Sim/Não) que decidem se a linha entra nos indicadores oficiais e no cartão de recebidos — usadas em vez de recalcular regras a cada gráfico |

### Dimensões usadas neste painel

- **`Dim_Programas_Mobilidade`** — as 4 modalidades (GCUB-MOB, ERASMUS+,
  Move La América, PROAFRI), com status da base (`Dados consolidados e
  revisados` / `Aguardando dados`) e público-alvo. Alimenta o painel
  "Programa de mobilidade" (botões coloridos, um por modalidade).
- **`Dim_Participantes_dados reais`** — uma linha por pessoa. Além dos
  campos restritos (ver Privacidade), tem as colunas `Nome_ABNT` (nome
  formatado como citação acadêmica, SOBRENOME, I. I. I. — uma inicial por
  nome/sobrenome do meio) e `Sexo_Genero` (categoria demográfica
  agregada) — as únicas duas informações desta dimensão usadas na página
  pública.
- Dimensões auxiliares não carregadas neste painel (redundantes com as
  colunas já desnormalizadas na fato, mas presentes no modelo Power BI
  original): `Dim_Paises`, `Dim_Programas`, `Dim_Edicoes`,
  `Dim_Calendario`.

### Medidas (equivalente às medidas DAX da aba `Medidas_DAX`)

Este painel recalcula essas medidas em JavaScript (`js/main.js`,
`computeKpis` e as funções `render*`) em vez de usar o motor do Power BI,
mas a lógica é a mesma:

| Medida | Regra |
|---|---|
| Total Cadastrados | `COUNT(Fato_Mobilidades)` no recorte de filtros ativo |
| Total Oficial | idem, filtrando `Incluir_Indicadores_Oficiais = "Sim"` |
| Recebidos Oficiais | idem, filtrando `Incluir_Recebidos = "Sim"` |
| % Recebidos | Recebidos Oficiais ÷ Total Oficial |
| Países de Origem / PPGs Envolvidos | contagem distinta de `Codigo_Pais_ISO2` / `Codigo_PPG`, só nos registros oficiais |

### Abas com dados pessoais — não usadas na página pública

`Dim_Participantes_dados reais` (nome completo, e-mail, telefone,
matrícula), `Fato_Acompanhamento` (status administrativo por pessoa:
Polícia Federal, banco, casa do estudante) e `Pendencias_Conferencia`
(pendência por pessoa, com nome) só entram na página **interna**
separada (`interno.html`, não publicada — ver Privacidade).
`Base_Original_` (cópia bruta da fonte, com CPF/passaporte/RNM/dados
bancários) não foi carregada em nenhuma das duas páginas — a própria
planilha orienta usá-la só para auditoria interna, fora do Power
BI/dashboard.

## Privacidade — duas páginas, dois níveis de acesso

Este projeto tem **duas páginas separadas**, deliberadamente não
interligadas por navegação, para que a página pública nunca vaze para a
restrita:

- **`index.html` (pública — é a que está publicada no link acima)** —
  usa `Fato_Mobilidades` + `Dim_Programas_Mobilidade` (`js/data.js`), e
  também as colunas `Nome_ABNT` e `Sexo_Genero` de `Dim_Participantes_dados
  reais` (na lista "Participantes" e no painel "Gênero", respectivamente).
  Ambas foram explicitamente autorizadas pela UEA/PROPESP para divulgação
  pública: o nome no formato ABNT (sobrenome + iniciais dos demais nomes)
  é uma forma de citação acadêmica, não o nome completo; o gênero é exibido
  apenas como contagem agregada, nunca associado a um nome individual na
  mesma visualização. Nenhum e-mail, telefone, matrícula ou documento é
  exibido aqui.

- **`interno.html` (uso interno — NÃO publicar)** — junta
  `Fato_Mobilidades` com `Dim_Participantes_dados reais` (nome completo,
  e-mail, telefone, matrícula), `Fato_Acompanhamento` (status
  administrativo: Polícia Federal, banco, casa do estudante etc.) e
  `Pendencias_Conferencia` (`js/data-interno.js`). Existe uma faixa de
  aviso vermelha fixa no topo da página e a tag `<meta name="robots"
  content="noindex, nofollow">`, mas isso não substitui cuidado humano:
  **não** hospedar em GitHub Pages, Netlify, Vercel ou qualquer link
  compartilhável; **não** commitar `js/data-interno.js` nem
  `interno.html` — ambos estão no `.gitignore`; acesso só localmente
  (`localhost`) por quem já tem autorização para ver esses dados na
  planilha de origem.

## Estrutura

```
index.html           página pública — painel principal, publicada em GitHub Pages
flowmap.html          página pública — mapa de fluxo animado (Manaus ⇄ países)
relatorio.html         página pública — relatório em HTML com opção de exportar PDF
interno.html          página de uso interno — nome completo, contato e acompanhamento por participante (gitignored, não publicar)
css/style.css         tokens + componentes (design system PROPESP/UEA) + estilos específicos (mapa de fluxo, relatório)
js/data.js            dados de Fato_Mobilidades + Nome_ABNT/Sexo_Genero + dimensão de modalidades (sem e-mail/telefone)
js/main.js            tema claro/escuro, filtros (persistidos em localStorage), mapa, gráficos, rankings, lista de participantes (index.html)
js/flowmap.js          linhas animadas Manaus ⇄ país; lê os mesmos filtros do Painel via localStorage (flowmap.html)
js/relatorio.js        tabelas + gráficos estáticos do recorte filtrado atual, botão "Exportar PDF" via window.print() (relatorio.html)
js/data-interno.js     join de Fato_Mobilidades + Dim_Participantes_dados reais + Fato_Acompanhamento + Pendencias_Conferencia (gitignored)
js/interno.js          busca, tabela e painel de detalhe (interno.html)
lib/                  d3.v7, topojson-client e o atlas mundial (countries-110m.json)
image/                logo PROPESP UEA
```

## Rodar localmente

```
cd "DASHBOARD MOBILIDADE"
python3 -m http.server 8080
# painel:                        http://localhost:8080/index.html
# mapa de fluxo:                 http://localhost:8080/flowmap.html
# relatório (exportar PDF):      http://localhost:8080/relatorio.html
# uso interno (restrito):        http://localhost:8080/interno.html
```

## Funcionalidades (páginas públicas)

### Filtros — combináveis e compartilhados entre as três páginas públicas

- **Clicáveis em praticamente todo painel**: programa de mobilidade,
  nível acadêmico, gênero, situação da participação, continente, PPG
  (sigla), país, fluxo/tipo de mobilidade e fonte de financiamento.
- **Nível acadêmico é multi-seleção** (clicar em "Mestrado" e depois em
  "Doutorado" mostra os dois somados) — tanto pelos cartões "Nível
  acadêmico" quanto pela legenda de "Evolução por edição", que
  controlam a mesma seleção. As demais dimensões continuam seleção
  única (clicar substitui a anterior).
- **Barra de filtros ativos** logo abaixo dos indicadores — um chip
  removível por filtro (dimensões multi-seleção geram um chip por
  valor escolhido) e um botão "Limpar filtros".
- **Persistem entre páginas via `localStorage`** (chave
  `"mobuea-filters"`): o recorte definido no Painel continua valendo ao
  abrir o Mapa de Fluxo ou o Relatório, nos dois sentidos — limpar um
  filtro no Mapa de Fluxo também limpa no Painel.
- **Ícones de ajuda (`!`)** no lugar do texto explicativo de cada
  painel — clique para abrir/fechar o popup (os cartões de KPI do
  topo, sem ícone próprio, usam hover parado por ~3s).

### Painel (`index.html`)

- Cartões de indicadores (Cadastrados, Total oficial, Recebidos, %
  Recebidos, Países de origem, PPGs envolvidos) centralizados no topo.
- "Programa de mobilidade" — um botão colorido por modalidade (GCUB-MOB
  verde escuro, ERASMUS+ azul royal, Move La América vermelho, PROAFRI
  amarelo), no topo da coluna esquerda.
- Mapa coroplético dos países de origem (indicadores oficiais) e
  "Continente de Origem" em barras (inclui América Central, distinta de
  América do Norte).
- "Evolução por edição": barras empilhadas (mestrado × doutorado ×
  graduação) + linha pontilhada de alunos acumulados no eixo direito.
  Botões "Barra" e "Linha" no cabeçalho ocultam cada série
  independentemente; a legenda dos níveis é clicável (filtro
  multi-seleção).
- "Ranking de países": barras horizontais alinhadas à direita, com
  nome do país e valor escritos dentro da própria barra.
- "PPG — Siglas": etiquetas com `Codigo_PPG`; passe o mouse numa sigla
  para ver o nome completo do programa.
- Cartões de Fluxo (IN/OUT) e Tipo de Mobilidade; ranking de fonte de
  financiamento.
- Lista de participantes com nome em formato ABNT, com busca por nome/
  país/PPG.
- Botão "Gerar relatório" no topo abre `relatorio.html` numa nova aba.
- Barra de aviso de privacidade em toda a largura, e assinatura
  "Powered by William Pinheiro" / fonte dos dados no rodapé.
- Tema claro/escuro persistido (localStorage), sem flash no carregamento
  e com todos os gráficos recolorindo corretamente ao trocar de tema.

### Mapa de Fluxo (`flowmap.html`)

Mapa-múndi onde cada país de origem/destino se liga a Manaus por uma
linha animada (efeito "formiguinha" via `stroke-dashoffset`), respeitando
os filtros ativos do Painel:

- Linha verde = fluxo **IN** (país de origem → Manaus); linha laranja =
  fluxo **OUT** (Manaus → país de destino) — hoje só há dados de IN
  (`Fluxo_Mobilidade` = `OUT` ainda não tem registros reais na base). A
  direção do desenho da linha (sem seta na ponta) já indica o sentido.
- Espessura da linha e tamanho do ponto do país são proporcionais ao
  número de estudantes.
- Coordenadas dos ~15 países envolvidos são aproximações fixas em
  `js/flowmap.js` (`COUNTRY_COORDS`) — a planilha de origem não traz
  latitude/longitude, só o país e o código ISO.
- Tooltip ao passar o mouse numa linha ou num ponto mostra o país e a
  quantidade.

### Relatório (`relatorio.html`)

Documento de uma coluna, aberto pelo botão "Gerar relatório" do Painel,
com o recorte de filtros ativo no momento:

- Resumo de KPIs, os gráficos de países de origem (mapa) e evolução por
  edição, e tabelas por programa de mobilidade, país, PPG, nível,
  gênero, situação, financiamento, continente e a lista completa de
  participantes.
- Botão "Exportar PDF" aciona a impressão do navegador — use "Salvar
  como PDF" no diálogo nativo para gerar o arquivo (evita chamar
  `window.print()` direto do Painel, que trava a página com o diálogo
  modal).

## Publicar / atualizar o GitHub Pages

O Pages já está configurado para servir a raiz da branch `main` nos dois
repositórios remotos (`origin` tem dois push URLs). Qualquer `git push`
atualiza os dois em alguns minutos:
- https://willpine1992.github.io/mobilidade-uea/
- https://internacionalizacao-create.github.io/mobilidade-uea/

Antes de commitar, confira `git status` para garantir
que `interno.html` e `js/data-interno.js` continuam fora (o `.gitignore`
já cuida disso, mas vale checar após qualquer renomeação de arquivo).

## Atualizar os dados

Quando a planilha de origem for atualizada (novas modalidades, novos
registros), regenerar `js/data.js` a partir de `Fato_Mobilidades`,
`Dim_Programas_Mobilidade` e das colunas `Nome_ABNT`/`Sexo_Genero` de
`Dim_Participantes_dados reais` — sem reintroduzir e-mail, telefone,
matrícula ou nome completo nesse arquivo.
