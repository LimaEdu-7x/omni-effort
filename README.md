# Omni Effort ⚡

> **Classificação de esforço, escolha de modelo e gestão de contexto para agentes de IA.**  
> 🇧🇷 **Projeto Brasileiro Open-Source** feito para decidir, antes de agir, quanto esforço cada tarefa merece e como executá-la com o menor custo que ainda entrega bem.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Origin: Brazil](https://img.shields.io/badge/Made%20in-Brazil%20%F0%9F%87%A7%F0%9F%87%B7-green.svg)](#-projeto-brasileiro-open-source)
[![Compatible: Claude Code](https://img.shields.io/badge/Compatible-Claude%20Code-purple.svg)](#-como-instalar)
[![Compatible: Maestri](https://img.shields.io/badge/Compatible-Maestri-orange.svg)](#-maestri)

Antes de qualquer ação, o agente classifica a tarefa como **baixo**, **médio** ou **alto** esforço, escolhe o modelo e o número de subagents, e sugere quando compactar (`/compact`) ou limpar (`/clear`) o contexto. Funciona na sessão atual e nos teammates do Maestri, cada um com seu modelo e effort.

---

## 🇧🇷 Projeto Brasileiro Open-Source

O **Omni Effort** faz parte da família Omni, skills open-source em português para agentes de IA. O foco é:
- **Custo proporcional:** tarefa simples não recebe modelo caro nem subagents desnecessários.
- **Contexto leve:** sinais claros de quando compactar e quando limpar, sem perder o estado da tarefa.
- **Ecossistema aberto:** funciona com Claude Code e com o Maestri, sem depender de serviço pago extra.

---

## 🎯 O que é o Omni Effort?

Agentes costumam usar o mesmo modelo, no mesmo esforço, para tudo. Isso gasta dinheiro em tarefa mecânica e deixa tarefa difícil com pouca profundidade.

O **Omni Effort** dá ao agente um método simples:
1. **Classificação antes de agir:** baixo, médio ou alto, com o motivo em uma linha.
2. **Modelo e subagents proporcionais:** cada nível tem um modelo sugerido e um limite de delegação.
3. **Delegação com critério:** só o que é independente vai para subagent, com objetivo, escopo e critério de pronto.
4. **Contexto com estado salvo:** antes de `/compact` ou `/clear`, o estado essencial é gravado em arquivo para continuar depois.
5. **Teammates do Maestri:** subir cada teammate com o modelo e o effort do nível da tarefa.

---

## 📊 Como funciona

| Nível | Quando | Modelo sugerido | Subagents |
| :--- | :--- | :--- | :--- |
| **Baixo** | Resposta direta, leitura de arquivo, mudança de uma linha, busca simples | Sessão atual ou `haiku` | Nenhum |
| **Médio** | Feature localizada, bug com causa provável, refactor em um módulo, revisão de diff | Sessão atual ou `sonnet` | 1–2, em paralelo quando independentes |
| **Alto** | Muitos arquivos, decisão de arquitetura, bug sem causa clara, migração, segurança | `opus` ou `fable` | Quantos forem necessários, cada um com escopo claro |

O nível sobe com risco (dados, segurança, produção), ambiguidade ou falha repetida, e desce quando a tarefa é mecânica.

---

## ⚡ Exemplos

Peça a tarefa normalmente, ou chame a skill diretamente:

```text
/omni-effort [tarefa]
```

Resposta esperada, no início:

```text
Esforço: médio (bug com causa provável em um módulo). Modelo: sonnet. Subagents: 2.
```

### Subir teammates no Maestri com modelo e effort definidos:
```bash
# Baixo: Haiku 5.5 com effort low
maestri recruit "Nome" --preset "Claude Code" --command "claude --model claude-haiku-5-5 --effort low"

# Médio: Sonnet 5.5 com effort medium
maestri recruit "Nome" --preset "Claude Code" --command "claude --model claude-sonnet-5-5 --effort medium"

# Trocar modelo ou effort de um teammate existente, sem perder conexões
maestri recruit "Nome" --replace "Nome" --preset "Claude Code" --command "claude --model claude-haiku-5-5 --effort low"
```

---

## 📦 Como instalar

### 1. Pelo instalador de skills (Recomendado)
```bash
npx skills add LimaEdu-7x/omni-effort
```

### 2. Instalação manual

#### 🟣 Claude Code
```bash
mkdir -p ~/.claude/skills/omni-effort
curl -sL https://raw.githubusercontent.com/LimaEdu-7x/omni-effort/main/SKILL.md -o ~/.claude/skills/omni-effort/SKILL.md
```

#### 🪐 Antigravity / Gemini CLI
```bash
mkdir -p ~/.gemini/config/skills/omni-effort
curl -sL https://raw.githubusercontent.com/LimaEdu-7x/omni-effort/main/SKILL.md -o ~/.gemini/config/skills/omni-effort/SKILL.md
```

---

## 🧭 Maestri

O Omni Effort foi pensado para o [Maestri](https://maestri.app), canvas que orquestra vários agentes em um mesmo espaço. Na prática:
- `maestri ask "Nome" "/model ..."` e `"/effort ..."` funcionam, mas gravam o padrão global do Claude Code. Para mudar só um teammate, use a flag `--effort` na subida.
- Envie slash commands pelo PowerShell. No Git Bash, `/model` vira caminho do Windows e o comando não é executado.
- `/clear` enviado via `maestri ask` zera o histórico do teammate.

---

## 🏆 Créditos & Referências

- **[Maestri](https://maestri.app)**: canvas visual para orquestração de múltiplos agentes de IA.
- **[Claude Code](https://docs.claude.com/en/docs/claude-code)**: agente de linha de comando que usa os níveis de effort e os comandos `/model`, `/effort`, `/compact` e `/clear`.

---

## 📄 Licença

Distribuído sob a licença **MIT**. Veja o arquivo [`LICENSE`](LICENSE) para mais detalhes.  
Feito com 💚 e 💛 no Brasil por [LimaEdu-7x](https://github.com/LimaEdu-7x).
