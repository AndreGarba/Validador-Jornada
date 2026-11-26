# 06 — Tela Principal e Biblioteca de Personas

## Objetivo
Definir os campos da tela principal, onde o usuário configura o teste, e as
regras da biblioteca de personas, onde ficam as personas já enviadas,
organizadas por produto.

Na implementação, os componentes visuais seguem o Orquestra primeiro e o Figma
como fallback (regras de UI do projeto).

## Acesso à Home
A Home só abre com uma chave de API válida salva no navegador; sem ela, o
usuário vai para a primeira tela de configuração da chave (`09`).

No cabeçalho da Home, à direita e discreto: ícone de chave + "Chave de API
••••{últimos 4}", que abre o modal para trocar ou remover a chave (`09`).

## Campos da tela principal
| Ordem | Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|---|
| 1 | Link do Figma ou do protótipo | Texto (URL) | Sim | Aceita link do Figma ou URL `https` (`04`). Embaixo do campo, a observação sobre separar a jornada numa página própria (`07`). |
| 2 | Objetivo principal da tarefa | Texto longo | Sim | O que deve ser concluído na jornada (ex.: "Contratar um seguro em menos de 3 minutos"). Vale nos dois modos. |
| 3 | Persona | Bloco com duas opções | Não | Subir persona nova **ou** escolher da biblioteca. Sem persona: modo somente heurísticas. |
| 4 | Validar componentes do Orquestra | Liga/desliga + Stack + Versão | Não | Desligado por padrão. Ligado, exige stack e versão. Regras em `08`. |

**Não existe seletor de heurísticas.** Todo teste avalia todas as heurísticas
do catálogo (`00`). A tela mostra só um texto informativo: "Este teste avalia
todas as heurísticas: Nielsen, Krug, Acessibilidade, Dark patterns e UX
cultural."

O botão **Iniciar teste** fica desabilitado até link e objetivo estarem
preenchidos, enquanto a persona escolhida estiver abaixo de 40%, e enquanto o
seletor do Orquestra estiver ligado sem stack ou versão.

A tela mostra sempre o modo resultante:
- com persona: "Persona + heurísticas — assertividade XX%";
- sem persona: "Somente heurísticas — assertividade 100%".
- com o seletor do Orquestra ligado, acrescenta: "+ componentes do Orquestra
  ({stack} {versão})".

Para link do Figma, a tela também mostra onde o resultado será escrito
(`07`): "Resultado em comentários do Figma", "Resultado em anotações do Dev
Mode" ou o aviso de que o resultado ficará só na plataforma.

## Bloco de persona
O usuário escolhe uma das duas opções. Escolher uma limpa a outra.

### Opção A — Subir persona nova
1. Campo de upload (`.md`, `.txt`), em qualquer formato de texto (`02`).
2. Campo **Produto**: lista dos produtos existentes, com a opção
   "+ Criar novo produto" (abre um campo de nome).
   - Obrigatório quando há arquivo.
   - Nome do produto único, sem diferenciar maiúsculas.
3. Ao enviar: normalização → confirmação (`02`) → assertividade (`03`) → a
   persona é salva na biblioteca, ligada ao produto, com o nome de exibição
   `{produto} — {nome}`.
4. Se o documento não trouxer o nome da persona, a tela de confirmação pede
   que o usuário informe um nome antes de salvar.

### Opção B — Escolher da biblioteca
1. Dropdown **Produto**: filtra as personas daquele produto (opção "Todos os
   produtos" por padrão).
2. Dropdown **Persona**: lista as personas do produto filtrado.
   Cada item mostra:
   - nome de exibição `{produto} — {nome}` (ex.: "App de Seguros — Marina
     Silva"), regra de `01`;
   - selo de assertividade (faixas de `03`);
   - ação **Baixar .md**;
   - ação **Excluir** (só para quem criou a persona).
3. Personas abaixo de 40% aparecem **desabilitadas** (não podem ser escolhidas),
   com o aviso "Assertividade insuficiente — baixe, enriqueça e envie de novo".
   As ações Baixar e Excluir continuam disponíveis.
4. Persona escolhida: não passa de novo pela normalização nem pela
   assertividade; usa o que foi salvo.

### Estado vazio
Se a biblioteca não tiver personas (ou o produto filtrado não tiver nenhuma),
o dropdown mostra "Nenhuma persona para este produto" e um atalho para a
Opção A.

## Biblioteca de personas
### Visibilidade
Todas as personas são visíveis para **todos os usuários** da plataforma.

### Baixar .md
- Baixa a versão **normalizada**, no template de `01`, com o `produto`
  preenchido no frontmatter.
- Nome do arquivo: `{produto}-{nome-da-persona}.md` (minúsculas, com hífen).

### Excluir
- Só **quem criou** a persona pode excluí-la. Para os demais usuários, a ação
  **Excluir** não aparece no item.
- Pede confirmação: "Excluir a persona {produto} — {nome}? Ela some da
  lista para todos os usuários. Testes já feitos com ela não mudam."
- Exclusão lógica (`04`): a persona some da lista e do dropdown, e não pode ser
  usada em testes novos.
- Testes já feitos mantêm a cópia da persona usada e continuam abrindo.

## Pontos em aberto (validar com o Garba)
- Se produtos podem ser renomeados ou excluídos, e o que acontece com as
  personas ligadas a eles.

## Critérios de validação desta etapa
- [ ] Home só abre com chave de API válida; acesso discreto para trocar a chave.
- [ ] Link e objetivo da tarefa obrigatórios.
- [ ] Observação de página própria exibida embaixo do campo de link (`07`).
- [ ] Seletor do Orquestra com stack e versão (`08`).
- [ ] Tela informa o canal de saída no Figma antes do início (`07`).
- [ ] Sem seletor de heurísticas; texto informa que o catálogo completo é avaliado.
- [ ] Objetivo da tarefa é um campo da tela, fora da persona.
- [ ] Persona opcional, com as opções "subir nova" e "escolher da biblioteca".
- [ ] Persona nova exige produto, com opção de criar um produto na hora.
- [ ] Dropdown filtra por produto e mostra assertividade de cada persona.
- [ ] Toda persona aparece como `{produto} — {nome}` na lista, no dropdown,
      na confirmação de exclusão e nos relatórios.
- [ ] Personas abaixo de 40% aparecem desabilitadas no dropdown.
- [ ] Baixar entrega o `.md` normalizado.
- [ ] Excluir só aparece para quem criou a persona, pede confirmação e não
      altera testes já feitos.
- [ ] Biblioteca visível para todos os usuários.
