---
produto: Validador de Jornada — Plataforma de Validação de UX com Personas Sintéticas
versao: 1.0
autor: Garba
status: draft
atualizado_em: 2026-10-14
---

# 00 — PRD: Visão Geral

## Visão
Plataforma que valida jornadas de UX (arquivo do Figma ou protótipo publicado e
navegável) simulando personas sintéticas. A reação da persona é cruzada com
heurísticas de usabilidade, e o resultado é um relatório estruturado com
marcações na interface, priorizadas por severidade.

## Problema que resolve
Testes de usabilidade com usuários reais são caros e lentos. A plataforma
antecipa problemas de UX simulando como uma persona específica reagiria a uma
interface, antes de investir em teste com usuários reais.

A plataforma **não substitui** teste com usuário real. Ela serve para chegar ao
teste real com os problemas óbvios já resolvidos.

## Modos de teste
| Modo | Quando | Persona | Peso | Assertividade |
|---|---|---|---|---|
| **Persona + heurísticas** | O usuário sobe uma persona nova ou escolhe uma da biblioteca. | Obrigatória | 70% persona / 30% heurística | Calculada pela qualidade da persona (`03`) |
| **Somente heurísticas** | O usuário não usa nenhuma persona. | Não existe | 100% heurística (sem 70/30) | Fixa em 100% |

O modo é definido pela presença de uma persona no teste: com persona, modo
persona + heurísticas; sem persona, modo somente heurísticas. Nos dois modos o
**objetivo principal da tarefa** é obrigatório.

A única escolha do usuário é **usar ou não uma persona**. Não existe seletor de
heurísticas: todo teste avalia **todas as heurísticas do catálogo** de uma vez.

## Objetivo principal da tarefa
O objetivo é da **tarefa testada**, e não da persona. Ele é informado num
campo próprio na tela principal (`06`), é obrigatório nos dois modos e guia a
avaliação:
- com persona: é o que a persona tenta concluir na jornada;
- sem persona: é a tarefa que o usuário precisa concluir, usada como critério
  da severidade crítica.

## Fluxo do usuário
Tela principal detalhada em `06`.
0. **Primeira tela:** informa a chave de API do provedor interno de IA, que é
   validada e salva só no navegador (ver `09`). Os testes consomem os tokens
   dessa chave. Com a chave válida, segue para a Home.
1. Informa o link: arquivo/frame do Figma **ou** URL de protótipo publicado.
2. Informa o **objetivo principal da tarefa** (obrigatório).
3. **Opcional:** define a persona de um destes jeitos:
   - sobe um arquivo novo em qualquer formato de texto (`.md`, `.txt`) e
     informa de qual produto ela é; ou
   - escolhe uma persona já salva na biblioteca (dropdown).
   Sem persona, o teste segue no modo somente heurísticas e pula para o passo 7.
4. Persona nova: o sistema **normaliza** para o modelo ideal (ver `02`).
5. Persona nova: o usuário **confirma** a persona normalizada, que é salva na
   biblioteca.
6. O sistema mostra a **assertividade** da persona (ver `03`). Abaixo de 40%, o
   teste é **bloqueado**: o usuário precisa enriquecer a persona, escolher
   outra ou seguir sem persona.
7. O sistema **captura** as telas (Figma via API; link via navegador headless).
8. O sistema avalia tela a tela contra **todas as heurísticas do catálogo**,
   simulando a persona (ou só aplicando as heurísticas, no modo somente
   heurísticas), e consolida a jornada (ver `05`).
9. O usuário recebe o **relatório** com marcações por severidade (ver `04`).
   No Figma, o resultado aparece no próprio arquivo, preso aos elementos da
   página enviada: comentários do Figma como prioridade, anotações do Dev Mode
   como alternativa (ver `07`).

**Opcional em qualquer modo:** ligar a validação de componentes do Orquestra,
escolhendo stack e versão. A validação roda pelo MCP do Orquestra (ver `08`).

## Peso da análise: 70% persona / 30% heurística
Vale apenas no modo **persona + heurísticas**. No modo somente heurísticas, a
análise é 100% heurística.

- **Persona é a lente:** define o que olhar, o que importa para aquele usuário,
  onde ele trava e o que o faz desistir.
- **Heurística é a régua:** dá o nome técnico e a severidade do problema,
  impedindo que a análise vire achismo.
- **Ordem forçada:** o modelo reage primeiro como a persona e só depois conecta
  o problema a uma heurística. Persona e heurística não competem; a persona
  "puxa" a heurística.
