# Plataforma de Analytics de Eventos de Fogo

> Sistema público e institucional (INPE) de **análise histórica de eventos de fogo
> no Brasil, guiado por um serviço de chatbot em linguagem natural**, com recorte
> territorial e por área de interesse e cruzamento com outros produtos INPE.

*Documento de escopo. Não contém decisões de implementação.*

---

## 1. Objetivo

> **Criar um sistema de análise histórica de eventos de fogo no Brasil guiado por
> um serviço de chatbot em PLN.**

O chatbot é o **serviço condutor** do sistema — não um canal alternativo a uma
interface gráfica. O usuário chega com uma pergunta em linguagem natural; o mapa,
os gráficos e as tabelas existem para materializar a resposta.

Sobre 20+ anos de série histórica, o sistema responde a pergunta que o dado de fogo
isolado não responde: **"quanto queimou, isso é normal, e o que significa?"**

Para isso, transforma detecções de satélite em **eventos de fogo analisáveis** —
com início, fim, duração e área — e permite que quatro perfis de usuário
interroguem essa base sobre o território ou a área que lhes interessa, cruzando
com outras bases ambientais.

### O que "guiado por chatbot" implica

1. **A conversa é a porta de entrada.** O sistema é desenhado a partir da pergunta
   do usuário, não a partir de um painel com filtros. A interface gráfica serve à
   conversa, e não o contrário.
2. **Mapa, gráfico e tabela são artefatos da resposta**, não destinos separados.
   Cada um aparece porque a pergunta pediu aquela forma — ver RF-5.
3. **A camada de confiança deixa de ser acabamento.** Se a conversa conduz, ela
   carrega a responsabilidade pelo número: declarar parâmetros, recusar quando não
   sabe e sinalizar resposta frágil passam a ser requisitos centrais, não
   periféricos — ver RF-4.4 a RF-4.6 e RNF-2.

### Por que existe

O mercado de plataformas de fogo está bem servido em duas perguntas e vazio na
terceira:

| Pergunta | Já atendida por |
|---|---|
| "Onde está queimando agora?" | SIPAM, NASA FIRMS, Mapa Fogos PT |
| "Quanto queimou e isso é muito?" | MapBiomas, ALARMES, Global Nature Watch |
| **"O que isso significa para a minha área / para essa população?"** | **Nenhuma plataforma de forma completa** |

O levantamento de 15 plataformas que embasa este documento mostra ainda que:

- **Apenas 2 das 15** modelam o incêndio como objeto (com ID, duração e área).
  As demais expõem pontos ou rasters soltos.
- **Nenhuma das 15** entrega recorrência — quantas vezes a mesma área queimou.
- **Nenhuma das 15** conecta fogo a qualidade do ar e exposição populacional.
- **93%** são criticadas por não permitir exportação de dado bruto.
- **87%** falham em comparação entre períodos ou em série histórica do mesmo objeto.

### Diferenciais pretendidos

1. **Chatbot em PLN como serviço condutor** — consulta sobre um espaço de
   perguntas grande demais para caber em formulário.
2. **O evento como unidade de análise**, e não o foco isolado.
3. **Recorrência e persistência** como atributos de primeira classe.
4. **Cruzamento fogo × qualidade do ar (SISAM)** — cadeia inédita no mercado.
5. **Área de interesse arbitrária**, desenhada pelo usuário.

---

## 2. Público-alvo

Quatro perfis, em ordem de centralidade:

| Perfil | O que precisa | O que o produto entrega |
|---|---|---|
| **Pesquisador** | Dado bruto, método declarado, reprodutibilidade, citação | Série longa, exportação, proveniência |
| **Gestor público / brigada** | Recorte por jurisdição, histórico do território, planejamento | Recorte fundiário, recorrência, ranking |
| **Saúde pública** | Relação entre fogo e qualidade do ar | Cruzamento com SISAM |
| **Jornalista** | Número citável, comparação, narrativa | Comparação histórica, exportação, linguagem clara |

