# pdf-to-maps-extractor

**Skill de extração e reconstrução geométrica de memoriais descritivos e plantas a partir de PDF, com projeção geodésica e exportação para SIG/CAD.**

[![Licença: MIT](https://img.shields.io/badge/licen%C3%A7a-MIT-green.svg)](LICENSE)
![Node.js](https://img.shields.io/badge/Node.js-%E2%89%A518-339933)
![SIRGAS2000](https://img.shields.io/badge/SIRGAS2000-UTM%20%2F%20Geo-0057b8)
![Offline](https://img.shields.io/badge/processamento-100%25%20local-lightgrey)

---

## Sumário

1. [O que é esta skill](#1-o-que-é-esta-skill)
2. [Escopo: o que faz e o que não faz](#2-escopo-o-que-faz-e-o-que-não-faz)
3. [Repositórios e linhagem do projeto](#3-repositórios-e-linhagem-do-projeto)
4. [Metodologia](#4-metodologia)
5. [Fundamentação matemática e normativa](#5-fundamentação-matemática-e-normativa)
6. [Instalação](#6-instalação)
7. [Uso](#7-uso)
8. [Estrutura do repositório](#8-estrutura-do-repositório)
9. [Formatos de saída](#9-formatos-de-saída)
10. [Conformidade e boas práticas](#10-conformidade-e-boas-práticas)
11. [Segurança da informação e LGPD](#11-segurança-da-informação-e-lgpd)
12. [Limitações e critérios de validação](#12-limitações-e-critérios-de-validação)
13. [Roteiro de evolução](#13-roteiro-de-evolução)
14. [Como contribuir](#14-como-contribuir)
15. [Autoria, créditos e licença](#15-autoria-créditos-e-licença)

---

## 1. O que é esta skill

A **pdf-to-maps-extractor** é uma skill de engenharia cartográfica que converte documentos PDF — memoriais descritivos georreferenciados, plantas, cadernetas digitalizadas e relatórios de vértices — em **geometria analítica validada**, pronta para uso em Sistemas de Informação Geográfica (SIG), CAD e plataformas de regularização fundiária.

O problema que ela resolve é recorrente na prática profissional brasileira: o dado geométrico do imóvel existe apenas em forma documental (texto e tabelas em PDF), frequentemente com coordenadas em graus, minutos e segundos (GMS), azimutes e distâncias de caminhamento, sem arquivo vetorial associado. A skill reconstrói esse perímetro de forma determinística, sem inferência estatística sobre a geometria, e entrega o polígono fechado, projetado e documentado.

**Princípios de projeto**

| Princípio | Consequência prática |
| --- | --- |
| Processamento 100% local e offline | Nenhum PDF, coordenada ou dado de cliente sai do computador do usuário |
| Determinismo | Mesma entrada produz mesma saída; nenhuma etapa depende de modelo generativo para calcular geometria |
| Rastreabilidade | Cada vértice extraído é vinculado à linha de origem no PDF e registrado em log técnico |
| Separação entre extração e decisão | A skill extrai, projeta, valida e exporta; o julgamento técnico (aceitar, corrigir, retificar) permanece com o responsável técnico |

> **Nota de escopo.** Esta skill não substitui a análise do responsável técnico nem a conferência contra o documento original. Ela reduz o trabalho manual de transcrição e padroniza a saída vetorial; a validação legal do perímetro continua sendo ato profissional.

---

## 2. Escopo: o que faz e o que não faz

**Faz**

- Leitura de PDF **nativo** (texto vetorial com posicionamento espacial) e de PDF **rasterizado** (imagem digitalizada, via OCR).
- Extração de coordenadas em **GMS** (graus, minutos, segundos) e em **graus decimais**, com higienização de separadores e símbolos (`°`, `'`, `"`, `”`, `,`, `.`).
- Extração de **caminhamento perimétrico**: vértice de origem, azimute ou rumo, distância e vértice de destino.
- Conversão entre **azimute e rumo** nos quatro quadrantes e cálculo de coordenadas por transporte sucessivo.
- **Projeção geodésica** de latitude/longitude para **UTM** sobre elipsoide GRS80/SIRGAS2000, com determinação automática de fuso e hemisfério.
- **Validação topológica**: fechamento do anel, detecção de vértices duplicados, verificação de auto-interseção e cálculo de erro de fechamento linear.
- **Cálculo geodésico** de área e perímetro (motor Turf.js) e geração de relatório técnico em texto.
- **Exportação** para GeoJSON, DXF (CAD), SVG e imagens de mapa 2D e 3D isométrico.
- Nomeação e organização automática de saídas por arquivo de origem.

**Não faz**

- Não corrige erros do documento de origem: divergências entre o memorial e a realidade de campo são **reportadas**, não arbitradas.
- Não executa ajustamento geodésico por mínimos quadrados nem transformação entre referenciais (por exemplo, SAD69 → SIRGAS2000); a conversão de datum exige parâmetros oficiais e é tratada como etapa externa.
- Não assina, não autentica e não substitui a **ART/TRT** nem o memorial descritivo assinado pelo profissional habilitado.
- Não envia dados para nuvem, API externa ou serviço de IA.
- Não realiza levantamento de campo: trabalha exclusivamente com o dado documental fornecido.

---

## 3. Repositórios e linhagem do projeto

| Papel | Repositório | Observação |
| --- | --- | --- |
| **Este projeto (skill)** | [`ErisonBarros/pdf-to-maps-extractor`](https://github.com/ErisonBarros/pdf-to-maps-extractor) | Repositório de distribuição da skill, ramo padrão `erison.barros`; documentação, empacotamento e evolução da skill |
| **Implementação de referência (upstream)** | [`D-H-O-R-A/pdf-to-maps`](https://github.com/D-H-O-R-A/pdf-to-maps) | Motor Node.js (ESM) v1.0.2, autoria de **Diego Oris**, licença MIT; base técnica do pipeline de extração, projeção e renderização |

**Relação entre os repositórios.** O projeto upstream é a implementação de referência do núcleo algorítmico (parser de PDF, extrator de vértices, projetor UTM, renderizadores e exportadores). Este repositório organiza a **skill** propriamente dita: instruções de acionamento, contrato de entrada e saída, critérios de validação, conformidade normativa brasileira e documentação de uso profissional. Alterações no upstream devem ser incorporadas de forma controlada, com registro do commit de origem.

**Sincronização.** O acompanhamento das atualizações do upstream é feito por automação dedicada: qualquer novo commit em `D-H-O-R-A/pdf-to-maps` dispara uma tarefa de verificação e incorporação das mudanças neste repositório, preservando o conteúdo próprio da skill.

---

## 4. Metodologia

O pipeline é organizado em oito etapas sequenciais, com ponto de falha explícito em cada uma. Nenhuma etapa "adivinha" geometria: quando o dado é insuficiente ou ambíguo, o processo para e reporta.

```text
[1] Ingestão            PDF nativo ou rasterizado
        │
[2] Leitura espacial    reconstrução de linhas por coordenadas X/Y do texto
        │
[3] Extração            regex estrito de vértices (GMS / decimal) e de caminhamento
        │
[4] Normalização        DMS → graus decimais; azimute ↔ rumo; unidades em metros
        │
[5] Validação           fechamento do anel, duplicidades, auto-interseção
        │
[6] Projeção            geodésica → UTM SIRGAS2000 (fuso e hemisfério automáticos)
        │
[7] Cálculo             área e perímetro geodésicos; erro de fechamento
        │
[8] Exportação          GeoJSON · DXF · SVG · PNG 2D/3D · relatório técnico
```

### 4.1 Ingestão e leitura espacial

Em PDFs nativos, o texto não chega na ordem de leitura humana: os operadores de desenho posicionam cada fragmento por coordenada. A etapa de leitura reconstrói as linhas lógicas ordenando os fragmentos por **Y** (de cima para baixo) e, dentro de cada linha, por **X** (da esquerda para a direita). Sem essa reconstrução, um vértice pode ser montado com fragmentos de linhas distintas — erro silencioso e grave.

Em PDFs rasterizados, a etapa equivalente é o OCR, com resultado intrinsecamente menos confiável: a qualidade depende da resolução, do contraste e da integridade da digitalização. Por isso, toda extração proveniente de raster é sinalizada no relatório para conferência obrigatória.

### 4.2 Extração

O extrator aplica padrões estritos para cadeias de vértice, aceitando variações usuais de separador e de hemisfério (`N/S`, `E/W`, `E/O`), e rejeitando cadeias incompletas. O resultado é uma lista de vértices com: identificador, latitude, longitude, altitude (quando presente) e a linha de origem no documento.

No caminhamento, a extração captura o par ordenado **azimute/rumo + distância** e o vértice de partida, permitindo reconstruir a poligonal por transporte sucessivo quando o documento não traz coordenadas absolutas de todos os vértices.

### 4.3 Normalização e validação

- Conversão de GMS para graus decimais com sinal definido pelo hemisfério.
- Conversão de azimute para rumo (e vice-versa) conforme o quadrante.
- Verificação de fechamento: o último vértice deve coincidir com o primeiro dentro da tolerância adotada.
- Detecção de vértices repetidos e de segmentos cruzados (auto-interseção), que invalidam a feição.
- Cálculo do erro de fechamento linear e comparação com a tolerância compatível com a classe do levantamento.

### 4.4 Projeção e cálculo

A projeção geodésica é feita com `proj4` sobre elipsoide GRS80 (parâmetros idênticos ao WGS84), com fuso e hemisfério determinados a partir da longitude e da latitude médias. Área e perímetro são calculados por formulação geodésica (`@turf/turf`), não por geometria plana — diferença relevante em imóveis de grande extensão.

### 4.5 Exportação

Cada PDF processado gera um diretório próprio em `./output/[NOME_DO_PDF]/`, com o conjunto completo de artefatos descrito na [seção 9](#9-formatos-de-saída). A organização por origem preserva a correspondência entre documento e geometria, requisito de auditoria.

---

## 5. Fundamentação matemática e normativa

### 5.1 Conversão de GMS para graus decimais

$$
DD = \operatorname{sgn}(D)\cdot\left(|D| + \frac{M}{60} + \frac{S}{3600}\right)
$$

Onde $D$ são os graus inteiros, $M$ os minutos inteiros e $S$ os segundos decimais. A função sinal é $-1$ quando o caractere de direção é `S`, `W` ou `O`, ou quando há sinal negativo explícito; caso contrário, $+1$.

### 5.2 Determinação de fuso e meridiano central

$$
\text{Fuso} = \left\lfloor \frac{\lambda + 180}{6} \right\rfloor + 1
\qquad\qquad
\lambda_0 = 6\cdot\text{Fuso} - 183
$$

Onde $\lambda$ é a longitude em graus decimais e $\lambda_0$ o meridiano central do fuso. O hemisfério define a falsa origem:

$$
N_{\text{falso}} =
\begin{cases}
10\,000\,000\ \text{m} & \text{se } \varphi < 0 \ \text{(hemisfério sul)}\\
0\ \text{m} & \text{se } \varphi \geq 0 \ \text{(hemisfério norte)}
\end{cases}
$$

O afastamento da falsa origem ($E_0 = 500\,000\ \text{m}$) é aplicado em ambos os hemisférios.

### 5.3 Azimute, rumo e transporte de coordenadas

Para um segmento entre os vértices $i$ e $i+1$:

$$
\Delta E = E_{i+1} - E_i \qquad \Delta N = N_{i+1} - N_i \qquad d = \sqrt{\Delta E^2 + \Delta N^2}
$$

$$
Az = \operatorname{atan2}(\Delta E, \Delta N) \quad \text{normalizado para } [0^\circ, 360^\circ)
$$

A conversão para rumo segue o quadrante: no 1º quadrante $R = Az$; no 2º, $R = 180^\circ - Az$; no 3º, $R = Az - 180^\circ$; no 4º, $R = 360^\circ - Az$, com prefixo e sufixo indicando os pontos cardeais.

### 5.4 Escala e enquadramento do desenho

Para desenhar o polígono em um canvas de largura $W$ e altura $H$ sem distorcer a proporção territorial, adota-se fator de escala uniforme $S$ com margem de segurança $P$:

$$
S = \min\left(\frac{W - 2P}{X_{\max} - X_{\min}},\ \frac{H - 2P}{Y_{\max} - Y_{\min}}\right)
$$

A escala uniforme é o que garante **aspecto 1:1** — qualquer fator independente por eixo produziria uma planta geometricamente falsa.

### 5.5 Projeção isométrica e sombreamento (mapa 3D)

A volumetria é gerada por translação para origem local (eliminando ruído numérico de coordenadas UTM de grande magnitude), rotação de guinada de $-45^\circ$ e projeção com arfagem de $35{,}264^\circ$ ($\sin\phi = 1/\sqrt{3}$). O sombreamento lateral usa produto escalar entre a normal da face e o vetor de luz, mapeado para luminosidade entre 35% e 100%.

### 5.6 Referências normativas aplicáveis

| Referência | Aplicação |
| --- | --- |
| **SIRGAS2000** (IBGE) | Referencial geodésico oficial do Brasil; base de toda projeção UTM na skill |
| **NBR 13133** (ABNT) | Execução de levantamento topográfico: classes, tolerâncias e critérios de fechamento |
| **NBR 14166** (ABNT) | Rede de referência cadastral municipal |
| **Decreto nº 89.817/1984** | Normas técnicas para a cartografia brasileira (PEC — Padrão de Exatidão Cartográfica) |
| **Lei nº 10.267/2001 e Decreto nº 4.449/2002** | Georreferenciamento de imóveis rurais e integração ao cadastro nacional |
| **Normas técnicas do INCRA (NTGIR)** | Especificações e normas para georreferenciamento de imóveis rurais |
| **SIGEF/INCRA** | Padrão de memorial descritivo e de vértices utilizado na validação de entrada |

> As tolerâncias numéricas aplicadas devem ser as da classe do levantamento declarada no documento. A skill não presume classe: quando o documento não a informa, o relatório registra a ausência e apresenta o erro de fechamento bruto para decisão do responsável técnico.

---

## 6. Instalação

### 6.1 Pré-requisitos

| Requisito | Versão mínima | Observação |
| --- | --- | --- |
| Node.js | 18 (LTS) | Recomendado 20 LTS ou superior |
| npm | 9 | Incluído no Node.js |
| Bibliotecas de build C++ | — | Necessárias para compilar `canvas` no Windows (Visual Studio Build Tools) e no Linux (`build-essential`, `libcairo2-dev`, `libpango1.0-dev`, `libjpeg-dev`, `libgif-dev`, `librsvg2-dev`) |
| Python | 3.8+ | Requerido pelo `node-gyp` durante a compilação de dependências nativas |
| Poppler (opcional) | — | Apenas para diagnóstico externo de PDFs |

### 6.2 Instalação do núcleo

```bash
git clone https://github.com/D-H-O-R-A/pdf-to-maps.git
cd pdf-to-maps
npm install
```

Dependências principais: `pdfjs-dist` (leitura de PDF), `canvas` (renderização 2D/3D), `proj4` (projeção geodésica), `@turf/turf` (cálculo geodésico) e `pdf-lib` (geração do PDF de exemplo).

### 6.3 Instalação como skill no Manus

Uma skill é um diretório com um arquivo `SKILL.md` (frontmatter `name` e `description`) e os materiais de apoio. Para instalar:

1. Clone este repositório no diretório de skills do ambiente:

   ```powershell
   git clone https://github.com/ErisonBarros/pdf-to-maps-extractor.git "$env:USERPROFILE\.manus\skills\pdf-to-maps-extractor"
   ```

2. Confirme a presença de `SKILL.md` na raiz do diretório clonado.
3. Reinicie a sessão para que a skill seja indexada.
4. Verifique o acionamento pedindo, por exemplo: *"extraia os vértices do memorial em anexo e gere GeoJSON e DXF"*.

> **Atenção ao ramo padrão.** Este repositório usa o ramo `erison.barros`. Se o clone não trouxer o conteúdo esperado, use `git clone -b erison.barros <url> <destino>`.

### 6.4 Verificação da instalação

```bash
node create_example.js      # gera exemplo_sigef.pdf (4 vértices, Coari/AM)
node bin/cli.js exemplo_sigef.pdf
```

Saída esperada: criação de `./output/exemplo_sigef/` com `vertices.json`, `mapa.png`, `mapa_3d.png`, `mapa.svg`, `mapa.geojson`, `mapa.dxf` e `log.txt`.

---

## 7. Uso

### 7.1 Linha de comando

```bash
# processa todos os PDFs da pasta padrão
node bin/cli.js

# processa uma pasta específica
node bin/cli.js "C:\processamento\memoriais"

# processa um arquivo isolado
node bin/cli.js "C:\processamento\memoriais\imovel_01.pdf"
```

### 7.2 Uso programático (ESM)

```js
import { readPDFAndReconstructLines } from "./src/parser/pdfReader.js";
import { extractVerticesFromReconstructedLines } from "./src/parser/vertexExtractor.js";
import { projectVerticesToUTM } from "./src/projection/utmProjector.js";
import { validateAndClosePolygon } from "./src/index.js";

async function extrairDados(pdfPath) {
  // 1. Reconstrói linhas por posição espacial no PDF
  const lines = await readPDFAndReconstructLines(pdfPath);

  // 2. Extrai vértices (GMS e decimal)
  const rawVertices = extractVerticesFromReconstructedLines(lines);

  // 3. Valida topologia e fecha o anel
  const validVertices = validateAndClosePolygon(rawVertices);

  // 4. Projeta para UTM SIRGAS2000
  const { projectedVertices, zone, isSouth } = projectVerticesToUTM(validVertices);

  console.log(`Zona ${zone}${isSouth ? "S" : "N"}`, projectedVertices);
}
```

### 7.3 Acionamento como skill (linguagem natural)

| Pedido | Resultado esperado |
| --- | --- |
| "Extraia os vértices deste memorial e gere GeoJSON e DXF" | Perímetro vetorial + relatório de fechamento |
| "Converta estas coordenadas GMS para UTM, fuso 23S" | Tabela de coordenadas projetadas |
| "Verifique se este caminhamento fecha e qual o erro linear" | Diagnóstico de fechamento com valor e tolerância |
| "Gere o mapa 2D e o bloco 3D do imóvel" | `mapa.png` e `mapa_3d.png` em alta resolução |
| "Produza o relatório técnico da área e do perímetro" | `log.txt` com área geodésica (ha e m²) e perímetro |

---

## 8. Estrutura do repositório

```text
pdf-to-maps-extractor/
├── README.md                  # este documento
├── LICENSE                    # MIT
├── SKILL.md                   # contrato de acionamento da skill (frontmatter + instruções)
├── assets/                    # imagens de demonstração (mapas 2D e 3D)
├── bin/
│   └── cli.js                 # interface de linha de comando
├── index.js                   # ponto de entrada ESM
├── create_example.js          # gerador do PDF SIGEF de teste
├── exemplo_sigef.pdf          # insumo público de demonstração
├── package.json               # metadados e dependências
└── src/
    ├── index.js               # coordenador de arquivos e concorrência
    ├── parser/
    │   ├── pdfReader.js       # leitura espacial do texto do PDF
    │   └── vertexExtractor.js # extrator estrito de vértices (GMS/decimal)
    ├── projection/
    │   └── utmProjector.js    # projeção geodésica → UTM SIRGAS2000
    ├── draw/
    │   ├── mapRenderer.js     # mapa 2D com grade e escala
    │   └── mapRenderer3D.js   # bloco isométrico 3D com sombreamento
    ├── export/
    │   ├── geojsonExporter.js # GeoJSON (OGC)
    │   ├── dxfExporter.js     # DXF R12 (POLYLINE)
    │   ├── svgExporter.js     # vetor SVG de alta definição
    │   └── logExporter.js     # relatório técnico (área/perímetro)
    └── utils/
        └── helper.js          # conversão DMS e gestão de diretórios
```

> A árvore acima reflete a implementação de referência. Arquivos adicionais da skill (por exemplo, `SKILL.md`, `references/` e `assets/`) são mantidos neste repositório conforme a evolução do pacote.

---

## 9. Formatos de saída

| Arquivo | Formato | Uso típico | Observações |
| --- | --- | --- | --- |
| `vertices.json` | JSON | Auditoria e integração | Vértices indexados com coordenadas geodésicas e projetadas |
| `mapa.png` | PNG 4000×4000 px | Peça técnica, relatório, apresentação | Grade métrica UTM, escala e identificação de vértices |
| `mapa_3d.png` | PNG | Visualização volumétrica | Bloco isométrico com sombreamento difuso |
| `mapa.svg` | SVG | Publicação web, impressão vetorial | Geometria escalável sem perda |
| `mapa.geojson` | GeoJSON (OGC) | QGIS, ArcGIS, Google Earth, web maps | Feição fechada, CRS geográfico |
| `mapa.dxf` | DXF R12 | AutoCAD, Civil 3D, QGIS | Polilinha em escala 1:1 métrica (UTM) |
| `log.txt` | Texto | Memorial técnico e conferência | Área (ha e m²), perímetro e conferência de fechamento |

**Formatos previstos na evolução da skill** (ainda não disponíveis na implementação de referência): **KML/KMZ** para Google Earth, **Shapefile** (`.shp/.shx/.dbf/.prj`) para fluxo ESRI e **visualização interativa em mapa web**. A inclusão deve preservar o mesmo contrato de validação das saídas atuais.

---

## 10. Conformidade e boas práticas

1. **Referencial.** Toda saída projetada usa SIRGAS2000 (GRS80). Ao receber documento em SAD69 ou Córrego Alegre, a transformação deve ser feita com parâmetros oficiais antes da importação — a skill não converte datum implicitamente.
2. **Fuso.** Confira o fuso informado no relatório contra o município do imóvel. Imóveis em divisa de fuso exigem decisão explícita do responsável técnico.
3. **Fechamento.** Erro de fechamento acima da tolerância da classe indica inconsistência no documento, não falha do software. O polígono não deve ser usado para fins legais antes da elucidação.
4. **Área.** Para imóveis rurais, a área a declarar é a **geodésica**, calculada sobre o elipsoide. Área plana UTM é aceitável apenas como referência de projeto.
5. **Rastreabilidade.** Preserve o `log.txt` junto ao PDF de origem: ele é a prova de qual documento gerou qual geometria.
6. **Conferência amostral.** Recomenda-se conferir manualmente ao menos três vértices por documento, preferencialmente os de menor e de maior coordenada, contra o texto original.

---

## 11. Segurança da informação e LGPD

Memoriais descritivos contêm dados de propriedade, confrontantes e, muitas vezes, dados pessoais. O projeto adota três salvaguardas:

- **Processamento local.** Não há envio de arquivos, coordenadas ou metadados para servidores externos.
- **Bloqueio de PDFs no versionamento.** O `.gitignore` de referência bloqueia `*.pdf` com exceção explícita do arquivo público de teste (`!exemplo_sigef.pdf`), impedindo commit acidental de documentos confidenciais.
- **Separação de diretórios.** Entradas em `inputs/`, artefatos intermediários em `.work/` e entregáveis em `outputs/`, evitando mistura entre dado sensível e resultado publicável.

**Recomendações operacionais:** manter os PDFs de clientes fora de pastas sincronizadas com nuvem pessoal; usar unidade criptografada quando o volume de dados exigir; e revisar o conteúdo de qualquer captura de tela antes de publicar em redes ou documentação.

---

## 12. Limitações e critérios de validação

| Limitação | Impacto | Mitigação |
| --- | --- | --- |
| PDF rasterizado com baixa resolução | OCR pode confundir dígitos (por exemplo, `5` e `6`, `1` e `7`) | Reprocessar com 300 dpi ou mais; conferência manual obrigatória |
| Layout não tabular (texto corrido, colunas) | Reconstrução de linha pode agrupar fragmentos distintos | Inspeção do `vertices.json` e conferência de amostra |
| Vértices com o mesmo identificador repetido | Ambiguidade na topologia | A validação sinaliza duplicidade e interrompe o fechamento |
| Caminhamento com azimute e distância apenas | Propagação de erro acumulada | Comparar erro de fechamento com a tolerância da classe |
| Documento sem altitude | Perda da componente vertical no 3D | Bloco 3D gerado com base plana (z = 0) |
| Divisa de fuso UTM | Distorção crescente fora do fuso | Escolher explicitamente o fuso do imóvel |

**Critérios mínimos antes de usar a geometria em peça técnica:** (i) anel fechado; (ii) ausência de auto-interseção; (iii) erro de fechamento dentro da tolerância declarada; (iv) conferência amostral de vértices contra o documento; (v) identificação do commit/versão da ferramenta utilizada.

---

## 13. Roteiro de evolução

| Prioridade | Item | Situação |
| --- | --- | --- |
| Alta | `SKILL.md` com contrato formal de acionamento e exemplos de entrada/saída | Planejado |
| Alta | Exportadores KML/KMZ e Shapefile | Planejado |
| Alta | Validação de auto-interseção com relatório de segmentos conflitantes | Planejado |
| Média | Leitura dedicada de cadernetas de caminhamento (azimute/rumo/distância) | Planejado |
| Média | Suporte a múltiplos fusos no mesmo lote, com alerta | Planejado |
| Média | Visualização interativa em mapa web (Leaflet) | Planejado |
| Baixa | Integração com QGIS via Processing Toolbox | Em avaliação |
| Baixa | Comparação automática entre memorial e base SIGEF (vértices certificados) | Em avaliação |

---

## 14. Como contribuir

1. Abra uma *issue* descrevendo o caso (tipo de PDF, padrão do memorial, resultado esperado) **sem anexar documentos confidenciais**.
2. Para código: crie um ramo a partir de `erison.barros`, mantenha o padrão ESM e adicione um caso de teste com PDF sintético.
3. Descreva no *pull request*: o que mudou, qual norma ou padrão motivou a mudança e como validar.
4. Alterações no núcleo algorítmico devem indicar o commit correspondente do upstream, quando aplicável.

**Versionamento.** O projeto segue versionamento semântico (`MAJOR.MINOR.PATCH`). Mudanças que alterem a geometria produzida para a mesma entrada são `MAJOR`, ainda que a interface permaneça igual.

---

## 15. Autoria, créditos e licença

**Skill e documentação.** Prof. Erison Rosa de Oliveira Barros — Universidade Federal de Pernambuco (UFPE), Departamento de Engenharia Cartográfica.

**Implementação de referência.** `pdf-to-maps` v1.0.2, desenvolvido por **Diego Oris** ([D-H-O-R-A](https://github.com/D-H-O-R-A)), licenciado sob MIT. Os fundamentos matemáticos, a estrutura de módulos e os formatos de saída descritos neste documento derivam dessa implementação.

**Licença.** Este projeto é distribuído sob a **Licença MIT**. Consulte o arquivo [`LICENSE`](LICENSE) para o texto integral. O uso comercial é permitido, mantidos os avisos de copyright.

**Como citar**

```text
BARROS, E. R. O. pdf-to-maps-extractor: extração e reconstrução geométrica de
memoriais descritivos em PDF para SIG e CAD. Recife: UFPE, 2026.
Disponível em: https://github.com/ErisonBarros/pdf-to-maps-extractor
```

**Aviso técnico.** A ferramenta apoia o trabalho profissional; não substitui a análise do responsável técnico, a conferência documental nem a Anotação de Responsabilidade Técnica (ART). Resultados devem ser verificados antes de qualquer uso legal, registral ou contratual.
