# 08 — Validação de Componentes do Orquestra

## Objetivo
Permitir que o usuário, além das heurísticas (e da persona, se houver), valide
se as telas usam corretamente os componentes do design system **Orquestra**.
A validação é feita pelo **MCP do Orquestra**.

## Seletor na tela principal
Campo novo na tela principal (`06`):

| Campo | Tipo | Padrão | Regra |
|---|---|---|---|
| Validar componentes do Orquestra | Liga/desliga | Desligado | Quando ligado, mostra os campos Stack e Versão. |
| Stack | Seleção única | — | `React`, `Angular` ou `Vue`. Obrigatório com o seletor ligado. |
| Versão do Orquestra | Seleção única | Versão mais recente | Lista vinda do MCP (`available-web-versions`) para a stack escolhida. Obrigatório com o seletor ligado. |

Regras:
- Funciona nos dois modos (com ou sem persona) e não muda a assertividade.
- Versões mínimas aceitas pelo MCP: React (`@orquestra-web/react`) 3.3.2;
  Angular (`@btgclient-web/orquestra`) 18.0.0.
- Se o MCP do Orquestra estiver indisponível, o seletor fica **desabilitado**
  com o aviso "MCP do Orquestra indisponível no momento".

## Ferramentas do MCP do Orquestra usadas
| Ferramenta | Uso |
|---|---|
| `available-web-versions` | Preenche a lista de versões da stack escolhida. |
| `list-web-components` | Traz o catálogo de componentes da stack e versão. |
| `info-web-component` | Traz props, variantes e exemplos de cada componente usado na tela. |

O catálogo e as informações dos componentes são consultados **uma vez por
teste** e reaproveitados em todas as telas.

## Como o componente do Figma é ligado ao Orquestra
A ligação usa os dois caminhos, nesta ordem:
1. **Code Connect:** se o componente do Figma tiver mapeamento de Code Connect
   para um componente do Orquestra, vale esse mapeamento (mais preciso).
2. **Nome na biblioteca do Figma:** sem Code Connect, o nome do componente
   principal é comparado com a lista de `list-web-components` (minúsculas,
   sem prefixo, palavras separadas por hífen).

Se nenhum dos dois achar correspondência, o elemento é tratado como elemento
próprio, e o modelo avalia se existe equivalente no Orquestra.

Cada item guarda o caminho usado (`vinculo`: `code_connect`, `nome` ou
`nenhum`), para medir a precisão da validação.

## O que é validado
Para cada elemento da árvore capturada (`04`, camada de captura):

| Situação | Exemplo | Severidade |
|---|---|---|
| Componente do Orquestra usado corretamente | Instância de Button com variante existente | — (conta como correto) |
| Variante ou propriedade que não existe na versão escolhida | Button com variante "ghost" que a versão não tem | alta |
| Elemento próprio ou desenhado à mão, com equivalente no Orquestra | Campo de texto feito com retângulo + texto | média |
| Instância do Orquestra desvinculada (detached) | Card desanexado da biblioteca | média |
| Elemento sem equivalente no Orquestra | Ilustração, gráfico específico | — (registrado como "sem equivalente", não é problema) |

Severidades aprovadas: alta para variante ou propriedade inexistente; média
para elemento com equivalente e para instância desvinculada.

Todo apontamento traz a **sugestão de troca**: qual componente do Orquestra
usar e com quais props/variantes, conforme `info-web-component`.

## Indicador de aderência
```
aderencia_orquestra = componentes corretos / (corretos + apontamentos) × 100
```
- Elementos "sem equivalente" ficam fora da conta.
- A aderência aparece no relatório e no resumo do Figma (`07`).
- Os apontamentos do Orquestra **não entram** na `nota_jornada` (`04`), que
  continua medindo persona e heurísticas.
- Os apontamentos ficam numa **lista separada** (`componentes_orquestra.apontamentos`,
  com ids `O1`, `O2`…) e **não entram** na lista geral `prioridades` do
  relatório, que continua só com problemas de persona/heurística.

## Onde aparece o resultado
- **Plataforma:** bloco `componentes_orquestra` no relatório da jornada (`04`).
- **Figma:** no mesmo canal do teste (`07`): comentário com prefixo
  `[Orquestra]` ou anotação com a categoria "Validador · Orquestra".

Texto do apontamento:
```
🟠 O3 · ALTA · [Orquestra] Variante inexistente
Elemento: botão Continuar (Button)
Problema: a variante "ghost" não existe no Orquestra React 3.3.2.
Ajuste: usar Button com variant "secondary" (variantes disponíveis: primary, secondary, tertiary).
[Validador de Jornada · teste {teste_id}]
```
Os valores de variante do exemplo são ilustrativos; os reais vêm do MCP.

## Formato JSON (bloco do relatório da jornada)
```json
"componentes_orquestra": {
  "validado": true,
  "stack": "react",
  "versao": "3.3.2",
  "aderencia": 78,
  "corretos": 18,
  "sem_equivalente": 3,
  "apontamentos": [
    {
      "id": "O3",
      "tela": "checkout-pagamento",
      "tipo": "variante_inexistente",
      "severidade": "alta",
      "componente_orquestra": "button",
      "vinculo": "code_connect",
      "localizacao": { "elemento": "botão Continuar", "node_id": "12:400" },
      "problema": "A variante \"ghost\" não existe na versão 3.3.2.",
      "sugestao": "Usar Button com variant \"secondary\"."
    }
  ]
}
```
`tipo` aceita: `variante_inexistente`, `propriedade_inexistente`,
`elemento_com_equivalente`, `instancia_desvinculada`.
`vinculo` aceita: `code_connect`, `nome`, `nenhum`.
Com o seletor desligado: `"componentes_orquestra": { "validado": false }`.

## Critérios de validação desta etapa
- [ ] Ligação Figma → Orquestra por Code Connect primeiro e nome depois, com
      `vinculo` registrado.
- [ ] Apontamentos em lista separada, fora de `prioridades`.
- [ ] Seletor desligado por padrão; ligado, exige stack e versão.
- [ ] Lista de versões vem do MCP do Orquestra.
- [ ] Seletor desabilitado quando o MCP do Orquestra está indisponível.
- [ ] Catálogo e props consultados pelo MCP, nunca escritos à mão.
- [ ] Cada apontamento traz a sugestão de componente/variante do Orquestra.
- [ ] Aderência calculada sem contar elementos sem equivalente.
- [ ] Apontamentos não alteram a `nota_jornada`.
- [ ] Resultado aparece na plataforma e no Figma, no canal do `07`.