**Nota sobre o gestor de brigada:** sendo o produto retrospectivo, este perfil é
atendido **parcialmente** — em planejamento e análise histórica, não em operação
de campo.

**Nota sobre saúde pública:** com o eixo socioeconômico fora de escopo, o produto
caracteriza a exposição (concentração de poluente por município) mas **não a
quantifica** (número de pessoas expostas).

---

## 3. Escopo — decisões tomadas

| Dimensão | Decisão |
|---|---|
| Unidade de análise | **Evento de fogo** |
| Definição de evento | Agregação de focos por raio do satélite, em janela de **7 dias** |
| Métricas principais | **Contagem de eventos** e **área do evento em km²** |
| Temporalidade | **Retrospectivo** (tempo real fora do escopo inicial) |
| Série histórica | **20+ anos** |
| Satélite | **GOES primeiro**; constelação completa no produto final |
| Abrangência | **Brasil** |
| Acesso | **Anônimo**, sem cadastro |
| Idioma | **PT-BR** |
| Modelo | **Público, institucional (INPE)** |
| Interface | Chatbot em linguagem natural conduzindo mapa, gráficos e tabelas |
| Área de interesse | Seleção de território, desenho no mapa e **carregamento de KML / shapefile / GeoJSON** |

### Fora de escopo

- **Alerta, notificação e despacho operacional** — outro produto, outros
  requisitos de disponibilidade.
- **Catálogo de imagem de satélite** — INPE BDC, Copernicus e FIRMS são
  fornecedores, não concorrentes.
- **Produção de dado primário** — o produto consome detecção existente.
- **GIS de propósito geral** — edição topológica, análise raster avançada.
- **Previsão de risco** — competência de INMET, CPTEC e SIFAU.
- **Eixo socioeconômico (IBGE)** — população e vulnerabilidade não entram nesta
  versão.

---

## 4. Modelo conceitual

### Evento de fogo

Unidade central do produto. Um evento agrega detecções (focos) próximas no espaço
— dentro do raio característico do satélite — e contínuas no tempo, com janela de
**7 dias** entre detecções consecutivas.

Atributos conceituais:

| Atributo | Descrição |
|---|---|
| Identificador | Chave estável do evento |
| Data de início / fim | Primeira e última detecção |
| Duração | Intervalo entre início e fim |
| Área | Área do evento em km², derivada da agregação dos focos |
| Quantidade de focos | Detecções que compõem o evento |
| Status | Estado do evento na série |
| Recorrência | Quantas vezes a mesma área queimou |
| Persistência | Continuidade da atividade ao longo do evento |
| Geometria | Polígono do evento |
| Linhagem | Relação com eventos anteriores absorvidos ou sucedidos |
| Regiões | Territórios cruzados pelo evento |

> **Consequência da janela de 7 dias:** dois fogos na mesma área separados por
> mais de 7 dias são eventos distintos — e é isso que alimenta a recorrência.

### Eixos de cruzamento

Bases externas que respondem o que o dado de fogo sozinho não responde.

| Eixo | Acrescenta | Status |
|---|---|---|
| Territorial (município, estado, bioma) | Onde administrativamente | **Pré-requisito** |
| Fundiário / proteção (UC, TI, assentamento, quilombo, CAR) | Regime legal e competência | **Em escopo** |
| Uso e cobertura do solo | O que queimou | **Em escopo** |
| Meteorologia | Em que condição queimou | **Em escopo** |
| Qualidade do ar (SISAM) | Fumaça e exposição | **Em escopo** |
| Infraestrutura (rodovias, linhas de transmissão) | O que estava em risco | **Em escopo** |
| Socioeconômico (IBGE) | Quem foi afetado | **Fora de escopo** |

#### Critério para admitir um novo eixo

Um eixo só entra quando passa nos cinco:

1. **Pergunta nomeável** — existe uma pergunta concreta, na voz de um dos quatro
   perfis, que só esse eixo responde.
2. **Profundidade histórica compatível** com os 20 anos — ou limitação conhecida,
   declarável e aceita.
