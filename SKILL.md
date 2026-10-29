---
name: zero-noise
description: "Escribe, refactoriza y revisa código con estándar de producción: cero código muerto, cero comentarios redundantes, cero placeholders/TODOs, cero imports o variables sin usar, cero duplicación, sin sobre-ingeniería y con errores controlados solo en los límites. Incluye un auditor determinista (scripts/audit_clean_code.py) que detecta código comentado, código muerto, prints de depuración, catch vacíos, funciones y archivos demasiado largos. Úsala cuando se pida escribir, generar, implementar, arreglar o refactorizar código y se exija calidad — «código limpio», «sin comentarios», «sin código muerto», «sin errores», «listo para producción», «sin basura», clean code, no dead code, no comments, production-ready — o cuando se pida auditar un archivo, módulo o repositorio en busca de ruido, imports sin usar, TODOs, código comentado o duplicación."
---

# Código limpio, sin ruido

## Principio

El código se lee diez veces más de lo que se escribe. Cada línea que dejas es una línea que alguien mantendrá, revisará y tendrá que entender. La calidad no se mide por lo que añades sino por lo que sobra.

Tres frases que gobiernan todo lo demás:

- **Antes de añadir, borra.** La mejor línea es la que no existe.
- **Antes de abstraer, espera.** La abstracción prematura cuesta más que la duplicación.
- **Antes de comentar, renombra.** Si necesitas explicar qué hace una línea, el problema es el nombre.

## Las diez reglas duras

### 1. Cero código muerto

Nada de lo que dejas puede quedar sin uso. Un solo caso justifica borrar:

- imports, variables, parámetros o constantes que nadie lee.
- funciones, clases, métodos o exportaciones sin llamadores.
- ramas `if` cuya condición es siempre verdadera o falsa.
- flags y opciones que ninguna ruta consume.
- claves de configuración que ningún código consulta.
- compatibilidad con versiones que el proyecto ya no soporta.
- código "por si acaso" o "para más adelante".

Antes de crear algo, comprueba que no existe: `rg "nombre"`.

### 2. Cero comentarios por defecto

El código debe explicarse solo. Un comentario que repite lo que la línea ya dice es ruido, y el ruido enseña a ignorar los comentarios que sí importan.

**Prohibido siempre:**

- explicar qué hace la línea siguiente (`// suma uno al contador`).
- código comentado. Para eso existe el control de versiones. Se borra.
- autor, fecha, "creado por", "modificado por".
- separadores decorativos (`// ===== SECCIÓN =====`).
- `// eslint-disable` sin justificar el motivo concreto.
- comentarios que han quedado obsoletos respecto al código.

**Las únicas tres excepciones:**

1. Cabecera de licencia o copyright que el proyecto exige.
2. Un "por qué" que no se deduce leyendo el código: un workaround de un bug concreto, con referencia (`// Chrome <120 descarta el canvas si...`, `// ver issue #412`).
3. Docstring de API pública, solo si el proyecto ya usa docstrings y el contrato no es evidente por la firma.

Si dudas, borra el comentario y mejora el nombre.

### 3. Cero placeholders

El código se entrega completo o no se entrega. Nunca:

- `TODO`, `FIXME`, `XXX`, `HACK`.
- `pass`, `...`, `NotImplemented`, `throw new Error("not implemented")`.
- funciones vacías "para que compile".
- datos de ejemplo hardcodeados (`"John Doe"`, `"test@test.com"`) fuera de tests.
- `return null` como "ya veremos".
- maquetación sin lógica real.

Si algo no se puede terminar, se dice explícitamente en la respuesta. No se esconde en el archivo.

### 4. Cero errores silenciados

- Nada de `catch {}` vacío.
- Nada de tragarse una excepción para "que no falle".
- Nada de `except Exception: pass`.
- Nada de `if err != nil { }` sin tratar.

Si un error no se puede manejar en ese punto, se propaga. Si se maneja, se hace algo real: reintentar, degradar, registrar con contexto o relanzar con información.