- Como o 70/30 entra na nota numérica está definido em `04`.

## Catálogo de heurísticas
Fonte: skills de heurísticas do projeto (`skills-heuristicas/`).

| Grupo | Itens | Fonte |
|---|---|---|
| Nielsen | As 10 heurísticas de usabilidade (N1 a N10) | `nielsen-heuristics-audit`, `ux-heuristics/references/nielsen-heuristics.md` |
| Krug | Princípios de "Don't Make Me Think" | `ux-heuristics/references/krug-principles.md` |
| Acessibilidade | Checklist WCAG | `ux-heuristics/references/wcag-checklist.md` |
| Dark patterns | Padrões enganosos | `ux-heuristics/references/dark-patterns.md` |
| UX cultural | Adequação cultural | `ux-heuristics/references/cultural-ux.md` |

Todo teste avalia o catálogo inteiro; o usuário não escolhe heurísticas. No
relatório, cada problema indica o grupo e o código da heurística ferida, e a
lista `heuristicas_avaliadas` traz sempre o catálogo completo e a versão dele.

## Escopo do MVP
- **Dentro:** entrada via Figma, objetivo da tarefa, biblioteca de personas
  por produto, normalização, validador de assertividade, simulação tela a tela,
  relatório JSON + marcações, histórico em banco.
- **Depois do MVP:** link navegável via navegador headless, dashboard de
  comparação entre versões, múltiplas personas no mesmo teste.

## Estrutura de pastas do projeto
| Pasta | Conteúdo |
|---|---|
| `PRD/` | Esta PRD (fonte oficial das regras do produto). |
| `Personas/` | Personas de exemplo e personas normalizadas (`.md`). |
| `Resultados/` | Relatórios gerados (`.json`) durante a validação manual. |

## Arquivos da PRD
| Arquivo | Tema |
|---|---|
| `00-prd-visao-geral.md` | Visão, fluxo, peso, catálogo, escopo |
| `01-modelo-ideal-persona.md` | Modelo ideal da persona e pesos dos campos |
| `02-normalizacao-input.md` | Reescrita de qualquer input para o modelo ideal |
| `03-validador-assertividade.md` | Cálculo da confiança do teste |
| `04-backend-arquitetura.md` | Camadas do back-end e formato do relatório |
| `05-prompts-mestres.md` | Prompts de normalização, profundidade e simulação |
| `06-tela-principal-biblioteca-personas.md` | Campos da tela principal e biblioteca de personas |
| `07-resultado-no-figma.md` | Resultado no próprio Figma: comentários ou anotações do Dev Mode |
| `08-validacao-componentes-orquestra.md` | Seletor e regras da validação de componentes do Orquestra |
| `09-chave-api-ia.md` | Primeira tela: chave de API da IA do usuário |
| `10-plugin-figma.md` | Versão alternativa: plugin do Figma com servidor e chave da equipe |
| `11-skill-agente-figma.md` | Versão alternativa: skill do agente do Figma, com créditos de IA de cada usuário |
| `12-principios-de-design.md` | Princípios de design do BTG Pactual (Orquestra, via Supernova) |
| `13-principios-de-motion.md` | Princípios de motion, tempos e curvas de animação (Orquestra, via Supernova) |
| `14-writing.md` | Guia de linguagem e tom de voz: regras de escrita e textos por componente (Orquestra, via Supernova) |

## Ordem de validação recomendada
1. `00` e `01` (fundação).
2. `05` testado manualmente no chat com 2 ou 3 telas (prova de conceito).
3. `02` e `03` (entrada confiável).
4. `04` (construção do back-end).

## Critérios de validação desta etapa
- [ ] Fluxo cobre entrada, normalização, validação, captura, simulação e saída.
- [ ] Nenhum teste roda sem chave de API válida do usuário (`09`).
- [ ] Dois modos definidos: persona + heurísticas e somente heurísticas.
- [ ] Peso persona/heurística definido e justificado (só no modo com persona).
- [ ] Sem seletor de heurísticas: todo teste avalia o catálogo completo.
- [ ] Escolhas do usuário: usar ou não uma persona e ligar ou não a validação
      do Orquestra.
- [ ] Objetivo principal da tarefa obrigatório nos dois modos, fora da persona.
- [ ] Catálogo de heurísticas definido e ligado às skills do projeto.
- [ ] Escopo do MVP delimitado (começar pelo Figma).