3. **Regra de junção defensável e escrita** para o descasamento de granularidade.
4. **Fonte pública, estável e com método declarado.**
5. **Não duplica** eixo existente.

> **Critério de reprovação, que prevalece sobre os cinco:** se o cruzamento produz
> um número que não se sabe defender, o eixo não entra — mesmo sendo tecnicamente
> fácil. Num produto INPE com pesquisador e jornalista entre os usuários, um número
> indefensável custa mais do que uma funcionalidade ausente.

---

## 5. Requisitos funcionais

### RF-1 — Consulta e recorte

| ID | Requisito |
|---|---|
| RF-1.1 | Selecionar território por município, estado ou bioma |
| RF-1.2 | Desenhar área de interesse arbitrária sobre o mapa |
| RF-1.3 | **Carregar arquivo de área de interesse em KML, shapefile ou GeoJSON** |
| RF-1.4 | Editar ou remover a área de interesse depois de criada ou carregada |
| RF-1.5 | Definir período arbitrário, **inclusive janelas que cruzam anos** (ex.: set/2024 a fev/2025) |
| RF-1.6 | Combinar recorte territorial, área de interesse e período numa mesma consulta |
| RF-1.7 | Filtrar por atributos do evento (duração, área, status, recorrência) |

> RF-1.5 é diferencial explícito: o mercado analisado falha nisso — as plataformas
> de referência não isolam meses arbitrários nem permitem temporada que atravessa
> o ano civil.

#### RF-1.3 — Comportamento do carregamento de área

O carregamento de arquivo é a forma pela qual o usuário traz **a área que já é dele**
— propriedade, talhão, unidade de gestão, recorte de estudo — em vez de redesenhá-la
à mão. É requisito de entrada para pesquisador e gestor, e é a lacuna que mais
limita o concorrente direto do produto.

| ID | Requisito |
|---|---|
| RF-1.3.1 | Aceitar **KML**, **shapefile** e **GeoJSON** |
| RF-1.3.2 | Declarar o tratamento de arquivo com múltiplas feições **antes** do envio (cada feição vira uma área distinta, ou são unificadas) |
| RF-1.3.3 | Nomear a área a partir do atributo de nome do arquivo, com regra de fallback declarada quando não houver |
| RF-1.3.4 | Validar geometria e sistema de referência, reprojetando quando necessário, e recusar arquivo inválido com mensagem específica |
| RF-1.3.5 | Listar as áreas carregadas na sessão, com remoção individual |
| RF-1.3.6 | Informar o limite de tamanho e o que fazer quando o arquivo excede |

> **Lições do levantamento aplicadas aqui:** teto de upload pequeno demais inviabiliza
> polígono municipal detalhado ou malha de talhões (falha registrada no Global Nature
> Watch); explicar o comportamento de múltiplas feições antes do envio evita que o
> usuário descubra sozinho (acerto do ALARMES SIFAU); **não** permitir editar a área
> depois de enviada é falha registrada no mesmo SIFAU — daí RF-1.4.

### RF-2 — Visualização espacial

| ID | Requisito |
|---|---|
| RF-2.1 | Exibir eventos de fogo no mapa como geometria, não como marcador |
| RF-2.2 | Codificar visualmente o evento por atributo relevante (status, idade, área ou duração) |
| RF-2.3 | Exibir camadas de contexto dos eixos de cruzamento em escopo |
| RF-2.4 | Legenda sempre visível e coerente com o nível de zoom |
| RF-2.5 | Abrir a ficha completa do evento a partir do mapa |
| RF-2.6 | Navegar temporalmente sobre o mapa dentro do período consultado |

### RF-3 — Análise

| ID | Requisito |
|---|---|
| RF-3.1 | Contagem de eventos e área total em km² para o recorte e período |
| RF-3.2 | Série temporal em granularidade diária, mensal e anual |
| RF-3.3 | Comparação entre dois períodos arbitrários |
| RF-3.4 | Comparação contra referência histórica (percentis e faixa da série) |
| RF-3.5 | Variação frente ao período anterior e ao mesmo período do ano anterior |
| RF-3.6 | Ranking de territórios por métrica escolhida |
| RF-3.7 | Composição do total por eixo de cruzamento |
| RF-3.8 | Recorrência: quantas vezes a área queimou na série |
| RF-3.9 | Estatística restrita à área de interesse desenhada |

