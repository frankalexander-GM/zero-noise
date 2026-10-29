# zero-noise — reglas completas

> Skill de IA, todo en un archivo. Pégalo en ChatGPT, Claude, Gemini, Cursor o Copilot.
> Las reglas van primero; el auditor determinista está en `scripts/audit_clean_code.py`.

**Cuándo aplicarlo:** Escribe, refactoriza y revisa código con estándar de producción: cero código muerto, cero comentarios redundantes, cero placeholders/TODOs, cero imports o variables sin usar, cero duplicación, sin sobre-ingeniería y con errores controlados solo en los límites. Incluye un auditor determinista (scripts/audit_clean_code.py) que detecta código comentado, código muerto, prints de depuración, catch vacíos, funciones y archivos demasiado largos. Úsala cuando se pida escribir, generar, implementar, arreglar o refactorizar código y se exija calidad — «código limpio», «sin comentarios», «sin código muerto», «sin errores», «listo para producción», «sin basura», clean code, no dead code, no comments, production-ready — o cuando se pida auditar un archivo, módulo o repositorio en busca de ruido, imports sin usar, TODOs, código comentado o duplicación.

**Instrucción de activación (pégala antes de tu código):**

> Aplica el estándar zero-noise: cero código muerto, cero comentarios redundantes,
> cero placeholders, cero errores silenciados, cero duplicación y cero sobre-ingeniería.
> Verifica antes de entregar y reporta qué ejecutaste.

## Código limpio, sin ruido

### Principio

El código se lee diez veces más de lo que se escribe. Cada línea que dejas es una línea que alguien mantendrá, revisará y tendrá que entender. La calidad no se mide por lo que añades sino por lo que sobra.

Tres frases que gobiernan todo lo demás:

- **Antes de añadir, borra.** La mejor línea es la que no existe.
- **Antes de abstraer, espera.** La abstracción prematura cuesta más que la duplicación.
- **Antes de comentar, renombra.** Si necesitas explicar qué hace una línea, el problema es el nombre.

### Las diez reglas duras

#### 1. Cero código muerto

Nada de lo que dejas puede quedar sin uso. Un solo caso justifica borrar:

- imports, variables, parámetros o constantes que nadie lee.
- funciones, clases, métodos o exportaciones sin llamadores.
- ramas `if` cuya condición es siempre verdadera o falsa.
- flags y opciones que ninguna ruta consume.
- claves de configuración que ningún código consulta.
- compatibilidad con versiones que el proyecto ya no soporta.
- código "por si acaso" o "para más adelante".

Antes de crear algo, comprueba que no existe: `rg "nombre"`.

#### 2. Cero comentarios por defecto

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

#### 3. Cero placeholders

El código se entrega completo o no se entrega. Nunca:

- `TODO`, `FIXME`, `XXX`, `HACK`.
- `pass`, `...`, `NotImplemented`, `throw new Error("not implemented")`.
- funciones vacías "para que compile".
- datos de ejemplo hardcodeados (`"John Doe"`, `"test@test.com"`) fuera de tests.
- `return null` como "ya veremos".
- maquetación sin lógica real.

Si algo no se puede terminar, se dice explícitamente en la respuesta. No se esconde en el archivo.

#### 4. Cero errores silenciados

- Nada de `catch {}` vacío.
- Nada de tragarse una excepción para "que no falle".
- Nada de `except Exception: pass`.
- Nada de `if err != nil { }` sin tratar.

Si un error no se puede manejar en ese punto, se propaga. Si se maneja, se hace algo real: reintentar, degradar, registrar con contexto o relanzar con información.

#### 5. Cero duplicación

La misma lógica dos veces es deuda. La tercera vez, extrae. Pero:

- no extraigas por dos líneas parecidas que cambian por motivos distintos.
- DRY es sobre conocimiento duplicado, no sobre texto duplicado.
- Si dos bloques se parecen hoy pero evolucionan por separado, déjalos separados.

#### 6. Cero sobre-ingeniería

No añadas lo que nadie pidió:

- capas de abstracción con una sola implementación.
- interfaces con un único consumidor.
- patrones de diseño sin problema que resolver.
- configurabilidad que nadie va a configurar.
- manejo de casos que no pueden ocurrir.
- dependencias nuevas para algo que la librería estándar ya hace.

La solución más simple que cumple el contrato gana. Siempre.

#### 7. Nombres que se explican solos

- Prohibidos: `data`, `temp`, `aux`, `res`, `x`, `obj`, `thing`, `manager`, `helper`, `utils` como cajón de sastre.
- Booleanos con prefijo: `is`, `has`, `can`, `should`, `needs`.
- Funciones con verbo: `calculateTotal`, `sendInvoice`, `parseHeader`.
- Sin abreviaturas que no sean obvias en el dominio.
- La longitud del nombre debe ser proporcional al alcance de la variable.

Si no encuentras un nombre honesto, la función hace más de una cosa.

#### 8. Funciones de un solo nivel de abstracción

- Una función, una responsabilidad. Si la descripción lleva "y", son dos.
- Orientativo: hasta ~30 líneas. Más allá, sospecha.
- Máximo 3 parámetros; a partir de ahí, un objeto con nombre.
- Sin banderas booleanas que cambian el comportamiento (`render(true)` es ilegible).
- Anidamiento máximo 3 niveles. Más profundo: guard clauses y salidas tempranas.

#### 9. Errores en los límites, no en cada línea

- Valida en la frontera: entrada de usuario, HTTP, fichero, base de datos, API externa.
- Dentro del dominio, confía en los tipos ya validados. No repitas comprobaciones.
- Nada de `try/catch` defensivo alrededor de código propio que no lanza.
- Mensajes de error con qué falló y con qué dato, sin filtrar secretos.

#### 10. Consistencia con el repositorio, por encima de tu gusto

El estilo del proyecto manda sobre tus preferencias.

- Lee `.editorconfig`, la configuración del linter y del formateador.
- Mira dos archivos vecinos antes de escribir: imports, orden, nombres, manejo de errores.
- Respeta el idioma del código existente (identificadores en inglés o español, el que ya se use). No mezcles.
- No reformatees líneas que no cambias: infla el diff y esconde el cambio real.

### Flujo de trabajo

#### Paso 1 — Leer antes de escribir

Nunca escribas sobre un archivo que no has leído. Y no basta con el archivo objetivo:

- el archivo que vas a modificar, completo.
- dos vecinos del mismo directorio, para captar convenciones.
- la configuración del linter y del formateador.
- el punto de entrada o el test que ejercita ese código.
- `rg "nombre"` para descubrir si lo que vas a crear ya existe.

#### Paso 2 — Fijar el contrato

Antes de teclear, ten claro: qué entra, qué sale, qué errores puede producir, quién lo llama. Si algo de eso no está claro, se pregunta o se lee. No se inventa.

#### Paso 3 — Escribir el mínimo

- Primero el camino feliz, lo más corto posible.
- Después solo las ramas que el contrato exige.
- Nada de ramas para casos imposibles.
- Nada de generalidad que el contrato no pide.

#### Paso 4 — Limpiar antes de verificar

Pasa el checklist de abajo. Es más rápido limpiar ahora que explicar después.

#### Paso 5 — Verificar de verdad

Ejecuta la comprobación real. Si no la ejecutaste, no está verificada, y así hay que decirlo.

Del repositorio, en este orden: lo que declara su `package.json` / `Makefile` / `pyproject.toml` / CI. Si no hay nada declarado, usa el defecto del lenguaje:

| Lenguaje | Comprobación mínima |
|---|---|
| TypeScript / JS | `tsc --noEmit`, `eslint .`, tests |
| Python | `ruff check .`, `mypy .` si hay tipos, `pytest` |
| Go | `go vet ./...`, `go build ./...`, `go test ./...` |
| Rust | `cargo clippy -- -D warnings`, `cargo test` |
| Java / C# | compilación del módulo y tests del módulo |

#### Paso 6 — Reportar en tres líneas

1. Qué cambió y en qué archivos.
2. Qué se borró (código muerto, comentarios, imports).
3. Qué se ejecutó para verificar y con qué resultado, incluido lo que falló.

