# 04 — Arquitetura do Back-end

## Visão das camadas
Modo persona + heurísticas, com persona nova:
```
Front ─▶ Entrada ─▶ Normalização ─▶ Assertividade ─▶ Biblioteca ─▶ Captura ─▶ Processamento ─▶ Saída
                         ▲                │
                         └─ confirmação ──┘ (usuário)
```
Modo persona + heurísticas, com persona da biblioteca:
```
Front ─▶ Entrada ─▶ Biblioteca (persona + assertividade salvas) ─▶ Captura ─▶ Processamento ─▶ Saída
```
Modo somente heurísticas:
```
Front ─▶ Entrada ─▶ Captura ─▶ Processamento ─▶ Saída
```

## 1. Camada de Entrada (API)
- Recebe:
  - chave de API da IA do usuário (obrigatória, em cabeçalho da requisição;
    regras em `09`);
  - link (obrigatório);
  - `objetivo_tarefa` (obrigatório, texto livre);
  - opcionalmente, **um** destes: arquivo de persona nova + `produto_id`, ou
    `persona_id` de uma persona da biblioteca.
- Não recebe lista de heurísticas: todo teste usa o **catálogo completo** (`00`).
  O teste grava a versão do catálogo usada (`catalogo_versao`), para manter a
  comparação entre testes.
- A chave é usada em todas as chamadas de IA daquele teste (Prompts 1 a 4),
  fica só em memória durante a execução e é descartada ao final. Nunca é
  gravada em banco, em log nem no relatório.
- Recusa o pedido se faltar chave, link ou objetivo, ou se vierem arquivo e
  `persona_id` ao mesmo tempo.
- Define o modo do teste:
  - com persona (nova ou da biblioteca) → `modo: persona_heuristicas`;
  - sem persona → `modo: somente_heuristicas`.
- Persona da biblioteca: lê a assertividade salva. Abaixo de 40%, recusa o
  teste (status `bloqueado`); a partir de 40%, status `pronto_para_captura`.
- Identifica o tipo de link:
  - `figma.com/design/...` ou `figma.com/proto/...` → fluxo Figma.
  - Qualquer outra URL `https` → fluxo navegável (pós-MVP).
- Valida o acesso ao link antes de seguir (token do Figma ou resposta da URL).
  No Figma, verifica as conexões disponíveis e define o canal de saída
  (`canal_figma`) pela ordem de `07`.
- Recebe também, opcionalmente, `validar_orquestra` com `stack` e `versao`
  (`08`). Recusa o pedido se vier ligado sem stack ou versão.
- Persona nova: converte o arquivo em texto, dispara a normalização (`02`) e
  cria o teste com status `aguardando_confirmacao`.
- Modo somente heurísticas: grava `assertividade: 100` e cria o teste com
  status `pronto_para_captura`.

## 2. Camada de Normalização e Assertividade
Só para persona nova.
- Roda o Prompt 1 (`05`) e devolve a persona normalizada para confirmação.
- Após a confirmação, roda o Prompt 2 (`05`) campo a campo e calcula a
  assertividade (`03`).
- Salva a persona na biblioteca com produto, assertividade e avaliação por
  campo, qualquer que seja a nota.
- Abaixo de 40%: status `bloqueado`. A API recusa qualquer pedido de captura ou
  processamento de teste bloqueado; não existe forma de forçar.
- Teste bloqueado só sai desse status de três jeitos: com uma persona
  reenviada (volta para `aguardando_confirmacao`), com a troca por outra
  persona da biblioteca a partir de 40%, ou com a remoção da persona (muda
  para `somente_heuristicas` e `pronto_para_captura`).
- A partir de 40%: status `pronto_para_captura`.

## Biblioteca de personas
Regras de tela em `06`. Visível para todos os usuários da plataforma.

### Endpoints
| Ação | Endpoint | Regra |
|---|---|---|
| Listar produtos | `GET /produtos` | Ordem alfabética. |
| Criar produto | `POST /produtos` | Nome obrigatório e único (sem diferenciar maiúsculas). |
| Listar personas | `GET /personas?produto_id=` | Agrupadas por produto; traz nome de exibição, produto, assertividade, se está habilitada e se o usuário atual pode excluir. |
| Baixar persona | `GET /personas/{id}/download` | Devolve o `.md` **normalizado** no template de `01`. |
| Excluir persona | `DELETE /personas/{id}` | Só quem criou (`criado_por`); outro usuário recebe 403. Exclusão lógica; some da lista e do dropdown. |

### Exclusão
- Pede confirmação na tela antes de excluir.
- Testes já feitos com a persona **não mudam**: cada teste guarda uma cópia da
  persona usada (`persona_snapshot`), e o relatório continua abrindo.