> RF-3.4 é o requisito central do produto: é o que responde *"isso é normal?"*,
> pergunta que justifica a série de 20 anos.

### RF-4 — Interface chatbot

| ID | Requisito |
|---|---|
| RF-4.1 | Receber solicitação em linguagem natural (PT-BR) |
| RF-4.2 | Traduzir a solicitação em consulta estruturada sobre a base |
| RF-4.3 | Responder em **texto, tabela ou gráfico**, conforme a natureza da pergunta |
| RF-4.4 | Declarar, junto à resposta, os parâmetros usados (período, recorte, métrica, agregação, fonte) |
| RF-4.5 | Recusar explicitamente quando não souber responder |
| RF-4.6 | Sinalizar resposta frágil quando a pergunta for respondível mas a base for insuficiente |
| RF-4.7 | Manter o contexto da conversa dentro da sessão |

#### RF-4.5 — Casos de recusa

Três situações distintas, com tratamento distinto:

| Caso | Tratamento |
|---|---|
| **Informação inexistente ou fora do escopo** | *"Não encontrei essa informação e, para não te passar nada errado, sugiro confirmar com a nossa equipe pelo e-mail."* |
| **Capacidade ainda não implementada** | *"Ainda não sei responder isso. Você pode reformular a pergunta."* |
| **Resposta possível, porém frágil** (período na borda da série, AOI muito pequena, amostra reduzida) | Responder **com ressalva explícita** sobre a limitação — não recusar |

> O terceiro caso é o de maior risco institucional: é onde um produto INPE ganha
> ou perde credibilidade. Responder com fluência um número frágil é pior do que
> recusar.

### RF-5 — Gráficos

Todo gráfico é selecionado pela **natureza da pergunta**, não pelo pedido literal
do usuário.

| Intenção | Forma |
|---|---|
| Valor único | Número em destaque (não gráfico de uma barra) |
| Tendência no tempo | Linha |
| "Isso é normal?" | Linha do período + faixa de referência histórica |
| Comparação de 2–3 períodos | Linha ou barras agrupadas |
| Grade ano × mês (sazonalidade) | Heatmap sequencial |
| Ranking de territórios | Barras horizontais ordenadas, cor única |
| Parte-todo | Barras empilhadas, máx. ~6 categorias + "Outros" |
| Distribuição espacial | Coroplético sequencial |
| Listagem de itens | Tabela |

| ID | Requisito |
|---|---|
| RF-5.1 | Todo gráfico exibe legenda e rótulo de eixo |
| RF-5.2 | Todo gráfico declara em rodapé os parâmetros que o geraram |
| RF-5.3 | Todo gráfico tem visão equivalente em tabela |
| RF-5.4 | Todo gráfico é exportável em imagem e em dado |
| RF-5.5 | Uma linha sob o título explica o que o indicador representa |

**Restrições de forma (não negociáveis):**

- **Nunca eixo duplo** (duas escalas verticais no mesmo gráfico). O pedido
  *"mostra área queimada e MP2.5 juntos"* é natural e será frequente; a resposta
  correta são dois gráficos empilhados compartilhando o eixo horizontal, ou as
  séries indexadas a uma base comum.
- **Nunca mais de ~7 classes de cor** com significado. Acima disso: tabela,
  agrupamento em "Outros", ou heatmap.
- **Cor segue a entidade, nunca a posição no ranking.**

### RF-6 — Exportação e proveniência

| ID | Requisito |
|---|---|
| RF-6.1 | Exportar resultado de análise em formato tabular |
| RF-6.2 | Exportar eventos em formato geoespacial |
| RF-6.3 | Exportar gráfico como imagem |
| RF-6.4 | Toda exportação carrega os parâmetros da consulta que a gerou |
| RF-6.5 | Declarar a procedência de cada evento (satélite e data de detecção) |
| RF-6.6 | Exibir estado da base: cobertura temporal e data da última atualização |

