# zero-noise

**Skill de IA que obliga a escribir código limpio: sin código muerto, sin comentarios, sin placeholders y sin errores silenciados.**

Funciona con Claude, ChatGPT, Gemini, Cursor, Copilot y cualquier agente que acepte un archivo de reglas. Incluye además un **auditor determinista en Python** que verifica que la IA cumplió de verdad, en lugar de fiarse de su palabra.

---

## El problema

Cuando le pides código a una IA, casi siempre obtienes lo mismo:

- comentarios que repiten lo que la línea ya dice;
- código comentado "por si acaso";
- `TODO`, `pass`, `NotImplemented`;
- imports y variables que nadie usa;
- un `catch {}` vacío que se come el error;
- abstracciones que nadie pidió;
- y un "listo, ya funciona" que nunca se ejecutó.

Esta skill convierte esas malas costumbres en reglas explícitas, verificables y con checklist.

## Las diez reglas duras

| # | Regla |
|---|---|
| 1 | Cero código muerto: imports, variables, parámetros, funciones y ramas sin uso |
| 2 | Cero comentarios por defecto, con solo tres excepciones justificadas |
| 3 | Cero placeholders: ni `TODO`, ni `pass`, ni `not implemented` |
| 4 | Cero errores silenciados: nada de `catch` vacío |
| 5 | Cero duplicación de conocimiento |
| 6 | Cero sobre-ingeniería: la solución más simple que cumple el contrato |
| 7 | Nombres que se explican solos |
| 8 | Funciones de un solo nivel de abstracción |
| 9 | Errores controlados en los límites, no en cada línea |
| 10 | Consistencia con el repositorio por encima del gusto personal |

Sobre los comentarios, la regla 2 solo admite tres casos: licencia, un "por qué" no deducible del código (con referencia al issue) y docstring de API pública cuando el proyecto ya los usa. Todo lo demás se borra y se resuelve renombrando.

## Instalación

| Herramienta | Qué hacer |
|---|---|
| **Claude Code** | `cp -r zero-noise ~/.claude/skills/` |
| **ChatGPT / Claude web / Gemini** | Pega `zero-noise-COMPLETO.md` como instrucciones personalizadas |
| **Cursor** | Copia ese archivo a `.cursor/rules/clean-code.mdc` |
| **GitHub Copilot** | Cópialo a `.github/copilot-instructions.md` |
| **Windsurf / Cline / Continue** | Úsalo como `.windsurfrules`, `.clinerules` o `CONVENTIONS.md` |
| **Cualquier otro agente** | Pásale `SKILL.md` y las referencias como contexto del proyecto |

## El auditor

Sin dependencias externas. Python 3.10 o superior.

```bash
python scripts/audit_clean_code.py <ruta>
python scripts/audit_clean_code.py . --only comments,dead-code
python scripts/audit_clean_code.py . --json
python scripts/audit_clean_code.py . --max-function-lines 30 --strict
```

Detecta:

| Regla | Qué busca |
|---|---|
| `comments/commented-code` | código comentado |
| `comments/todo` | `TODO`, `FIXME`, `XXX`, `HACK` |
| `comments/decorative` | banners y separadores |
| `dead-code/unused-import` | imports sin usar (Python, JS, TS) |
| `dead-code/empty-catch` | `catch {}` / `except: pass` |
| `dead-code/placeholder` | `NotImplemented`, `not implemented` |
| `debug/statement` | `console.log`, `print`, `dd()`, `debugger` |
| `size/long-function` | funciones por encima del umbral |
| `size/long-file` | archivos por encima del umbral |
| `style/deep-nesting` | anidamiento real por bloques, no por sangría |
| `style/trailing-whitespace` | espacios al final de línea |

Sale con **código 1** si encuentra algo, así que se puede usar en CI o en un pre-commit.

### Directivas

```python
# audit:allow debug        silencia esa familia de reglas en el archivo
# audit:ignore             silencia la línea
# audit:ignore-file        excluye el archivo entero
```

## Ejemplo

Entrada:

```python
# ===== UTILIDADES =====
import json

# Calcula el total
def calcular_total(items):
    total = 0
    for item in items:
        total += item["precio"]
    # total = total * 1.19
    # print(total)
    return total

def sin_terminar(dato):
    # TODO: validar
    pass
```

Salida del auditor:

```
sucio.py
  ERR      2  dead-code/unused-import  import sin usar: json
  ERR      3  comments/commented-code  codigo comentado
  WARN     5  comments/decorative      comentario decorativo
  ERR     10  comments/commented-code  codigo comentado
  ERR     11  comments/commented-code  codigo comentado
  ERR     15  comments/todo            marcador pendiente en el codigo

6 errores, 1 avisos, 0 informativos
```

Código limpio equivalente:

```python
def calcular_total(items: list[dict]) -> float:
    return sum(item["precio"] for item in items)
```

## Estructura

```
zero-noise/
├── SKILL.md                        reglas, flujo de trabajo y checklist
├── zero-noise-COMPLETO.md   todo en un archivo, para pegar
├── references/
│   ├── anti-patterns.md            12 pares antes/después
│   ├── language-rules.md           linters y detectores por lenguaje
│   └── review-checklist.md         revisión por gravedad
└── scripts/
    └── audit_clean_code.py         auditor determinista
```

## Nota sobre el propio auditor

El script se audita a sí mismo y sale limpio:

```bash
python scripts/audit_clean_code.py scripts/audit_clean_code.py
# 1 archivos analizados
# 0 errores, 0 avisos, 0 informativos
# Limpio.
```

Durante el desarrollo encontró cuatro fallos reales en su primera versión: dos falsos positivos (`from __future__` y sus propias regex), un archivo de 437 líneas contra su límite de 400 y funciones de 89 líneas contra el límite de 40. Los cuatro están corregidos.

## Licencia

MIT. Ver [LICENSE](LICENSE).

---

## English

**AI skill that enforces clean code: no dead code, no comments, no placeholders, no silenced errors.**

A rule set for coding agents plus a deterministic Python auditor that checks whether the model actually complied. Works with Claude Code, ChatGPT, Gemini, Cursor, Copilot and any agent that accepts a project rules file.

Zero dead code · zero redundant comments · zero placeholders · zero empty catch blocks · zero over-engineering · boundary-only error handling · conventions over personal taste.

MIT licensed.
