# 05 — Prompts Mestres

Prompts na ordem em que rodam. Variáveis entre `{chaves}` são preenchidas pelo
back-end. Todos os prompts são versionados (ex.: `p1-v1`).

## Quais prompts rodam em cada modo
| Prompt | Persona + heurísticas | Somente heurísticas |
|---|---|---|
| 1 — Normalização | Sim | Não |
| 2 — Profundidade | Sim | Não |
| 3A — Simulação da persona | Sim | Não |
| 3B — Avaliação heurística | Não | Sim |
| 4 — Consolidação da jornada | Sim (variante com persona) | Sim (variante sem persona) |

## Os três truques que fazem funcionar (modo com persona)
1. **Ordem forçada persona → heurística:** impede que o modelo vire um consultor
   genérico e garante os 70% da persona.
2. **Escala de severidade fixa e ancorada:** garante comparabilidade entre rodadas.
3. **Primeira pessoa na reação:** "eu não entendi onde clicar" em vez de "o
   usuário pode se confundir". Fica concreto e acionável.

---

## Prompt 1 — Normalização do input (`02`)
```
Você recebe um documento de persona em formato livre. Reescreva-o no modelo
ideal abaixo, mapeando cada informação para o campo correto.

REGRAS:
- Use SOMENTE informação presente no documento original.
- Se um campo não tiver correspondência, deixe-o vazio ("").
- Não infira, não complete, não embeleze.
- Preserve a intenção original; apenas reorganize.
- Para cada campo preenchido, registre o trecho original de onde veio.
- Toda informação que não couber em nenhum campo vai para "nao_mapeado".
- A persona NÃO tem campo de objetivo de tarefa. Se o documento trouxer um
  objetivo de tarefa (o que ela quer concluir no produto), coloque o trecho em
  "nao_mapeado" com o motivo "objetivo de tarefa: informe no campo Objetivo
  principal da tarefa". Preferências que valem para qualquer tarefa vão para
  "criterios_sucesso".
- Deixe "produto" vazio no frontmatter; ele é preenchido pelo sistema.

MODELO IDEAL:
{template de 01-modelo-ideal-persona.md}

DOCUMENTO DO USUÁRIO:
{conteúdo enviado}

Responda SOMENTE em JSON:
{
  "persona_md": "persona completa no template do modelo ideal",
  "campos": {
    "comportamentos": { "conteudo": "", "origem": "" },
    "dores_frustracoes": { "conteudo": "", "origem": "" },
    "contexto_uso": { "conteudo": "", "origem": "" },
    "nivel_digital": { "conteudo": "", "origem": "" },
    "conhecimento_previo": { "conteudo": "", "origem": "" },
    "criterios_sucesso": { "conteudo": "", "origem": "" },
    "demograficos": { "conteudo": "", "origem": "" }
  },
  "nao_mapeado": []
}
```

---

## Prompt 2 — Avaliação de profundidade (`03`)
Roda uma vez por campo preenchido.
```
Você avalia a QUALIDADE de um campo de persona, não o conteúdo em si.

CAMPO: {nome do campo}
O QUE ESTE CAMPO DEVE RESPONDER: {descrição do campo em 01}
CONTEÚDO: {texto do campo}

Avalie em três eixos (0 a 10 cada):
- Especificidade: é concreto e detalhado, ou genérico?
- Acionabilidade: dá para simular comportamento real a partir disso?
- Singularidade: descreve ESTA persona, ou serviria para qualquer uma?

Texto longo não vale mais por ser longo. Se o conteúdo serviria para qualquer
persona, a singularidade deve ser baixa, independente do tamanho.

Calcule a média e classifique:
- 0 a 3,9: RASO
- 4 a 6,9: ADEQUADO
- 7 a 10: RICO

Responda SOMENTE em JSON:
{
  "campo": "",
  "especificidade": 0,
  "acionabilidade": 0,
  "singularidade": 0,
  "media": 0,
  "classificacao": "RASO|ADEQUADO|RICO",
  "sugestao": "o que falta para subir de nível, em uma frase"
}
```

---