### Auditoría mecánica

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

### Checklist antes de entregar

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

### Cuándo la regla cede

Aplica el estándar al código que escribes. No conviertas la limpieza en un refactor no pedido.

- **Código legacy que no tocas en esta tarea:** déjalo. No reformatees, no limpies de paso. El diff debe contener solo tu cambio.
- **Código generado** (protobuf, ORM, clientes de API): no se edita a mano; si tiene ruido, se ajusta el generador o se excluye del linter.
- **Migraciones de base de datos:** se escriben una vez y quedan como histórico. No se "limpian".
- **Tests:** la claridad gana a DRY. Un test puede repetir setup a propósito si así se lee de un vistazo.
- **Comentarios que exige una herramienta:** directivas de linter, `# type: ignore`, `// @ts-expect-error`. Se quedan, pero cada una debe tener su motivo al lado.

### Referencias

- `references/anti-patterns.md` — pares de antes/después para los casos que más se repiten.
- `references/language-rules.md` — linters, detectores de código muerto e idiomas de error por lenguaje.
- `references/review-checklist.md` — checklist ampliada para revisar código ajeno.

---

## Anti-patrones: antes y después

Casos que se repiten en casi toda revisión. Cada par muestra el problema y el arreglo, no solo el estilo.

### 1. Comentario que repite el código

Antes:

```python
# Recorre la lista de usuarios
for usuario in usuarios:
    # Si el usuario está activo
    if usuario.activo:
        # Lo añade a la lista
        activos.append(usuario)
```

Después:

```python
for usuario in usuarios:
    if usuario.activo:
        activos.append(usuario)
```

Los tres comentarios desaparecen. Lo que queda se lee igual de bien y no puede quedar obsoleto.

### 2. Código comentado como "respaldo"

Antes:

```js
function calcularTotal(items) {
  // const iva = 0.21;
  // return items.reduce((acc, i) => acc + i.precio * 1.21, 0);
  return items.reduce((acc, i) => acc + i.precio, 0);
}
```

Después:

```js
function calcularTotal(items) {
  return items.reduce((acc, item) => acc + item.precio, 0);
}
```

Si esa versión con IVA hace falta, está en el historial de Git. Un archivo no es un cajón de recuerdos.

### 3. `catch` que se come el error

Antes:

```ts
try {
  await guardarPedido(pedido);
} catch (error) {}
```

Después:

```ts
try {
  await guardarPedido(pedido);
} catch (error) {
  logger.error("No se pudo guardar el pedido", { pedidoId: pedido.id, error });
  throw new Error(`Guardado fallido para el pedido ${pedido.id}`, { cause: error });
}
```

Un error silenciado convierte un fallo visible en una corrupción silenciosa.

### 4. Validación defensiva en todas partes

Antes:

```python
def aplicar_descuento(precio, porcentaje):
    if precio is None:
        return 0
    if not isinstance(precio, (int, float)):
        return 0
    if porcentaje is None:
        porcentaje = 0
    if not isinstance(porcentaje, (int, float)):
        porcentaje = 0
    if porcentaje < 0:
        porcentaje = 0
    return precio * (1 - porcentaje / 100)
```

Después:

```python
def aplicar_descuento(precio: float, porcentaje: float) -> float:
    return precio * (1 - porcentaje / 100)
```

La validación va en el límite: donde entra el dato del usuario o de la API. Dentro del dominio, los tipos ya están garantizados y repetir la comprobación oculta el bug real.

### 5. Parámetro bandera

Antes:

```js
function renderizar(items, compacto) {
  if (compacto) {
    return items.map((i) => i.nombre).join(", ");
  }
  return items.map((i) => `<li>${i.nombre}</li>`).join("");
}
```

Después:

```js
function renderizarLista(items) {
  return items.map((i) => `<li>${i.nombre}</li>`).join("");
}

function renderizarResumen(items) {
  return items.map((i) => i.nombre).join(", ");
}
```

Dos funciones con nombre propio se leen en la llamada; `renderizar(items, true)` no dice nada.

### 6. Anidamiento en pirámide