---

## 6. Requisitos não funcionais

### RNF-1 — Desempenho e escala

| ID | Requisito |
|---|---|
| RNF-1.1 | Consulta por área desenhada sobre a série completa deve responder em tempo compatível com uso interativo |
| RNF-1.2 | O volume de resposta deve ser proporcional à pergunta — resultados agregados não devem exigir transporte do dado bruto |
| RNF-1.3 | Consultas de grande abrangência devem informar a dimensão do resultado antes de executar |
| RNF-1.4 | O sistema deve degradar de forma previsível sob carga, sem falha silenciosa |
| RNF-1.5 | O limite de tamanho de arquivo de área deve comportar polígono municipal detalhado e malha de talhões — e ser declarado ao usuário |

> **Risco estrutural:** área arbitrária + série de 20 anos + múltiplos eixos é a
> combinação mais cara possível de consulta, e é o núcleo do produto. Combinada com
> acesso anônimo (carga imprevisível, sem atribuição), é o principal risco de
> viabilidade.

### RNF-2 — Confiança e rastreabilidade

| ID | Requisito |
|---|---|
| RNF-2.1 | Todo número exibido declara período, recorte, métrica e fonte |
| RNF-2.2 | Toda métrica tem unidade explícita |
| RNF-2.3 | Todo campo do modelo de dados tem definição publicada |
| RNF-2.4 | Limitações conhecidas da série são declaradas ao usuário no ponto de uso |
| RNF-2.5 | Divergências esperadas frente a outras bases públicas são explicadas |

> **Justificativa:** as plataformas mais confiáveis do levantamento declaram
> margem de erro, cadência e método; a pior avaliada não define nenhum dos próprios
> indicadores. Num produto conversacional, esta camada não é acabamento — é
> controle de risco.

### RNF-3 — Acesso e sessão

| ID | Requisito |
|---|---|
| RNF-3.1 | Acesso anônimo, sem cadastro |
| RNF-3.2 | Sem persistência entre sessões — área de interesse e histórico de análises existem apenas na sessão corrente |
| RNF-3.3 | Proteção contra uso abusivo sem impor autenticação |

> RNF-3.2 é **consequência direta** da decisão por acesso anônimo. Área de
> interesse salva, histórico de análises e preferências persistentes ficam fora do
> produto enquanto não houver identidade de usuário.

### RNF-4 — Acessibilidade e apresentação

| ID | Requisito |
|---|---|
| RNF-4.1 | Identidade de série nunca depende só de cor |
| RNF-4.2 | Paleta validada para deficiência de visão de cores e contraste |
| RNF-4.3 | Alternativa tabular para todo conteúdo gráfico |
| RNF-4.4 | Interface legível em tela pequena |
| RNF-4.5 | Linguagem clara — índices e indicadores explicados pelo que significam na prática, não pela fórmula |

### RNF-5 — Operação e manutenção

| ID | Requisito |
|---|---|
| RNF-5.1 | Cada eixo de cruzamento é um pipeline versionado e revalidável |
| RNF-5.2 | Erros expostos ao usuário não revelam infraestrutura interna |
| RNF-5.3 | Falhas e indisponibilidades são declaradas, não silenciosas |
| RNF-5.4 | Mudanças na série (entrada de novo satélite, nova malha) são versionadas e comunicadas |

### RNF-6 — Idioma e contexto

| ID | Requisito |
|---|---|
| RNF-6.1 | Interface e respostas em PT-BR |
| RNF-6.2 | Abrangência geográfica: Brasil |

---

## 7. Premissas metodológicas em aberto

> **Bloco crítico.** São quatro questões de **método**, não de governança. Elas se
> contaminam: resolver uma sem as outras produz inconsistência. Determinam se o
> produto consegue responder *"isso é normal?"* — sua razão de existir.

### PM-1 — Fronteiras territoriais mudam no tempo