## Prompt 3A — Simulação da persona, uma tela por vez (`04`)
Só no modo persona + heurísticas.
```
Você é um avaliador de experiência que SIMULA uma persona específica navegando
por uma interface. Você NÃO dá opinião genérica de especialista: você reage
como ESTA persona reagiria e só então usa as heurísticas para nomear
tecnicamente cada problema.

## PERSONA QUE VOCÊ ESTÁ SIMULANDO
{persona normalizada e confirmada}

## OBJETIVO DA TAREFA (o que a persona está tentando concluir)
{objetivo_tarefa}

## ONDE ELA ESTÁ NA JORNADA
Tela {ordem} de {total}: {nome da tela}
Resumo das telas anteriores: {reações e problemas já encontrados}

## COMO AGIR
1. Olhe a tela pelos olhos da persona, tentando cumprir o objetivo da tarefa,
   com o contexto e os comportamentos dela.
2. Pergunte-se: "O que EU tentaria fazer aqui? Onde eu travaria? O que me faria
   desistir?"
3. SÓ DEPOIS de identificar a reação, conecte o problema a uma das heurísticas
   do catálogo. Se nenhuma se aplicar, não registre o problema.
4. Não aponte problemas que esta persona não perceberia ou que não a afetariam.

## CATÁLOGO DE HEURÍSTICAS (a régua técnica — todas valem)
{catálogo completo: grupo, código, nome e definição de cada heurística}

## ESCALA DE SEVERIDADE
- critica (4): impede a persona de concluir o objetivo da tarefa.
- alta (3): conclui com muito esforço ou quase desiste.
- media (2): gera atrito ou dúvida, mas segue.
- baixa (1): cosmético; não muda o comportamento dela.

## TELA PARA AVALIAR
{imagem da tela}
{árvore de elementos com node_id, texto e coordenadas}

## TAREFA
Para cada problema:
1. reacao_persona: em primeira pessoa, como ela sentiria.
2. heuristica: grupo, código e nome.
3. severidade (e o número na escala).
4. localizacao: elemento, node_id e coordenadas da árvore.
5. sugestao: a correção.

Dê também:
- nota_persona (0 a 10): chance de ela atravessar esta tela avançando no
  objetivo da tarefa, sem atrito.
- nota_heuristica (0 a 10): aderência a todas as heurísticas do catálogo.
- pontos_positivos: o que funcionou para ela.

Responda SOMENTE no formato JSON "avaliação de uma tela" de
04-backend-arquitetura.md. Não escreva texto fora do JSON.
```

---

## Prompt 3B — Avaliação heurística, uma tela por vez (`04`)
Só no modo somente heurísticas. Não há persona e não há regra 70/30.
```
Você é um especialista em usabilidade fazendo uma avaliação heurística de uma
interface. Avalie a tela contra TODAS as heurísticas do catálogo abaixo.

## OBJETIVO DA TAREFA (o que o usuário está tentando concluir)
{objetivo_tarefa}

## ONDE A TELA ESTÁ NA JORNADA
Tela {ordem} de {total}: {nome da tela}
Resumo das telas anteriores: {problemas já encontrados}

## COMO AGIR
1. Considere o que o usuário precisa fazer nesta tela para avançar no
   objetivo da tarefa.
2. Percorra a tela verificando cada heurística do catálogo.
3. Registre só problemas ligados a uma heurística do catálogo.

## CATÁLOGO DE HEURÍSTICAS (todas valem)
{catálogo completo: grupo, código, nome e definição de cada heurística}

## ESCALA DE SEVERIDADE
- critica (4): impede o usuário de concluir o objetivo da tarefa.
- alta (3): o usuário conclui com muito esforço.
- media (2): gera atrito ou dúvida, mas o usuário segue.
- baixa (1): cosmético; não afeta a tarefa.

## TELA PARA AVALIAR
{imagem da tela}
{árvore de elementos com node_id, texto e coordenadas}

## TAREFA
Para cada problema:
1. observacao: descrição técnica do problema.
2. heuristica: grupo, código e nome.
3. severidade (e o número na escala).
4. localizacao: elemento, node_id e coordenadas da árvore.
5. sugestao: a correção.

Dê também:
- nota_heuristica (0 a 10): aderência a todas as heurísticas do catálogo.
- pontos_positivos: o que a tela faz bem em relação às heurísticas.
- persona e nota_persona: null.
- nota_tela: igual a nota_heuristica.

Responda SOMENTE no formato JSON "avaliação de uma tela" de
04-backend-arquitetura.md (variante sem persona). Não escreva texto fora do JSON.
```

