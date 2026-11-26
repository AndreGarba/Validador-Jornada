# 07 — Resultado do Teste no Figma

## Objetivo
Entregar o resultado do teste **dentro do próprio arquivo do Figma** que o
usuário enviou: cada ajuste e cada ponto crítico aparece preso ao item que
precisa ser ajustado, na própria página enviada, sem duplicar telas.

Vale para os problemas de heurística/persona (`04`) e para os apontamentos de
componentes do Orquestra (`08`).

## Quando vale
- **Link do Figma:** o resultado é escrito no arquivo (esta PRD) e também fica
  salvo na plataforma (`04`).
- **Link navegável (pós-MVP):** não há arquivo do Figma; o resultado fica só
  na plataforma.

## Pré-requisito: jornada em uma página própria
As anotações são feitas **na própria página** enviada, sem duplicar telas. Por
isso, o usuário precisa separar a jornada (ou a tela) a ser validada em uma
página própria do Figma antes de enviar o link.

### Observação embaixo do campo de link (tela principal, `06`)
> Antes de enviar, separe a jornada ou a tela a ser validada em uma página
> própria no Figma. As anotações do teste serão feitas direto nessa página,
> sem duplicar as telas.

### Regras do link
- O link deve apontar para a página da jornada (ou para um frame dentro dela).
  A página inteira é tratada como a jornada.
- São avaliados só os frames de primeiro nível da página.

## Regra de ouro: não alterar o design
O Validador **não edita, não move, não cria e não duplica** camadas do
usuário. Ele só escreve comentários ou anotações presos aos elementos.

## Canal de saída: comentários primeiro, anotações como alternativa
O sistema escolhe o canal **antes de rodar o teste**, nesta ordem:

| Prioridade | Canal | Quando é usado | Como é escrito |
|---|---|---|---|
| 1 | **Comentários do Figma** | Existe conexão com o Figma que permite criar comentários no arquivo. | API REST do Figma (endpoint de comentários), com token com permissão de escrita de comentários. |
| 2 | **Anotações do Dev Mode** | Não existe conexão para comentários, mas existe conexão com o MCP do Figma. | Plugin API pelo MCP do Figma (`use_figma`), gravando anotações nos elementos. |
| 3 | **Só na plataforma** | Não existe nenhuma das duas conexões. | O relatório fica só na plataforma, com aviso. |

Regras:
- Um teste usa **um canal só**; não mistura comentários e anotações.
- O canal usado fica gravado no teste (`canal_figma`: `comentarios`,
  `anotacoes_devmode` ou `nenhum`) e aparece no relatório.
- Se o canal escolhido falhar no meio da escrita, o sistema tenta o próximo
  canal da lista para os itens que faltaram e avisa o usuário.
- No canal 3, a tela principal avisa antes do início: "Sem conexão com o Figma
  para escrever comentários ou anotações. O resultado ficará só na
  plataforma."

## Canal 1 — Comentários do Figma
Comentários geram **notificação real** para quem acompanha o arquivo; por isso
são a prioridade.

### Comentário de problema
- Um comentário por problema, preso ao elemento (`node_id`) no ponto da
  `localizacao` do relatório (`04`).
- Texto do comentário:

```
🔴 P1 · CRÍTICA · Nielsen N1 — Visibilidade do status do sistema
Elemento: botão Continuar
Persona: "Com pressa, achei que travou. Não percebi que o botão estava ativo."
Ajuste: Aumentar o contraste do estado ativo e mostrar feedback de carregamento.
[Validador de Jornada · teste {teste_id}]
```

- Prefixo de severidade: 🔴 crítica, 🟠 alta, 🟡 média, 🔵 baixa.
- No modo somente heurísticas, a linha "Persona" é trocada por "Observação",
  com o texto técnico do problema.
- Apontamentos do Orquestra (`08`) usam o prefixo `[Orquestra]` no lugar da
  heurística.

### Comentário de resumo
Um comentário preso ao primeiro frame da jornada, com: modo, persona
(`{produto} — {nome}`) ou "Somente heurísticas", objetivo da tarefa,
assertividade, nota da jornada, objetivo concluído, problemas por severidade,
3 primeiras prioridades, aderência ao Orquestra (quando validado) e link para
o relatório completo.

## Canal 2 — Anotações do Dev Mode
### Categorias
O sistema cria (se ainda não existirem no arquivo) categorias de anotação por
severidade:

| Categoria | Cor |
|---|---|
| Validador · Crítica | vermelho |
| Validador · Alta | laranja |
| Validador · Média | amarelo |
| Validador · Baixa | azul |
| Validador · Orquestra | violeta |
| Validador · Resumo | verde |

### Anotação de problema
- Uma anotação por problema, gravada no próprio elemento (`node_id`), com a
  categoria da severidade.
- Mesmo texto do comentário do canal 1.
- Apontamentos do Orquestra usam a categoria "Validador · Orquestra" e trazem a
  severidade no texto.

### Anotação de resumo
Gravada no primeiro frame da jornada, categoria "Validador · Resumo", com o
mesmo conteúdo do comentário de resumo.

### Anotações já existentes
O Validador não altera nem apaga anotações que não são dele. Ele só mexe nas
anotações com as categorias "Validador · …".

## Rodar o teste de novo na mesma página
Para não duplicar:
- **Comentários:** o sistema apaga os comentários do teste anterior naquela
  página (ids guardados no teste) antes de escrever os novos.
- **Anotações:** o sistema remove as anotações das categorias "Validador · …"
  da página antes de escrever as novas.
- O histórico de todos os testes continua salvo na plataforma (`04`).

## Elemento não encontrado
Se um `node_id` não existir mais (a tela mudou entre a captura e a escrita), o
item é preso ao frame da tela, com o aviso "Elemento não encontrado:
{elemento}".

## Permissões
- Canal 1: o token precisa poder criar e apagar comentários no arquivo.
- Canal 2: o usuário conectado ao MCP do Figma precisa ter permissão de
  **edição** no arquivo.
- Sem permissão no canal escolhido, o sistema passa para o próximo canal.

## Critérios de validação desta etapa
- [ ] Observação sobre página própria aparece embaixo do campo de link.
- [ ] Nenhuma camada do usuário é editada, movida, criada ou duplicada.
- [ ] Canal escolhido na ordem: comentários → anotações do Dev Mode → só
      plataforma.
- [ ] Um teste usa um canal só, gravado em `canal_figma`.
- [ ] Cada problema fica preso ao elemento, com prefixo ou categoria de
      severidade.
- [ ] Resumo da jornada preso ao primeiro frame.
- [ ] Rodar de novo na mesma página não duplica comentários nem anotações.
- [ ] Anotações de terceiros nunca são alteradas.
