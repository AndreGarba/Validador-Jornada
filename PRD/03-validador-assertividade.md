# 03 — Validador de Assertividade

## Objetivo
Medir a confiança do teste com base na qualidade da persona normalizada, e
mostrar ao usuário o que fazer para aumentar essa confiança.

Roda **depois** da confirmação da normalização (`02`) e **antes** da captura e
da simulação. O resultado fica salvo com a persona na biblioteca (`06`): ao
escolher uma persona já salva, a assertividade não é recalculada.

A assertividade mede só a persona. O objetivo principal da tarefa fica fora do
cálculo, porque é obrigatório na tela principal (`06`).

## Assertividade por modo de teste
| Modo | Assertividade | Regra 70/30 |
|---|---|---|
| Persona + heurísticas | Calculada pela fórmula abaixo | Sim |
| Somente heurísticas | **Fixa em 100%** | Não existe (não há persona) |

No modo somente heurísticas, o objetivo do usuário é validar a interface apenas
contra todas as heurísticas do catálogo. Como não há persona para medir, a confiança
do teste é **recalculada para esse objetivo** e sobe para 100%. O cálculo
abaixo não roda, e o relatório indica `modo: somente_heuristicas`.

As seções a seguir valem só para o modo **persona + heurísticas**.

## Duas dimensões
### Dimensão 1 — Presença
O campo existe e tem conteúdo? Campo ausente vale zero.
Ex.: faltando "Comportamentos" (30%) e "Contexto de uso" (15%), o teste perde
45% de cara.

### Dimensão 2 — Profundidade
O campo tem substância? Cada campo presente é avaliado em três eixos, de 0 a 10:

| Eixo | Pergunta | Genérico | Específico |
|---|---|---|---|
| Especificidade | É concreto e detalhado? | "Tem pressa" | "Usa entre reuniões, abandona se demora mais de 30 s" |
| Acionabilidade | Dá para simular comportamento a partir disso? | "É impaciente" | "Lê por cima e pula textos longos" |
| Singularidade | Descreve **esta** persona ou qualquer uma? | "Gosta de coisas fáceis" | "Desconfia de pedir cartão antes de ver avaliações" |

**Singularidade é o eixo anti-trapaça:** pega texto longo mas genérico, escrito
só para inflar o medidor.

Média dos três eixos → classificação → fator de profundidade:

| Média | Classificação | Fator |
|---|---|---|
| campo ausente | VAZIO | 0,00 |
| 0 a 3,9 | RASO | 0,40 |
| 4 a 6,9 | ADEQUADO | 0,75 |
| 7 a 10 | RICO | 1,00 |

## Cálculo da assertividade
```
assertividade = Σ (peso_do_campo × fator_de_profundidade)
```
- Os pesos vêm de `01` e somam 100%.
- A presença já está embutida: campo ausente tem fator 0.
- Resultado arredondado para inteiro (0 a 100%).

### Exemplos
| Cenário | Cálculo | Assertividade |
|---|---|---|
| Todos os campos RICOS | 100 × 1,00 | 100% |
| Todos os campos ADEQUADOS | 100 × 0,75 | 75% |
| Todos presentes, todos RASOS | 100 × 0,40 | 40% |
| Essenciais RICOS, enriquecimento ausente | 60 × 1,00 + 40 × 0 | 60% |
| Sem Comportamentos e Contexto, resto ADEQUADO | 55 × 0,75 | 41% |

## Faixas de confiança e comportamento
| Faixa | Rótulo | Comportamento |
|---|---|---|
| 0 a 39% | Insuficiente | **Bloqueia** o teste, sem opção de forçar. Mostra o que falta. |
| 40 a 69% | Baixa | Permite com **aviso forte**; o relatório leva o selo "confiança baixa". |
| 70 a 84% | Moderada | Permite com aviso leve. |
| 85 a 100% | Alta | Permite sem aviso. |

A assertividade sempre aparece no relatório final (`04`), junto da nota.

### Teste bloqueado
Abaixo de 40%, o usuário **não consegue** rodar o teste com essa persona. Não
existe opção de forçar. Para seguir, ele tem dois caminhos:
1. **Enriquecer a persona** com base no feedback e reenviar (volta para `02`).
2. **Escolher outra persona** da biblioteca com assertividade a partir de 40%.
3. **Remover a persona** e rodar no modo somente heurísticas (assertividade 100%).

Personas abaixo de 40% ficam salvas na biblioteca, para poderem ser baixadas e
enriquecidas, mas aparecem **desabilitadas** no dropdown de seleção (`06`).

## Feedback na tela
Não basta o número: o validador diz o que fazer, campo a campo, ordenado pelo
ganho potencial (peso × quanto falta para RICO).

```
Assertividade do teste: 68% ⚠️  Confiança baixa

Para aumentar a confiança:
- Comportamentos (RASO): descreva como a persona navega e decide.  +18%
- Critérios de sucesso (VAZIO): defina o que ela consideraria "bom". +17%
- Contexto de uso (RICO) ✓
```

## Pontos em aberto (validar com o Garba)
- Fatores 0,40 / 0,75 / 1,00 e limites das faixas são uma proposta inicial.
  Calibrar depois dos testes manuais.

## Critérios de validação desta etapa
- [ ] Cálculo combina presença e profundidade sem contar duas vezes.
- [ ] Eixo singularidade implementado como anti-trapaça.
- [ ] Piso mínimo de 40% bloqueia o teste, sem opção de forçar.
- [ ] Teste bloqueado oferece: enriquecer, escolher outra persona ou seguir sem persona.
- [ ] Assertividade salva com a persona e não recalculada ao reutilizá-la.
- [ ] Modo somente heurísticas tem assertividade fixa em 100% e não usa 70/30.
- [ ] Feedback campo a campo, ordenado pelo ganho potencial.
- [ ] Mesma persona avaliada 3 vezes varia no máximo ±5% (estabilidade).