Antes:

```python
def procesar(pedidos):
    for pedido in pedidos:
        if pedido.validado:
            for linea in pedido.lineas:
                if linea.cantidad > 0:
                    if linea.stock > 0:
                        enviar(linea)
```

Después:

```python
def procesar(pedidos):
    lineas_validas = (
        linea
        for pedido in pedidos
        if pedido.validado
        for linea in pedido.lineas
        if linea.cantidad > 0 and linea.stock > 0
    )
    for linea in lineas_validas:
        enviar(linea)
```

O, si el `for` anidado se prefiere por claridad, salidas tempranas con `continue` en vez de cuatro niveles.

### 7. Números mágicos

Antes:

```python
if intentos > 3:
    esperar(86400)
```

Después:

```python
MAX_INTENTOS = 3
SEGUNDOS_ESPERA_TRAS_BLOQUEO = 24 * 60 * 60

if intentos > MAX_INTENTOS:
    esperar(SEGUNDOS_ESPERA_TRAS_BLOQUEO)
```

Un número sin nombre obliga a adivinar si 86400 es un día, catorce horas o un typo.

### 8. Cajón de sastre `utils`

Antes:

```
utils/
  helpers.js        // 400 líneas, 30 funciones sin relación
  misc.py
  common.ts
```

Después:

```
pricing/
  calcular-descuento.ts
pedidos/
  formatear-direccion.ts
```

Un `utils` crece sin límite porque nadie sabe qué hay dentro. Cada función junto a su dominio se encuentra antes y se borra sin miedo.

### 9. Placeholder que viaja a producción

Antes:

```js
export async function enviarFactura(factura) {
  // TODO: implementar el envío real
  return { ok: true };
}
```

Después: o se implementa, o se elimina la función y el código que la llama. No existe una tercera opción honesta. Si hay que posponerlo, se dice en la conversación, no se disfraza de código terminado.

### 10. Duplicación que se desincroniza

Antes:

```python
def validar_email_admin(email):
    if "@" not in email or "." not in email.split("@")[-1]:
        raise ValueError("email invalido")

def validar_email_cliente(email):
    if "@" not in email or "." not in email.split("@")[-1]:
        raise ValueError("email invalido")
```

Después:

```python
def validar_email(email: str) -> None:
    if "@" not in email or "." not in email.split("@")[-1]:
        raise ValueError(f"email invalido: {email}")
```

Dos copias idénticas divergen en cuanto alguien corrige una y olvida la otra. Dos copias que cambian por motivos distintos se dejan separadas.

### 11. Abstracción con una sola implementación

Antes:

```ts
interface ProveedorDePrecios {
  obtenerPrecio(sku: string): Promise<number>;
}

class ProveedorDePreciosImpl implements ProveedorDePrecios {
  async obtenerPrecio(sku: string): Promise<number> {
    return this.api.consultar(sku);
  }
}
```

Después:

```ts
class ClienteDePrecios {
  async obtenerPrecio(sku: string): Promise<number> {
    return this.api.consultar(sku);
  }
}
```

La interfaz se añade cuando existe la segunda implementación o cuando hay que sustituirla en tests por un doble. Antes de eso, solo añade un archivo que hay que mantener.

### 12. `try` alrededor de código propio

Antes:

```python
try:
    total = sumar(precios)
except Exception:
    total = 0
```

Después:

```python
total = sumar(precios)
```

`sumar` no lanza. Ese `try` solo garantiza que, el día que lance un `TypeError` por un bug real, nadie se entere.

---

## Reglas por lenguaje

Lo que cambia de un lenguaje a otro no es el principio —cero muerto, cero ruido— sino la herramienta con la que se comprueba y la forma idiomática de expresar lo mismo.

### TypeScript / JavaScript

**Verificación:** `tsc --noEmit`, `eslint .`, tests.

**Código muerto:**

- `ts-prune` o `knip` para exportaciones sin uso.
- `eslint` con `no-unused-vars`, `no-unused-imports`.
- Activá `noUnusedLocals` y `noUnusedParameters` en `tsconfig.json`.

