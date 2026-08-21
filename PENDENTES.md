# Pendências — Gavira

Estado em 21/08/2026, antes da migração de servidor. Escrito para ser aplicado
DEPOIS que a infra voltar. Nada aqui foi aplicado em produção.

---

## 1. Prompt do agente da Marcela — pronto, falta subir no node

**Arquivo:** `prompt_marcela.md` (este repo) — é a versão testada e aprovada.
**Onde vai:** workflow `[ GAVIRA ] - OUVIDO` (`qxRmKy3XC9078i2I`), node **AGENTE MARCELA**,
campo `options.systemMessage` (com o `=` na frente, é campo de expressão). Publicar depois.

A produção está rodando a versão anterior, que tem o bug abaixo.

**O bug que isso corrige** (visto nas conversas 157/158, 20 e 21/08): a Marcela responde
"Ok" depois de o agente relatar o que fez; o agente trata como comando ambíguo, pergunta
"Ok sobre qual assunto?", ela responde "Este" — e o agente **manda a mensagem para o
paciente de novo**. O paciente recebeu duas vezes a mesma coisa, em dois casos diferentes
(Fabio em 20/08, Rosane em 21/08).

**A regra nova:** "Ok"/"certo"/"beleza"/"👍" depois do relato é encerramento, não pedido —
responde no máximo "Certo." e não chama ferramenta nenhuma. Instrução curta ("pode sim")
se aplica ao caso mais recente; só pergunta qual quando há dois ou mais casos abertos.

**Testado** no fluxo `[ TESTE ] - AGENTE MARCELA (isolado)` (`JjiKltSdM8t2ykzc`),
que lê este mesmo arquivo do GitHub:
- "Ok" depois do relato → "Certo." (antes: "Ok sobre qual assunto?" + reenvio)
- "pode sim" com um caso aberto → age no caso, pede só o dado que falta
- "pode fazer" com dois casos abertos → "Faço qual dos dois?"

⚠️ Existe também um `prompt_Marcela.md` (M maiúsculo) no repo — é a versão ANTIGA,
de antes da reescrita. O arquivo válido é o de M minúsculo.

---

## 2. Fila de correções levantadas no monitoramento — NÃO aprovadas

O Felipe decidiu em 20/08 não aplicar nenhuma destas. Ficam registradas com a evidência;
só aplicar se ele mandar.

**2.1 Cancelou → o próximo horário é criar, não remarcar** *(prompt da secretária)*
Conversa 308 (Vanessa), 19/08: a secretária cancelou a consulta de 20/08, a paciente
escolheu 24/08 às 16h15, e a secretária pediu à Agenda para REMARCAR — mas o evento não
existia mais. A Agenda varreu 90 dias, não achou nada e escalou. A paciente ficou sem
consulta nenhuma; a Marcela resolveu na mão 3 minutos depois. É o mais caro dos quatro.

**2.2 Trava de `ia_off` relida antes do envio** *(código, não prompt)*
Conversa 390 (Carlos), 20/08: a etiqueta foi aplicada às 17:21:45; a execução tinha
começado às 17:20:57 e a resposta saiu às 17:21:46, por cima da conversa que a humana
conduzia. A checagem acontece no início do fluxo. Correção: reler a etiqueta (ou uma
marca no Redis) no node imediatamente antes do POST no Chatwoot e abortar se apareceu.

**2.3 Não apresentar quem resolve + resumo repetido carrega o pedido novo** *(prompt)*
Cinco ocorrências de "encaminhei pra Marcela" a quem não perguntou pela Marcela (nem
sempre — em outras ela disse "pro responsável", então não é sistemático). E na conversa
390 o segundo resumo repetiu o primeiro (a garrafa d'água) e perdeu o pedido novo, que
era a pergunta sobre o terceiro exame.

**2.4 Fim de conversa não pode sair vazio** *(código, de preferência)*
Conversa 392 (Ariane), 20/08: ela respondeu "Não obrigada" ao "posso ajudar em mais
alguma coisa?", o agente devolveu vazio e a conversa morreu sem despedida. Cuidado: o
vazio é CERTO quando o paciente agradece depois de uma despedida de verdade (conversas
394 e 238). A correção precisa olhar o turno anterior: se foi pergunta, responde; se foi
tchau, cala.

---

## 3. Observações soltas, sem correção definida

- **Vocativo escapou uma vez** (conversa 391, 20/08): "Perfeito, Alice." O prompt proíbe
  chamar pela pessoa pelo nome. Uma ocorrência só; se repetir, a regra precisa endurecer.
- **Follow-up usa vocativo por desenho** ("Oi, Fulana") enquanto a secretária tem a regra
  oposta — os dois falam com o mesmo paciente. Vale alinhar.
- **`CHATWOOT_IMPORT_DATABASE_CONNECTION_URI` fora do ar** no container do Evolution
  (2662 erros ENOTFOUND). Quebra a gravação do `source_id`, o que impede distinguir com
  segurança mensagem da IA de mensagem digitada pela Marcela no painel. Precisa da
  credencial do Postgres do Chatwoot e restart do serviço, fora do horário de clínica.
- **Conversa começando no meio**: no fluxo isolado, uma primeira mensagem tipo "já mandei
  a foto da carteirinha", sem histórico, fez a secretária responder "Qual dos horários
  fica melhor pra você?" — inventou um passo. Em produção ela tem o histórico, o que
  reduz o risco. Observar.

---

## Já aplicado e no ar (para conferir depois da migração)

| o quê | onde | versão |
|---|---|---|
| Prompt da secretária v5.3 | node SECRETÁRIA do `NFj9ZvJdscOa0txC` | `e10cbe59` |
| `RESUMO:` removido do recado | nodes "Enviar resumo" e "ENVIA MSG1" | idem |
| Follow-up: pedido no presente ("Consegue") | node Redator do `TaYLzq6ht9PunFr5` | `dc8b093a` |
| Credencial OpenAI "Gavira" em todos os nodes | todos os fluxos do Gavira | — |
| Prompt do agente da Marcela v2 (sem o fix do "Ok") | node AGENTE MARCELA do `qxRmKy3XC9078i2I` | `61070141` |

O `prompt_secretaria.md` deste repo é igual ao que está no node em produção
(conferido por diff em 21/08). O `prompt_marcela.md` está À FRENTE da produção —
é o item 1 desta lista.
