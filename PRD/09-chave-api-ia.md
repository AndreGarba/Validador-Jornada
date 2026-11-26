# 09 — Chave de API da IA

## Objetivo
O Validador não tem IA própria. Cada usuário roda os testes com a **sua
chave de API** do provedor interno de IA da organização, consumindo os tokens
da própria conta. Sem chave válida, nenhum teste roda.

## Provedor
- **Provedor interno da organização**, que oferece vários modelos e agentes.
- O modelo usado nos testes precisa **ler imagem**, porque as telas são
  avaliadas por screenshot (`04`).

## Primeira tela: configurar a chave
É a primeira tela da plataforma. Aparece sempre que o navegador não tem uma
chave salva.

| Campo / ação | Regra |
|---|---|
| Título | "Conecte sua chave de IA" |
| Texto de apoio | "Os testes usam a sua chave de API do provedor interno de IA e consomem os tokens da sua conta. A chave fica salva só neste navegador." |
| Chave de API | Campo de senha (texto mascarado), com botão de mostrar/ocultar. Obrigatório. |
| Validar e continuar | Valida a chave com uma chamada simples ao provedor. Desabilitado com o campo vazio. |

### Validação da chave
- **Chave válida:** salva no navegador e leva para a Home (`06`).
- **Chave inválida:** fica na tela com a mensagem "Chave inválida. Confira a
  chave e tente de novo."
- **Provedor fora do ar:** fica na tela com a mensagem "Não foi possível
  validar a chave agora. Tente de novo em alguns minutos."
- **Chave sem acesso a modelo que lê imagem:** fica na tela com a mensagem
  "Esta chave não tem acesso a um modelo compatível com análise de telas."

## Onde a chave fica guardada
- **Só no navegador do usuário.** Não é salva no servidor nem no banco.
- É enviada ao back-end **a cada teste**, junto com o pedido (`04`).
- O back-end usa a chave só durante a execução daquele teste, em memória, e a
  descarta ao final. A chave **nunca** é gravada em banco, em log ou no
  relatório.
- Trocar de navegador ou de máquina exige informar a chave de novo.
- Na tela, a chave salva aparece só mascarada, com os 4 últimos caracteres
  (ex.: `••••••••a3f9`).

## Trocar a chave (acesso discreto na Home)
- No cabeçalho da Home, à direita e em tamanho pequeno: ícone de chave +
  texto "Chave de API ••••a3f9".
- Ao clicar, abre um modal com:
  - a chave atual mascarada;
  - campo para a nova chave + botão **Validar e salvar** (mesma validação da
    primeira tela);
  - link **Remover chave**, que apaga a chave do navegador e volta para a
    primeira tela.
- Um teste em andamento continua com a chave com que começou; a nova chave
  vale para os próximos testes.

## Erros de chave durante o teste
Se a chave deixar de funcionar no meio de um teste (revogada, expirada ou sem
saldo de tokens):
- o teste para com status `erro_chave`;
- a tela mostra "Sua chave de IA parou de funcionar ({motivo}). Troque a chave
  e rode o teste de novo." com atalho para o modal de troca;
- o que já tiver sido escrito no Figma (`07`) é mantido, e o teste pode ser
  refeito, o que substitui as anotações (`07`).

## Pontos em aberto (validar com o Garba)
- Endereço (URL), formato da chamada e forma de autenticação da API do
  provedor interno. Preciso da documentação para fechar a validação da chave e
  as chamadas dos prompts (`05`).
- Se o usuário escolhe o modelo ou o agente usado nos testes (o provedor tem
  vários), ou se a plataforma fixa um modelo padrão que lê imagem.
- Se o relatório deve mostrar os tokens consumidos no teste, caso o provedor
  informe esse dado.

## Critérios de validação desta etapa
- [ ] Sem chave no navegador, a primeira tela é a de configurar a chave.
- [ ] Chave validada com o provedor antes de liberar a Home.
- [ ] Chave salva só no navegador; nunca em banco, log ou relatório.
- [ ] Chave enviada a cada teste e descartada pelo back-end ao final.
- [ ] Acesso discreto no cabeçalho da Home para trocar ou remover a chave.
- [ ] Chave exibida sempre mascarada, com os 4 últimos caracteres.
- [ ] Erro de chave durante o teste para o teste com mensagem e atalho para troca.
