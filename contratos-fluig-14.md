---
name: contratos-fluig
description: "Converter arquivos PDF (formulários, contratos, termos de RH) em HTML compatível com o ambiente Fluig TOTVS, pronto para colar no editor de formulários de contratos da plataforma. Suporta variáveis de dois ERPs: RM e Protheus. Sempre usar esta skill quando o usuário mencionar Fluig, contrato TOTVS, HTML para Fluig, variáveis RM ou variáveis Protheus."
---

# 🧠 Skill: PDF → HTML Fluig TOTVS

> Skill consolidada e validada em produção real — conversão fiel de estrutura, imagens base64, variáveis RM/Protheus automáticas, abordagem híbrida CSS validada para preview e PDF de assinatura.

---

> ## 🚨 REGRA DE BLOQUEIO DE ENTREGA
>
> **Antes de finalizar qualquer resposta com HTML gerado por esta skill, verificar obrigatoriamente:**
>
> - Todo elemento de texto tem `font-size` definido inline?
> - Todo elemento de texto tem `line-height` definido inline?
> - Todo bloco de conteúdo tem `margin-bottom` definido inline?
>
> **Se qualquer resposta for NÃO → NÃO FINALIZAR. Corrigir primeiro.**
> Todos os exemplos e trechos HTML desta skill seguem esse padrão sem exceção.

---

## 🔀 Seleção de ERP — Obrigatória no Início de Cada Conversão

**Antes de mapear qualquer variável, perguntar ao usuário:**

> "Este contrato usa variáveis de qual ERP — **RM** ou **Protheus**?"