**Errores:** en las fronteras, `unknown` en lugar de `any`; se estrecha con guardas. Nada de `catch (e: any)`. Los errores de dominio se modelan como clases propias o como uniones discriminadas, no como strings sueltos.

**Comentarios:** JSX admite `{/* */}`; también es ruido. Nada de `// eslint-disable-next-line` sin el motivo al lado.

**Trampas frecuentes:**

- `console.log` olvidado dentro de un `useEffect`.
- `useEffect` con dependencias vacías que oculta un bug de datos.
- `?.` encadenado para tapar un estado que no debería ser nulo.

### Python

**Verificación:** `ruff check .`, `mypy .` si hay anotaciones, `pytest`.

**Código muerto:**

- `vulture` para funciones, variables e imports sin uso.
- `ruff` con reglas `F401` (import sin usar) y `F841` (variable asignada y no usada).
- `deadcode` como alternativa.

**Errores:** excepciones propias que heredan de una base del dominio; `raise ... from err` para no perder la causa. Nunca `except: pass`. Nunca `except Exception` salvo en el borde superior de un worker o un CLI, y ahí se registra.

**Comentarios:** docstring solo en API pública, y solo si el proyecto ya los usa. `# type: ignore` y `# noqa` se permiten con el motivo al lado.

**Trampas frecuentes:**

- `if x is not None` repetido tras una firma que ya lo garantiza.
- `dict.get(k)` devolviendo `None` y luego comparando con cadenas.
- Argumentos mutables por defecto (`def f(x=[])`).

### Go

**Verificación:** `go vet ./...`, `go build ./...`, `go test ./...`.

**Código muerto:** el compilador ya falla con imports y variables sin usar. Para lo demás, `deadcode ./...` y `staticcheck`.

**Errores:** `if err != nil { return fmt.Errorf("contexto: %w", err) }`. Nunca `_ = err`. Nunca `if err != nil {}` vacío.

**Comentarios:** los comentarios de documento empiezan por el nombre del identificador y son obligatorios en paquetes exportados por convención de `golint`. Fuera de eso, cero.

**Trampas frecuentes:**

- `fmt.Println` de depuración que se queda en el código de producción.
- Interfaz definida en el paquete que la implementa en lugar del que la consume.

### Rust

**Verificación:** `cargo clippy -- -D warnings`, `cargo test`.

**Código muerto:** el compilador avisa con `dead_code`; no lo silencies con `#[allow(dead_code)]` salvo en código generado.

**Errores:** `Result` con `?`. `unwrap()` solo donde la imposibilidad está demostrada y comentada con el porqué. `expect()` con mensaje que explique la invariante.

**Comentarios:** `///` para documentación de API pública; `//` solo para el porqué no evidente. Nada de `// TODO`.

### Java / Kotlin

**Verificación:** compilación del módulo, `spotbugs`, `detekt` (Kotlin), tests.

**Código muerto:** el IDE lo marca, pero el build no falla. Revisa con `-Xlint:all` y `-Werror` en Java; `detekt` con `UnusedPrivateMember` en Kotlin.

**Errores:** excepciones específicas, nunca `catch (Exception e) {}`. En Kotlin, `Result` o excepciones, pero no las dos a la vez en la misma capa.

**Comentarios:** Javadoc/KDoc solo en API pública y sin repetir la firma (`@param id el id` no aporta nada).

### C# / .NET

**Verificación:** `dotnet build`, `dotnet test`, analizadores de Roslyn.

**Código muerto:** activa `TreatWarningsAsErrors` y las reglas `IDE0051`, `IDE0052`, `IDE0005`.

**Errores:** excepciones específicas; `catch (Exception) { }` está prohibido. `ArgumentNullException.ThrowIfNull` en las fronteras públicas.

**Comentarios:** XML doc solo en API pública. Nada de `// TODO`.

### Shell

**Verificación:** `shellcheck`.

**Reglas propias:** `set -euo pipefail` al principio. Comillas en todas las variables (`"$var"`). Nada de `echo` para depurar que se quede en el script.

### Regla transversal

