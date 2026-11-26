# 10 — Validador como Plugin do Figma

## Objetivo
Levar o Validador para dentro do Figma, como um plugin. O teste roda no próprio
arquivo, na página da jornada, e o usuário **não informa nenhuma chave de IA**:
as chamadas de IA passam por um servidor da equipe, que guarda uma chave única.

Esta PRD descreve uma **versão alternativa** à plataforma web. Ela não altera
as PRDs `00` a `09`; a tabela abaixo diz o que vale para o plugin.

## Por que um servidor, e não a IA do Figma
- A Plugin API do Figma não dá acesso aos modelos de IA do Figma. Um plugin não
  consegue rodar os prompts do Validador com créditos do Figma.
- O código de um plugin pode ser lido por quem tem acesso a ele. Por isso a
  chave de IA **nunca** fica no plugin.
- Solução: o plugin envia as telas para o servidor do Validador, que chama a
  IA com a chave da equipe e devolve o resultado.

## Relação com as outras PRDs
| PRD | Na versão plugin |
|---|---|
| `00` Visão geral | Vale, com o fluxo desta PRD no lugar do fluxo web. Link navegável fica fora. |
| `01` Modelo da persona | Vale sem mudança. |
| `02` Normalização | Vale sem mudança. |
| `03` Assertividade | Vale sem mudança. |
| `04` Back-end | Vale, com as mudanças de "Servidor" abaixo. A captura passa a ser feita pelo plugin. |
| `05` Prompts | Vale sem mudança. |
| `06` Tela principal | Substituída pela tela do plugin desta PRD. Bloco de persona e biblioteca valem iguais. |
| `07` Resultado no Figma | Vale a regra de não alterar o design e de não duplicar. O canal muda (ver "Resultado no arquivo"). |
| `08` Orquestra | Vale, com a ligação de componentes descrita abaixo. |
| `09` Chave de API | **Não se aplica.** O usuário não informa chave. |

## Arquitetura
```
Plugin (no arquivo do Figma) ─▶ Servidor do Validador ─▶ IA (chave única da equipe)
        │  captura telas e estrutura         │  normalização, assertividade,
        │  escreve anotações                 │  simulação, consolidação
        ◀──────────── resultado em JSON ──────┘  biblioteca e histórico (banco)
```

### Plugin
- Plugin **privado da organização**, instalado pelo Figma.
- Lê a página atual, exporta as telas e monta a estrutura de elementos.
- Escreve o resultado como anotações nos elementos (Plugin API).
- Não guarda nenhuma chave de IA nem segredo no código.
- Só acessa os domínios declarados no manifesto: o servidor do Validador e o
  CDN do Orquestra (fontes e tema da interface).

### Servidor
- Guarda a **chave única da equipe** do provedor interno de IA como segredo do
  servidor. Ela nunca vai para o plugin, para o banco, para o log ou para o
  relatório.
- Mantém as camadas de `04`: entrada, normalização, assertividade, biblioteca,
  processamento e saída. A captura deixa de ser feita no servidor.
- Recebe do plugin as imagens e a estrutura das telas, em vez de um link.
- Só atende pedidos com sessão válida de um usuário da organização (ver
  "Acesso").
- Registra o consumo de IA por usuário e por teste (tokens, quando o provedor
  informar).

## Acesso
- Na primeira vez, o plugin mostra a tela **Entrar**, que abre o login
  corporativo no navegador.
- O login só aceita contas `@btgpactual.com` e `@grupopan.com`.
- Depois do login, o plugin guarda a sessão no armazenamento local do plugin
  no Figma. A sessão expira e pede login de novo.
- A identidade do login é a usada em `criado_por` (biblioteca e testes, `04`).

## Fluxo do usuário
1. Abre o arquivo e vai para a **página da jornada**. A regra de página
   própria de `07` continua valendo.
2. Abre o plugin. Sem sessão, faz o login (ver "Acesso").
3. O plugin mostra a página atual e quantos frames de primeiro nível serão
   avaliados.
4. Informa o **objetivo principal da tarefa** (obrigatório, `00`).
5. **Opcional:** escolhe a persona: sobe uma nova ou escolhe da biblioteca
   (regras de `06`). Persona nova passa por normalização, confirmação e
   assertividade (`02`, `03`). Abaixo de 40%, o teste é bloqueado.
6. **Opcional:** liga a validação de componentes do Orquestra, com stack e
   versão (`08`).
7. Clica em **Iniciar teste**. O plugin captura as telas e envia ao servidor.
8. O plugin mostra o andamento tela a tela.
9. Ao terminar, o plugin escreve as anotações na página e mostra o resumo do
   resultado, com link para o relatório completo.

