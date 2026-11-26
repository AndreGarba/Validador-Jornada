# 11 — Validador como Skill do Agente do Figma

## Objetivo
Rodar o Validador dentro do Figma Design com o **agente do Figma**, em forma de
skill. Cada usuário gasta os **próprios créditos de IA do Figma** a cada teste,
sem informar nenhuma chave e sem servidor próprio.

Esta PRD descreve uma **versão alternativa**. Ela não altera as PRDs `00` a
`10`. A PRD `10` (plugin + servidor com chave da equipe) continua como outra
alternativa.

## Por que skill, e não plugin
- A Plugin API do Figma não dá acesso à IA do Figma. Um plugin clássico só usa
  IA chamando uma API externa, com a chave de alguém.
- Plugins generativos gastam créditos só para serem criados; ao rodar, não usam
  IA e não podem chamar APIs externas.
- Quem gasta os créditos de IA de cada usuário é o **agente do Figma**, que
  passou a consumir créditos em 6 de outubro de 2026.
- O agente aceita **skills personalizadas**: um arquivo `.md` com instruções,
  chamado no chat por um comando (ex.: `/validador-de-jornada`).

## Arquivo da skill
- Fica em `Skill/validador-de-jornada.md`, na raiz do projeto.
- Um único arquivo `.md` no padrão Agent Skills (frontmatter com `name` e
  `description`). O Figma não aceita pastas de apoio (`scripts/`,
  `references/`, `assets/`), então todas as regras ficam no próprio arquivo.
- O `name` define o comando: `/validador-de-jornada`.

## Relação com as outras PRDs
| PRD | Na versão skill |
|---|---|
| `00` Visão geral | Vale: modos, objetivo da tarefa obrigatório, ordem "persona puxa a heurística" e catálogo completo. Link navegável fica fora. **Exceção:** o peso 70% persona / 30% heurística foi substituído por pesos pela origem da persona (ver "Pesos das notas"). |
| `04` Back-end (notas) | **Exceção:** a fórmula `0,7 × nota_persona + 0,3 × nota_heuristica` e o desconto genérico por severidade foram substituídos (ver "Pesos das notas"). |
| `01` Modelo da persona | Vale: template, campos e pesos estão na skill. |
| `02` Normalização | Vale: regras de não inventar e "Não mapeado" estão na skill. A confirmação é feita no chat. |
| `03` Assertividade | Vale: fórmula, fatores, faixas e bloqueio abaixo de 40% estão na skill. |
| `04` Back-end | **Não se aplica.** Não há servidor, banco nem histórico. Valem a escala de severidade, as notas e a regra da nota limitada a 4,0. |
| `05` Prompts | Vale: os prompts 1, 2, 3A, 3B e 4 viraram as etapas 2 a 5 da skill. Com persona, os problemas são separados em Usuário e Heurísticas (ver "Avaliadores e assuntos"). |
| `06` Tela principal | **Não se aplica.** As informações do teste são coletadas no chat. Não há biblioteca de personas. |
| `07` Resultado no Figma | Vale o canal de anotações do Dev Mode, com textos e resumo. Comentários do Figma não são usados nesta versão. **Exceções:** as categorias são por assunto, e não por severidade (a severidade fica no prefixo do texto); a skill cria os frames "Validador · Relatório", "Validador · Persona", "Validador · Design System" e "Validador · Princípios de Design" (ver "Resultado"). |
| `12` Princípios de Design | Vale: os cinco princípios entram como avaliador próprio (ver "Princípios de Design na skill"). |
| `13` Princípios de Motion | **Ainda não se aplica.** Fica para quando a validação puder rodar num link de protótipo funcional com motion. |
| `14` Writing | **Ainda não se aplica.** Será pensado depois. |
| `08` Orquestra | **Substituída nesta versão**, por decisão. O avaliador Design System avalia o uso da **biblioteca do Figma** (componentes, detach e tokens), e não o código pelo MCP do Orquestra. Ligação, severidades, aderência e ids de `08` não se aplicam; vale só a regra de não entrar na nota da jornada nem nas prioridades. |
| `09` Chave de API | **Não se aplica.** O usuário não informa chave. |
| `10` Plugin + servidor | Alternativa. Não se aplica a esta versão. |

## Escolha da validação
A primeira mensagem da skill é sempre esta pergunta de múltipla escolha:

