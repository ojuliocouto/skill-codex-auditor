---
name: "codex-auditor"
description: >
  Auditor de código usando OpenAI Codex CLI como segunda perspectiva.
  Roda automaticamente após alterações de código para pegar erros que
  o Claude Code não identificou. Usa codex review e codex exec.
  Triggers: "auditar com codex", "codex review", "segunda opinião",
  "revisar com codex", "codex audit", "/codex-auditor"
---

# Codex Auditor - Segunda Perspectiva de Código

O Claude Code constrói. O Codex audita. Perspectivas diferentes pegam erros diferentes.

## Pré-requisitos (onboarding)

1. Instalar o Codex CLI da OpenAI: `npm install -g @openai/codex`.
2. Entrar na conta: `codex login` (conta ChatGPT ou chave de API da OpenAI).
3. Conferir: `codex --version` responde.
4. Rodar dentro de um repositório git, para o `git diff` achar os arquivos alterados.

Sem o Codex instalado e logado, a skill não roda. Cada chamada consome a cota ou os créditos da conta OpenAI.

## Quando Usar

- Após implementar features complexas
- Antes de deploy para produção
- Quando o código toca em: auth, pagamentos, dados sensíveis, APIs públicas
- Para revisar arquitetura e decisões técnicas
- Quando quiser uma segunda opinião independente

## Como Funciona

### 1. Review de Código (arquivos alterados)

Detecta os arquivos modificados e pede ao Codex para revisar:

```bash
# Review dos arquivos alterados no git
codex review

# Review de arquivo específico
codex exec "Review this code for bugs, security issues, and performance problems. Be ruthless: $(cat path/to/file.js)"
```

### 2. Auditoria Direcionada

```bash
# Segurança
codex exec "Security audit this code. Check for: injection, XSS, auth bypass, secret exposure, OWASP top 10. Code: $(cat path/to/file.js)"

# Performance
codex exec "Performance review: find N+1 queries, memory leaks, unbounded loops, missing indexes, unnecessary allocations. Code: $(cat path/to/file.js)"

# Arquitetura
codex exec "Architecture review: evaluate coupling, cohesion, separation of concerns, error handling patterns, edge cases. Code: $(cat path/to/file.js)"
```

### 3. Workflow Padrão (Claude Code + Codex)

```
Claude Code implementa alteração
         ↓
Identifica arquivos modificados (git diff --name-only)
         ↓
Para cada arquivo modificado:
  codex exec "Review this diff for bugs and security issues: $(git diff path/to/file)"
         ↓
Consolida achados do Codex
         ↓
Categoriza: MUST FIX / SHOULD FIX / INFO
         ↓
Se MUST FIX → Claude Code corrige antes de entregar
Se SHOULD FIX → reporta ao usuário
Se INFO → registra no relatório
```

## Comando de Auditoria Completa

Quando o usuário pedir auditoria com Codex, executar:

```bash
# 1. Pegar arquivos alterados
CHANGED=$(git diff --name-only HEAD~1 2>/dev/null || git diff --name-only --staged 2>/dev/null || echo "")

# 2. Se não tem git diff, pedir o arquivo específico
if [ -z "$CHANGED" ]; then
  echo "Nenhum arquivo alterado detectado. Especifique o arquivo."
  exit 1
fi

# 3. Rodar review
for FILE in $CHANGED; do
  if [ -f "$FILE" ]; then
    echo "=== Auditando: $FILE ==="
    codex exec "You are a senior code reviewer. Review this code thoroughly for:
1. Bugs and logic errors
2. Security vulnerabilities (injection, XSS, auth issues)
3. Performance problems (N+1, memory, loops)
4. Edge cases not handled
5. Breaking changes

Be specific with line references. Categorize each finding as MUST_FIX, SHOULD_FIX, or INFO.

File: $FILE
Code:
$(cat "$FILE")
"
  fi
done
```

## Integração como Hook (opcional)

Para rodar automaticamente após cada alteração de código:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "command": "codex review 2>/dev/null || true"
      }
    ]
  }
}
```

**Nota:** O hook automático consome tokens da OpenAI a cada edição. Recomenda-se usar sob demanda (trigger manual) em vez de automático, exceto em código crítico.

## Modos de Uso

| Modo | Comando | Quando |
|------|---------|--------|
| Review rápido | `codex review` | Após alterações |
| Audit completo | Workflow acima | Antes de deploy |
| Segunda opinião | `codex exec "Analyze..."` | Decisão técnica |
| Segurança | Audit direcionado | Auth, pagamentos, PII |

## Limitações

- Codex usa tokens da OpenAI (custo por chamada)
- Não substitui testes reais
- Respostas podem ser genéricas sem contexto suficiente - sempre incluir o código completo
- Para arquivos muito grandes (>500 linhas), dividir em seções
