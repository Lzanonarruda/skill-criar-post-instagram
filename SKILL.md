---
description: Gera post único para Instagram — HTML com identidade visual do cliente + exportação PNG 1080×1350px via Playwright.
when_to_use: cria um post, faz uma arte pro Instagram, post de depoimento, post educativo, post de oferta, marco numérico, data comemorativa, prova social com foto
---

# Post Instagram

Gera posts únicos com a identidade visual do cliente aplicada: HTML para aprovação + exportação em PNG 1080×1350px via Playwright.

Skills relacionadas: [[criar-carrossel-instagram]] · [[criar-copy-gmn]] · [[calendário-conteudo]] · [[validar-arte-visual]] · [[criar-legenda-instagram]] · [[classificar-icones]]

Os layouts (tipos A–H), componentes reutilizáveis, mecânica de exportação e validação de dimensão desta skill vêm de `_sistema/referencias/templates-post-instagram.md` (referência genérica, reutilizável por qualquer skill de post). O schema de dados do brand-profile está em `_sistema/referencias/brand-profile-conteudo-visual.md`. Este arquivo documenta só o que é específico desta skill: fluxo de coleta, integração com o banco de imagens, humanização e regras de exportação de post único.

---

## Pré-requisito 0 — Qual cliente

1. Se o usuário nomear o cliente/pasta explicitamente na mensagem → usar essa pasta como `{cliente}`.
2. Senão, procurar as pastas de topo do vault que contêm `brand-profile.md` na raiz:
   - Exatamente 1 encontrada → usar essa, sem perguntar.
   - 0 encontradas → perguntar ao usuário o nome do cliente/pasta (esperado: `{pasta}/brand-profile.md`).
   - 2+ encontradas → perguntar qual cliente usar antes de prosseguir.
3. Todos os paths e exemplos deste documento usam `{cliente}` como placeholder.

---

## Pré-requisito — Brand Profile

Antes de qualquer geração, carregar `{cliente}/brand-profile.md` (schema completo em `_sistema/referencias/brand-profile-conteudo-visual.md`) e extrair:

- Paleta de cores e tokens (`{brand_primary}`, `{brand_primary_rgb}`, `{brand_accent}`, `{brand_bg}`, `{brand_ink}`, `{card_white}`, `{border}`)
- Tipografia por elemento (fonte, peso, tamanho, cor)
- Regras de fundo e posição de logo
- Filtros de copy e CTA padrão
- Vocabulário do público e padrões de humanização
- Vocabulário de ícones temáticos (mapeamento tema → ícone)
- Contexto do cliente (segmento, público-alvo, tom)
- Caminhos do ambiente (`{path_html_temp}`, `{path_logo}`, `{path_output}`, `{path_temp_dir}`)
- Handle e nome do perfil para o frame de preview (`{brand_handle}`, `{brand_profile_name}`)

**Dado factual (endereço, telefone, WhatsApp, horário, serviço específico ou diferencial):** sempre consultar a seção "Dados da empresa" do brand-profile e usar exatamente o que está lá. Nunca inventar, aproximar ou supor serviço, estrutura ou informação que não conste nessa seção — se o post precisar de um dado que não está lá, perguntar ao usuário em vez de completar sozinho. **Horário em especial:** nunca assumir que existe um único campo genérico de "horário" — negócios com serviço-fim diferente do atendimento ao público (ver nota em `brand-profile-conteudo-visual.md` → "Dados da empresa") podem ter mais de um horário registrado; usar o campo que corresponde ao assunto do post, não o primeiro que aparecer.

---

## Passo 0 — Vocabulário do público (espelho de linguagem)

Antes de montar o copy, consultar a seção "Vocabulário do público" do brand-profile para entender como o público descreve o problema, não como keywords a inserir, mas como espelho de vocabulário e ângulo.