- Se **RM** → usar exclusivamente [🔧 Variáveis RM](#-variáveis-rm--mapeamento-completo)
- Se **Protheus** → usar exclusivamente [🔷 Variáveis Protheus](#-variáveis-protheus--mapeamento-completo)
- **Nunca misturar** variáveis dos dois ERPs no mesmo documento

> Se o usuário não souber qual ERP usa: contratos do módulo **RM Labore** usam variáveis RM; contratos do **SIGAADV / SIGAFOPAG (Protheus)** usam variáveis Protheus.

---

## 📋 Sumário

1. [🔄 Pipeline de Execução](#-pipeline-de-execução)
2. [🏗️ Estrutura Obrigatória do HTML](#️-estrutura-obrigatória-do-html)
3. [🆕 Fluig 2.0 — Entregar Apenas o Corpo](#-fluig-20--entregar-apenas-o-corpo-sem-bloco-de-estilo-separado)
4. [🚨 Regras Absolutas](#-regras-absolutas)
5. [👁️ Análise Visual Obrigatória do PDF](#️-análise-visual-obrigatória-do-pdf)
6. [🔤 Tipografia, Line-Height e Margin (Obrigatórios)](#-tipografia-line-height-e-margin-obrigatórios)
7. [🚫 Tabela vs. Texto Corrido — Regra de Decisão](#-tabela-vs-texto-corrido--regra-de-decisão)
8. [📝 Campos de Texto Corrido (sem tabela)](#-campos-de-texto-corrido-sem-tabela)
9. [📊 Conteúdo Dentro de `<td>`](#-conteúdo-dentro-de-td)
10. [🖼️ Inclusão de Imagens (Logo / Rodapé)](#️-inclusão-de-imagens-logo--rodapé)
11. [📷 Coluna Vertical com rowspan (FOTO)](#-coluna-vertical-com-rowspan-foto)
12. [📏 Dimensionamento por Tipo de Dado](#-dimensionamento-por-tipo-de-dado)
13. [🗓️ Datas por Extenso](#️-datas-por-extenso)
14. [🪜 Quando Separar em Tabelas Distintas](#-quando-separar-em-tabelas-distintas)
15. [🔧 Variáveis RM — Mapeamento Completo](#-variáveis-rm--mapeamento-completo)
16. [🔷 Variáveis Protheus — Mapeamento Completo](#-variáveis-protheus--mapeamento-completo)
17. [✅ Checklist Final](#-checklist-final)
18. [🚨 Erros Comuns e Soluções](#-erros-comuns-e-soluções)

---

## 🔄 Pipeline de Execução

> **Seguir esta ordem em toda conversão. Não pular etapas.**

| Etapa | O que fazer | Quando concluir |
|---|---|---|
| **1. Análise visual** | Executar a [Análise Visual Obrigatória](#️-análise-visual-obrigatória-do-pdf) completa — alinhamentos, espaçamentos, quebras de linha, hierarquia, densidade | Antes de escrever qualquer linha de HTML |
| **2. Extração da estrutura** | Percorrer o PDF linha por linha e classificar cada elemento (tabela real, texto corrido, imagem, título, assinatura), aplicando a [Regra de Decisão](#-tabela-vs-texto-corrido--regra-de-decisão) para cada bloco candidato a `<table>` | Antes de montar o esqueleto HTML |
| **3. Identificação dos componentes** | Mapear cada campo para o componente HTML correto: `<div>` + `<span>` (texto corrido), `<table>` (grade real), `<img>` (logo/rodapé), alíneas, parágrafos, cláusulas, assinaturas | Antes de começar a codificar |
| **4. Reprodução do HTML** | Montar o HTML com o [esqueleto e style padrão](#️-estrutura-obrigatória-do-html), aplicando a abordagem híbrida (classe `f-` + `style=""` inline) e os três obrigatórios (`font-size`+`line-height`+`margin-bottom`) em todo elemento | Quando toda a estrutura visual estiver codificada |
| **5. Aplicação das variáveis** | Confirmar o ERP com o usuário e substituir cada campo por `{{variavel}}` usando [Variáveis RM](#-variáveis-rm--mapeamento-completo) ou [Variáveis Protheus](#-variáveis-protheus--mapeamento-completo). Campos sem variável mantêm `<span border-bottom>` manual | Quando todo campo variável estiver mapeado |
| **6. Validação final** | Executar o [Checklist Final](#-checklist-final) + a [Regra de Bloqueio de Entrega](#-regra-de-bloqueio-de-entrega). Revisar cada `<table>` contra a Regra de Decisão. Perguntar se o ambiente é **Fluig 2.0** — se for, entregar **somente o corpo** conforme a seção [🆕 Fluig 2.0](#-fluig-20--entregar-apenas-o-corpo-sem-bloco-de-estilo-separado) | Antes de entregar a resposta |

> ⚠️ A resposta só é válida quando todas as 6 etapas forem concluídas na ordem. Entregar HTML sem passar pela etapa 6 é proibido.

---

## 🏗️ Estrutura Obrigatória do HTML

### Contexto do Fluig — dois motores, um só HTML

| Contexto | Como renderiza | O que usa |
|---|---|---|
| **Preview** (criação do contrato) | iframe isolado sem `<head>` | Apenas `style=""` inline nos elementos |
| **PDF de assinatura** (geração final) | Motor PDF completo com `<head>` | Classes CSS do `<style>` no `<head>` |

**Solução validada em produção: abordagem híbrida.** Cada elemento recebe **simultaneamente** a classe CSS (para o PDF) e o `style=""` inline (para o preview). Os dois coexistem sem conflito, e nenhum dos dois pode faltar.

### Esqueleto padrão que o Fluig já entrega ao criar um contrato

```html
<!DOCTYPE html>
<html>
<head>
  <title></title>
  <link href="/style-guide/css/animalia-icons.min.css" rel="stylesheet" type="text/css">
  <link href="/style-guide/css/flat-icons.min.css" rel="stylesheet" type="text/css">
  <link href="/style-guide/css/fluig-style-guide-flat.min.css" rel="stylesheet" type="text/css">
  <style type="text/css">
    body{padding: 10px !important; word-wrap: break-word !important;}
  </style>
  <style type="text/css">
    body { padding: 0px !important; }
  </style>
  <style type="text/css">
    hr.wordpagebreak { border: none; border-top: 2px dashed #a6a6a6 !important; margin: 10px 0; page-break-before: always; }
  </style>
  <style type="text/css">
    @page { @bottom-right { content: "Página " counter(page) " de " counter(pages); } }
  </style>
  <style type="text/css">
    @page { size: A4 portrait; }
  </style>
</head>
<body class="fluig-style-guide"></body>
</html>
```

**Nunca remover ou alterar esse esqueleto.** Nossa contribuição é sempre:
1. Adicionar **um único `<style>` extra** com nossas classes — depois dos `<style>` do Fluig
2. Preencher o `<body>` com o conteúdo do formulário

> No **Fluig 2.0** esse esqueleto já vem preenchido de outra forma (sem os 5 `<style>` acima) — ver a seção [🆕 Fluig 2.0](#-fluig-20--entregar-apenas-o-corpo-sem-bloco-de-estilo-separado) antes de entregar qualquer conversão nesse ambiente.

### Nosso `<style>` — bloco único de classes `f-`

Adicionar este bloco completo, sem alterações, logo após os `<style>` do Fluig (esqueleto genérico) ou dentro do `<head>` já preenchido:

```html
<style type="text/css">
  body { font-family: Arial, Helvetica, sans-serif; font-size: 12px; color: #000; background: #fff; }

  /* Imagens */
  .f-img-header { width: 100%; display: block; margin-bottom: 0; }
  .f-img-footer  { width: 100%; display: block; margin-top: 20px; }

  /* Tipografia e blocos */
  .f-titulo       { text-align: center; font-size: 15px; font-weight: bold; line-height: 1.2; margin: 18px 0 16px 0; }
  .f-linha        { margin-bottom: 6px; text-align: justify; font-size: 12px; line-height: 1.2; }
  .f-linha-opcao  { margin-bottom: 4px; font-size: 12px; line-height: 1.2; }
  .f-paragrafo    { margin-bottom: 12px; text-align: justify; font-size: 12px; line-height: 1.4; }
  .f-paragrafo-sm { margin-bottom: 10px; text-align: justify; font-size: 12px; line-height: 1.4; }
  .f-alinea       { margin-bottom: 6px;  padding-left: 32px; text-align: justify; font-size: 12px; line-height: 1.4; }
  .f-alinea-last  { margin-bottom: 14px; padding-left: 32px; text-align: justify; font-size: 12px; line-height: 1.4; }
  .f-item         { margin-bottom: 6px;  padding-left: 24px; text-align: justify; font-size: 12px; line-height: 1.4; }
  .f-item-last    { margin-bottom: 14px; padding-left: 24px; text-align: justify; font-size: 12px; line-height: 1.4; }

  /* Campos especiais */
  .f-ul-vte   { display: inline-block; width: 140px; border-bottom: 1px solid #000; margin-left: 4px; }
  .f-ul-vales { display: inline-block; width: 90px;  border-bottom: 1px solid #000; margin: 0 4px; }

  /* Tabelas */
  .f-tabela    { width: 100%; border-collapse: collapse; border: 1px solid #000; table-layout: fixed; margin-bottom: 8px; }
  .f-tabela td { border: 1px solid #000; padding: 4px; vertical-align: middle; }
  .f-th { font-size: 11px; font-weight: bold; line-height: 1.1; word-wrap: break-word; overflow-wrap: break-word; white-space: normal; width: 100%; display: block; }
  .f-td { font-size: 11px; line-height: 1.2; word-wrap: break-word; overflow-wrap: break-word; white-space: normal; width: 100%; display: block; }

  /* Rodapé do formulário */
  .f-nota            { font-size: 11px; line-height: 1.2; margin-bottom: 12px; }
  .f-firmo           { margin-bottom: 20px; font-size: 12px; line-height: 1.2; }
  .f-data            { margin-bottom: 40px; font-size: 12px; line-height: 1.2; font-weight: bold; }
  .f-assinatura-wrap { text-align: center; margin-bottom: 4px; }
  .f-assinatura-linha { display: inline-block; width: 60%; border-top: 1px solid #000; padding-top: 6px; font-size: 12px; line-height: 1.2; }

  /* Cláusulas (contratos) */
  .f-clausula        { font-size: 12px; font-weight: bold; line-height: 1.2; margin-bottom: 6px; }
  .f-subcláusula     { margin-bottom: 10px; text-align: justify; font-size: 12px; line-height: 1.4; }
  .f-paragrafo-unico { margin-bottom: 14px; text-align: justify; font-size: 12px; line-height: 1.4; padding-left: 16px; }
</style>
```

### Template base completo — copiar para cada novo contrato (fora do Fluig 2.0)

```html
<!DOCTYPE html>
<html>
<head>
  <title></title>
  <link href="/style-guide/css/animalia-icons.min.css" rel="stylesheet" type="text/css">
  <link href="/style-guide/css/flat-icons.min.css" rel="stylesheet" type="text/css">
  <link href="/style-guide/css/fluig-style-guide-flat.min.css" rel="stylesheet" type="text/css">
  <style type="text/css">
    body{padding: 10px !important; word-wrap: break-word !important;}
  </style>
  <style type="text/css">
    body { padding: 0px !important; }
  </style>
  <style type="text/css">
    hr.wordpagebreak { border: none; border-top: 2px dashed #a6a6a6 !important; margin: 10px 0; page-break-before: always; }
  </style>
  <style type="text/css">
    @page { @bottom-right { content: "Página " counter(page) " de " counter(pages); } }
  </style>
  <style type="text/css">
    @page { size: A4 portrait; }
  </style>

  <!-- ✅ NOSSO STYLE (bloco acima) — adicionado após os do Fluig -->

</head>
<body class="fluig-style-guide">

  <!-- CONTEÚDO DO FORMULÁRIO AQUI -->

</body>
</html>
```

### Regra híbrida — como aplicar nos elementos

**Cada elemento deve ter SEMPRE os dois: classe + inline.** A classe serve o PDF de assinatura; o inline serve o preview.

```html
<!-- ✅ CORRETO — híbrido -->
<div class="f-titulo"
     style="text-align:center; font-size:15px; line-height:1.2; font-weight:bold; margin-bottom:16px; font-family:Arial,Helvetica,sans-serif;">
  TÍTULO DO FORMULÁRIO
</div>

<table class="f-tabela"
       width="100%" border="1" cellpadding="4" cellspacing="0"
       style="border-collapse:collapse; border:1px solid #000; table-layout:fixed; margin-bottom:8px;">
  <tr>
    <td style="width:45%; border:1px solid #000; padding:4px; text-align:center; vertical-align:middle;">
      <div class="f-th" style="font-size:11px; line-height:1.1; font-weight:bold; display:block; width:100%; font-family:Arial,Helvetica,sans-serif;">CABEÇALHO</div>
    </td>
  </tr>
</table>

<!-- ❌ ERRADO — só classe (preview quebra) -->
<div class="f-titulo">TÍTULO</div>

<!-- ❌ ERRADO — só inline (PDF de assinatura pode quebrar) -->
<div style="text-align:center; font-size:15px;">TÍTULO</div>
```

### Converter imagem para base64

```bash
base64 /caminho/da/imagem.png | tr -d '\n'
```

```html
<!-- Logo — primeiro elemento do body -->
<img src="data:image/png;base64,BASE64_AQUI"
     class="f-img-header"
     style="width:100%; display:block; margin-bottom:0;"
     alt="Logo da empresa" />

<!-- Rodapé — último elemento do body -->
<img src="data:image/png;base64,BASE64_AQUI"
     class="f-img-footer"
     style="width:100%; display:block; margin-top:20px;"
     alt="Rodapé" />
```

---

## 🆕 Fluig 2.0 — Entregar Apenas o Corpo (Sem Bloco de Estilo Separado)

> **Causa raiz confirmada em produção:** no editor de formulários do **Fluig 2.0**, o esqueleto já existe quando o contrato é criado, e o próprio Fluig **já preenche automaticamente o início** (`<head>` com os `<link>` padrão da plataforma). Colar o **documento completo** (com `<!DOCTYPE>`, `<html>`, `<head>` e `<body>` próprios) por cima do esqueleto existente cria tags **duplicadas/aninhadas**. O editor falha ao salvar essa estrutura inválida — o formulário **não salva** e, por consequência, **não gera preview para download**.

> Esse problema é específico do editor do Fluig 2.0 e não altera nenhuma outra regra desta skill — a estrutura de HTML gerada (tabelas, `<div>`, variáveis, abordagem híbrida) continua exatamente a mesma. Muda apenas **o que é entregue e colado**.

### Esqueleto confirmado do Fluig 2.0

> Diferente do esqueleto genérico da seção anterior (5 blocos `<style>`), o **Fluig 2.0 preenche automaticamente apenas isto** — sem nenhum `<style>`:

```html
<!DOCTYPE html>
<html>
<head>
  <title></title>
  <link href="/style-guide/css/animalia-icons.min.css" rel="stylesheet" type="text/css" />
  <link href="/style-guide/css/flat-icons.min.css" rel="stylesheet" type="text/css" />
  <link href="/style-guide/css/fluig-style-guide-flat.min.css" rel="stylesheet" type="text/css" />
</head>
<body class="fluig-style-guide">
</body>
</html>
```

### ✅ Regra obrigatória — entregar somente o corpo

**Nunca instruir o usuário a colar o `<!DOCTYPE>`/`<html>`/`<head>`/`<body>` completos, nem um bloco `<style>` separado, por cima do esqueleto que o Fluig 2.0 já criou.** A entrega deve ser **um único bloco**:

| Conteúdo | Onde cola no Fluig 2.0 |
|---|---|
| Apenas o conteúdo que vai **dentro** do `<body>` (tabelas, títulos, campos, assinatura, imagens) — sem `<body>`/`</body>`, sem `<head>`, sem `<style>` extra | Dentro do `<body class="fluig-style-guide">` que o Fluig 2.0 já criou |

Como a abordagem híbrida já exige `font-size`+`line-height`+`margin-bottom` inline em **todo** elemento, o corpo entregue já é autossuficiente visualmente — não depende de nenhum `<style>` externo para renderizar corretamente no preview nem no PDF de assinatura.

### 🚫 Nunca fazer no Fluig 2.0
- Colar `<!DOCTYPE html>`, uma segunda tag `<html>`/`<head>`/`<body>` — já existem no editor
- Repetir os 3 `<link>` do Fluig — já estão lá
- Entregar um bloco `<style>` separado para colar no `<head>` — o Fluig 2.0 já preenche essa parte e o corpo já é 100% inline

### ✅ Sempre fazer no Fluig 2.0
- Gerar o HTML normalmente seguindo todo o restante desta skill (pipeline, tabelas, variáveis, abordagem híbrida)
- Antes de entregar, **extrair apenas o conteúdo interno do `<body>`** como um único bloco/arquivo
- Informar explicitamente: "cole este conteúdo dentro do `<body>` que o Fluig já criou"
- Se o Fluig usado **não for 2.0** (versão que aceita substituição total do código-fonte), o documento completo (esqueleto padrão + nosso `<style>` + corpo, tudo junto) continua sendo uma entrega válida — entregar só o corpo é regra específica do Fluig 2.0

---

## 🚨 Regras Absolutas

### HTML e CSS

| ❌ Nunca | ✅ Sempre |
|---|---|
| Remover o esqueleto do Fluig | Manter os `<style>`/`<link>` originais do Fluig intactos (fora do Fluig 2.0) |
| Adicionar `<style>` antes dos do Fluig | Nosso `<style>` sempre **após** os do Fluig |
| `display:flex` ou `display:grid` | Layout sempre via `<table>` |
| Frameworks externos (Bootstrap, Tailwind) | Sem dependências externas |
| `value="..."` em inputs | Inputs sempre vazios |
| Classes sem prefixo `f-` | Prefixo `f-` em todas as classes para evitar conflito com o CSS do Fluig |
| Elemento só com classe ou só com inline | **Sempre híbrido**: classe + `style=""` inline juntos |

### Tabelas

| ❌ Nunca | ✅ Sempre | Motivo |
|---|---|---|
| `cellpadding="0"` | `cellpadding="4"` | `0` colapsa linhas a altura zero |
| `height` fixo em `<div>` de valor | `&nbsp;` + padding | Height colapsa no Fluig |
| `overflow:hidden` direto no `<td>` | Sempre via `<div>` filho | `<td>` não respeita `overflow` |
| `text-overflow:ellipsis` | `word-wrap:break-word; white-space:normal;` | Ellipsis corta conteúdo |
| Tabela interna dentro de `<td>` | `<td>` reais na linha principal | Tabelas internas vazam |

### Fonte e bordas

- `font-family:Arial, Helvetica, sans-serif` declarado no `body` via CSS de classe **e** inline em cada elemento — o Fluig tem CSS global que sobrescreve herança
- `border:1px solid #000` sempre preto sólido, nunca `#999`/`#ccc`
- `border-collapse:collapse` em todas as tabelas, declarado no `<table>` **e** em cada `<td>`

---

## 👁️ Análise Visual Obrigatória do PDF

> **Este modelo possui capacidade multimodal.** A interpretação do PDF deve ser baseada **principalmente na análise visual da página**, não apenas no texto extraído. Esta etapa é obrigatória e deve ser concluída **ANTES** de escrever qualquer HTML.

### Pré-análise — Composição Visual (10 verificações)

| # | Verificar | O que procurar |
|---|---|---|
| 1 | **Página inteira** | Composição geral, proporções, margens |
| 2 | **Hierarquia visual** | Títulos maiores → subtítulos → corpo → labels → notas |
| 3 | **Agrupamentos** | Blocos de campos que pertencem ao mesmo grupo lógico |
| 4 | **Alinhamentos** | `left` / `center` / `right` / `justify` de cada bloco |
| 5 | **Espaçamentos** | Distância entre blocos — compacto, padrão, seção, assinatura |
| 6 | **Linhas** | Quebras de linha intencionais, separadores, underlines de campo |
| 7 | **Bordas** | Linhas verticais e horizontais formando grades reais |
| 8 | **Tabelas** | Grades com bordas visíveis (ambos critérios da Regra de Decisão) |
| 9 | **Caixas** | Retângulos, áreas delimitadas, blocos com borda externa |
| 10 | **Imagens** | Logos, rodapés, fotos, ícones |

> ⚠️ Somente após compreender completamente a composição visual, iniciar a geração do HTML.

### Detalhamento por dimensão visual

| Dimensão | Regra |
|---|---|
| **Alinhamentos** | Nunca assumir `justify` como padrão universal — verificar bloco a bloco (títulos: left/center/right? parágrafos: justify ou left? valores em célula: left/center/right?) |
| **Espaçamentos** | Blocos muito próximos → `margin-bottom:2-4px`; padrão entre parágrafos → `8-12px`; entre seções → `16-24px`; antes de assinatura → `32-48px`. Nunca uniformizar |
| **Linhas e quebras** | Preservar toda quebra intencional (`<br>` ou `margin-bottom` maior); separação entre grupos de campos vira `<div>` separado, nunca `<br>` |
| **Hierarquia visual** | Título principal: `15-16px bold center`; subtítulo/cláusula: `12px bold`; corpo: `12px normal`; label de célula: `9px #555`; nota: `11px`. Nunca nivelar |
| **Densidade** | Região densa (campos lado a lado): `padding:2px`, `margin-bottom:2px`, `10-11px`; região normal: `padding:4px`, `margin-bottom:6px`, `12px`; região espaçada (cláusulas): `margin-bottom:10-14px`, `12px` |

### Regra de Fidelidade Visual

> **O HTML deve replicar a aparência do PDF — não apenas a estrutura.**

| ❌ Proibido | ✅ Obrigatório |
|---|---|
| Reorganizar campos para ficarem "mais limpos" | Manter a ordem e posição exata do PDF |
| Simplificar uma tabela com muitas colunas | Reproduzir todas as colunas com os `width%` corretos |
| Trocar alinhamento para "padronizar" | Usar o alinhamento exato que o PDF mostra |
| Unir parágrafos/linhas próximos em um só | Manter blocos `<div>` separados se o PDF os separa |
| Reduzir `margin-bottom` para "economizar espaço" | Preservar o espaçamento visual do PDF |
| Omitir quebra de linha por parecer redundante | Incluir toda quebra de linha que existe no PDF |
| Otimizar o layout para parecer "mais moderno" | O objetivo é cópia fiel, não melhoria estética |
| Ajustar font-size para parecer mais legível | Usar o tamanho proporcional ao que o PDF apresenta |

---

## 🔤 Tipografia, Line-Height e Margin (Obrigatórios)

> **`font-size` + `line-height` + `margin-bottom` são obrigatórios em TODO elemento de texto, sem exceção**, sempre com base na análise visual do PDF — nunca por padrão ou conveniência. Ausência de qualquer um implica HTML inválido.

| Propriedade | Guia de valores | Regra crítica |
|---|---|---|
| **`font-size`** | Notas/legendas: `10-11px` · Corpo/parágrafos/campos: `11-12px` · Destaque/subtítulos: `13-14px` · Título principal: `14-16px` | Nunca herdar do `body`; nunca padronizar tudo em `12px` — variar conforme a hierarquia do PDF |
| **`line-height`** | Texto denso (labels, campos compactos): `1.1` · Texto padrão: `1.2` · Parágrafos/cláusulas corridas: `1.4` | Nunca omitir — sem ele o navegador usa `~1.6` e o documento "estica", perdendo fidelidade |
| **`margin-bottom`** | Elementos muito compactos: `2-4px` · Espaçamento padrão: `6-12px` · Entre seções: `12-16px` · Antes de assinatura: `24-40px` | Nunca omitir nem usar um valor único genérico para tudo — variar pela percepção visual do PDF |

```html
<!-- ✅ CORRETO — os três obrigatórios sempre juntos, valores variando por elemento -->
<div class="f-titulo"
     style="font-size:15px; line-height:1.2; margin-bottom:16px; font-weight:bold; text-align:center; font-family:Arial,Helvetica,sans-serif;">
  TÍTULO
</div>

<div class="f-nota"
     style="font-size:10px; line-height:1.2; margin-bottom:12px; font-family:Arial,Helvetica,sans-serif;">
  Nota de rodapé menor
</div>

<div class="f-data"
     style="font-size:12px; line-height:1.2; margin-bottom:40px; font-family:Arial,Helvetica,sans-serif;">
  Local e data...
</div>

<!-- ❌ ERRADO — falta font-size (herda do body) -->
<div style="margin-bottom:6px; text-align:justify;">Texto</div>

<!-- ❌ ERRADO — tudo no mesmo tamanho/espaçamento, sem variação -->
<div style="font-size:12px; margin-bottom:6px;">Nota</div>
<div style="font-size:12px; margin-bottom:6px;">Título</div>
<div style="font-size:12px; margin-bottom:6px;">Cláusula</div>
```

---

## 🚫 Tabela vs. Texto Corrido — Regra de Decisão

> **Nem todo conteúdo alinhado visualmente no PDF é uma tabela.** Colunas, campos lado a lado ou alinhamento visual **não significam** que existe uma grade real — este é um dos erros mais frequentes na conversão.

### Mapeamento de elementos

| O PDF tem... | Converter para... |
|---|---|
| Grade com bordas visíveis (linhas E colunas) | `<table>` com `<tr>` e `<td>` |
| Texto seguido de linha `___` | `<div>` + `<span style="border-bottom:1px solid #000">` |
| Parágrafo de texto normal | `<div>` parágrafo |
| Título centralizado | `<div style="text-align:center">` |
| Lista a), b), 1., 2. | `<div>` com `padding-left` |
| Linha de assinatura | `<span>` com `border-top` ou `border-bottom` |
| Logo/imagem | `<img>` com `width:100%` |
| Campos lado a lado sem borda / labels+valores em texto corrido / blocos verticais sem grade / borda externa decorativa sem divisórias internas | `<div>` + `<span>` inline (nunca `<table>`) |

### 🧠 Regra de Decisão — as duas perguntas obrigatórias

**Executar este fluxo antes de escrever qualquer estrutura HTML:**

```
1. É tabela real — tem grade visível com bordas?
   ├── NÃO → usar <div> + <span>. Parar aqui.
   └── SIM → continuar
        2. Existe divisão clara de colunas E linhas com bordas separando o conteúdo?
           ├── NÃO → usar <div> + <span>. Parar aqui.
           └── SIM → usar <table>
```

> ⚠️ As duas condições precisam ser SIM para justificar `<table>`. **Na dúvida, sempre `<div>`** — usar `<table>` quando não devia causa dano à fidelidade visual; usar `<div>` quando poderia ser `<table>` é inofensivo e seguro.

### Por que isso importa

Tabela usada onde não deveria causa: espaçamento artificial (o `<td>` adiciona padding/altura mínima não controlada), desalinhamento entre preview e PDF de assinatura (os dois motores calculam largura de coluna de forma diferente) e documento mais longo que o PDF original. **Esse é um erro estrutural — não tem correção por CSS** (ajustar `padding`/`margin`/`width` não resolve); a única correção válida é substituir `<table>` por `<div>`.

### Exemplos

```html
<!-- ✅ CORRETO — campos lado a lado sem grade → texto corrido -->
<div class="f-linha"
     style="font-family:Arial,Helvetica,sans-serif; font-size:12px; line-height:1.2; margin-bottom:6px;">
  Nome: <strong>{{pf_nome}}</strong>&nbsp;&nbsp;
  CPF: <strong>{{pf_cpf}}</strong>
</div>

<!-- ✅ CORRETO — label + valor sem borda → texto corrido com underline -->
<div class="f-linha"
     style="font-family:Arial,Helvetica,sans-serif; font-size:12px; line-height:1.2; margin-bottom:6px;">
  Cargo: <span style="display:inline-block; width:40%; border-bottom:1px solid #000; margin:0 4px;">&nbsp;</span>
  Setor: <span style="display:inline-block; width:30%; border-bottom:1px solid #000; margin:0 4px;">&nbsp;</span>
</div>

<!-- ✅ CORRETO — bloco vertical de dados sem grade → divs sequenciais -->
<div class="f-linha" style="font-family:Arial,Helvetica,sans-serif; font-size:12px; line-height:1.2; margin-bottom:4px;">
  <strong>Data Nascimento:</strong> {{pf_dtnascimento}}
</div>
```

```html
<!-- ❌ ERRADO — criou tabela para campos sem bordas → erro estrutural, refazer -->
<table>
  <tr>
    <td>Nome:</td>
    <td>{{pf_nome}}</td>
    <td>CPF:</td>
    <td>{{pf_cpf}}</td>
  </tr>
</table>
```

### Validação final obrigatória — antes de entregar

1. Listar todas as `<table>` criadas no documento
2. Para cada uma, confirmar as duas condições da Regra de Decisão (linhas E colunas com borda visível?)
3. Se qualquer `<table>` não atender aos dois critérios → refatorar para `<div>` antes de entregar

---

## 📝 Campos de Texto Corrido (sem tabela)

### Larguras de referência para campos inline

| Campo | Width |
|---|---|
| Nome completo (linha inteira) | `85%` |
| Nome (dividindo linha) | `55-65%` |
| RG | `18%` |
| CPF | `18%` |
| Matrícula | `15%` |
| Emitido pelo(a) | `22%` |
| Cargo | `30%` |
| Setor | `35%` |
| CEP | `15%` |
| Cidade | `30%` |
| Rua (linha inteira) | `85%` |
| Quantidade numérica | `80-100px` fixo |

### Alíneas a), b) / Itens 1. 2.

```html
<div class="f-alinea"
     style="margin-bottom:6px; padding-left:32px; text-align:justify; font-size:12px; line-height:1.4; font-family:Arial,Helvetica,sans-serif;">
  a)&nbsp;&nbsp;Texto da alínea.
</div>

<div class="f-item"
     style="margin-bottom:6px; padding-left:24px; text-align:justify; font-size:12px; line-height:1.4; font-family:Arial,Helvetica,sans-serif;">
  1.&nbsp;&nbsp;Texto do item.
</div>
```

### Linha de assinatura

> **Padrão de fechamento completo** — quando o PDF contiver o bloco "Por ser verdade, firmo o presente. / Cidade(UF), data / Assinatura / Nome", reproduzir **todas** as partes abaixo na ordem exata.

```html
<!-- "Por ser verdade, firmo o presente." -->
<div class="f-firmo"
     style="margin-bottom:20px; font-size:12px; line-height:1.2; font-family:Arial,Helvetica,sans-serif;">
  Por ser verdade, firmo o presente.
</div>

<!-- Local e data manual (sem variáveis) -->
<div class="f-data"
     style="margin-bottom:40px; font-size:12px; line-height:1.2; font-family:Arial,Helvetica,sans-serif;">
  <span style="display:inline-block; width:130px; border-bottom:1px solid #000;">&nbsp;</span>
  (BA),
  <span style="display:inline-block; width:35px; border-bottom:1px solid #000;">&nbsp;</span>
  /
  <span style="display:inline-block; width:60px; border-bottom:1px solid #000;">&nbsp;</span>
  de
  <span style="display:inline-block; width:50px; border-bottom:1px solid #000;">&nbsp;</span>.
</div>

<!-- Local e data via variável RM (preferível) -->
<div class="f-data"
     style="margin-bottom:40px; font-size:12px; line-height:1.2; font-weight:bold; font-family:Arial,Helvetica,sans-serif;">
  {{dgf_pf_cidade_desc}}/({{pf_estado}}), {{dgf_dt_admissao_extenso}}
</div>

<!-- Assinatura centralizada -->
<div class="f-assinatura-wrap" style="text-align:center; margin-bottom:4px;">
  <span class="f-assinatura-linha"
        style="display:inline-block; width:60%; border-top:1px solid #000; padding-top:6px; font-size:12px; line-height:1.2; font-family:Arial,Helvetica,sans-serif;">
    Assinatura do(a) colaborador(a)
  </span>
</div>

<!-- Nome do candidato / funcionário abaixo da assinatura -->
<div class="f-assinatura-wrap" style="text-align:center; margin-bottom:4px;">
  <span style="font-size:12px; line-height:1.2; font-family:Arial,Helvetica,sans-serif;">
    {{pf_nome}}
  </span>
</div>
```

---

## 📊 Conteúdo Dentro de `<td>`

> Em tabelas do Fluig, o `<td>` **NÃO garante herança consistente de estilo** — o comportamento varia entre o preview (iframe limitado) e o PDF de assinatura (motor próprio). `font-size`, `line-height`, `text-align` e espaçamento interno definidos só no `<td>` podem ser ignorados por um dos dois motores.

### ✅ Regra obrigatória — estilo sempre no `<div>` filho, nunca no `<td>`

Todo texto em célula deve estar encapsulado em `<div>` com estilo inline completo (`font-family`, `font-size`, `line-height`, `text-align`, `margin-bottom` quando há mais de um `<div>`), **nunca** texto direto no `<td>`.

**Fórmula de contenção obrigatória** em todo `<div>` de texto dentro de tabela:
```css
word-wrap:break-word; overflow-wrap:break-word; white-space:normal; width:100%; display:block;
```

```html
<!-- ✅ CORRETO — célula simples (valor único) -->
<td style="border:1px solid #000; padding:4px; vertical-align:middle;">
  <div style="font-family:Arial,Helvetica,sans-serif; font-size:11px; line-height:1.2; text-align:left; margin-bottom:0; word-wrap:break-word; overflow-wrap:break-word; white-space:normal; width:100%; display:block;">
    {{pf_nome}}
  </div>
</td>

<!-- ✅ CORRETO — célula com label (9px/1.1) + valor (11px/1.2) -->
<td style="border:1px solid #000; padding:4px; vertical-align:top;">
  <div style="font-family:Arial,Helvetica,sans-serif; font-size:9px; line-height:1.1; text-align:left; color:#555; margin-bottom:2px; word-wrap:break-word; overflow-wrap:break-word; white-space:normal; width:100%; display:block;">NOME COMPLETO</div>
  <div style="font-family:Arial,Helvetica,sans-serif; font-size:11px; line-height:1.2; text-align:left; color:#000; margin-bottom:0; word-wrap:break-word; overflow-wrap:break-word; white-space:normal; width:100%; display:block;">{{pf_nome}}</div>
</td>

<!-- ✅ CORRETO — célula centralizada (ex: cabeçalho de coluna) -->
<td style="border:1px solid #000; padding:4px; vertical-align:middle;">
  <div style="font-family:Arial,Helvetica,sans-serif; font-size:11px; line-height:1.1; font-weight:bold; text-align:center; margin-bottom:0; word-wrap:break-word; overflow-wrap:break-word; white-space:normal; width:100%; display:block;">CABEÇALHO</div>
</td>

<!-- ❌ ERRADO — estilo no <td>, ou texto direto sem <div> encapsulando -->
<td style="border:1px solid #000; padding:4px; font-size:11px; line-height:1.2;">
  Conteúdo da célula
</td>
```

---

## 🖼️ Inclusão de Imagens (Logo / Rodapé)

Converter para base64 (comando na seção [🏗️ Estrutura Obrigatória](#️-estrutura-obrigatória-do-html)) e inserir como `<img>` híbrido: logo como primeiro elemento do `<body>` com classe `f-img-header`, rodapé como último elemento com classe `f-img-footer` — sempre com `display:block` (evita espaço branco abaixo da imagem) e `data:image/...;base64,` como `src` (nunca caminho de arquivo local).

---

## 📷 Coluna Vertical com rowspan (FOTO)

Apenas quando a tabela existe realmente no PDF.

```html
<table class="f-tabela" width="100%" border="1" cellpadding="4" cellspacing="0"
       style="border-collapse:collapse; border:1px solid #000; table-layout:fixed;">
  <tr>
    <td rowspan="8"
        style="width:22%; border:1px solid #000; vertical-align:middle; text-align:center; padding:6px; background:#f5f5f5;">
      <div style="font-size:11px; line-height:1.2; margin-bottom:0; font-family:Arial,Helvetica,sans-serif;">&nbsp;</div>
    </td>
    <td style="width:38%; border:1px solid #000; vertical-align:top; padding:4px;">
      <div style="font-size:9px; line-height:1.1; color:#555; margin-bottom:2px; word-wrap:break-word; overflow-wrap:break-word; white-space:normal; width:100%; display:block; font-family:Arial,Helvetica,sans-serif;">LABEL</div>
      <div style="font-size:11px; line-height:1.2; margin-bottom:0; word-wrap:break-word; overflow-wrap:break-word; white-space:normal; width:100%; display:block; font-family:Arial,Helvetica,sans-serif;">&nbsp;</div>
    </td>
    <td style="width:40%; border:1px solid #000; vertical-align:top; padding:4px;">
      <div style="font-size:9px; line-height:1.1; color:#555; margin-bottom:2px; word-wrap:break-word; overflow-wrap:break-word; white-space:normal; width:100%; display:block; font-family:Arial,Helvetica,sans-serif;">LABEL</div>
      <div style="font-size:11px; line-height:1.2; margin-bottom:0; word-wrap:break-word; overflow-wrap:break-word; white-space:normal; width:100%; display:block; font-family:Arial,Helvetica,sans-serif;">&nbsp;</div>
    </td>
  </tr>
  <!-- LINHAS 2-N: sem repetir o <td> da foto -->
</table>
```

---

## 📏 Dimensionamento por Tipo de Dado

| Categoria | Largura | Exemplos |
|---|---|---|
| **Muito curto** (1-4 chars) | **6-8%** | UF, Zona, Sexo |
| **Curto fixo** (5-12 chars) | **10-12%** | Nº, Órgão emissor |
| **Médio-fixo** (13-18 chars) | **14-18%** | CPF, CNPJ, RG, Salário |
| **Datas por extenso** | **20-22%** | Data nascimento, admissão |
| **Longo** | **25-35%** | Local nascimento, Horário |
| **Linha inteira** | **100%** | Endereço, Rua |

**Proporção longo + curto: sempre 75/25, nunca 50/50.**

---

## 🗓️ Datas por Extenso

Reservar no mínimo **20%** para campos de data. Usar `{{dgf_dt_admissao_extenso}}` quando disponível — retorna o formato completo, eliminando campos separados de dia/mês/ano.

---

## 🪜 Quando Separar em Tabelas Distintas

- **SEPARAR:** blocos com grades de coluna diferentes ou lógica de divisão incompatível
- **MANTER:** seções com a mesma grade lógica que fluem naturalmente

---

## 🔧 Variáveis RM — Mapeamento Completo

> Ao converter qualquer contrato, identificar cada campo variável e substituir pela variável RM no formato `{{variavel}}`. Campos **sem** variável disponível mantêm `<span border-bottom>` para preenchimento manual.

### 🏢 Dados da Empresa (Matriz)

| Campo | Variável RM |
|---|---|
| Razão Social | `{{dgf_pf_cl_frmnome}}` |
| Nome Fantasia | `{{dgf_pf_cl_nomefantasia}}` |
| CNPJ | `{{dgf_pf_cl_cnpj}}` |
| Código e Descrição | `{{dgf_pf_empresa_com_desc_cod}}` |
| Endereço | `{{dgf_pf_cl_rua}}` |
| Número | `{{dgf_pf_cl_numero}}` |
| Complemento | ❌ Não possui variável |
| Bairro | `{{dgf_pf_cl_bairro}}` |
| Cidade | `{{dgf_pf_cl_cidade}}` |
| Estado | `{{dgf_pf_cl_estado}}` |
| País | ❌ Não possui variável |
| CEP | `{{dgf_pf_cl_cep}}` |
| Emissão da Ficha | `{{dgf_dt_admissao}}` |

### 🏥 Dados da Filial

| Campo | Variável RM |
|---|---|
| Razão Social | `{{dgf_pf_fl_frmnome}}` |
| Nome Fantasia | `{{dgf_pf_fl_nomefantasia}}` |
| CNPJ | `{{dgf_pf_fl_cnpj}}` |
| Código e Descrição | `{{dgf_pf_filial_com_desc_cod}}` |
| Endereço | `{{dgf_pf_fl_rua}}` |
| Número | `{{dgf_pf_fl_numero}}` |
| Complemento | `{{dgf_pf_fl_complemento}}` |
| Bairro | `{{dgf_pf_fl_bairro}}` |
| Cidade | `{{dgf_pf_fl_cidade}}` |
| Estado | `{{dgf_pf_fl_estado}}` |
| País | `{{dgf_pf_fl_pais}}` |
| CEP | `{{dgf_pf_fl_cep}}` |

### 👤 Dados do Candidato / Empregado

| Campo | Variável RM |
|---|---|
| Nome | `{{pf_nome}}` |
| Matrícula / Chapa | `{{pf_chapa}}` |
| Sexo | `{{pf_sexo}}` |
| Data de Nascimento | `{{pf_dtnascimento}}` |
| Estado Civil (código) | `{{pf_estadocivil}}` |
| Estado Civil (descrição) | `{{dgf_pf_estadocivil_desc}}` |
| Grau de Instrução (código) | `{{pf_grauinstrucao}}` |
| Grau de Instrução (descrição) | `{{dgf_pf_grauinstrucao_desc}}` |
| Nacionalidade (código) | `{{pf_nacionalidade}}` |
| Nacionalidade (descrição) | `{{dgf_pf_nacionalidade_desc}}` |
| Naturalidade (código) | `{{pf_naturalidade}}` |
| Naturalidade (descrição) | `{{dgf_pf_naturalidade_desc}}` |
| Estado Natal | `{{pf_estadonatal}}` |
| Nº de Filhos | `{{pf_nfilhos}}` |
| Nome da Mãe | `{{dgf_pd_mnome}}` |
| Nome do Pai | `{{dgf_pd_pnome}}` |
| E-mail | `{{dgf_email_contato}}` |
| Celular | `{{dgf_celular_contato}}` |

### 📄 Documentos do Candidato / Empregado

| Campo | Variável RM |
|---|---|
| CPF | `{{pf_cpf}}` |
| RG / Identidade | `{{pf_cartidentidade}}` |
| Órgão Emissor RG (código) | `{{pf_orgemissorident}}` |
| Órgão Emissor RG (descrição) | `{{dgf_pf_orgemissident_desc}}` |
| Emissão RG | `{{pf_dtemissaoident}}` |
| Carteira de Trabalho | `{{pf_carteiratrab}}` |
| Série CTPS | `{{pf_seriecarttrab}}` |
| Emissão CTPS | `{{pf_dtcarttrab}}` ou `{{pf_dtexpcarttrab}}` |
| PIS/PASEP | `{{pf_pispasep}}` |
| CNH | `{{pf_cartmotorista}}` |
| Emissão CNH | `{{pf_dtemissaocnh}}` |
| Validade CNH | `{{pf_dtvenchabilit}}` |
| UF CNH | `{{pf_ufcnh}}` |
| Categoria CNH | `{{pf_tipocarthabilit}}` |
| Título de Eleitor | `{{pf_tituloeleitor}}` |
| Título de Eleitor Zona | `{{pf_zonatiteleitor}}` |
| Título de Eleitor Seção | `{{pf_secaotiteleitor}}` |
| Cert. Reservista | `{{pf_certifreserv}}` |
| Categoria Militar | `{{pf_categmilitar}}` |
| Registro Profissional | `{{pp_numeroregistro}}` |

### 📍 Endereço do Candidato / Empregado

| Campo | Variável RM |
|---|---|
| Tipo do Logradouro | `{{dgf_pf_codtiporuad_desc}}` |
| Nome do Logradouro / Rua | `{{pf_rua}}` |
| Número | `{{pf_numero}}` |
| Complemento | `{{pf_complemento}}` |
| Bairro | `{{pf_bairro}}` |
| Cidade | `{{dgf_pf_cidade_desc}}` |
| UF / Estado | `{{pf_estado}}` |
| CEP | `{{pf_cep}}` |

### 👨‍👩‍👧 Dados de Dependentes

> Incrementar o sufixo numérico conforme a ordem (`_1`, `_2`, `_3`...).

#### Dependentes Gerais

| Campo | Variável RM |
|---|---|
| Nome | `{{pd_nome_1}}` |
| Data de Nascimento | `{{pd_dtnascimento_1}}` |
| Sexo | `{{pd_sexo_1}}` |
| Parentesco | `{{dgf_pd_grauparentesco_desc_1}}` |
| CPF | `{{dgf_depend_cpf_1}}` |
| Nome da Mãe | `{{pd_mnome_1}}` |
| RG | `{{pd_cartidentidade_1}}` |
| Estado Civil | `{{dgf_pd_estadocivil_desc_1}}` |

#### Dependentes para IR

| Campo | Variável RM |
|---|---|
| Nome | `{{pd_nome_ir_1}}` |
| Data de Nascimento | `{{pd_dtnascimento_ir_1}}` |
| Sexo | `{{pd_sexo_ir_1}}` |
| Parentesco | `{{dgf_pd_grauparentesco_desc_ir_1}}` |
| CPF | `{{dgf_depend_cpf_ir_1}}` |
| Nome da Mãe | `{{pd_mnome_ir_1}}` |
| RG | `{{pd_cartidentidade_ir_1}}` |
| Estado Civil | `{{dgf_pd_estadocivil_desc_ir_1}}` |

#### Dependentes para Salário Família

| Campo | Variável RM |
|---|---|
| Nome | `{{pd_nome_sf_1}}` |
| Data de Nascimento | `{{pd_dtnascimento_sf_1}}` |
| Sexo | `{{pd_sexo_sf_1}}` |
| Parentesco | `{{dgf_pd_grauparentesco_desc_sf_1}}` |
| CPF | `{{dgf_depend_cpf_sf_1}}` |
| Nome da Mãe | `{{pd_mnome_sf_1}}` |
| RG | `{{pd_cartidentidade_sf_1}}` |
| Estado Civil | `{{dgf_pd_estadocivil_desc_sf_1}}` |

#### Dependentes para Plano de Saúde

| Campo | Variável RM |
|---|---|
| Nome | `{{pd_nome_b1_1}}` |
| Data de Nascimento | `{{pd_dtnascimento_b1_1}}` |
| Sexo | `{{pd_sexo_b1_1}}` |
| Parentesco | `{{dgf_pd_grauparentesco_desc_b1_1}}` |
| CPF | `{{dgf_depend_cpf_b1_1}}` |
| Nome da Mãe | `{{pd_mnome_b1_1}}` |
| RG | `{{pd_cartidentidade_b1_1}}` |
| Estado Civil | `{{dgf_pd_estadocivil_desc_b1_1}}` |

#### Dependentes para Plano Odontológico

| Campo | Variável RM |
|---|---|
| Nome | `{{pd_nome_b2_1}}` |
| Data de Nascimento | `{{pd_dtnascimento_b2_1}}` |
| Sexo | `{{pd_sexo_b2_1}}` |
| Parentesco | `{{dgf_pd_grauparentesco_desc_b2_1}}` |
| CPF | `{{dgf_depend_cpf_b2_1}}` |
| Nome da Mãe | `{{pd_mnome_b2_1}}` |
| RG | `{{pd_cartidentidade_b2_1}}` |
| Estado Civil | `{{dgf_pd_estadocivil_desc_b2_1}}` |

### 💼 Dados de Contratação

| Campo | Variável RM |
|---|---|
| Salário (valor numérico) | `{{pf_salario}}` |
| Salário Hora | `{{pf_hora}}` |
| Salário por Extenso | `{{dgf_vl_salario_extenso}}` |
| Data de Admissão | `{{pf_dataadmissao}}` |
| Data de Admissão por Extenso | `{{dgf_dt_admissao_extenso}}` |
| Opção FGTS | `{{pf_dtopcaofgts}}` |
| Forma de Pagamento (descrição) | `{{dgf_pf_codformapagto_desc}}` |
| Centro de Custo (descrição) | `{{dgf_pf_codccusto_desc}}` |
| Centro de Custo (código e descrição) | `{{dgf_pf_codccusto_desc_cod}}` |
| Função / Cargo (descrição) | `{{dgf_pf_codfuncao_desc}}` |
| Função / Cargo (código e descrição) | `{{dgf_pf_codfuncao_desc_cod}}` |
| Seção (descrição) | `{{dgf_pf_codsecao_desc}}` |
| Seção (código e descrição) | `{{dgf_pf_codsecao_desc_cod}}` |
| Jornada (descrição e código) | `{{dgf_jornada_desc_cod}}` |
| Turno de Trabalho | `{{dgf_pf_codhorario_desc}}` |
| Vencimento da 1ª Experiência | `{{dgf_pf_experiencia_inicio}}` |
| Dias de Vencimento da 1ª Exp. | `{{dpf_dias_venc_exp1}}` |
| Vencimento da 2ª Experiência | `{{dgf_pf_experiencia_fim}}` |
| Dias de Vencimento da 2ª Exp. | `{{dpf_dias_venc_exp2}}` |

### 🌍 Dados de Estrangeiro

> Usar apenas quando o empregado for estrangeiro. Campos sem variável mantêm `<span border-bottom>` manual.

| Campo | Variável RM |
|---|---|
| Data de Chegada | `{{pf_datachegada}}` |
| Cônjuge Brasileiro(s) | `{{pf_conjugebrasil}}` |
| Nº Cart. Identidade Estrangeiro | `{{pf_nroreggeral}}` |
| Tipo de Visto | `{{pf_codclassiftrabestrang}}` |
| Nº Registro Geral | `{{pf_nroreggeral}}` |
| Nº Decreto | ❌ Não possui variável |
| Naturalizado | ❌ Não possui variável |
| Validade Cart. Identidade | `{{pf_dtvencident}}` |
| Validade Cart. Trabalho | ❌ Não possui variável |

### 🚌 Vale Transporte

> Incrementar o sufixo numérico conforme a linha (`_1`, `_2`, `_3`...).
> Sufixo `_1` = trajeto **para a empresa**; sufixo `_2` = trajeto **para a residência**.

#### Para a Empresa (sufixo `_1`)

| Campo | Variável RM |
|---|---|
| Destino | `{{dgf_pv_destino___1}}` |
| Tipo | `{{dgf_pv_tipo___1}}` |
| Código da Linha | `{{dgf_pv_codlinha___1}}` |
| Nome da Linha | `{{dgf_pv_nomelinha___1}}` |
| Empresa de Transporte | `{{dgf_pv_empresesatrans___1}}` |
| Número do Cartão | `{{dgf_pv_numcartao___1}}` |
| Valor do Cartão | `{{dgf_pv_valor_tarifa_cartao___1}}` |
| Valor | `{{dgf_pv_valor___1}}` |

#### Para a Residência (sufixo `_2`)

| Campo | Variável RM |
|---|---|
| Destino | `{{dgf_pv_destino___2}}` |
| Tipo | `{{dgf_pv_tipo___2}}` |
| Código da Linha | `{{dgf_pv_codlinha___2}}` |
| Nome da Linha | `{{dgf_pv_nomelinha___2}}` |
| Empresa de Transporte | `{{dgf_pv_empresesatrans___2}}` |
| Número do Cartão | `{{dgf_pv_numcartao___2}}` |
| Valor do Cartão | `{{dgf_pv_valor_tarifa_cartao___2}}` |
| Valor | `{{dgf_pv_valor___2}}` |

### ⚡ Mapeamento Rápido — Campos mais usados em contratos

| Campo | Variável RM |
|---|---|
| Nome do empregado | `{{pf_nome}}` |
| Matrícula / Chapa | `{{pf_chapa}}` |
| CPF do empregado | `{{pf_cpf}}` |
| RG do empregado | `{{pf_cartidentidade}}` |
| Órgão Emissor RG | `{{pf_orgemissorident}}` ou `{{dgf_pf_orgemissident_desc}}` |
| Endereço (rua) do empregado | `{{pf_rua}}` |
| CNPJ da empresa | `{{dgf_pf_cl_cnpj}}` |
| CNPJ da filial | `{{dgf_pf_fl_cnpj}}` |
| Cargo / Função | `{{dgf_pf_codfuncao_desc}}` |
| Seção | `{{dgf_pf_codsecao_desc}}` |
| Salário | `{{pf_salario}}` |
| Salário Hora | `{{pf_hora}}` |
| Salário por extenso | `{{dgf_vl_salario_extenso}}` |
| Data de admissão | `{{pf_dataadmissao}}` |
| Data de admissão por extenso | `{{dgf_dt_admissao_extenso}}` |
| Opção FGTS | `{{pf_dtopcaofgts}}` |
| Forma de Pagamento | `{{dgf_pf_codformapagto_desc}}` |
| Jornada | `{{dgf_jornada_desc_cod}}` |
| Turno de Trabalho | `{{dgf_pf_codhorario_desc}}` |
| Vencimento 1ª experiência (45 dias) | `{{dgf_pf_experiencia_inicio}}` |
| Vencimento 2ª experiência (90 dias) | `{{dgf_pf_experiencia_fim}}` |

---

## 🔷 Variáveis Protheus — Mapeamento Completo

> Usar **somente** quando o cliente utiliza Protheus (SIGAADV/SIGAFOPAG). Formato: `{{variavel}}`.
> Campos marcados com ❌ não possuem variável disponível — manter `<span style="border-bottom:1px solid #000">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span>`.

### 🏢 Dados da Empresa (Matriz)

| Campo | Variável Protheus |
|---|---|
| Razão Social | `{{dgf_pf_cl_frmnome}}` |
| Nome Fantasia | `{{dgf_pf_cl_nomefantasia}}` |
| CNPJ | `{{dgf_pf_cl_cnpj}}` |
| Código e Descrição | `{{dgf_pf_empresa_com_desc_cod}}` |
| Endereço | `{{dgf_pf_cl_rua}}` |
| Número | `{{dgf_pf_cl_numero}}` |
| Complemento | ❌ Não possui variável |
| Bairro | `{{dgf_pf_cl_bairro}}` |
| Cidade | `{{dgf_pf_cl_cidade}}` |
| Estado | `{{dgf_pf_cl_estado}}` |
| País | ❌ Não possui variável |
| CEP | `{{dgf_pf_cl_cep}}` |

### 🏥 Dados da Filial

| Campo | Variável Protheus |
|---|---|
| Razão Social | `{{dgf_pf_fl_frmnome}}` |
| Nome Fantasia | `{{dgf_pf_fl_nomefantasia}}` |
| CNPJ | `{{dgf_pf_fl_cnpj}}` |
| Código e Descrição | `{{dgf_pf_filial_com_desc_cod}}` |
| Endereço | `{{dgf_pf_fl_rua}}` |
| Número | `{{dgf_pf_fl_numero}}` |
| Complemento | `{{dgf_pf_fl_complemento}}` |
| Bairro | `{{dgf_pf_fl_bairro}}` |
| Cidade | `{{dgf_pf_fl_cidade}}` |
| Estado | `{{dgf_pf_fl_estado}}` |
| País | `{{dgf_pf_fl_pais}}` |
| CEP | `{{dgf_pf_fl_cep}}` |

### 👤 Dados do Candidato / Empregado

| Campo | Variável Protheus |
|---|---|
| Nome | `{{pf_ra_nome}}` |
| Sexo | `{{pf_ra_sexo}}` |
| Data de Nascimento | `{{pf_ra_nasc}}` |
| Estado Civil (código) | `{{pf_ra_estcivi}}` |
| Estado Civil (descrição) | `{{dgf_pf_estadocivil_desc}}` |
| Grau de Instrução (código) | `{{pf_grauinstrucao}}` |
| Grau de Instrução (descrição) | `{{dgf_pf_grauinstrucao_desc}}` |
| Nacionalidade (código) | `{{pf_ra_naciona}}` |
| Nacionalidade (descrição) | `{{dgf_pf_nacionalidade_desc}}` |
| Naturalidade | `{{dgf_pf_naturalidade_desc}}` |
| Estado Natal | `{{pf_ra_estado}}` |
| Nome da Mãe | `{{pf_ra_mae}}` |
| Nome do Pai | `{{pf_ra_pai}}` |
| E-mail | `{{dgf_email_contato}}` |
| Celular | `{{dgf_celular_contato}}` |

### 📄 Documentos do Candidato / Empregado

| Campo | Variável Protheus |
|---|---|
| CPF | `{{pf_ra_cic}}` |
| RG | `{{pf_ra_rg}}` |
| Órgão Emissor RG | `{{dgf_pf_orgemissident_desc}}` |
| Carteira de Trabalho | `{{pf_ra_numcp}}` |
| Série CTPS | `{{pf_ra_sercp}}` |
| Emissão CTPS | `{{pf_ra_dtcpexp}}` |
| PIS/PASEP | `{{pf_ra_pis}}` |
| CNH | `{{pf_ra_habilit}}` |
| Emissão CNH | `{{pf_ra_dtemcnh}}` |
| Validade CNH | `{{pf_ra_dtvccnh}}` |
| UF CNH | `{{pf_ra_ufcnh}}` |
| Categoria CNH | `{{pf_ra_catcnh}}` |
| Título de Eleitor | `{{pf_ra_tituloe}}` |
| Título de Eleitor Zona | `{{pf_ra_zonasec}}` |
| Título de Eleitor Seção | `{{pf_ra_secao}}` |
| Reservista | `{{pf_ra_reservi}}` |

### 📍 Endereço do Candidato / Empregado

| Campo | Variável Protheus |
|---|---|
| Tipo do Logradouro | `{{dgf_pf_codtiporuad_desc}}` |
| Nome do Logradouro / Rua | `{{pf_ra_logrdsc}}` |
| Número | `{{pf_ra_logrnum}}` |
| Complemento | `{{pf_ra_complem}}` |
| Bairro | `{{pf_ra_bairro}}` |
| Cidade | `{{dgf_pf_cidade_desc}}` |
| UF / Estado | `{{pf_ra_estado}}` |
| CEP | `{{pf_ra_cep}}` |

### 👨‍👩‍👧 Dados de Dependentes

> Incrementar o sufixo numérico conforme a ordem (`_1`, `_2`, `_3`...).

#### Dependentes Gerais

| Campo | Variável Protheus |
|---|---|
| Nome | `{{pd_nome_1}}` |
| Data de Nascimento | `{{pd_dtnascimento_1}}` |
| Sexo | `{{pd_sexo_1}}` |
| Parentesco | `{{dgf_pd_grauparentesco_desc_1}}` |
| CPF | `{{dgf_depend_cpf_1}}` |
| Nome da Mãe | `{{pd_mnome_1}}` |
| RG | `{{pd_cartidentidade_1}}` |
| Estado Civil | `{{dgf_pd_estadocivil_desc_1}}` |

#### Dependentes para IR

| Campo | Variável Protheus |
|---|---|
| Nome | `{{pd_rb_nome_ir_1}}` |
| Data de Nascimento | `{{pd_rb_dtnasc_ir_1}}` |
| Sexo | `{{pd_sexo_ir_1}}` |
| Parentesco | `{{dgf_pd_grauparentesco_desc_ir_1}}` |
| CPF | `{{dgf_depend_cpf_ir_1}}` |
| Nome da Mãe | `{{pd_mnome_ir_1}}` |
| RG | `{{pd_cartidentidade_ir_1}}` |
| Estado Civil | `{{dgf_pd_estadocivil_desc_ir_1}}` |

#### Dependentes para Salário Família

| Campo | Variável Protheus |
|---|---|
| Nome | `{{pd_rb_nome_sf_1}}` |
| Data de Nascimento | `{{pd_rb_dtnasc_sf_1}}` |
| Sexo | `{{pd_sexo_sf_1}}` |
| Parentesco | `{{dgf_pd_grauparentesco_desc_sf_1}}` |
| CPF | `{{dgf_depend_cpf_sf_1}}` |
| Nome da Mãe | `{{pd_mnome_sf_1}}` |
| RG | `{{pd_cartidentidade_sf_1}}` |
| Estado Civil | `{{dgf_pd_estadocivil_desc_sf_1}}` |

#### Dependentes para Plano de Saúde

| Campo | Variável Protheus |
|---|---|
| Nome | `{{pd_nome_b1_1}}` |
| Data de Nascimento | `{{pd_dtnascimento_b1_1}}` |
| Sexo | `{{pd_sexo_b1_1}}` |
| Parentesco | `{{dgf_pd_grauparentesco_desc_b1_1}}` |
| CPF | `{{dgf_depend_cpf_b1_1}}` |
| Nome da Mãe | `{{pd_mnome_b1_1}}` |
| RG | `{{pd_cartidentidade_b1_1}}` |
| Estado Civil | `{{dgf_pd_estadocivil_desc_b1_1}}` |

#### Dependentes para Plano Odontológico

| Campo | Variável Protheus |
|---|---|
| Nome | `{{pd_nome_b2_1}}` |
| Data de Nascimento | `{{pd_dtnascimento_b2_1}}` |
| Sexo | `{{pd_sexo_b2_1}}` |
| Parentesco | `{{dgf_pd_grauparentesco_desc_b2_1}}` |
| CPF | `{{dgf_depend_cpf_b2_1}}` |
| Nome da Mãe | `{{pd_mnome_b2_1}}` |
| RG | `{{pd_cartidentidade_b2_1}}` |
| Estado Civil | `{{dgf_pd_estadocivil_desc_b2_1}}` |

### 💼 Dados de Contratação

| Campo | Variável Protheus |
|---|---|
| Salário (valor numérico) | `{{pf_ra_salario}}` |
| Salário por Extenso | `{{dgf_vl_salario_extenso}}` |
| Data de Admissão | `{{pf_ra_admissa}}` |
| Data de Admissão por Extenso | `{{dgf_dt_admissao_extenso}}` |
| Centro de Custo (descrição) | `{{dgf_pf_codccusto_desc}}` |
| Centro de Custo (código e descrição) | `{{dgf_pf_codccusto_desc_cod}}` |
| Função (descrição) | `{{dgf_pf_codfuncao_desc}}` |
| Função (código e descrição) | `{{dgf_pf_codfuncao_desc_cod}}` |
| Vencimento da 1ª Experiência | `{{dgf_pf_experiencia_inicio}}` |
| Dias de Vencimento da 1ª Exp. | `{{dpf_dias_venc_exp1}}` |
| Vencimento da 2ª Experiência | `{{dgf_pf_experiencia_fim}}` |
| Dias de Vencimento da 2ª Exp. | `{{dpf_dias_venc_exp2}}` |
| Seção | `{{dgf_pf_codsecao_desc_cod}}` |
| Turno de Trabalho | `{{dgf_pf_codhorario_desc}}` |

### ⚡ Mapeamento Rápido — Campos mais usados (Protheus)

| Campo | Variável Protheus |
|---|---|
| Nome do empregado | `{{pf_ra_nome}}` |
| CPF do empregado | `{{pf_ra_cic}}` |
| RG do empregado | `{{pf_ra_rg}}` |
| Endereço (rua) do empregado | `{{pf_ra_logrdsc}}` |
| CNPJ da filial | `{{dgf_pf_fl_cnpj}}` |
| Cargo / Função | `{{dgf_pf_codfuncao_desc}}` |
| Salário | `{{pf_ra_salario}}` |
| Salário por extenso | `{{dgf_vl_salario_extenso}}` |
| Data de admissão | `{{pf_ra_admissa}}` |
| Data de admissão por extenso | `{{dgf_dt_admissao_extenso}}` |
| Vencimento 1ª experiência (45 dias) | `{{dgf_pf_experiencia_inicio}}` |
| Vencimento 2ª experiência (90 dias) | `{{dgf_pf_experiencia_fim}}` |
| Turno / Jornada de trabalho | `{{dgf_pf_codhorario_desc}}` |

---

## ✅ Checklist Final

### Antes de começar — análise visual do PDF (obrigatória)

- [ ] Executei as 10 verificações da Pré-análise (composição visual)
- [ ] Identifiquei o alinhamento real de cada bloco (left / center / right / justify)
- [ ] Medi o espaçamento visual entre cada seção e converti para `margin-bottom` proporcional
- [ ] Localizei todas as quebras de linha intencionais (distintas do wrap natural de texto)
- [ ] Mapeei a hierarquia visual: título principal → subtítulos → corpo → labels → notas
- [ ] Avaliei a densidade de cada região (densa / normal / espaçada)
- [ ] Determinei o `font-size`, `line-height` e `margin-bottom` de cada bloco com base no PDF

### Antes de começar — análise estrutural do PDF

- [ ] Li o PDF linha por linha e classifiquei cada elemento (tabela real vs. texto corrido)
- [ ] Para cada bloco que parece tabela: executei a **Ordem de Decisão** (existem linhas verticais E horizontais com borda? divisão clara de colunas?)
- [ ] Para cada bloco com qualquer dúvida na identificação: optei por `<div>` (fallback seguro)
- [ ] Para cada bloco sem **ambos** os critérios confirmados: usei `<div>` + `<span>` em vez de `<table>`
- [ ] Nenhuma `<table>` foi criada apenas por alinhamento visual ou campos lado a lado
- [ ] Identifiquei quais linhas são campos inline com underline
- [ ] Identifiquei quais blocos são tabelas reais com bordas visíveis
- [ ] Identifiquei imagens a incluir (logo, rodapé) e converti para base64
- [ ] Identifiquei todos os campos variáveis e mapeei para as variáveis RM correspondentes

### Validação final — todas as `<table>` do documento

- [ ] Listei todas as `<table>` criadas no documento
- [ ] Cada `<table>` tem linhas verticais E horizontais formando células com borda visível (critério 1)
- [ ] Cada `<table>` tem divisão clara de colunas com bordas separando o conteúdo (critério 2)
- [ ] Nenhuma `<table>` passou apenas um dos dois critérios
- [ ] Qualquer `<table>` com dúvida foi refatorada para `<div>` antes desta entrega

### Estrutura HTML

- [ ] Esqueleto do Fluig (5 `<style>` + 3 `<link>`) mantido intacto
- [ ] Nosso `<style>` adicionado **após** os do Fluig
- [ ] `<body class="fluig-style-guide">` mantido
- [ ] Sem `display:flex`, `display:grid` para layout
- [ ] Sem `value=""` nos inputs

### Abordagem híbrida

- [ ] **Todo elemento tem classe `f-` E `style=""` inline simultaneamente**
- [ ] Prefixo `f-` em todas as classes para evitar conflito com CSS do Fluig
- [ ] Imagens com classe `f-img-header` / `f-img-footer` + `style=""` inline

### Fidelidade visual e estrutural

- [ ] HTML e PDF parecem o mesmo documento quando abertos lado a lado
- [ ] Cada bloco tem o alinhamento correto (não assumido — verificado no PDF)
- [ ] Espaçamentos entre seções são proporcionalmente iguais ao PDF
- [ ] Todas as quebras de linha intencionais estão presentes
- [ ] Hierarquia visual (tamanhos, pesos) foi preservada integralmente
- [ ] Densidade das regiões (compacta / normal / espaçada) foi respeitada
- [ ] Nenhum campo foi reorganizado, simplificado ou "otimizado"
- [ ] Nenhuma tabela criada onde o PDF não tem tabela
- [ ] Campos de texto corrido usam `<div>` + `<span style="border-bottom:1px solid #000">`
- [ ] Parágrafos de texto são `<div>` simples
- [ ] Alíneas e itens com `padding-left` correto
- [ ] Títulos centralizados usam `text-align:center`

### Variáveis (RM ou Protheus)

- [ ] ERP definido com o usuário antes de mapear qualquer variável (RM ou Protheus)
- [ ] Apenas variáveis do ERP escolhido foram utilizadas — nenhuma mistura
- [ ] Todos os campos com variável disponível usam `{{variavel}}`
- [ ] Campos sem variável mantêm `<span style="border-bottom:1px solid #000">` manual
- [ ] Data por extenso usa `{{dgf_dt_admissao_extenso}}` quando aplicável
- [ ] Dependentes com sufixo numérico correto (`_1`, `_2`...)

### Tabelas (quando existem no PDF)

- [ ] `cellpadding="4"` em todas (nunca `0`)
- [ ] `cellspacing="0"` em todas
- [ ] `table-layout:fixed` em tabelas com múltiplas colunas
- [ ] Soma dos `width%` = exatamente 100%
- [ ] `border:1px solid #000` no `<table>` e em cada `<td>`
- [ ] Todo conteúdo dentro de `<td>` está em `<div>` filho — nunca texto direto no `<td>`
- [ ] O `<div>` filho define `font-family`, `font-size`, `line-height` e `text-align` inline
- [ ] Nenhum estilo tipográfico depende de herança do `<td>` ou do `<table>`

### Padrão de célula (quando em tabela)

- [ ] Fórmula de contenção em todo `<div>` de texto
- [ ] Labels: `font-size:9px; color:#555`
- [ ] Valores: `font-size:11px; color:#000`
- [ ] `&nbsp;` em todos os `<div>` de valor vazios

---

---

## 🚨 Erros Comuns e Soluções

| Sintoma | Causa | Solução |
|---|---|---|
| **Fluig 2.0: formulário não salva / sem preview para download** | Documento completo (`<!DOCTYPE>`/`<html>`/`<head>`/`<body>`/`<style>` extra) colado por cima do esqueleto que o Fluig 2.0 já criou e já preenche automaticamente → tags duplicadas/aninhadas | Entregar **apenas o conteúdo do `<body>`**, sem `<head>` e sem `<style>` separado, para colar dentro do `<body>` que o Fluig 2.0 já criou — ver seção [🆕 Fluig 2.0](#-fluig-20--entregar-apenas-o-corpo-sem-bloco-de-estilo-separado) |
| **Preview bugado** — texto corrido sem formatação | Elemento só com classe, sem `style=""` inline | Aplicar abordagem híbrida: classe + inline em todo elemento |
| **PDF de assinatura sem estilo** | Elemento só com inline, sem classe | Adicionar classe `f-` correspondente em cada elemento |
| **Campo gigantesco** | Transformou texto corrido em tabela | Remover a tabela; usar `<div>` + `<span border-bottom>` |
| **Campo ausente** | Esqueceu campo ao criar células | Reler o PDF linha a linha |
| **Variável RM não aplicada** | Usou underline onde havia variável | Consultar tabela de variáveis RM e substituir |
| Conflito de classes com Fluig | Classe sem prefixo (ex: `.titulo`) | Renomear com prefixo `f-` (ex: `.f-titulo`) |
| Tabela invisível / linhas colapsando | `cellpadding="0"` | Usar `cellpadding="4"` |
| Imagem não aparece | Caminho local de arquivo | Converter para base64 como data URI |
| Imagem com espaço branco abaixo | `display:inline` padrão | Adicionar `display:block` |
| Colunas desalinhadas | Soma de width% ≠ 100% | Recalcular para somar 100% |
| **Tabela criada onde não existe** | Não analisou o PDF antes | Ler o PDF linha por linha antes de escrever HTML |
| **`<table>` criada sem passar pela Ordem de Decisão** | Pulou o fluxo de verificação | Aplicar os dois critérios: linhas verticais E horizontais com borda? → se NÃO: refazer com `<div>` |
| **Labels e valores em `<td>` sem borda visível no PDF** | Confundiu alinhamento visual com grade | Substituir por `<div class="f-linha">` com campos inline — erro estrutural, refazer |
| **Bloco vertical de dados convertido em tabela** | Interpretou lista de campos como grade | Usar `<div>` sequenciais com `margin-bottom` calibrado — erro estrutural, refazer |
| **Documento mais longo que o PDF** | `<table>` usada onde não deveria (altura mínima de `<td>` acumula) **ou** `line-height`/`margin-bottom` excessivos | Identificar `<table>` indevidas pela Ordem de Decisão e substituir por `<div>`; ajustar `line-height`/`margin-bottom` conforme a análise visual do PDF |
| **Desalinhamento entre preview e PDF de assinatura em campos** | `<td>` processa largura de forma diferente nos dois motores | Refazer com `<div>` + `<span>` inline quando não há grade real |
| **`<table>` usada em estrutura ambígua sem certeza** | Não aplicou o fallback de segurança | Na dúvida, sempre `<div>` — `<table>` só quando inequívoco |
| **Validação final não executada** | HTML finalizado sem revisar todas as `<table>` | Revisar cada `<table>` contra a Ordem de Decisão e passar pelo Checklist Final antes de entregar |
| **Correção com CSS em vez de refatoração estrutural** | Tentou ajustar `padding`, `margin` ou `width` para compensar `<table>` indevida | O erro estrutural não tem correção por CSS — única solução é substituir `<table>` por `<div>` |
| **Layout visualmente diferente do PDF** | Espaçamentos padronizados sem observar o PDF | Executar a Análise Visual Obrigatória (seção 3) antes de escrever HTML |
| **Alinhamento incorreto** | Assumiu `justify` para todo texto | Verificar alinhamento real de cada bloco no PDF |
| **Hierarquia visual achatada** | Todos elementos com mesmo `font-size`/`font-weight` | Mapear os 5 níveis hierárquicos e aplicar CSS correspondente |
| **Seções reorganizadas** | "Melhorou" a ordem para parecer mais lógica | Reverter para a ordem exata do PDF sem alterações |
| **Espaçamento uniforme** | Aplicou `margin-bottom` padrão em tudo | Variar o `margin-bottom` conforme a separação visual do PDF |
| **Quebras de linha ausentes** | Omitiu `<br>` por parecer desnecessário | Preservar todas as quebras intencionais do PDF |
| **font-size ausente no inline** | Omitiu `font-size` esperando herança do body | Declarar `font-size` inline em todo elemento de texto |
| **Todo conteúdo em 12px** | Padronizou sem observar a hierarquia do PDF | Variar `font-size` por nível: notas `10-11px`, corpo `11-12px`, destaque `13-14px`, título `14-16px` |
| **Texto "estufado" / mais espaçado que o PDF** | `line-height` não declarado (browser usa ~1.6) | Declarar `line-height:1.2` (padrão) ou `1.4` (parágrafos) ou `1.1` (denso) em todo elemento |
| **margin-bottom ausente** | Omitiu achando que o browser gerencia | Declarar `margin-bottom` inline em todo bloco de conteúdo |
| **Espaçamento uniforme entre todos os blocos** | Aplicou `margin-bottom:6px` em tudo | Variar: compacto `2-4px`, padrão `6-12px`, seções `12-16px`, assinatura `24-40px` |
| **Estilo diferente entre preview e PDF de assinatura em tabelas** | Estilo definido no `<td>` dependendo de herança | Mover todos os estilos tipográficos para o `<div>` filho dentro do `<td>` |
| **Texto direto em `<td>` sem `<div>`** | Conteúdo inserido sem encapsulamento | Sempre encapsular em `<div>` com estilo inline completo |
| **font-size/line-height correto no preview, errado no PDF** | Herança do `<td>` funciona no browser mas não no motor do PDF | Declarar `font-size` e `line-height` no `<div>` filho — não no `<td>` |
| **Alinhamento de célula inconsistente** | `text-align` definido no `<td>`, não herdado pelo motor do PDF | Declarar `text-align` no `<div>` filho dentro do `<td>` |

---
