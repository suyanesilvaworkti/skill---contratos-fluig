
# contratos-fluig

Skill que converte **PDFs de formulários, contratos e termos de RH** em **HTML compatível com o Fluig TOTVS**, pronto para colar no editor de formulários de contratos da plataforma. Já substitui os campos por variáveis do ERP (**RM** ou **Protheus**).

## Quando usar

Use quando a conversa mencionar Fluig, contrato TOTVS, HTML para Fluig, variáveis RM ou variáveis Protheus. Para chamar manualmente, digite `/contratos-fluig`.

## Como usar

1. Chame a skill e envie o PDF do contrato.
2. Informe qual ERP o contrato usa: **RM** ou **Protheus**. A skill pergunta isso no início de cada conversão. Se você não souber, contratos do módulo RM Labore usam variáveis RM, e contratos do SIGAADV/SIGAFOPAG usam variáveis Protheus.
3. Informe se o ambiente é **Fluig 2.0**.
4. Receba o HTML e cole no editor do Fluig.

## O que ela entrega

- Conversão fiel da estrutura do PDF: alinhamentos, espaçamentos, quebras de linha, hierarquia de títulos e densidade do texto.
- Imagens (logo e rodapé) embutidas em base64.
- Campos substituídos por variáveis no formato `{{variavel}}`, por exemplo `{{pf_nome}}` e `{{pf_cpf}}` no RM.
- Campos sem variável disponível mantidos como linha de preenchimento (`<span>` com sublinhado).
- Datas e valores por extenso quando há variável para isso, como `{{dgf_dt_admissao_extenso}}`.

## Pipeline (6 etapas, sempre nesta ordem)

| Etapa | O que acontece |
|---|---|
| 1. Análise visual | Examina o PDF antes de escrever qualquer HTML |
| 2. Extração da estrutura | Classifica cada elemento: tabela real, texto corrido, imagem, título, assinatura |
| 3. Identificação dos componentes | Escolhe o componente HTML de cada campo |
| 4. Reprodução do HTML | Monta o HTML com classe `f-` e `style=""` inline em cada elemento |
| 5. Aplicação das variáveis | Confirma o ERP e mapeia os campos |
| 6. Validação final | Checklist e regra de bloqueio de entrega |

## Regras principais

- **Abordagem híbrida:** o Fluig usa dois motores, um para o preview e outro para o PDF de assinatura. Por isso cada elemento recebe a classe CSS (prefixo `f-`, para o PDF) e o estilo inline (para o preview).
- **Três propriedades inline obrigatórias** em todo elemento de texto: `font-size`, `line-height` e `margin-bottom`. Se faltar alguma, a entrega é bloqueada até corrigir.
- **Tabela só quando há grade real:** `<table>` apenas se o PDF tiver linhas verticais e horizontais com borda visível. Campos lado a lado e blocos de dados em lista viram `<div>` com `<span>` sublinhado. Na dúvida, usa `<div>`.
- **Dentro de `<td>`:** todo conteúdo fica em um `<div>` filho com o estilo completo, nunca texto direto no `<td>`.
- **Tabelas:** `cellpadding="4"`, `cellspacing="0"`, `border-collapse:collapse`, borda `1px solid #000` e soma das larguras igual a 100%.
- **Sem** `display:flex`, `display:grid`, frameworks externos nem `value=""` nos inputs.
- **Nunca misturar** variáveis RM e Protheus no mesmo documento.

## Fluig 2.0: entrega só o corpo

No editor do Fluig 2.0 o esqueleto do HTML já existe. Colar um documento completo por cima cria tags duplicadas, e o formulário não salva nem gera preview. Nesse caso a skill entrega **apenas o conteúdo de dentro do `<body>`**, para colar no `<body class="fluig-style-guide">` que o Fluig já criou. Em versões que aceitam substituir todo o código-fonte, o documento completo continua válido.

## Variáveis

A skill traz o mapeamento completo para os dois ERPs, organizado por grupos: empresa, filial, candidato/empregado, documentos, endereço, dependentes (sufixos `_1`, `_2`...) e contratação. O RM tem ainda estrangeiro e vale transporte. Cada ERP tem também uma tabela de campos mais usados em contratos.

Exemplos do RM:

| Campo | Variável |
|---|---|
| Nome do empregado | `{{pf_nome}}` |
| CPF | `{{pf_cpf}}` |
| Cargo / Função | `{{dgf_pf_codfuncao_desc}}` |
| Salário por extenso | `{{dgf_vl_salario_extenso}}` |
| Data de admissão | `{{pf_dataadmissao}}` |

## Problemas comuns

| Sintoma | Causa provável |
|---|---|
| Formulário não salva no Fluig 2.0 | Documento completo colado por cima do esqueleto |
| Preview sem formatação | Elemento só com classe, sem estilo inline |
| PDF de assinatura sem estilo | Elemento só com estilo inline, sem classe |
| Imagem não aparece | Caminho local em vez de base64 |
| Documento mais longo que o PDF | `<table>` usada onde não devia, ou `line-height` e `margin-bottom` exagerados |

A skill traz uma lista maior de erros, com a solução de cada um.

## Estrutura

```
contratos-fluig/
└── SKILL.md   (instruções completas da skill)
```