---

## Prompt 4 — Consolidação da jornada (`04`)
### Variante com persona
```
Você recebe as avaliações de todas as telas de uma jornada, feitas simulando a
persona abaixo. Consolide a visão da jornada inteira.

PERSONA: {persona normalizada}
OBJETIVO DA TAREFA: {objetivo_tarefa}
AVALIAÇÕES DAS TELAS: {lista de JSONs do Prompt 3A}

TAREFA:
- objetivo_concluido: a persona concluiria o objetivo da tarefa? (sim | com_dificuldade | nao)
- ponto_de_abandono: se não concluísse, em que tela e por quê.
- resumo_persona: até 5 frases, em primeira pessoa, sobre a jornada.
- prioridades: ids dos problemas, do mais para o menos urgente, considerando
  severidade e o quanto cada um afasta a persona do objetivo.
- Não crie problemas novos; use só os que vieram nas avaliações.

Responda SOMENTE no formato JSON "relatório da jornada" de
04-backend-arquitetura.md. Não escreva texto fora do JSON.
```

### Variante sem persona (somente heurísticas)
```
Você recebe as avaliações heurísticas de todas as telas de uma jornada.
Consolide a visão da jornada inteira.

OBJETIVO DA TAREFA: {objetivo_tarefa}
HEURÍSTICAS AVALIADAS: catálogo completo ({catalogo_versao})
AVALIAÇÕES DAS TELAS: {lista de JSONs do Prompt 3B}

TAREFA:
- objetivo_concluido: o usuário conseguiria concluir o objetivo da tarefa?
  (sim | com_dificuldade | nao)
- ponto_de_abandono: se não conseguisse, em que tela e por quê.
- resumo_heuristico: até 5 frases sobre a aderência da jornada às heurísticas.
- prioridades: ids dos problemas, do mais para o menos urgente, por severidade
  e por quantas telas o problema afeta.
- persona, produto e resumo_persona: null.
- assertividade: 100; confianca: "alta"; modo: "somente_heuristicas".
- Não crie problemas novos; use só os que vieram nas avaliações.

Responda SOMENTE no formato JSON "relatório da jornada" de
04-backend-arquitetura.md (variante sem persona). Não escreva texto fora do JSON.
```

---

## Teste manual antes do back-end
### Modo persona + heurísticas
1. Salvar a persona de exemplo de `01` em `Personas/`.
2. Rodar o Prompt 1 com essa persona escrita em texto corrido e comparar com o
   template.
3. Rodar o Prompt 2 nos campos e conferir a assertividade calculada.
4. Rodar o Prompt 3A em 2 ou 3 telas reais do Figma, com um objetivo de
   tarefa definido (ex.: "Contratar um seguro em menos de 3 minutos").
5. Salvar os JSONs em `Resultados/` e revisar: a reação parece da persona? A
   heurística foi bem aplicada? A severidade faz sentido?

### Modo somente heurísticas
1. Rodar o Prompt 3B nas mesmas telas e com o mesmo objetivo, sem persona.
2. Comparar com o resultado do modo com persona: o 3B deve trazer problemas
   técnicos sem reação em primeira pessoa, e o 3A deve priorizar o que afeta a
   persona.

## Critérios de validação desta etapa
- [ ] Prompts versionados e testados manualmente antes do back-end.
- [ ] Modo somente heurísticas roda só o 3B e o 4 (variante sem persona).
- [ ] Normalização não inventa dados (Prompt 1).
- [ ] Objetivo da tarefa vem da tela principal em 3A, 3B e 4; nunca da persona.
- [ ] Profundidade estável: mesmo campo, 3 rodadas, média varia no máximo 1 ponto.
- [ ] Simulação valida o conceito no chat antes de qualquer código.
- [ ] Todas as saídas são JSON válido no formato de `04`.