**Regra:** keywords não entram literalmente no texto. O que muda é o **vocabulário e o ângulo**: escrever do jeito que o público pensa no assunto, não do jeito que a empresa descreve o serviço.

---

## Passo 0.5 — Criar checklist de execução (obrigatório)

Antes de coletar os dados do post, criar uma lista TodoWrite com os gates obrigatórios desta skill: Passo 1.8 (Humanizar o texto), Passo 2.6 (Autoavaliação visual), Passo 2.7 (Criar legenda). Ver [[auditoria-execucao-skills]] — marcar cada item `completed` só quando a etapa de fato rodar.

---

## Passo 1 — Coletar dados do post

Perguntar ao usuário antes de gerar:

1. **Tipo de post** — identificar pelo conteúdo (ver tabela abaixo)
2. **Conteúdo** — texto, dados, nome do protagonista, etc.
3. **Imagem** — por padrão, antes de perguntar ao usuário se ele tem foto própria, tentar buscar no banco de imagens via MCP `banco-imagens` (ver [[mcp-banco-imagens]]):

- Chamar `buscar_imagem(cliente: "{cliente}", consulta: "<tema do post>")`. Por padrão só retorna imagens **ainda não usadas** em nenhum post anterior (nunca repete)
- **Score alto não é sinônimo de coerência temática — checar os dois antes de sugerir.** `score` ≥ 0.65 mede similaridade textual da consulta, não garante que a cena bate com a emoção/situação específica do post. Ex: buscar por "pânico, travar na prova" pode retornar cenas de condução tranquila e confiante (score 0.65–0.67) que são tecnicamente próximas mas contradizem o tema. Antes de sugerir, avaliar a imagem em si contra o tema real do slide/post — se não bater, não sugerir só porque o score passou do threshold
- Se a imagem bater tanto em score quanto em coerência temática, apresentá-la ao usuário como sugestão ("Achei esta imagem no banco, quer usar?") antes de embutir; só usa após confirmação
- **Antes de desistir e cair pro tipográfico, tentar reformular a busca** (outra consulta, outra categoria de `listar_indice` que pareça plausível pro tema) em vez de decidir sozinho na primeira tentativa sem sucesso. Uma consulta genérica pode não achar nada relevante enquanto uma categoria mais específica tem exatamente a foto certa
- **Só cai para o fluxo manual (perguntar se o usuário tem foto própria, ou seguir com ícone puro) depois de tentar reformular e mesmo assim não haver imagem com contexto relevante** — banco vazio, todas as imagens já usadas, nenhuma reformulação resolveu, ou MCP indisponível. Ao cair pro fluxo manual, informar ao usuário rapidamente o que foi tentado (ex: "busquei por X e Y no banco, não achei nada coerente com o tema"), para que ele possa apontar uma categoria/pasta específica se souber de uma. **Nunca bloquear a geração do post por causa do banco de imagens.**
- Imagem fornecida pelo usuário (não veio do banco) deve ser indexada com `indexar_imagem`, para engordar o banco ao longo do tempo

| Conteúdo do post | Tipo |
|---|---|
| Avaliação, depoimento de cliente (texto) | A — Depoimento |
| Print real de avaliação do Google | A2 — Avaliação Google |
| Dica, curiosidade, fato educativo | B — Educativo |
| Pergunta para engajar seguidores | C — Pergunta/Engajamento |
| Foto real de protagonista (cliente, aluno, usuário) em conquista, entrega, bastidor com equipe | D — Conquista/Prova Social com Foto |
| Promoção, oferta, preço, condição especial (sem foto) | E — Oferta |
| Mesma situação de E, mas com foto real disponível | E2 — Oferta com Foto |
| Marco numérico (anos, clientes, avaliações) | G — Marco Numérico |
| Data comemorativa, homenagem sazonal | F — Data Comemorativa |
| Aviso de horário, funcionamento, feriado, mudança de agenda | H — Comunicado/Aviso Rápido |