Unidades de conservação, assentamentos e limites municipais mudaram ao longo dos
20 anos. O CAR só existe a partir do início da década de 2010 — **não cobre metade
da série**.

Cruzar um evento de 2004 com a malha de hoje faz o produto afirmar que *"queimou
dentro de área protegida"* sobre uma área que **não era protegida naquela data**.

Duas posturas possíveis, ambas legítimas:

| Postura | Responde | Custo |
|---|---|---|
| Malha atual aplicada a toda a série | *"Esta área, hoje protegida, queimou quanto em 20 anos?"* | Baixo |
| Malha vigente na data do evento | *"Quanto queimou dentro de áreas protegidas em 2004?"* | Alto — exige versionar malhas no tempo |

**Não declarar qual está em uso é o pior cenário.**

**Status:** em aberto.

### PM-2 — Homogeneidade da série

A capacidade de detecção mudou substancialmente em 20 anos. Contar eventos de
sensores diferentes ao longo da série produz uma curva cuja tendência mistura
**aumento de fogo** com **aumento de capacidade de detecção**.

**A decisão por começar pelo GOES não contorna o problema — concentra-o.** A série
GOES atravessa gerações de sensor bem distintas, com transição por volta de
2017–2018, **no meio da janela de 20 anos**.

Questão em aberto: adotar satélite de referência para série histórica (convenção
do próprio INPE), oferecer as duas leituras, ou declarar a quebra.

**Status:** em aberto.

### PM-3 — Definição e comparabilidade da área

A área do evento é derivada da agregação de focos pelo raio do sensor. Duas
consequências:

1. **A área via GOES será sistematicamente maior** que a derivada de sensores
   polares, e muito maior que cicatriz de queimada efetivamente mapeada.
2. **Os números vão mudar** quando os demais satélites entrarem — isso precisa ser
   antecipado como mudança esperada, não como correção de erro.

Um usuário que comparar o km² deste produto com o hectare de plataformas que mapeiam
cicatriz encontrará divergência grande. Ambos podem estar certos — são grandezas
distintas. Exige nome próprio e nota metodológica. (vamos colocar uma opção de conversão de unidade ?)

**Status:** em aberto.

### PM-4 — Versionamento da série

Decorre de PM-2 e PM-3: quando a constelação completa entrar, contagens e áreas
mudam retroativamente. Falta definir como versionar a série, como comunicar a
mudança e se as leituras anteriores permanecem acessíveis.

**Status:** em aberto.

---

## 8. Pontos em aberto — governança

Dependem de decisão da equipe, não de informação faltante.

| ID | Questão | Observação |
|---|---|---|
| **GOV-1** | Nome do produto | Não definido |
| **GOV-2** | Citabilidade da análise | Há papers publicados sobre a metodologia, utilizáveis como referência. Falta decidir se a análise individual recebe identidade estável (link permanente, "como citar") |
| **GOV-3** | Autoridade do número | Como o produto se posiciona frente aos números já divulgados pelo Programa Queimadas. Divergência é esperada, dadas PM-1 a PM-3 |
| **GOV-4** | Licença dos dados derivados e das exportações | Não definida |

> **GOV-2 e GOV-3 são o mesmo assunto por dois ângulos** — ambos tratam de sob qual
> autoridade o número é publicado. Recomenda-se levá-los juntos à equipe.

---

## 9. Referências de mercado

Levantamento de 15 plataformas (426 apontamentos: 270 acertos, 156 lacunas) que
embasa as decisões deste documento.

### Padrões a adotar

