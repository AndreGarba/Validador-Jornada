# 02 — Normalização do Input da Persona

## Objetivo
Aceitar **qualquer** formato de documento de persona e reescrevê-lo no modelo
ideal (`01`) antes da validação. O usuário não precisa conhecer o template.

## Quando roda
Só quando o usuário sobe uma **persona nova**. Esta etapa é pulada:
- no modo somente heurísticas (não há persona);
- quando o usuário escolhe uma persona da biblioteca (`06`), que já está
  normalizada e confirmada.

## Fluxo
1. O usuário sobe um documento em qualquer estrutura (texto corrido, tópicos,
   outro template, transcrição de entrevista etc.).
2. O modelo lê e **mapeia** o conteúdo para os campos do modelo ideal.
3. Campos sem correspondência ficam **explicitamente vazios**. Nada é inventado.
4. O documento normalizado é exibido ao usuário, lado a lado com o original.
5. O usuário **confirma** ou **edita** antes de seguir para a validação (`03`).
6. Se o usuário editar, a versão editada é a que segue para a validação.
7. Depois da confirmação, a persona é salva na biblioteca (`06`), ligada ao
   produto informado.

## Objetivo de tarefa no documento enviado
A persona não tem campo de objetivo (`01`). Se o documento enviado trouxer um
objetivo de tarefa (ex.: "quer contratar um plano em 3 minutos"), esse trecho
vai para `nao_mapeado` com o motivo "objetivo de tarefa: informe no campo
Objetivo principal da tarefa". A normalização não preenche o campo de objetivo
da tela principal.

Preferências que valem para qualquer tarefa (ex.: "quer ver o preço antes de
dar dados") continuam indo para `criterios_sucesso`.

## Regra de ouro: não inventar
A normalização **reorganiza** o que existe; nunca cria conteúdo. Se o documento
não fala de contexto de uso, o campo fica vazio e perde pontos na assertividade.

Inventar dado mascara a qualidade real do input e corrompe o teste: um documento
raso pareceria rico. Normalização e validador trabalham juntos: **normaliza o
que existe, penaliza o que falta.**

### O que é permitido
- Mover uma frase para o campo correto.
- Separar um parágrafo em tópicos, preservando o sentido.
- Traduzir para português, preservando o sentido.
- Juntar no mesmo campo informações que estavam espalhadas.

### O que é proibido
- Inferir idade, ocupação ou nível digital que não estejam escritos.
- Completar um campo com "o comum" para aquele perfil.
- Reescrever frases genéricas como se fossem específicas.
- Descartar informação do original sem registrar.

## Rastreabilidade
Toda informação mapeada guarda o trecho de origem. Toda informação do original
que não couber em nenhum campo vai para `nao_mapeado`, para o usuário decidir.

## Entradas e saídas
- **Entrada:** texto do arquivo enviado (`.md` ou `.txt`; outros formatos são
  convertidos para texto antes).
- **Saída:** persona no template de `01` + objeto de rastreabilidade (formato
  em `05`, Prompt 1).

## Por que a confirmação do usuário importa
Mostrar o resultado normalizado antes de validar dá ao usuário a chance de
corrigir um mapeamento errado e cria transparência: ele vê exatamente o que
será testado.

## Prompt de normalização
Texto completo em `05-prompts-mestres.md`, Prompt 1.

## Critérios de validação desta etapa
- [ ] Normalização nunca inventa dados ausentes.
- [ ] Campos sem correspondência ficam vazios e perdem pontos na assertividade.
- [ ] Todo campo mapeado guarda o trecho de origem.
- [ ] Informação que não coube em nenhum campo aparece em `nao_mapeado`.
- [ ] Usuário confirma (ou edita) o documento normalizado antes de validar.
- [ ] Objetivo de tarefa encontrado no documento vai para `nao_mapeado`.
- [ ] Persona confirmada é salva na biblioteca com o produto informado.
- [ ] Testado com 3 personas em formatos diferentes (texto corrido, tópicos,
      outro template).