Ver a matriz completa de escolha de layout em `_sistema/referencias/templates-post-instagram.md` se o conteúdo não encaixar claramente numa linha acima.

Se o usuário disser "faz um post sobre X" sem especificar o tipo, inferir e confirmar antes de gerar.

---

## Passo 1.2 — Estrutura do post educativo (só para Tipo B)

Invocar `/gerador-de-post-educativo-para-instagram` com os dados coletados no Passo 1,
pré-preenchendo o contexto do nicho a partir do brand-profile:

```
Tema do post: [tema do Passo 1]
Nicho: [segmento do cliente — ver brand-profile]
Público-alvo: [ver brand-profile]
Nível do público: iniciante/intermediário
Formato desejado: imagem única com legenda longa
Tom: [ver brand-profile — filtros de tom]
Objetivo: gerar salvamentos, construir autoridade
```

**Extrair e mapear o output — só o texto que vai NA arte:**

| Output do gerador | Destino |
|---|---|
| Frase de impacto (texto da imagem) | → Título do Layout B |
| Corpo educativo (1–2 pontos, máx. 40 palavras cada) | → Texto do Layout B |

A legenda completa e as hashtags que o gerador também devolve **não são usadas aqui** — descartar essa parte do output. A legenda de verdade é criada depois da arte pronta e validada, por `/criar-legenda-instagram` (ver Passo 2.7).

Avançar imediatamente para o Passo 1.5.

---

**Tipos A e A2 (Depoimento / Avaliação Google):** o texto da arte não vem de um gerador de copy — usa o print (A2) ou o depoimento bruto humanizado (A), coletado no Passo 1. Avançar diretamente para o Passo 1.5.

**Tipo A2 — preparar o print antes de embutir (obrigatório, nunca pular):** prints de avaliação (Google e afins) quase sempre vêm com barras pretas de letterbox nas bordas (screenshot com fundo preto ao redor do card branco). Antes de qualquer coisa:
1. Medir a caixa exata do conteúdo real (card branco) via varredura de pixel — nunca "no olho". Usar `mcp__playwright__browser_run_code_unsafe` com um canvas: carregar a imagem, ler `getImageData`, varrer linhas/colunas até achar onde o preto (`r,g,b < 40`) termina
2. Recortar exatamente essa caixa (outro canvas, `drawImage` com `sx,sy,sw,sh`) e salvar o resultado via `page.waitForEvent('download')` + `download.saveAs(...)` — nunca usar o arquivo bruto
3. Usar as dimensões do recorte pra calcular a proporção do container de destino (ver Regra 5 de "Imagens fornecidas pelo usuário" em `templates-post-instagram.md`) — evita cortar texto do print por incompatibilidade de aspect ratio, especialmente nas variações com moldura/rotação

---

## Passo 1.5 — Ícone temático como elemento de arte

O ícone (PNG próprio do Banco de Ícones, ou Font Awesome 6 como fallback) é um **elemento visual da arte**, não apenas um rótulo. Deve ser aplicado em **todos os tipos de post** (A, B, C, D, E, E2, F, G, H), com papel e posição escolhidos conforme o conteúdo e a composição visual (ver "Papéis do ícone temático" em `templates-post-instagram.md`).

**Tipo A2:** usar o **logo oficial do Google** (SVG dos 4 arcos coloridos, não um ícone monocromático) como badge circular fixo no header — nunca `fa-solid fa-star` ou ícone genérico. É print de avaliação do Google especificamente; o logo real deixa isso inequívoco à primeira vista, o que uma estrela genérica não comunica. Path SVG de referência:

```html
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 48 48" width="26" height="26">
  <path fill="#EA4335" d="M24 9.5c3.54 0 6.71 1.22 9.21 3.6l6.85-6.85C35.9 2.38 30.47 0 24 0 14.62 0 6.51 5.38 2.56 13.22l7.98 6.19C12.43 13.72 17.74 9.5 24 9.5z"/>
  <path fill="#4285F4" d="M46.98 24.55c0-1.57-.15-3.09-.38-4.55H24v9.02h12.94c-.58 2.96-2.26 5.48-4.78 7.18l7.73 6c4.51-4.18 7.09-10.36 7.09-17.65z"/>
  <path fill="#FBBC05" d="M10.53 28.59c-.48-1.45-.76-2.99-.76-4.59s.27-3.14.76-4.59l-7.98-6.19C.92 16.46 0 20.12 0 24c0 3.88.92 7.54 2.56 10.78l7.97-6.19z"/>
  <path fill="#34A853" d="M24 48c6.48 0 11.93-2.13 15.89-5.81l-7.73-6c-2.15 1.45-4.92 2.3-8.16 2.3-6.26 0-11.57-4.22-13.47-9.91l-7.98 6.19C6.51 42.62 14.62 48 24 48z"/>
</svg>
```

Colocar num círculo branco (`background:#fff`, borda sutil `{border}`) em vez de fundo `{brand_primary}` — o logo já tem cor própria, fundo colorido compete com ele.

### Mapeamento tema → ícone

Antes de escolher o ícone, invocar `classificar-icones` — se houver PNG pendente em `{cliente}/Identidade Visual/Ícones/_entrada/`, ele é classificado e nomeado ali, ficando disponível pra esta escolha; se a pasta estiver vazia, a skill não faz nada e segue o fluxo normal.

Vocabulário (qual ícone usar por tema, PNG próprio ou Font Awesome) e a técnica de CSS mask pra recolorir o PNG: ver `{cliente}/brand-profile.md` → "Vocabulário de ícones temáticos". Regra de fallback está lá também: se o tema não estiver na tabela, escolher o ícone mais próximo semanticamente, ou `fa-solid fa-circle-info` em caso de dúvida.

---

## Passo 1.8 — Humanizar o texto (obrigatório)

Antes de montar o HTML, invocar a skill `/humanizer` no texto planejado para o post.

**Como executar:**

1. Escrever o rascunho do texto (tag, título, corpo, CTA)
2. Consultar brand-profile, seção "Humanização", para padrões específicos do nicho a corrigir
3. Invocar `/humanizer` passando esse rascunho como input
4. Usar o texto final retornado pelo humanizer no HTML, nunca o rascunho direto

**Critério de aprovação:** o texto passa se alguém poderia ter escrito assim numa mensagem de WhatsApp.

---

## Passo 2 — Gerar o HTML

Usar `_sistema/referencias/templates-post-instagram.md` para:

- Escolher o layout do tipo definido no Passo 1 (A, A2, B, C, D, E, E2, F, G ou H)
- Aplicar os componentes reutilizáveis (aspas, faixas, CTA — textual/botão/selo, variações de ícone)
- Nos tipos D e E2 (foto real), seguir também "Composição com foto real" (Bloco 2) — crop, sombra, overlay, preservação de rosto
- Seguir as regras de embutir imagem do usuário em base64
- Montar o frame Instagram de preview
- Validar as dimensões do layout (obrigatório antes de apresentar)

Substituir todos os tokens (`{brand_primary}`, `{path_logo}` etc.) pelos valores extraídos do brand-profile no Pré-requisito.

---

## Passo 2.6 — Autoavaliação visual (obrigatório antes de apresentar)

**Objetivo: reduzir ao máximo, idealmente zerar, a necessidade de o usuário apontar problemas de acabamento.** Esta etapa acontece depois da validação de dimensões e antes de mostrar qualquer versão ao usuário (Fluxo de revisão). Nunca pular, mesmo em iterações rápidas do loop.

