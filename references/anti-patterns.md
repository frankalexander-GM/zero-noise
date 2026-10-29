# Anti-patrones: antes y después

Casos que se repiten en casi toda revisión. Cada par muestra el problema y el arreglo, no solo el estilo.

## 1. Comentario que repite el código

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

## 2. Código comentado como "respaldo"

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

## 3. `catch` que se come el error

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

## 4. Validación defensiva en todas partes

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

## 5. Parámetro bandera

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

## 6. Anidamiento en pirámide

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

## 7. Números mágicos

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

## 8. Cajón de sastre `utils`

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

## 9. Placeholder que viaja a producción

Antes:

```js
export async function enviarFactura(factura) {
  // TODO: implementar el envío real
  return { ok: true };
}
```

Después: o se implementa, o se elimina la función y el código que la llama. No existe una tercera opción honesta. Si hay que posponerlo, se dice en la conversación, no se disfraza de código terminado.

## 10. Duplicación que se desincroniza

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

## 11. Abstracción con una sola implementación

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

## 12. `try` alrededor de código propio

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
