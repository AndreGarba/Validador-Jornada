# 01 — Modelo Ideal da Persona

## Objetivo
Definir o modelo ideal (a referência de 100%) do arquivo de persona. Ele é a
base de quatro coisas:
- o **alvo da normalização** (`02`): todo input é reescrito neste formato;
- a **régua da assertividade** (`03`): cada campo tem um peso;
- o **contexto da simulação** (`05`): é o que o modelo lê para "ser" a persona;
- o **arquivo baixado** da biblioteca de personas (`06`).

## A persona não tem objetivo de tarefa
O objetivo principal da tarefa **não faz parte da persona**. Ele é informado
por teste, num campo próprio da tela principal (`06`). Assim, a mesma persona
pode ser usada para testar tarefas diferentes do mesmo produto.

A persona descreve **quem** ela é e **como** ela age. O teste diz **o que**
ela vai tentar fazer.

## Campos essenciais (60% da assertividade)
| Campo | Chave | Peso | Por que importa |
|---|---|---|---|
| Comportamentos | `comportamentos` | 30% | Gera reação realista em vez de opinião genérica. **Campo mais crítico.** |
| Dores e frustrações | `dores_frustracoes` | 15% | Define onde a persona trava e desiste. |
| Contexto de uso | `contexto_uso` | 15% | Pressa, dispositivo, ambiente e momento mudam a interpretação. |

## Campos de enriquecimento (40% da assertividade)
| Campo | Chave | Peso | Por que importa |
|---|---|---|---|
| Critérios de sucesso | `criterios_sucesso` | 17% | Régua do que ela consideraria uma jornada "boa", seja qual for a tarefa. |
| Nível digital | `nivel_digital` | 10% | Calibra o que é "óbvio" para ela. |
| Conhecimento prévio do domínio | `conhecimento_previo` | 8% | Separa novato de experiente no assunto do produto. |
| Demográficos | `demograficos` | 5% | Idade e ocupação contextualizam, mas pesam menos que comportamento. |

**Total: 100%.** Os campos `nome`, `produto` e `dispositivo_principal` não
pontuam sozinhos: `nome` e `produto` são identificação, e
`dispositivo_principal` faz parte do `contexto_uso`.

## O que conta em cada campo
| Campo | Deve responder |
|---|---|
| Comportamentos | Como navega, como lê, como decide, o que a faz confiar ou abandonar. |
| Dores e frustrações | O que a irrita, o que a faz desconfiar, experiências ruins anteriores. |
| Contexto de uso | Onde, quando, com que pressa, em que dispositivo e com que distrações. |
| Critérios de sucesso | O que faz ela considerar uma jornada concluída e satisfatória (ex.: ver o preço antes de dar dados, não voltar etapas). |
| Nível digital | Familiaridade com apps e padrões de interface. |
| Conhecimento prévio | O que já sabe e o que não sabe sobre o domínio (ex.: investimentos, crédito). |
| Demográficos | Idade, ocupação e outros dados que mudem o comportamento. |

## Template de referência
Este é o formato exato que a normalização produz e que é baixado da biblioteca.

```markdown
---
nome:
produto:
idade:
ocupacao:
nivel_digital:
dispositivo_principal:
contexto_uso:
---

## Comportamentos

## Dores e frustrações

## Contexto de uso

## Critérios de sucesso

## Conhecimento prévio
```

## Nome de exibição
Toda persona é identificada na plataforma pelo nome de exibição
`{produto} — {nome}` (ex.: "App de Seguros — Marina Silva"), para que fique
claro a qual produto ela pertence e em que testes foi usada. O nome de
exibição é montado pelo sistema a partir dos campos `produto` e `nome`; o
usuário não digita o prefixo.

Regras do template:
- No frontmatter, `contexto_uso` é o resumo em uma linha; a seção
  `## Contexto de uso` traz o detalhe. Para pontuar, vale o conteúdo somado dos
  dois.
- `produto` é preenchido com o produto escolhido na tela principal (`06`), e
  não pela normalização.

## Exemplo preenchido (nível RICO)

```markdown
---
nome: Marina Silva
produto: App de Seguros
idade: 34
ocupacao: Gerente de marketing
nivel_digital: intermediário
dispositivo_principal: smartphone Android
contexto_uso: no trabalho, entre reuniões, com pressa
---

## Comportamentos
- Lê por cima; pula qualquer texto com mais de duas linhas.
- Confia em prova social (avaliações, selos) antes de informar dados.
- Abandona se não entende o próximo passo em poucos segundos.
- Usa o celular com uma mão só, geralmente andando.

## Dores e frustrações
- Perde a paciência com formulários longos.
- Desconfia quando pedem dados de cartão antes de mostrar o preço final.

## Contexto de uso
Intervalos de 5 a 10 minutos entre reuniões, conexão 4G instável, notificações
interrompendo o tempo todo.

## Critérios de sucesso
Ver o preço final antes de qualquer dado pessoal, concluir sem voltar etapas e
sem precisar falar com atendente.

## Conhecimento prévio
Já contratou produtos parecidos em concorrentes; não conhece os termos técnicos
do contrato.
```

Exemplo de objetivo da tarefa para testar com essa persona (informado na tela
principal, não na persona): "Contratar um seguro em menos de 3 minutos".

## Critérios de validação desta etapa
- [ ] Todos os campos têm peso definido e somam 100%.
- [ ] Persona não contém objetivo de tarefa.
- [ ] Campo "Comportamentos" tratado como o mais crítico para realismo.
- [ ] Template de referência e exemplo preenchido aprovados.
- [ ] Persona de exemplo salva em `Personas/` para os testes manuais.