## Tela do plugin
| Ordem | Campo | Obrigatório | Regra |
|---|---|---|---|
| 1 | Página da jornada | — | Só leitura: nome da página atual e número de frames. Atualiza quando o usuário troca de página. |
| 2 | Objetivo principal da tarefa | Sim | Igual a `06`. |
| 3 | Persona | Não | Igual ao bloco de persona de `06`. |
| 4 | Validar componentes do Orquestra | Não | Igual a `06` e `08`. |

- Não existe campo de link e não existe seletor de heurísticas (`00`).
- O botão **Iniciar teste** fica desabilitado sem objetivo, sem frames na
  página, com persona abaixo de 40%, ou com o Orquestra ligado sem stack ou
  versão.
- O modo resultante aparece como em `06`.
- Na implementação, a interface segue o Orquestra primeiro e o Figma como
  fallback (regras de UI do projeto).

## Captura pelo plugin
- Avalia os frames de primeiro nível da página atual (`07`).
- Ordem da jornada: fluxos do protótipo da página; sem fluxo, ordem dos frames.
- Para cada frame:
  - imagem PNG exportada pelo plugin;
  - estrutura dos elementos: `node_id`, nome, tipo, texto, posição e tamanho;
  - componentes: nome do componente principal e da biblioteca, variantes e
    propriedades, e se a instância foi desvinculada.
- Não precisa de token do Figma: o plugin já lê o arquivo aberto.

## Resultado no arquivo
- Canal da versão plugin: **anotações do Dev Mode**, escritas pelo próprio
  plugin, com as categorias, os textos e o resumo de `07`.
- Comentários do Figma não são usados nesta versão: a Plugin API não cria
  comentários.
- Rodar de novo na mesma página remove as anotações "Validador · …" antes de
  escrever as novas (`07`).
- Escrever anotações exige permissão de **edição** no arquivo. Sem ela, o
  plugin avisa antes do início e o resultado fica só no servidor.
- `canal_figma` do teste passa a ser `anotacoes_devmode` ou `nenhum`.

### Plugin fechado antes do fim
- O teste continua no servidor.
- Ao abrir o plugin de novo na mesma página, ele mostra "Resultado pronto para
  aplicar" com o botão **Aplicar anotações**.
- Se os elementos mudaram, vale a regra de elemento não encontrado de `07`.

## Validação do Orquestra no plugin
- O plugin envia os dados dos componentes de cada frame; o servidor consulta o
  MCP do Orquestra (`08`).
- Ligação com o Orquestra: pelo nome do componente na biblioteca do Figma.
  Code Connect fica como ponto em aberto (abaixo).

## Erros
| Situação | Mensagem no plugin |
|---|---|
| Sessão expirada | "Sua sessão expirou. Entre de novo para continuar." |
| Servidor fora do ar | "Não foi possível falar com o Validador agora. Tente de novo em alguns minutos." |
| IA da equipe sem saldo ou fora do ar | "A IA do Validador está indisponível no momento. Tente de novo mais tarde." |
| Página sem frames | "Esta página não tem frames para avaliar." |
| Sem permissão de edição | "Você não tem permissão de edição neste arquivo. O resultado ficará só no relatório." |

## Pontos em aberto (validar com o Garba)
- Qual serviço da ADA guarda a chave da equipe (LLM BUS é a candidata) e o
  formato da chamada com imagem.
- Limite de uso por usuário (por exemplo, testes por dia), já que o consumo sai
  da conta da equipe.
- Se o relatório mostra os tokens consumidos no teste.
- Se a organização no Figma permite publicar plugin privado, e quem aprova.
- Code Connect na ligação com o Orquestra: exige token do Figma no servidor,
  porque o plugin não lê o Code Connect.
- Limite de frames por teste e tamanho das imagens enviadas.
- Se o usuário pode avaliar só os frames selecionados, em vez da página
  inteira.

## Critérios de validação desta etapa
- [ ] Usuário não informa nenhuma chave de IA.
- [ ] Chave da equipe só no servidor; nunca no plugin, banco, log ou relatório.
- [ ] Servidor só atende usuários logados da organização.
- [ ] Plugin avalia os frames de primeiro nível da página atual.
- [ ] Captura feita pelo plugin, sem token do Figma.
- [ ] Objetivo, persona, biblioteca e Orquestra seguem `00`, `03`, `06` e `08`.
- [ ] Resultado escrito como anotações do Dev Mode, sem alterar o design.
- [ ] Rodar de novo não duplica anotações.
- [ ] Resultado pode ser aplicado depois, se o plugin for fechado no meio.
- [ ] Consumo de IA registrado por usuário e por teste.