Antes de discutir estilo, ejecuta el linter del proyecto. Si el proyecto no tiene linter configurado, propón uno en la conversación y no lo instales por tu cuenta dentro de una tarea que no lo pedía.

---

## Checklist de revisión

Para revisar código ajeno. Los ítems están ordenados por gravedad: un fallo correcto es más urgente que un nombre mejorable, y decirlo en ese orden evita que la revisión parezca una lista de gustos.

### Cómo usarla

1. Ejecuta primero `scripts/audit_clean_code.py` sobre la ruta cambiada. Lo mecánico se resuelve sin discutir.
2. Lee el diff completo antes de escribir un solo comentario.
3. Recorre las secciones de arriba abajo.
4. Agrupa los hallazgos por gravedad en la respuesta, no por archivo.
5. Si algo es cuestión de gusto y no de corrección, dilo como sugerencia y no bloquees la revisión por ello.

### Bloqueante — el código no puede entrar así

- [ ] **No compila o no pasa el linter.** Sin excepciones.
- [ ] **Tests en rojo** que no estaban en rojo antes del cambio.
- [ ] **Error silenciado**: `catch` vacío, `except: pass`, `_ = err`, promesa sin `await` en una ruta de error.
- [ ] **Placeholder en producción**: `TODO`, `FIXME`, `pass`, `NotImplemented`, retorno falso que finge éxito.
- [ ] **Datos sensibles** en logs, en mensajes de error o en el repositorio (tokens, contraseñas, rutas con usuario).
- [ ] **Validación ausente en el límite**: entrada de usuario, HTTP, fichero o API externa sin comprobar.
- [ ] **Cambio de comportamiento no pedido** mezclado con el cambio solicitado.

### Grave — debería corregirse en esta revisión

- [ ] **Código muerto**: imports, variables, parámetros, funciones, ramas inalcanzables.
- [ ] **Duplicación de lógica** que ya existe en otro módulo.
- [ ] **Anidamiento** que obliga a leer cuatro niveles para entender una condición.
- [ ] **Función que hace más de una cosa**, o con más de tres parámetros sin justificar.
- [ ] **Error del dominio representado como string** suelto en lugar de un tipo.
- [ ] **Estado mutable compartido** entre hilos, peticiones o componentes sin control.
- [ ] **Consulta dentro de un bucle** (N+1) o trabajo pesado en el camino de una petición.
- [ ] **Recurso sin cerrar**: fichero, conexión, transacción.

### Mejora — no bloquea, pero se agradece

- [ ] **Nombres** que no describen: `data`, `temp`, `res`, `manager`, `util`.
- [ ] **Comentarios** que repiten el código, decorativos o desactualizados.
- [ ] **Números y cadenas mágicas** sin constante con nombre.
- [ ] **Abstracción con una sola implementación** que nadie pidió.
- [ ] **Dependencia nueva** para algo que la librería estándar cubre.
- [ ] **Formato inconsistente** con el resto del archivo.
- [ ] **Reformateo de líneas no relacionadas**, que infla el diff.

### Preguntas que conviene hacerse en cualquier cambio

- ¿Qué pasa si esto se ejecuta dos veces? ¿Es idempotente donde debe serlo?
- ¿Qué pasa si llega una lista vacía? ¿Y si llega `null`?
- ¿Qué se registra si falla, y basta para diagnosticarlo mañana?
- ¿El test nuevo falla si quito el arreglo? Si no falla, no prueba nada.
- ¿Hay algún caso que solo funciona en mi máquina?
- ¿El cambio toca algo que otro equipo consume sin avisar?

### Formato de la respuesta

Ordena por gravedad y sé concreto. Un comentario útil dice qué, dónde y por qué:

> `pedidos.ts:42` — el `catch` vacío se come el error de `guardarPedido`. Si falla la escritura, la API responde 200 igual y el pedido se pierde. Registrar con contexto y relanzar.

Un comentario inútil dice qué sin por qué:

> `pedidos.ts:42` — mejora el manejo de errores.

Cierra con un veredicto explícito: aprobado, aprobado con cambios menores, o cambios requeridos. Quien lee la revisión debe saber si puede fusionar sin adivinar.
