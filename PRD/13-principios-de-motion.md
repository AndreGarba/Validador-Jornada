# 13 — Princípios de Motion

## Objetivo
Registrar os princípios de motion do BTG Pactual e as regras de tempo e curvas
de animação que o Orquestra define a partir deles.

## Fonte
- Supernova, design system **Orquestra** ("Design System oficial do BTG
  Pactual"), página **Primeiros passos / Princípios** ("Valores que norteiam o
  Design System"), seção "Motion".
- Consultado em 27/10/2026, na versão Shared draft.
- O texto abaixo é o da documentação, com correções de escrita. Se a
  documentação mudar, esta PRD deve ser atualizada a partir dela.

## Conceito
Expressar a expansão e segurança financeira que o banco proporciona, com
movimentos ágeis e fluidos que remetem a crescimento.

## Princípios conceituais
| # | Princípio | Descrição |
|---|---|---|
| 1 | **Humano** | Nós respeitamos as leis da física e da gravidade com movimentos naturais, que são familiares ao usuário e podem ser entendidos facilmente. Com animações, tornamos as experiências mais simples e acessíveis. |
| 2 | **Informado** | Usamos o motion como uma ferramenta para tornar visíveis as transformações da tela e destacar aquilo que realmente importa, eliminando distrações desnecessárias. Também ajudamos a contextualizar o que acontece de forma clara e informativa, oferecendo feedbacks sobre as interações. |
| 3 | **Consistente** | Esse é o nosso jeito BTG de fazer motion: criamos animações orquestradas que obedecem sempre a mesma lógica e que são capazes de expressar a nossa identidade de forma consistente. |
| 4 | **Encantador** | Nossos movimentos são leves, podem conter pitadas de humor ou trazer algo inusitado. Eles refinam e tornam a experiência surpreendente e encantadora. Assim, com sutileza e fluidez, nos aproximamos de nosso cliente, respeitando a nossa personalidade. |
| 5 | **Eficiente** | Valorizamos muito o tempo dos nossos clientes e, por isso, nossas animações são fluidas, rápidas e precisas. A prioridade delas é sempre colaborar com a produtividade do usuário e não buscar protagonismo. |
| 6 | **Empático** | Usamos o motion para demonstrar empatia e ajudar nossos usuários a se sentirem acolhidos durante jornadas difíceis. Nós também celebramos com eles suas vitórias e momentos importantes. |

Os cinco primeiros têm o mesmo nome dos princípios de design (`12`);
**Empático** existe só em motion.

## Como os princípios se aplicam
- Dois estilos de animação, seguindo o caminho do Carbon: um para animações
  diretas, que precisam de resposta rápida, e outro para momentos em que é
  preciso chamar a atenção do usuário para o que está acontecendo na tela.
- Os tokens de animação têm tempo máximo total baixo, mesmo nas animações mais
  informativas.
- Animações encantadoras ficam para feedbacks em telas específicas: confirmação
  de ação, loader ou microinteração complexa.

## Variação de tempo (dynamic duration)
O tempo de uma animação pode variar com o tamanho do componente ou a distância
percorrida na tela. A variação pode partir do valor de um token, somando um
tempo conforme o tamanho, ou ser usada à parte dos tokens.

No Orquestra, a variação vale para animações de **posição** e **escala**:
- **Short distance** equivale a 6 colunas no eixo X.
- Tempo base de short distance e variação por coluna a mais ou a menos:

| Grid | Tempo base | Variação por coluna |
|---|---|---|
| `grid-sm-default` | 150 ms | 2% |
| `grid-medium` | 200 ms | 4% |
| `grid-large` | 250 ms | 5% |

## Curvas de animação (easing)
| Ideia | Uso |
|---|---|
| Ease (in-out) | Elementos que já estão em cena: aceleram e desaceleram durante a animação. |
| Ease Out | Elementos entrando em cena: já chegam com velocidade e desaceleram até a posição final. |
| Ease In | Elementos saindo de cena: aceleram até sair, sem desacelerar. |

Curvas do Orquestra:

| Easing | Cubic-bezier | After Effects |
|---|---|---|
| Standard Ease | 0.25, 0, 0.25, 1 | Incoming: 75% / Outgoing: 25% |
| Informative Ease | 0.55, 0, 0.25, 1 | Incoming: 75% / Outgoing: 55% |
| Ease Out (desaceleração) | 0, 0, 0.25, 1 | Incoming: 75% / Outgoing: 0% |
| Ease In (aceleração) | 0.75, 0, 1, 1 | Incoming: 0% / Outgoing: 75% |
| Linear | 0, 0, 1, 1 | Incoming: 0% / Outgoing: 0% |

- **Informative Ease:** só quando for preciso dar ênfase à animação em tela;
  tem tempo maior que as outras.
- **Linear:** só em animações de cor e opacidade.
- Referências citadas na documentação: Material Design (emphasized easing) e
  Carbon (grupos productive e expressive).

## Uso no Validador de Jornada
Ainda não entra. O Validador avalia telas estáticas do Figma e hoje não
analisa animações nem protótipos com motion. Decidido em 27/10/2026: estes
princípios ficam para depois, quando a validação puder rodar num link de
protótipo funcional com motion.

## Critérios de validação desta etapa
- [ ] Os seis princípios correspondem à página do Supernova.
- [ ] Tempos, variações e curvas conferem com a documentação.
- [ ] A fonte e a data da consulta estão registradas.
- [ ] O uso no Validador está decidido.