**Antes de invocar `/validar-arte-visual`, se o HTML usar algum elemento posicionado de forma absoluta/flutuante perto de outro bloco (badge, pill, selo, tag sobre um card, foto ou print, ou texto de CTA/corpo próximo a um logo posicionado em `position:absolute` no rodapé):** medir as caixas delimitadoras dos elementos envolvidos (via `getBoundingClientRect` no browser, comparando `top`/`bottom`/`left`/`right` de cada um) e confirmar que não há sobreposição não intencional antes de considerar a arte pronta pra autoavaliação. Não confiar só na leitura visual do preview — uma sobreposição de poucos pixels no preview vira uma faixa visível na exportação final em escala maior, e já passou despercebida tanto na geração quanto na autoavaliação visual numa sessão anterior.

Com o post já renderizado na resolução final (1080×1350, mesma regra do Fluxo de revisão), primeiro rodar a "Verificação automática de contraste (medição por pixel)" de `_sistema/referencias/templates-post-instagram.md` — corrigir qualquer bloco de texto reprovado antes de seguir. Só depois invocar `/validar-arte-visual` (ver [[validar-arte-visual]]) passando:

- O caminho do PNG renderizado
- O contexto de marca (paleta e princípios do brand-profile)
- Canal: feed Instagram
- **Elementos obrigatórios: logo, se essa for a regra fixa da marca do cliente** (brand-profile, seção Princípios de design — verificar se logo é obrigatório ou opcional). `validar-arte-visual` trata logo como opcional por padrão pra qualquer cliente; se a marca do cliente exige logo sempre, passar essa informação explicitamente
- CTA obrigatório, exceto no Tipo H (comunicado/aviso), que pode dispensar CTA por padrão

- Veredito "Aprovada" ou "Aprovada com observações" → seguir pro Fluxo de revisão (repassar as observações ao usuário, se houver)
- Veredito "Solicitar ajustes antes de aprovar" ou "Reprovada" → corrigir os itens de prioridade alta/média do relatório e repetir a autoavaliação, sem esperar o usuário apontar

**Teto de segurança: 7 tentativas.** Se depois de 7 rodadas de correção o veredito ainda não for "Aprovada" ou "Aprovada com observações", parar o loop e apresentar a arte ao usuário mesmo assim — junto com o relatório da última autoavaliação e uma nota explícita do que não foi possível resolver sozinho. Nunca insistir indefinidamente sem informar o usuário.

### Guia de correção rápida (específico deste template HTML/CSS)

Quando o relatório de `validar-arte-visual` apontar um destes problemas, o ajuste técnico neste template é:

- **Bloco reprovado na Verificação automática de contraste** (razão medida abaixo do threshold) → não tentar adivinhar a cor certa; escurecer/clarear o texto na direção do fim "ink" da rampa da marca (nunca cinza genérico) e remedir até passar do threshold. Se o fundo for uma foto ou gradiente, considerar uma faixa/overlay sólido atrás do texto em vez de só mudar a cor da fonte
- **Respiro concentrado num vazio só** (critério 4) → nunca usar `margin-bottom:auto` sozinho pra empurrar o rodapé; definir gaps fixos entre cada bloco e conferir a soma
- **Sombra de foto pesada** (critério 9) → reduzir opacidade/blur do `box-shadow`; na dúvida entre dois valores, escolher o mais sutil
- **CTA e logo desproporcionais** (Regra D) → ajustar `font-size` do CTA e `width` do logo até ficarem parceiros visuais no rodapé
- **Corte desconfortável de rosto** (Regra B) → ao mexer em `object-position` ou na altura da caixa da foto pra resolver outro problema (ex: fundo bagunçado), sempre reconferir se a mudança não cortou olhos/boca/queixo/topo da cabeça de alguém
- **Selfie muito próxima** (Regra C) → não force crop artificial; registrar como limitação do insumo e, se fizer sentido, sugerir ao usuário fotos futuras com mais margem ao redor das pessoas
- **Elemento decorativo fraco demais** (Regra A) → decidir entre aumentar contraste/espessura ou remover, nunca deixar ambíguo
- **Dado suspeito ou incorreto** (Regra G) → conferir manualmente número de WhatsApp, telefone, endereço, horário, serviço ou data que entrou no texto contra `{cliente}/brand-profile.md` → "Dados da empresa" antes de fechar o HTML; se não der pra confirmar, não afirmar que está certo — sinalizar ao usuário em vez de deixar passar
- **Elemento perto demais da borda** (Regra H) → respeitar o padding lateral mínimo do layout escolhido (ver tabela de cada tipo em `templates-post-instagram.md`); nunca reduzir o padding só pra caber mais texto
- **Imagem cortando conteúdo essencial (texto de print, rosto)** → provável incompatibilidade entre a proporção do container (`object-fit:cover`) e a proporção real da imagem-fonte; medir a imagem e ajustar a altura do container antes de ajustar qualquer outra coisa (ver Regra 5 de "Imagens fornecidas pelo usuário" em `templates-post-instagram.md`)
- **Foto de produto/objeto do banco de imagens (item isolado em fundo branco) cortando ou "sobrando" espaço vazio demais dentro do card** → nunca insistir alternando só `cover`/`contain`/`object-position` no mesmo card pequeno — primeiro checar se a foto tem excesso de fundo próprio ao redor do assunto (recortar pela caixa real do conteúdo antes de embutir) e, se o assunto precisar ficar 100% visível, migrar para a variação "Foto de topo (full-bleed, sem sobreposição)" do Layout E2 em vez de forçar num card — ver Regra 6 de "Imagens fornecidas pelo usuário" em `templates-post-instagram.md`
- **Número gigante (hero) parecendo desalinhado/solto do resto do título** → montar número + frase como um só bloco de texto corrido (spans inline), não como linha separada + parágrafo à parte (ver "Modo Hero vs Modo Seguro" em `templates-post-instagram.md`)
- **Texto do botão de CTA quebrando estranho em duas linhas** → o botão deve ter só o `{cta_text}` padrão da marca, curto; qualquer ideia extra vai no corpo de texto ou na legenda, nunca dentro do botão (ver "CTA — escolher a variação" em `templates-post-instagram.md`)
- **Vazio grande entre CTA e logo em slide de fechamento/gradiente** → padding do container de conteúdo deve ser assimétrico (`padding-bottom` maior que `padding-top`) quando o conteúdo é centralizado verticalmente e o logo é elemento fixo à parte (ver mesma seção de CTA em `templates-post-instagram.md`)
- **Badge/pill/selo flutuante sobrepondo o canto de um card, foto ou print** → não é um efeito de "carimbo sobre a peça" válido por padrão; separar com uma faixa de respiro real (posicionar o elemento flutuante inteiramente acima ou ao lado do bloco, nunca cruzando a borda dele) e confirmar por medição de bounding box, não só olhando o preview
- **Salto grande de tamanho entre dois textos vizinhos** (ex: citação decorativa gigante ao lado de uma legenda pequena) → aproximar as escalas (reduzir o elemento maior, aumentar levemente o menor) até a transição parecer parte do mesmo sistema visual, não dois elementos de peças diferentes coladas
- **Elemento centralizado no restante da composição mas um bloco (ex: header/tag) ficou alinhado à esquerda por padrão** → revisar se todos os blocos da mesma peça compartilham o mesmo eixo de alinhamento; numa composição centralizada, cabeçalho/tag/rótulo também centralizam, não só o conteúdo principal
- **Texto de CTA (ou último elemento de um bloco de conteúdo centralizado) sobrepondo o logo no rodapé** → sempre que o layout tiver logo `position:absolute` no rodapé (canto inferior direito, por exemplo) E o bloco de conteúdo acima usar `justify-content:center` numa altura fixa, medir via `getBoundingClientRect` antes de considerar pronto — não confiar na leitura visual do preview. Se não houver folga de pelo menos ~15-20px entre o fim do CTA e o topo do logo, aumentar o `padding-bottom` do container de conteúdo (calibrar por medição real, não estimativa — no caso real corrigido, 52px de padding-bottom resolveu um caso de ~22px de sobreposição). Erro real encontrado em produção (25/07/2026): a primeira versão saiu com o CTA sobrepondo o logo, só detectado porque o Léo apontou visualmente — a autoavaliação informal não pegou