```
Qual será a validação da tela/jornada?
1 - Heurísticas + Persona (caso inclua uma persona)
2 - Heurísticas (caso não tenha uma persona)
3 - Design System
4 - Princípios de design
5 - Todas

Você pode combinar opções (ex.: "2, 3 e 4").
```

Decisão de 27/10/2026:
- Aceita combinações, mas 1 e 2 não se combinam: a persona sempre roda junto
  com as heurísticas. **Não existe mais a validação só de persona.**
- "5 - Todas" equivale a 3 e 4 mais a opção 1 (com persona) ou a 2 (sem
  persona). Se a persona não veio anexada, a skill pergunta se vai usar uma.
- Só rodam os avaliadores escolhidos. Anotações, frames, resumo, chat e JSON
  mostram só o que foi escolhido.

| Escolha | Modo | Nota da tela |
|---|---|---|
| 1 | Persona + heurísticas | `peso_persona × nota_persona + peso_heuristica × nota_heuristica` (pesos pela origem da persona) |
| 2 | Heurísticas | `nota_heuristica` |
| Só 3 e/ou 4 | Design System e/ou Princípios de design | Sem nota da tela nem da jornada; só as notas próprias desses avaliadores |

- Os modos voltam a ser os de `00` ("persona + heurísticas" e "somente
  heurísticas"); o modo só Persona desta versão foi removido.
- O objetivo da tarefa é obrigatório com a opção 1 ou 2. **Só Design System
  e/ou Princípios de design não pedem objetivo** e não geram nota da jornada,
  objetivo concluído, ponto de abandono, prioridades nem o frame "Validador ·
  Relatório".
- Com a opção 1, a skill pede o anexo se ele não veio. Persona anexada com a
  opção 2 não é usada (a skill pergunta se quer trocar para a 1).
- Abaixo de 40% de assertividade, o usuário enriquece a persona ou troca a
  opção 1 pela 2.

## Avaliadores e assuntos
Todo resultado é separado em quatro assuntos, nas anotações e nos frames:

| Assunto | O que entra | Quando roda | Ids |
|---|---|---|---|
| Usuário | Problemas que a persona percebe, com a reação dela em primeira pessoa e a heurística ligada. | Opção 1 | U1, U2… |
| Heurísticas | Problemas técnicos que a persona não perceberia (ex.: contraste, rótulo para leitor de tela). Sem persona, todos os problemas ficam aqui. Um problema não aparece em Usuário e em Heurísticas ao mesmo tempo. | Opções 1 e 2 | H1, H2… |
| Design System | Uso da biblioteca do Figma: componentes com detach, falta de tokens e componentes locais (ver "Design System na skill"). | Opção 3, com biblioteca de referência | DS1, DS2… |
| Princípios de Design | Só o que está fora dos princípios de design de `12`, incluindo inconsistências entre telas (ver "Princípios de Design na skill"). | Opção 4 | PD1, PD2… |

- `nota_heuristica` desconta só os problemas de Heurísticas; `nota_persona`
  vem da simulação e reflete os problemas de Usuário. Um problema conta uma
  vez só (sem dupla penalização). O limite de 4,0 com problema crítico
  continua valendo.

### Pesos das notas
Decisão de 26/10/2026, a partir da análise de metodologia feita com o
ChatGPT. Substitui o 70/30 de `00` e o desconto genérico de `04` nesta versão.

**Persona x heurísticas (modo Persona + heurísticas)**, pela origem da
persona, que a skill sempre pergunta ao usuário (não deduz pelo texto):

| Origem da persona | Heurísticas | Persona |
|---|---|---|
| Pesquisa (entrevistas, testes ou dados reais) | 60% | 40% |
| Pesquisa complementada por IA | 70% | 30% |
| Só IA, sem pesquisa | 80% | 20% |
**`nota_heuristica` com fator de risco por grupo** (revisado em 26/10/2026;
substitui a média ponderada 43/21/14/14/8, que deixava um grupo sem
problemas compensar as falhas de outro). A nota começa em 10, sem ficar
abaixo de 0, e cada problema H desconta:

```
desconto = severidade de Nielsen (crítica 4, alta 3, média 2, baixa 1) × fator do grupo × 0,5
```

| Grupo | Fator | Embasamento |
|---|---|---|
| Acessibilidade (WCAG) | 1,5 | Obrigatória por lei (LBI, art. 63) e falha mais comum na web (WebAIM Million 2026). |
| Dark patterns | 1,5 | Risco ao consumidor e legal (Mathur et al., 2019; FTC, 2022; CDC, art. 37; LGPD, art. 8º). |
| Nielsen | 1,0 | Base da avaliação heurística. |
| UX cultural | 1,0 | Impacto no desempenho (Reinecke e Bernstein, 2011). |
| Krug | 0,75 | Sobrepõe parte de Nielsen; evita contar o mesmo problema em dobro. |

- Exemplo (Page 17, Template Estudo): um problema alto e um médio de
  contraste descontam 2,25 + 1,5, e a nota fica 6,3. Na média ponderada,
  ficaria 9,0.
- Só o problema crítico limita a nota da jornada a 4,0, em qualquer grupo.
- O uso de pesos por relevância segue Talero-Sarmiento et al. (2024); os
  valores dos fatores são calibração do time.

- As notas de Usuário e de Heurísticas aparecem sempre separadas, além da
  combinada, para uma nota boa não esconder problemas da outra.
- O relatório mostra o desconto de cada problema e a conta da nota da tela.
- `heuristic-conflicts.md` e `audit-template.md` não têm peso: o primeiro
  resolve conflitos entre heurísticas, o segundo dá o formato do relatório.
- As prioridades juntam U e H. Design System fica em lista própria, com nota
  própria, sem entrar na nota da jornada.

### Possível abandono de jornada
- Em cada tela, a skill verifica se a pessoa desistiria da jornada ali (com
  persona, pelos comportamentos, dores e contexto dela; sem persona, quando
  não há como avançar no objetivo).
- O problema que causa o abandono é sempre **crítico** (nota da jornada
  limitada a 4,0), é a primeira prioridade e o objetivo concluído vira "não".
- A avaliação **continua** nas telas seguintes, com todos os avaliadores.
  Essas telas são marcadas como "Avaliada após o possível abandono".
- Destaque "⛔ POSSÍVEL ABANDONO DE JORNADA":
  - na anotação do problema (primeira linha) e numa anotação presa ao frame
    da tela;
  - na primeira linha da anotação de resumo e da resposta no chat;
  - no relatório: faixa vermelha logo abaixo do cabeçalho, tag no plano de
    ajuste e card da tela com borda vermelha;
  - no JSON (`ponto_de_abandono`, `possivel_abandono`, `apos_abandono`).

### Dispositivo da persona x formato das telas
Regra nova desta versão, para não deixar passar uma persona de celular
avaliando telas de desktop (e vice-versa).

- Formato de cada frame pela largura: até 599 px celular, de 600 a 1023 px
  tablet, 1024 px ou mais desktop.
- O dispositivo vem de `dispositivo_principal` e do Contexto de uso da
  persona (`01`). Se a persona não informar, a skill pergunta ao usuário qual
  considerar. A resposta vale só para o teste: não entra na persona nem muda
  a assertividade.
- A conferência acontece depois da assertividade e antes da avaliação das
  telas. Se algum frame estiver em outro formato, a skill avisa e pergunta se
  segue ou se o usuário vai trocar os frames.
- Se seguir, a incompatibilidade vira o problema **U1** (assunto Usuário),
  na jornada inteira:
  - heurística Krug K7 quando a persona usa celular ou tablet; Nielsen N4
    quando a persona usa desktop e as telas são de celular ou tablet;
  - **crítica** se a persona usa só esse dispositivo (nota da jornada
    limitada a 4,0); **alta** se ela também usa o formato das telas;
  - as telas são avaliadas como a persona as veria no dispositivo dela, sem
    repetir o problema de incompatibilidade em cada tela.
- A compatibilidade aparece na anotação de resumo, no cabeçalho do relatório,
  no frame da persona, no chat e no JSON (`dispositivo`).

### Design System na skill
Avalia o uso da biblioteca do design system **no Figma**, não o código. O MCP
do Orquestra não é usado.

- **Bibliotecas de referência:** a skill lista todas as bibliotecas publicadas
  ativadas no arquivo e o usuário marca quais valem (o Orquestra costuma estar
  dividido em várias, como `01_styles`, `01_token` e `03_web_components`). Se
  houver um frame "Validador · Design System" anterior na página, a skill
  oferece repetir a mesma lista. Componentes locais do arquivo nunca são
  referência. Sem biblioteca, segue sem esse avaliador e registra o motivo.
- **Unidades avaliadas** (regra fixa, para a nota ser comparável entre
  testes): cada instância de componente conta uma vez, com seus elementos
  internos; cada texto solto conta uma vez; cada contêiner visual próprio
  (preenchimento, borda ou raio) conta uma vez. Não contam o frame da tela,
  frames só de organização, camadas ocultas e o conteúdo de imagens.
- **Wrapper local:** um componente local cuja estrutura principal é uma
  instância de uma biblioteca de referência (ex.: "Header GoPan" com o
  "Header" da biblioteca dentro) conta como **correto**, se a instância
  interna não teve detach e não há valores fixos no wrapper nem nela.
  Senão, vale a tabela abaixo.
- **O que verifica:** se o elemento é instância da biblioteca, componente
  local ou desenhado à mão; se houve detach; se há componente similar na
  biblioteca; e se cores, textos, efeitos, espaçamentos e raios usam estilos
  e variáveis da biblioteca (inclusive valores sobrescritos em instâncias).
- Cada elemento recebe um resultado; se couber mais de um, vale o mais grave:

| Situação | Severidade | Peso na nota |
|---|---|---|
| Instância da biblioteca sem valores fixos | Correto | 1,00 |
| Componente com detach | Crítica | 0,00 |
| Valores fixos em vez de tokens (cor, texto, espaçamento, raio, estilo) | Alta | 0,25 |
| Componente local ou elemento à mão com similar na biblioteca | Alta | 0,25 |
| Componente local sem similar, feito sem tokens | Média | 0,50 |
| Componente local sem similar, feito com tokens | Baixa | 0,75 |

- **Nota de uso do Design System:** por tela,
  `10 × (soma dos pesos ÷ unidades avaliadas)`; da jornada, a média das
  telas. Fica separada da nota da jornada.
- **Inventário obrigatório:** todas as unidades de cada tela, numeradas, com
  o resultado de cada uma, para conferir diferenças entre testes.
- O resultado completo vai para o frame "Validador · Design System"; o
  relatório mostra só a nota.

### Princípios de Design na skill
Decisão de 27/10/2026. Avalia a tela ou a jornada pelos cinco princípios de
design do BTG Pactual (`12`): Humano, Informado, Encantador, Consistente e
Eficiente. Os princípios de motion (`13`) ficam para depois, quando a
validação puder rodar num link de protótipo funcional com motion; o Writing
(`14`) também fica para depois.

- **Só o que está fora:** o avaliador registra apenas o que contraria um
  princípio. O que está de acordo não vira apontamento.
- **O que verifica:** a skill traz, para cada princípio, a definição de `12`
  e uma lista do que verificar, tirada do próprio texto do princípio (ex.:
  Informado verifica informação decisiva escondida, letras miúdas e falta de
  limite ou saldo na hora de definir um valor).
- **Tela única ou jornada:** com uma tela, Consistente verifica a tela e os
  padrões da plataforma. Com várias telas, compara as telas entre si (ação
  principal, títulos, termos, ícones, componentes para a mesma função e tom).
  A referência é a primeira tela em que o padrão aparece; o apontamento fica
  na tela que diverge. A jornada ganha uma lista de **inconsistências entre
  telas**, agrupada por padrão.
- **Relação com os outros assuntos:** um problema que já está em Usuário ou
  Heurísticas e também fere um princípio é registrado em Princípios citando o
  id ("ver H2"). As notas são separadas, então não há dupla penalização.
- **Severidade:** crítica (quebra a confiança ou impede uma decisão segura),
  alta (contraria claramente o princípio num elemento central), média
  (elemento secundário ou parcial) e baixa (desvio pequeno).
- **Nota**, no mesmo modelo de pesos do Design System: cada tela tem 5
  unidades, uma por princípio; cada princípio recebe correto ou a severidade
  do pior apontamento dele na tela.

  | Resultado | Peso |
  |---|---|
  | Correto | 1,00 |
  | Baixa | 0,75 |
  | Média | 0,50 |
  | Alta | 0,25 |
  | Crítica | 0,00 |

  `nota_pd_tela = 10 × (soma dos pesos dos 5 princípios ÷ 5)`; a da jornada é
  a média das telas. Exemplo: um apontamento alto em Consistente e um médio
  em Informado dão 10 × (1 + 0,50 + 1 + 0,25 + 1) ÷ 5 = 7,5.
- A nota fica separada da nota da jornada e os apontamentos ficam fora das
  prioridades, como no Design System.
- O resultado completo vai para o frame "Validador · Princípios de Design";
  o relatório mostra só a nota.

## Quem pode usar
- Usuário com **assento Full**. Assentos View, Dev e Collab só usam o agente em
  Drafts.
- Permissão de **edição** no arquivo, porque o agente escreve anotações.
- Créditos de IA do Figma disponíveis. O consumo varia com o tamanho da
  jornada e com o que o agente processa.

## Instalação e distribuição
1. No chat do agente, em **Skills → Add skill**, enviar o arquivo
   `Skill/validador-de-jornada.md`.
2. Para a equipe, publicar a skill como **privada** para o time ou para a
   organização (**Manage skills → Publish**).
3. Cada nova versão do arquivo é publicada com **Publish changes**.

## Fluxo do usuário
1. Separa a jornada numa página própria do Figma (regra de `07`) e abre essa
   página.
2. Abre o agente e chama `/validador-de-jornada`. A primeira resposta é a
   pergunta "Qual será a validação da tela/jornada?" e o usuário escolhe uma
   ou mais opções.
3. Conforme a escolha, informa o objetivo da tarefa (opções 1 e 2) e anexa o
   arquivo da persona com o produto dela e a origem (pesquisa, pesquisa + IA
   ou só IA) (opção 1).
4. O agente confirma a página, a ordem dos frames, o objetivo, o modo e os
   avaliadores. Com Design System, lista as bibliotecas ativadas no arquivo
   e o usuário marca quais valem como referência.
5. Com persona:
   - o agente mostra a persona normalizada e o que não foi mapeado, e espera a
     confirmação;
   - o agente mostra a assertividade e o feedback campo a campo;
   - o agente compara o dispositivo da persona com o formato das telas e,
     se não baterem, avisa e pergunta se segue;
   - abaixo de 40%, o teste é bloqueado: o usuário anexa a persona enriquecida
     ou troca a opção 1 pela 2.
6. O agente avalia tela a tela, marca possíveis abandonos sem parar a
   avaliação e consolida a jornada (com Princípios, também as
   inconsistências entre telas).
7. O agente remove as anotações "Validador · …" de testes anteriores da página
   e escreve as novas: uma por problema e um resumo no primeiro frame.
8. O agente cria, da esquerda para a direita e substituindo os anteriores:
   "Validador · Relatório" à direita da jornada; "Validador · Persona" (com
   persona); "Validador · Design System" (com Design System ativo); e
   "Validador · Princípios de Design" (com Princípios).
9. O agente responde no chat com o resumo do resultado. O relatório em JSON é
   entregue quando o usuário pedir.

## Resultado
- **Anotações:** anotações do Dev Mode separadas por assunto:

  | Categoria | Cor | Conteúdo |
  |---|---|---|
  | Validador · Usuário | roxo | Problemas U, com a reação da persona |
  | Validador · Heurísticas | azul | Problemas H, com observação técnica |
  | Validador · Design System | verde | Apontamentos DS (detach, tokens, componentes locais), com o componente, estilo ou variável da biblioteca a usar |
  | Validador · Princípios de Design | rosa | Apontamentos PD, com o princípio, o que foi encontrado, a tela de referência (em inconsistências) e o ajuste |
  | Validador · Resumo | cinza | Resumo no primeiro frame, com contagens por assunto e as notas de Design System e de Princípios de Design |

  A severidade aparece no prefixo do texto: 🔴 crítica, 🟠 alta, 🟡 média,
  🔵 baixa.
- **Frames:** "Validador · Relatório", "Validador · Persona", "Validador ·
  Design System" e "Validador · Princípios de Design" (ver abaixo).
- **No chat:** modo, persona, objetivo, assertividade, nota da jornada,
  objetivo concluído, ponto de abandono, 3 primeiras prioridades com assunto,
  nota por tela, resultado do Design System, resultado de Princípios de
  Design, anotações por assunto e onde ficaram os frames.
- Rodar de novo na mesma página substitui as anotações "Validador · …" e os
  frames do Validador, e não mexe em mais nada. Um novo teste sem persona
  apaga o frame "Validador · Persona" anterior; sem Design System, apaga o
  "Validador · Design System"; sem Princípios, apaga o "Validador ·
  Princípios de Design"; sem Heurísticas, apaga o "Validador · Relatório".

## Frame de relatório
Um frame chamado **"Validador · Relatório"**, criado na página da jornada, que
organiza o resultado para o designer saber o que ajustar.

### Regras
- Ele e os frames "Validador · Persona", "Validador · Design System" e
  "Validador · Princípios de Design" são as **únicas camadas** que a skill
  cria. As camadas do designer continuam intocadas.
- Fica à direita de todos os frames da página, a 200 px, alinhado ao topo do
  primeiro frame, sem sobrepor nada. Largura de 1200 px; altura pelo conteúdo.
- Ao rodar de novo, o frame anterior com esse nome é apagado e um novo é
  criado.
- **Estilo fixo do Validador**, igual em qualquer arquivo e nos quatro frames
  (Relatório, Persona, Design System e Princípios de Design). Não usa estilos, variáveis nem
  componentes das bibliotecas do arquivo, por decisão, para o resultado não
  variar entre testes.
  - Fonte Inter: títulos em Bold (frame 32, seção 22, assunto 18 na cor do
    assunto, bloco 16), corpo Regular 14, rótulos Semi Bold 12.
  - Cores dos assuntos: Usuário `#6E3FD6`, Heurísticas `#0B5CD5`, Design
    System `#1E7F4F`, Princípios de Design `#B8336A`. Severidades: crítica `#C9372C`, alta `#D35F00`, média
    `#8A6100`, baixa `#2C6ECB`, cada uma com fundo claro na tag.
  - Fundo geral branco; todo conteúdo em blocos cinza `#F4F5F7` (padding 24,
    raio 12); divider de 1 px `#DFE1E6` entre seções e entre assuntos; itens
    com barra de 4 px na cor do assunto.
  - Modelo de referência: frame "Validador · Modelo de estilo" no arquivo
    [Template-Validador-Jornada](https://www.figma.com/design/zpNptFjwXaKm52Ztcp1rID/Template-Validador-Jornada?node-id=2-2).
    Os valores exatos ficam na skill.
- Sem gráficos, porque o agente não cria visualizações de dados no Figma
  Design. Notas aparecem como números e, opcionalmente, barras simples.
- Todo texto é camada de texto editável.
- Se o agente não conseguir criar o frame, o mesmo conteúdo vai para o chat.

### Seções
| Ordem | Seção | Conteúdo |
|---|---|---|
| 1 | Cabeçalho | Título com o nome da página, modo, persona (`{produto} — {nome}`) ou "Somente heurísticas", objetivo da tarefa, data, assertividade, confiança e compatibilidade do dispositivo. |
| 2 | Alerta de abandono | Só com possível abandono: faixa vermelha "⛔ POSSÍVEL ABANDONO DE JORNADA", com tela, motivo, problema, reação da persona e ajuste. |
| 3 | Placar | Nota da jornada em destaque, objetivo concluído, ponto de abandono e um bloco por assunto: Usuário (nota da persona e contagem por severidade), Heurísticas (nota e contagem), Design System (nota de uso e aviso de que os detalhes estão no frame "Validador · Design System", ou "Não avaliado" com o motivo) e Princípios de Design (nota, contagem e aviso de que os detalhes estão no frame "Validador · Princípios de Design"). |
| 4 | Plano de ajuste | Checklist de Usuário e de Heurísticas na ordem das prioridades (problema de abandono primeiro, com tag; críticos e altos; médios e baixos em "Ajustes de refinamento"). |
| 5 | Tela a tela | Um card por tela: notas (incluindo as de Design System e de Princípios de Design), ids dos problemas de Usuário e de Heurísticas e pontos positivos. Tela com possível abandono em destaque; telas seguintes marcadas. |
| 6 | Resumo | Com persona: resumo em primeira pessoa e aviso de que os dados estão no frame "Validador · Persona". Sem persona: resumo técnico. |
| 7 | Como aumentar a confiança | Só com persona e assertividade abaixo de 85%: feedback campo a campo de `03`. |
| 8 | Como as notas foram calculadas | Desconto de cada problema de Heurísticas (severidade × fator do grupo × 0,5), `nota_persona`, fórmula da nota da tela com os pesos da origem da persona e nota da jornada. |

## Frame da persona
Um frame chamado **"Validador · Persona"**, criado só nos testes com persona,
para o designer saber por quais olhos a jornada foi avaliada.

- Fica à direita do frame de relatório, a 200 px, alinhado ao topo dele, sem
  sobrepor nada. Largura de 800 px; altura pelo conteúdo. Mesmo estilo fixo
  do relatório, com destaques na cor de Usuário.
- Usa o texto da persona confirmada na normalização (`02`), sem resumir nem
  completar. Campo vazio aparece como "Não informado".

| Ordem | Seção | Conteúdo |
|---|---|---|
| 1 | Identificação | `{produto} — {nome}`, origem da persona, idade, ocupação, nível digital, dispositivo principal e contexto de uso. |
| 2 | Assertividade | Percentual, confiança e a classificação de cada campo (VAZIO, RASO, ADEQUADO ou RICO) com o peso de `01`. |
| 3 a 7 | Campos | Comportamentos, Dores e frustrações, Contexto de uso, Critérios de sucesso e Conhecimento prévio. |
| 8 | Não mapeado | Trechos que não couberam nos campos, com o motivo. |

## Frame de Design System
Um frame chamado **"Validador · Design System"**, criado só quando o avaliador
Design System roda, com o resultado completo do uso da biblioteca.

- Fica à direita do último frame do Validador (persona ou, sem persona,
  relatório), a 200 px, alinhado ao topo dele, sem sobrepor nada. Largura de
  1200 px; altura pelo conteúdo. Mesmo estilo fixo do relatório, com
  destaques na cor de Design System.

| Ordem | Seção | Conteúdo |
|---|---|---|
| 1 | Cabeçalho | Título com o nome da página, data e a lista exata das bibliotecas de referência marcadas. |
| 2 | Nota de uso do Design System | Nota da jornada em destaque, unidades avaliadas, corretas (com quantos wrappers locais) e apontamentos por severidade. |
| 3 | Nota por tela | Um card por tela: nota, unidades avaliadas, corretas e ids dos apontamentos. |
| 4 | Apontamentos | Agrupados por tipo, na ordem: detach, falta de tokens, componentes locais com similar, locais sem tokens e locais com tokens. Cada linha com id, severidade, tela, elemento, o que foi encontrado e ajuste. |
| 5 | Inventário por tela | Todas as unidades, numeradas, com o resultado de cada uma. |
| 6 | Como ler a nota | Tabela de pesos, fórmula com os números do teste e regra de contagem. |

## Frame de Princípios de Design
Um frame chamado **"Validador · Princípios de Design"**, criado só quando o
avaliador Princípios de design roda, com **só o que está fora dos
princípios**.

- Fica à direita do último frame do Validador criado no teste, a 200 px,
  alinhado ao topo dele; se for o único frame do Validador, à direita da
  jornada. Largura de 1200 px; altura pelo conteúdo. Mesmo estilo fixo do
  relatório, com destaques na cor de Princípios de Design.

| Ordem | Seção | Conteúdo |
|---|---|---|
| 1 | Cabeçalho | Título com o nome da página, data, "Tela única" ou "Jornada com {n} telas" e os princípios avaliados. |
| 2 | Nota de Princípios de Design | Nota da jornada em destaque, apontamentos por severidade e um bloco por princípio com a quantidade de apontamentos. |
| 3 | Inconsistências entre telas | Só com várias telas: cada padrão, a tela de referência, as telas que divergem e os ids. |
| 4 | Apontamentos por princípio | Só os princípios com apontamento; cada linha com id, severidade, tela, elemento, o que foi encontrado, tela de referência, relacionado e ajuste. |
| 5 | Nota por tela | Um card por tela: nota, princípios fora (com severidade) e ids. |
| 6 | Como ler a nota | Tabela de pesos, regra das 5 unidades por tela e a conta com os números do teste. |

## Limitações desta versão
- **Sem histórico:** cada resultado fica só no arquivo e no chat. Não há
  comparação entre testes.
- **Sem biblioteca de personas:** a persona é anexada a cada teste. A
  assertividade é calculada de novo a cada vez.
- **Design System depende da biblioteca no arquivo:** o avaliador só roda se
  houver uma biblioteca de referência ativada. A avaliação é do uso da
  biblioteca no Figma, não da fidelidade ao código.
- **Resultado menos estável:** o agente não é determinístico. A mesma jornada
  pode ter variações de nota e de problemas entre rodadas.
- **Sem link navegável:** só jornadas do próprio arquivo do Figma. Por isso
  os princípios de motion (`13`) ainda não são avaliados.

## Pontos em aberto (validar com o Garba)
- Quanto um teste consome de créditos, medido nos primeiros testes com 3 a 5
  telas.
- Se o agente consegue criar as categorias de anotação com cor. Se não
  conseguir, o assunto vai no rodapé do texto ("[Validador de Jornada ·
  Usuário]") e a severidade no prefixo.
- Se existe limite de tamanho para a skill. Hoje o arquivo tem cerca de 60 KB,
  com o catálogo de heurísticas resumido.
- A lista "o que verificar" de cada princípio de design foi tirada do texto
  dos princípios (`12`) e é uma base inicial: validar com o time de design
  nos primeiros testes, principalmente Encantador, que é o mais subjetivo.
- Estabilidade da nota de Princípios de Design entre rodadas: repetir o teste
  da mesma jornada e comparar.
- Estabilidade da nota de Design System: dois testes da mesma tela (Template
  Estudo, 26/10/2026) deram 9,3 e 8,6 porque o Header e o Menu (wrappers
  locais) foram classificados de formas diferentes e as bibliotecas de
  referência não eram as mesmas. As regras de wrapper, contagem e lista de
  bibliotecas foram fixadas; repetir o teste para confirmar que a nota
  coincide.
- Precisão do agente para detectar **detach**: o Figma não guarda a origem de
  um componente desvinculado, então a skill identifica pelo nome e pela
  estrutura. Validar falsos positivos nos primeiros testes.
- Se o agente consegue ler as variáveis e os estilos ligados a cada camada
  (incluindo espaçamento e raio) para apontar a falta de tokens.
- Se a skill é publicada para um time ou para a organização inteira.
- Os pesos 60/40, 70/30 e 80/20 e os fatores dos grupos são uma base
  experimental: comparar os achados com testes com usuários reais e
  recalibrar.

## Critérios de validação desta etapa
- [ ] Usuário não informa nenhuma chave; o teste consome os créditos do Figma
      de quem roda.
- [ ] A skill é um único `.md` e roda pelo comando `/validador-de-jornada`.
- [ ] A primeira mensagem é a pergunta de múltipla escolha da validação;
      combinações e "Todas" funcionam, 1 e 2 não se combinam, não existe
      validação só de persona e só rodam os avaliadores escolhidos.
- [ ] Objetivo da tarefa obrigatório com a opção 1 ou 2; só Design System
      e/ou Princípios de design não pedem objetivo nem criam o frame de
      relatório.
- [ ] Persona normalizada sem inventar dados e confirmada no chat antes de
      seguir.
- [ ] Assertividade calculada pela fórmula de `03`; abaixo de 40% bloqueia.
- [ ] Todas as heurísticas do catálogo avaliadas em toda tela.
- [ ] Nenhuma camada do usuário é alterada.
- [ ] Uma anotação por problema, presa ao elemento, na categoria do assunto
      (Usuário, Heurísticas, Design System ou Princípios de Design), e um
      resumo no primeiro frame.
- [ ] Com persona, nenhum problema aparece em Usuário e em Heurísticas ao
      mesmo tempo; sem persona, não há Usuário.
- [ ] Persona de celular com telas de desktop (ou o contrário) gera aviso
      antes da avaliação e o problema U1, crítico ou alto conforme a regra.
- [ ] Possível abandono gera problema crítico, não interrompe a avaliação e
      aparece com "⛔ POSSÍVEL ABANDONO DE JORNADA" nas anotações, no resumo,
      no relatório e no chat.
- [ ] Dois testes da mesma tela, com as mesmas bibliotecas marcadas, dão a
      mesma nota de uso do Design System e o mesmo inventário.
- [ ] Design System usa as bibliotecas marcadas pelo usuário, aponta detach, falta de tokens e componentes locais com as
      severidades definidas e pode ser pulado.
- [ ] Nota de uso do Design System por tela e da jornada, separada da nota da
      jornada; Design System fora das prioridades.
- [ ] Frame "Validador · Relatório" criado à direita da jornada, sem sobrepor
      camadas, com as 7 seções.
- [ ] Frame "Validador · Persona" criado só com persona, à direita do
      relatório, com os dados da persona confirmada.
- [ ] Frame "Validador · Design System" criado só com Design System ativo,
      à direita do último frame do Validador.
- [ ] Princípios de design avaliados em tela única e em jornada; o frame
      "Validador · Princípios de Design" mostra só o que está fora dos
      princípios e, com várias telas, as inconsistências entre telas.
- [ ] Nota de Princípios de Design com 5 unidades por tela e os pesos do
      Design System, separada da nota da jornada e fora das prioridades.
- [ ] Os quatro frames seguem o estilo fixo do Validador (Inter, cores dos
      assuntos e severidades, blocos cinza, dividers) e saem iguais em testes
      diferentes e em arquivos diferentes.
- [ ] Rodar de novo não duplica anotações nem frames, e não altera anotações
      de terceiros.
- [ ] Testado em 2 ou 3 jornadas reais, nos dois modos.