### 5. Cero duplicación

La misma lógica dos veces es deuda. La tercera vez, extrae. Pero:

- no extraigas por dos líneas parecidas que cambian por motivos distintos.
- DRY es sobre conocimiento duplicado, no sobre texto duplicado.
- Si dos bloques se parecen hoy pero evolucionan por separado, déjalos separados.

### 6. Cero sobre-ingeniería

No añadas lo que nadie pidió:

- capas de abstracción con una sola implementación.
- interfaces con un único consumidor.
- patrones de diseño sin problema que resolver.
- configurabilidad que nadie va a configurar.
- manejo de casos que no pueden ocurrir.
- dependencias nuevas para algo que la librería estándar ya hace.

La solución más simple que cumple el contrato gana. Siempre.

### 7. Nombres que se explican solos

- Prohibidos: `data`, `temp`, `aux`, `res`, `x`, `obj`, `thing`, `manager`, `helper`, `utils` como cajón de sastre.
- Booleanos con prefijo: `is`, `has`, `can`, `should`, `needs`.
- Funciones con verbo: `calculateTotal`, `sendInvoice`, `parseHeader`.
- Sin abreviaturas que no sean obvias en el dominio.
- La longitud del nombre debe ser proporcional al alcance de la variable.

Si no encuentras un nombre honesto, la función hace más de una cosa.

### 8. Funciones de un solo nivel de abstracción

- Una función, una responsabilidad. Si la descripción lleva "y", son dos.
- Orientativo: hasta ~30 líneas. Más allá, sospecha.
- Máximo 3 parámetros; a partir de ahí, un objeto con nombre.
- Sin banderas booleanas que cambian el comportamiento (`render(true)` es ilegible).
- Anidamiento máximo 3 niveles. Más profundo: guard clauses y salidas tempranas.

### 9. Errores en los límites, no en cada línea

- Valida en la frontera: entrada de usuario, HTTP, fichero, base de datos, API externa.
- Dentro del dominio, confía en los tipos ya validados. No repitas comprobaciones.
- Nada de `try/catch` defensivo alrededor de código propio que no lanza.
- Mensajes de error con qué falló y con qué dato, sin filtrar secretos.

### 10. Consistencia con el repositorio, por encima de tu gusto

El estilo del proyecto manda sobre tus preferencias.

- Lee `.editorconfig`, la configuración del linter y del formateador.
- Mira dos archivos vecinos antes de escribir: imports, orden, nombres, manejo de errores.
- Respeta el idioma del código existente (identificadores en inglés o español, el que ya se use). No mezcles.
- No reformatees líneas que no cambias: infla el diff y esconde el cambio real.

## Flujo de trabajo

### Paso 1 — Leer antes de escribir

Nunca escribas sobre un archivo que no has leído. Y no basta con el archivo objetivo:

- el archivo que vas a modificar, completo.
- dos vecinos del mismo directorio, para captar convenciones.
- la configuración del linter y del formateador.
- el punto de entrada o el test que ejercita ese código.
- `rg "nombre"` para descubrir si lo que vas a crear ya existe.

### Paso 2 — Fijar el contrato

Antes de teclear, ten claro: qué entra, qué sale, qué errores puede producir, quién lo llama. Si algo de eso no está claro, se pregunta o se lee. No se inventa.

### Paso 3 — Escribir el mínimo

- Primero el camino feliz, lo más corto posible.
- Después solo las ramas que el contrato exige.
- Nada de ramas para casos imposibles.
- Nada de generalidad que el contrato no pide.

### Paso 4 — Limpiar antes de verificar

Pasa el checklist de abajo. Es más rápido limpiar ahora que explicar después.

### Paso 5 — Verificar de verdad

Ejecuta la comprobación real. Si no la ejecutaste, no está verificada, y así hay que decirlo.

Del repositorio, en este orden: lo que declara su `package.json` / `Makefile` / `pyproject.toml` / CI. Si no hay nada declarado, usa el defecto del lenguaje:

