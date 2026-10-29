# Reglas por lenguaje

Lo que cambia de un lenguaje a otro no es el principio —cero muerto, cero ruido— sino la herramienta con la que se comprueba y la forma idiomática de expresar lo mismo.

## TypeScript / JavaScript

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

## Python

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

## Go

**Verificación:** `go vet ./...`, `go build ./...`, `go test ./...`.

**Código muerto:** el compilador ya falla con imports y variables sin usar. Para lo demás, `deadcode ./...` y `staticcheck`.

**Errores:** `if err != nil { return fmt.Errorf("contexto: %w", err) }`. Nunca `_ = err`. Nunca `if err != nil {}` vacío.

**Comentarios:** los comentarios de documento empiezan por el nombre del identificador y son obligatorios en paquetes exportados por convención de `golint`. Fuera de eso, cero.

**Trampas frecuentes:**

- `fmt.Println` de depuración que se queda en el código de producción.
- Interfaz definida en el paquete que la implementa en lugar del que la consume.

## Rust

**Verificación:** `cargo clippy -- -D warnings`, `cargo test`.

**Código muerto:** el compilador avisa con `dead_code`; no lo silencies con `#[allow(dead_code)]` salvo en código generado.

**Errores:** `Result` con `?`. `unwrap()` solo donde la imposibilidad está demostrada y comentada con el porqué. `expect()` con mensaje que explique la invariante.

**Comentarios:** `///` para documentación de API pública; `//` solo para el porqué no evidente. Nada de `// TODO`.

## Java / Kotlin

**Verificación:** compilación del módulo, `spotbugs`, `detekt` (Kotlin), tests.

**Código muerto:** el IDE lo marca, pero el build no falla. Revisa con `-Xlint:all` y `-Werror` en Java; `detekt` con `UnusedPrivateMember` en Kotlin.

**Errores:** excepciones específicas, nunca `catch (Exception e) {}`. En Kotlin, `Result` o excepciones, pero no las dos a la vez en la misma capa.

**Comentarios:** Javadoc/KDoc solo en API pública y sin repetir la firma (`@param id el id` no aporta nada).

## C# / .NET

**Verificación:** `dotnet build`, `dotnet test`, analizadores de Roslyn.

**Código muerto:** activa `TreatWarningsAsErrors` y las reglas `IDE0051`, `IDE0052`, `IDE0005`.

**Errores:** excepciones específicas; `catch (Exception) { }` está prohibido. `ArgumentNullException.ThrowIfNull` en las fronteras públicas.

**Comentarios:** XML doc solo en API pública. Nada de `// TODO`.

## Shell

**Verificación:** `shellcheck`.

**Reglas propias:** `set -euo pipefail` al principio. Comillas en todas las variables (`"$var"`). Nada de `echo` para depurar que se quede en el script.

## Regla transversal

Antes de discutir estilo, ejecuta el linter del proyecto. Si el proyecto no tiene linter configurado, propón uno en la conversación y no lo instales por tu cuenta dentro de una tarea que no lo pedía.