- Persona excluída não pode ser escolhida em testes novos.

### Modelo de dados (Supabase)
| Tabela | Campos principais |
|---|---|
| `produtos` | `id`, `nome`, `criado_por`, `criado_em` |
| `personas` | `id`, `produto_id`, `nome`, `nome_exibicao` (`{produto} — {nome}`), `persona_md` (normalizada), `original_texto`, `assertividade`, `avaliacao_campos` (JSON do Prompt 2), `criado_por`, `criado_em`, `excluida_em` |
| `testes` | `id`, `modo`, `link`, `objetivo_tarefa`, `catalogo_versao`, `validar_orquestra`, `orquestra_stack`, `orquestra_versao`, `canal_figma`, `figma_itens_ids` (ids dos comentários/anotações escritos), `persona_id`, `persona_snapshot`, `assertividade`, `status`, `criado_por`, `criado_em` |

## 3. Camada de Captura
### Figma (MVP)
- Puxa os frames da jornada pela API do Figma: imagem de cada tela + árvore de
  elementos (nome, tipo, texto, posição e tamanho).
- A ordem da jornada segue os fluxos do protótipo; sem fluxo definido, segue a
  ordem dos frames na página.
- Componentes do design system ficam identificados na árvore, para a análise
  separar problema de componente de problema de uso.

### Link navegável (pós-MVP)
- Navegador headless (Playwright) percorre a jornada e tira screenshot de cada
  passo, guardando também a árvore de acessibilidade da página.
- Links atrás de login exigem credencial de teste informada pelo usuário.

### Regra comum
Toda tela capturada vira: `imagem + estrutura + posição na jornada`. Imagem
avalia o visual; estrutura dá a lógica e as coordenadas das marcações.

## 4. Camada de Processamento
- Avalia **uma tela por vez**:
  - modo persona + heurísticas: Prompt 3A (`05`) com objetivo da tarefa +
    persona normalizada + catálogo completo de heurísticas + tela;
  - modo somente heurísticas: Prompt 3B (`05`) com objetivo da tarefa +
    catálogo completo de heurísticas + tela.
- Com `validar_orquestra` ligado, valida os componentes de cada tela pelo MCP
  do Orquestra (`08`), em paralelo à avaliação de heurísticas.
- Depois de todas as telas, roda o Prompt 4 (`05`) para consolidar a jornada.
- Exige saída em JSON; resposta fora do formato é rejeitada e reprocessada
  (até 2 tentativas).
- Se o provedor recusar a chave no meio do teste, o teste para com status
  `erro_chave` (`09`).

## 5. Camada de Saída
- Persiste tudo em banco (Supabase): teste, persona normalizada, assertividade,
  telas, problemas e relatório da jornada.
- Gera as marcações na interface a partir das coordenadas dos problemas.
- Link do Figma: escreve o resultado no próprio arquivo, preso aos elementos
  da página enviada, sem duplicar telas. Canal por prioridade: comentários do
  Figma → anotações do Dev Mode → só plataforma (regras em `07`).
- Na validação manual (antes do back-end), os relatórios ficam em `Resultados/`.

## Escala de severidade
Alinhada à escala de Nielsen usada nas skills de heurística (0 a 4).

| Severidade | Nielsen | Critério (com persona) | Critério (somente heurísticas) |
|---|---|---|---|
| crítica | 4 | Impede a persona de concluir o objetivo da tarefa. | Impede o usuário de concluir o objetivo da tarefa. |
| alta | 3 | A persona conclui com muito esforço ou quase desiste. | O usuário conclui com muito esforço. |
| média | 2 | Gera atrito ou dúvida, mas a persona segue. | Gera atrito ou dúvida, mas o usuário segue. |
| baixa | 1 | Cosmético; não muda o comportamento da persona. | Cosmético; não afeta a tarefa. |

## Nota por modo
### Modo persona + heurísticas (70/30)
Cada tela recebe duas notas de 0 a 10:
- `nota_persona`: chance de a persona atravessar a tela avançando no objetivo
  da tarefa, sem atrito, pelos critérios de sucesso dela.
- `nota_heuristica`: aderência a todas as heurísticas do catálogo, descontando os
  problemas pela severidade.

```
nota_tela    = 0,7 × nota_persona + 0,3 × nota_heuristica
nota_jornada = média das notas das telas
```

### Modo somente heurísticas (sem 70/30)
Não existe persona, então não existe `nota_persona` nem a regra 70/30.
```
nota_tela    = nota_heuristica
nota_jornada = média das notas das telas
```