---

## Passo 2.7 — Criar legenda (depois da arte aprovada na autoavaliação)

Só depois que o Passo 2.6 aprovar a arte (veredito "Aprovada" ou "Aprovada com observações"), invocar `/criar-legenda-instagram` (ver [[criar-legenda-instagram]]) passando:

- **Formato:** post único (feed Instagram)
- **Ângulo:** o Tipo definido no Passo 1, mapeado conforme a tabela abaixo
- **O que de fato ficou na arte final:** título, corpo, dados e CTA validados no Passo 2.6 — nunca o rascunho planejado no Passo 1, caso algo tenha mudado nas correções
- **Contexto de marca:** brand-profile (Pré-requisito)
- **Provas sociais disponíveis** (Tipos A, A2, D): depoimento, nome, avaliação, resultado — coletados no Passo 1
- **CTA desejado:** se o usuário tiver pedido um CTA específico; senão, o CTA padrão do brand-profile

| Tipo (Passo 1) | Ângulo pra `/criar-legenda-instagram` |
|---|---|
| A — Depoimento | Depoimento / prova social |
| A2 — Avaliação Google | Depoimento / prova social |
| B — Educativo | Educativo |
| C — Pergunta/Engajamento | Institucional / engajamento puro |
| D — Conquista/Prova Social com Foto | Depoimento / prova social |
| E — Oferta | Oferta / urgência |
| E2 — Oferta com Foto | Oferta / urgência |
| F — Data Comemorativa | Institucional / data comemorativa |
| G — Marco Numérico | Marco numérico |
| H — Comunicado/Aviso Rápido | Institucional / comunicado |

O bloco de legenda + hashtags retornado é apresentado junto com o preview no Fluxo de revisão (abaixo) — nunca só o PNG sozinho.

---

## Fluxo de revisão e exportação

**Auditoria antes de apresentar (obrigatório):** reconferir a lista TodoWrite do Passo 0.5 — todos os itens devem estar `completed` antes de mostrar qualquer preview ao usuário. Se algum estiver pendente, executar a etapa faltante primeiro (ver [[auditoria-execucao-skills]]).

Seguir o "Fluxo de revisão — loop obrigatório" e a "Exportação — perfil Feed 4:5 (PNG 1080×1350px)" de `_sistema/referencias/templates-post-instagram.md`. Apresentar a legenda do Passo 2.7 junto com o preview da arte — se o feedback do usuário mudar texto ou dado que a legenda referencia, repetir o Passo 2.7 antes de exportar.

**Adendo específico desta skill:**
- Se o post usou uma imagem vinda do banco (`buscar_imagem`), chamar `marcar_imagem_usada(cliente: "{cliente}", caminho: <caminho da imagem>)` logo após a exportação bem-sucedida, nunca antes (só quando o post é de fato exportado, não apenas sugerido/rejeitado no meio do loop de revisão).
- Salvar a legenda final do Passo 2.7 em `legenda.md` na mesma pasta do `post.png` exportado (`{cliente}/Posts/<AAAA-MM>/Postagens/<AAAA-MM-DD>-<tema>/legenda.md`), logo após a exportação bem-sucedida — nunca antes de o post ser de fato exportado. Formato:

```markdown
---
data: <AAAA-MM-DD>
tema: <tema>
tipo: <Tipo do Passo 1>
status: pronto
---

# Legenda — <tema>

<texto da legenda>

---

**Hashtags:**
<hashtags>
```

---

## Princípios de design e restrições visuais

Ver `{cliente}/brand-profile.md`, seções "Princípios de design" e "Restrições visuais".