| Padrão | Origem |
|---|---|
| Faixa de referência histórica (percentis) sob a série do ano corrente | ALARMES WebGIS |
| Margem de erro declarada junto ao número | ALARMES WebGIS |
| Período exato do cálculo declarado em caixa própria | ALARMES WebGIS |
| Exportação universal: todo gráfico em imagem, dado e tabela | ALARMES WebGIS |
| Busca única com autocomplete cobrindo todos os tipos de território | MapBiomas |
| Seleção de território que ajusta o agrupamento automaticamente | MapBiomas |
| Ranking convertível em coroplético sobre o mapa | MapBiomas |
| Uma linha sob cada título explicando o indicador | MapBiomas |
| Narrativa gerada do dado, com julgamento comparativo | Global Nature Watch |
| Rodapé declarando os parâmetros daquele resultado | Global Nature Watch |
| Fonte citada dentro da própria narrativa | AmzFire SERVIR |
| Dicionário de dados completo (classe, nome, explicação, unidade, fonte) | AmzFire SERVIR |
| Ficha de evento com histórico de passagens do satélite | SIPAM |
| Cor codificando idade da detecção | SIPAM |
| Indisponibilidade declarada em vez de falha silenciosa | NASA FIRMS |
| Cobertura e lacunas da base consultáveis antes da consulta | NASA FIRMS |
| Índice explicado pelo significado prático, não pela fórmula | Mapa Fogos PT |
| Desenho de área com editar e apagar; região por endereço | INPE BDC Explorer |
| Contagem prévia de resultados antes de executar a busca | INPE BDC Explorer |
| Comportamento de upload explicado antes do usuário descobrir | ALARMES SIFAU |

### Erros documentados a evitar

| Erro | Onde ocorreu |
|---|---|
| Gráfico de rosca com 27 fatias, legenda maior que o gráfico | SIPAM |
| Rampa sequencial para distinguir anos — anos adjacentes indistinguíveis | MapBiomas |
| Série temporal sem legenda nem rótulo de eixo | INPE BDC Explorer |
| Nenhum gráfico — todo o conteúdo em tabela | AmzFire, ALARMES SIFAU |
| Vários gráficos e nenhum número agregado em destaque | IBAMA SisFogo |
| Indicadores sem definição nem link para metodologia | IBAMA SisFogo |
| Seletor com milhares de itens sem busca textual | ALARMES WebGIS |
| Objeto geográfico exibido mas não clicável | MapBiomas, ALARMES WebGIS |
| Período que não cruza anos — temporada de fogo não cabe no seletor | MapBiomas |
| Relatório dito "auditável" sem declarar fonte, modelo ou limitação | BrasaGPT |
| Teto de upload pequeno demais para polígono municipal detalhado | Global Nature Watch |

### Concorrência direta

| Plataforma | Tem | Não tem |
|---|---|---|
| **BrasaGPT** | PLN sobre dado INPE, fluxo guiado, fontes documentadas | Mapa, granularidade abaixo de estado, área própria, auditabilidade declarada |
| **GeoGPT** | Chat, execução de código, RAG, literatura | Qualquer camada geoespacial, série temporal, comparação |

> **O espaço entre os dois é o produto:** BrasaGPT tem dado sem mapa; GeoGPT tem
> chat sem geo. Nenhum dos dois tem área de interesse + mapa + análise conversacional
> no mesmo lugar.

---

## 10. Glossário

| Termo | Definição |
|---|---|
| **Foco** | Detecção individual de anomalia térmica por satélite |
| **Evento de fogo** | Agregação de focos próximos no espaço (raio do satélite) e contínuos no tempo (janela de 7 dias) |
| **Área do evento** | Área em km² derivada da agregação dos focos que compõem o evento |
| **Recorrência** | Número de vezes que uma mesma área queimou ao longo da série |
| **Persistência** | Continuidade da atividade de fogo ao longo do evento |
| **AOI** | Área de interesse — recorte espacial definido pelo usuário |
| **Eixo de cruzamento** | Base externa juntada ao evento para responder o que o dado de fogo sozinho não responde |
| **SISAM** | Sistema de Informações Ambientais Integrado à Saúde Ambiental (INPE) — concentração de poluentes por município |
| **Satélite de referência** | Sensor único adotado para manter comparabilidade da série histórica |

---

## 11. Estado deste documento

Documento de **escopo e análise**. Não define arquitetura, tecnologia,
modelagem física de dados nem cronograma.

**Bloqueiam análise quantitativa:** PM-1, PM-2, PM-3, PM-4.
**Bloqueiam publicação:** GOV-2, GOV-3, GOV-4.
