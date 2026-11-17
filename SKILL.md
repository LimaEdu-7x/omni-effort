---
name: omni-effort
description: Classifica automaticamente o esforço de cada tarefa não trivial (baixo, médio, alto) e escolhe modelo, número de subagents, quando sugerir /compact e quando sugerir /clear. Aplique em toda tarefa de código, bug, feature, refactor, revisão, pesquisa ou orquestração de agentes, sem esperar pedido do usuário. Compatível com o Maestri, com modelo e effort por teammate.
user-invocable: true
---

# Omni Effort ⚡

Esta skill decide, antes de agir, quanto esforço uma tarefa merece e como executá-la com o menor custo que ainda entregue bem. Ela vale para a sessão atual e para os subagents que você criar, inclusive teammates do Maestri.

## Modo automático

- Aplique esta skill sozinha, no início de toda tarefa não trivial. O usuário não precisa chamá-la nem digitar comandos.
- Só tarefas triviais (uma pergunta curta, uma correção de digitação) ficam fora. Na dúvida, aplique.
- O usuário só precisa digitar `/model`, `/effort`, `/compact` ou `/clear` quando você sugerir. Nunca peça para ele rodar um comando que você possa rodar sozinho (ex.: `maestri recruit`, `maestri ask`, gravar o arquivo de estado).
- Limite real: a sessão principal não troca de modelo nem de effort sozinha. Quando o nível pedir outro modelo, diga a linha exata para o usuário digitar e continue com o que já pode fazer.

## Limites que você precisa respeitar

- **Você não troca o modelo nem o effort da sessão principal.** `/model`, `/effort`, `/clear` e `/compact` são comandos do usuário. Quando precisar de um deles, diga ao usuário qual comando digitar e por quê. Não finja que executou.
- **Com o Maestri, `maestri ask "Nome" "/model ..."` e `"/effort ..."` funcionam**, mas gravam o padrão global do Claude Code. Para mudar só um teammate, use a flag na subida (seção 5).
- **Subagents aceitam `model` na chamada do Agent tool.** Os valores são `haiku`, `sonnet`, `opus` e `fable`.
- **Modelos atuais:** Fable 5.2 (`claude-fable-5-2`), Opus 5.5 (`claude-opus-5-5`), Sonnet 5.5 (`claude-sonnet-5-5`), Haiku 5.5 (`claude-haiku-5-5`).

## 1. Classifique o esforço

Antes de qualquer ação, responda em uma linha: **baixo**, **médio** ou **alto**, e o motivo.

| Nível | Quando | Modelo sugerido | Subagents |
|-------|--------|-----------------|-----------|
| **Baixo** | Resposta direta, leitura de arquivo, mudança de uma linha, busca simples, formatação | Sessão atual ou `haiku` | Nenhum |
| **Médio** | Feature localizada, bug com causa provável, refactor em um módulo, revisão de um diff | Sessão atual ou `sonnet` | 1–2, em paralelo quando as partes forem independentes |
| **Alto** | Muitos arquivos, decisão de arquitetura, bug sem causa clara, migração, segurança ou dinheiro | `opus` ou `fable` | Quantos forem necessários, cada um com escopo claro |

Suba o nível quando houver risco (dados, segurança, produção), ambiguidade ou falha repetida. Desça quando a tarefa for mecânica.

## 2. Delegue com critério

- Delegue só o que é **independente**. Tarefa sequencial, em que uma etapa depende da anterior, fica com você.
- Cada subagent recebe: objetivo, arquivos ou áreas permitidos, critério de pronto e formato da resposta. Sem isso ele reinventa o contexto.
- Não há limite fixo de subagents. Crie quantos a tarefa justificar, e prefira poucos bem definidos a muitos vagos.
- Um subagent que volta com algo inconsistente é descartado ou refeito. Não o remende em silêncio.

## 3. Contexto: quando compactar ou limpar

Acompanhe o tamanho da conversa. Sinais de que é hora de agir:

- **Sugira `/compact`** quando a conversa estiver longa, ainda no mesmo assunto, e o histórico antigo (saídas grandes de ferramentas, tentativas já descartadas) não for mais necessário. Também antes de uma etapa grande que vai encher o contexto de novo. Grave o estado essencial antes (seção 4).
- **Não sugira `/compact`** em conversa curta: o Claude Code responde "Not enough messages to compact."
- **Sugira `/clear`** quando a tarefa acabou e a próxima é outro assunto, ou quando o contexto ficou poluído com tentativas erradas que confundem o modelo. Antes, grave o estado essencial.
- **Nunca** sugira `/clear` no meio de uma tarefa sem o estado salvo.
- Em teammate do Maestri, `/clear` enviado via `maestri ask` zera o histórico dele (limpa a tela e o contexto).
- Os critérios de `/compact` acima são hipótese: ainda não há medição do efeito numa conversa longa. Ajuste se a prática mostrar outra coisa.

Ao sugerir qualquer um dos dois, escreva a linha exata para o usuário: "Digite `/compact` (ou `/clear`) para continuar com contexto leve."

## 4. Memória e persistência

Antes de compactar ou limpar, grave um arquivo de estado em `<projeto>/.claude/esforco/estado.md` (ou no diretório temporário da sessão, se não houver projeto). Conteúdo mínimo:

- Objetivo atual e critério de pronto.
- Decisões tomadas e o motivo de cada uma.
- Arquivos alterados e pendências.
- Próximo passo exato.

Regras:

- Não grave segredos, tokens ou dados pessoais.
- Não grave o que o código ou o git já registram.
- Sobrescreva o arquivo ao fim de cada tarefa. Não acumule histórico ali.
- Ao começar uma tarefa, leia o arquivo se ele existir e siga o próximo passo.
- Se algo valer para o longo prazo (preferência, decisão), salve pela memória persistente do assistente, não aqui.

## 5. Compatibilidade com o Maestri

Use esta seção só quando o `maestri` estiver disponível (ou quando a sessão indicar o Maestri).

1. **Antes de recrutar, rode `maestri list`.** Se já existe um teammate com o papel certo, use `maestri ask` em vez de criar outro.
2. **Subir um teammate com modelo e effort definidos**, sem gravar o padrão global:
   `maestri recruit "Nome" --preset "Claude Code" --command "claude --model claude-haiku-5-5 --effort low"`
   Para médio, use `--model claude-sonnet-5-5 --effort medium`. Para alto, `--model claude-opus-5-5` ou `claude-fable-5-2`, com o effort que a tarefa pedir.
3. **Trocar o modelo ou o effort de um teammate existente**: `maestri recruit "Nome" --replace "Nome" --preset "Claude Code" --command "claude --model <id> --effort <nível>"`. Use `--replace`, nunca `dismiss` seguido de `recruit`, porque o `dismiss` apaga o nó e pode deixar notas ou portais órfãos. O `--replace` preserva conexões e posição, mas reinicia o processo.
4. **O histórico de chat se perde no reinício.** Depois de trocar o modelo ou o papel, envie um briefing com `maestri ask` contendo o estado do arquivo `estado.md`.
5. **Slash commands via `maestri ask`**: envie pelo PowerShell. No Git Bash, `/model` vira caminho do Windows e o comando não roda.
6. **Use o mesmo formato de comando do preset.** Coloque as flags depois do programa, por exemplo `claude --model opus`. Um wrapper escondido impede a retomada da sessão.
7. **Ao delegar em lote**, use `maestri ask --batch` para tarefas independentes. Assim elas rodam em paralelo.
8. **Termine cada tarefa enviada ao teammate** com a linha de retorno: `maestri ask "Seu Nome" "<resultado>"`.

## 6. Formato da resposta ao usuário

- Comece com a classificação: `Esforço: médio (motivo curto). Modelo: sonnet. Subagents: 2.`
- Diga o que foi delegado e o que ficou com você.
- Se precisar de `/model`, `/effort`, `/compact` ou `/clear`, diga o comando exato no fim da resposta.
- Ao final, informe o que não foi verificado, se houver algo.
