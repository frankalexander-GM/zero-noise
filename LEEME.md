# Cómo usar esta skill con otras IA

Nombre de la skill: **zero-noise**

## 1. IA de chat (ChatGPT, Claude web, Gemini, Grok)

Abre `zero-noise-COMPLETO.md`, cópialo entero y pégalo al inicio de la conversación
(o guárdalo como "instrucciones personalizadas" / "custom instructions" / GPT personalizado).

Después pide lo que necesites: el modelo ya tendrá las reglas cargadas.

## 2. Cursor

Copia el contenido de `zero-noise-COMPLETO.md` a `.cursor/rules/clean-code.mdc`
o a `.cursorrules` en la raíz de tu proyecto.

## 3. Claude Code

```
mkdir -p ~/.claude/skills
cp -r zero-noise ~/.claude/skills/
```

O añade el contenido de `SKILL.md` a tu `CLAUDE.md` del proyecto.

## 4. GitHub Copilot

Crea `.github/copilot-instructions.md` con el contenido de `zero-noise-COMPLETO.md`.

## 5. Windsurf / Cline / Continue / Aider

Usa `zero-noise-COMPLETO.md` como archivo de reglas del proyecto
(`.windsurfrules`, `.clinerules`, `.continuerules`, `CONVENTIONS.md`).

## 6. El auditor (opcional, pero es lo mejor)

Necesita Python 3.10 o superior. Sin dependencias externas.

```bash
python scripts/audit_clean_code.py <ruta>
python scripts/audit_clean_code.py . --only comments,dead-code
python scripts/audit_clean_code.py . --json
```

Sale con código 1 si encuentra problemas, así que puedes meterlo en un pre-commit
o en un paso de CI. Ninguna IA lo necesita para funcionar: es para verificar
que la IA cumplió.

## Estructura

```
zero-noise/
├── SKILL.md                        reglas + flujo de trabajo
├── zero-noise-COMPLETO.md   todo en un archivo, para pegar
├── LEEME.md                        esto
├── references/
│   ├── anti-patterns.md
│   ├── language-rules.md
│   └── review-checklist.md
└── scripts/
    └── audit_clean_code.py
```