| Lenguaje | Comprobación mínima |
|---|---|
| TypeScript / JS | `tsc --noEmit`, `eslint .`, tests |
| Python | `ruff check .`, `mypy .` si hay tipos, `pytest` |
| Go | `go vet ./...`, `go build ./...`, `go test ./...` |
| Rust | `cargo clippy -- -D warnings`, `cargo test` |
| Java / C# | compilación del módulo y tests del módulo |

### Paso 6 — Reportar en tres líneas

1. Qué cambió y en qué archivos.
2. Qué se borró (código muerto, comentarios, imports).
3. Qué se ejecutó para verificar y con qué resultado, incluido lo que falló.

## Auditoría mecánica

Para revisar un directorio o un archivo completo, ejecuta el auditor incluido. Es determinista y no necesita que leas el código a mano:

```bash
python scripts/audit_clean_code.py <ruta>
python scripts/audit_clean_code.py <ruta> --json          # salida para procesar
python scripts/audit_clean_code.py <ruta> --max-function-lines 40
python scripts/audit_clean_code.py <ruta> --only dead-code,comments
```

Detecta: código comentado, `TODO`/`FIXME`, prints de depuración, `catch` vacíos, placeholders, funciones largas, archivos largos, anidamiento profundo, espacios al final de línea e imports presumiblemente sin usar.

Sale con código 1 si encuentra problemas, así que sirve en CI. Interpreta los hallazgos con criterio: es un heurístico, no un compilador, y un falso positivo se descarta explicando por qué.

Equivalente rápido con `rg` cuando no quieras ejecutar el script:

```bash
rg -n "TODO|FIXME|XXX|HACK" .
rg -n "console\.(log|debug|dir)|debugger|print\(|pp\(|var_dump|dd\(" .
rg -n "^\s*(//|#)\s*(const|let|var|function|def|class|if|for|while|return|import)\b" .
```

## Checklist antes de entregar

Recorre las doce preguntas. Si alguna es "no", no está listo.

1. ¿He borrado todos los imports, variables, parámetros y funciones sin uso?
2. ¿He borrado todos los comentarios que no sean una de las tres excepciones?
3. ¿He borrado el código comentado?
4. ¿Queda algún `TODO`, `FIXME`, `pass`, `...` o `not implemented`?
5. ¿Queda algún `console.log`, `print`, `dd()` o `debugger`?
6. ¿Hay algún `catch` o `except` vacío o que solo registra y sigue?
7. ¿Hay duplicación de lógica que debería estar en un solo sitio?
8. ¿He añadido abstracciones, capas o dependencias que nadie pidió?
9. ¿Los nombres describen lo que la cosa es, sin abreviaturas ni cajones de sastre?
10. ¿Alguna función pasa de ~30 líneas o de 3 parámetros sin justificación?
11. ¿El código compila, pasa el linter y los tests?
12. ¿Lo que digo en la respuesta coincide exactamente con lo que ejecuté?

## Cuándo la regla cede

Aplica el estándar al código que escribes. No conviertas la limpieza en un refactor no pedido.

- **Código legacy que no tocas en esta tarea:** déjalo. No reformatees, no limpies de paso. El diff debe contener solo tu cambio.
- **Código generado** (protobuf, ORM, clientes de API): no se edita a mano; si tiene ruido, se ajusta el generador o se excluye del linter.
- **Migraciones de base de datos:** se escriben una vez y quedan como histórico. No se "limpian".
- **Tests:** la claridad gana a DRY. Un test puede repetir setup a propósito si así se lee de un vistazo.
- **Comentarios que exige una herramienta:** directivas de linter, `# type: ignore`, `// @ts-expect-error`. Se quedan, pero cada una debe tener su motivo al lado.

## Referencias

- `references/anti-patterns.md` — pares de antes/después para los casos que más se repiten.
- `references/language-rules.md` — linters, detectores de código muerto e idiomas de error por lenguaje.
- `references/review-checklist.md` — checklist ampliada para revisar código ajeno.