### Regra comum
Se houver um problema **crítico**, a nota da jornada fica limitada a 4,0.

## Formato JSON — avaliação de uma tela
```json
{
  "teste_id": "uuid",
  "tela": { "id": "12:345", "nome": "checkout-pagamento", "ordem": 3 },
  "persona": "App de Seguros — Marina Silva",
  "nota_persona": 6.0,
  "nota_heuristica": 7.5,
  "nota_tela": 6.5,
  "problemas": [
    {
      "id": "P1",
      "reacao_persona": "Com pressa, achei que travou. Não percebi que o botão estava ativo.",
      "heuristica": { "grupo": "Nielsen", "codigo": "N1", "nome": "Visibilidade do status do sistema" },
      "severidade": "alta",
      "severidade_nielsen": 3,
      "localizacao": { "elemento": "botão Continuar", "node_id": "12:400", "x": 320, "y": 540, "largura": 280, "altura": 48 },
      "sugestao": "Aumentar o contraste do estado ativo e mostrar feedback de carregamento."
    }
  ],
  "pontos_positivos": ["Preço final visível antes dos dados do cartão."]
}
```
No modo somente heurísticas: `persona` e `nota_persona` vêm como `null`,
`nota_tela` é igual a `nota_heuristica`, e cada problema traz `observacao`
(descrição técnica do problema) no lugar de `reacao_persona`.

## Formato JSON — relatório da jornada
```json
{
  "teste_id": "uuid",
  "modo": "persona_heuristicas",
  "objetivo_tarefa": "Contratar um seguro em menos de 3 minutos",
  "produto": "App de Seguros",
  "persona": "App de Seguros — Marina Silva",
  "assertividade": 82,
  "confianca": "moderada",
  "catalogo_versao": "v1",
  "heuristicas_avaliadas": "catalogo_completo",
  "objetivo_concluido": "com_dificuldade",
  "ponto_de_abandono": { "tela": "checkout-pagamento", "motivo": "..." },
  "nota_jornada": 6.8,
  "resumo_persona": "Texto curto, em primeira pessoa, sobre a jornada inteira.",
  "problemas_por_severidade": { "critica": 0, "alta": 2, "media": 4, "baixa": 3 },
  "prioridades": ["P1", "P4", "P2"],
  "canal_figma": "comentarios",
  "componentes_orquestra": { "validado": false },
  "telas": ["...avaliações de cada tela..."]
}
```
`modo` aceita: `persona_heuristicas`, `somente_heuristicas`.
`canal_figma` aceita: `comentarios`, `anotacoes_devmode`, `nenhum` (`07`).
`componentes_orquestra`: formato completo em `08`.
`objetivo_concluido` aceita: `sim`, `com_dificuldade`, `nao`.

No modo somente heurísticas: `persona`, `produto` e `resumo_persona` vêm como
`null`; `assertividade` é `100` e `confianca` é `alta`. No lugar de
`resumo_persona` vem `resumo_heuristico` (até 5 frases sobre a aderência da
jornada às heurísticas). Como o objetivo da tarefa existe nos dois modos,
`objetivo_concluido` e `ponto_de_abandono` são preenchidos nos dois.

## Ordem de construção
1. Validar os prompts (`05`) manualmente no chat com 2 ou 3 telas do Figma.
2. Fluxo Figma: entrada → normalização → assertividade → captura → simulação.
3. Persistência, biblioteca de personas e marcações.
4. Front: tela principal (`06`) e tela de confirmação da persona.
5. Link navegável com navegador headless.
6. Dashboard de comparação entre versões.

## Critérios de validação desta etapa
- [ ] Camadas bem separadas, com status do teste em cada passagem.
- [ ] Saída sempre em JSON estruturado e comparável.
- [ ] Avaliação de uma tela por vez, com consolidação da jornada no fim.
- [ ] Escala de severidade única, ligada à escala de Nielsen.
- [ ] Assertividade aparece no relatório final.
- [ ] Modo do teste definido na entrada pela presença de persona (nova ou da biblioteca).
- [ ] Objetivo da tarefa obrigatório na entrada e presente no relatório.
- [ ] Biblioteca: listar, criar produto, baixar `.md` normalizado e excluir.
- [ ] Exclusão de persona não altera testes já feitos (`persona_snapshot`).
- [ ] Só quem criou a persona consegue excluí-la (validado na API).
- [ ] Chave de API recebida a cada teste e nunca persistida nem registrada em log.
- [ ] Teste bloqueado é recusado pela API, sem forma de forçar.
- [ ] Modo somente heurísticas pula normalização e assertividade e não usa 70/30.
