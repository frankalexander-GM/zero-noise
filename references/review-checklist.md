# Checklist de revisión

Para revisar código ajeno. Los ítems están ordenados por gravedad: un fallo correcto es más urgente que un nombre mejorable, y decirlo en ese orden evita que la revisión parezca una lista de gustos.

## Cómo usarla

1. Ejecuta primero `scripts/audit_clean_code.py` sobre la ruta cambiada. Lo mecánico se resuelve sin discutir.
2. Lee el diff completo antes de escribir un solo comentario.
3. Recorre las secciones de arriba abajo.
4. Agrupa los hallazgos por gravedad en la respuesta, no por archivo.
5. Si algo es cuestión de gusto y no de corrección, dilo como sugerencia y no bloquees la revisión por ello.

## Bloqueante — el código no puede entrar así

- [ ] **No compila o no pasa el linter.** Sin excepciones.
- [ ] **Tests en rojo** que no estaban en rojo antes del cambio.
- [ ] **Error silenciado**: `catch` vacío, `except: pass`, `_ = err`, promesa sin `await` en una ruta de error.
- [ ] **Placeholder en producción**: `TODO`, `FIXME`, `pass`, `NotImplemented`, retorno falso que finge éxito.
- [ ] **Datos sensibles** en logs, en mensajes de error o en el repositorio (tokens, contraseñas, rutas con usuario).
- [ ] **Validación ausente en el límite**: entrada de usuario, HTTP, fichero o API externa sin comprobar.
- [ ] **Cambio de comportamiento no pedido** mezclado con el cambio solicitado.

## Grave — debería corregirse en esta revisión

- [ ] **Código muerto**: imports, variables, parámetros, funciones, ramas inalcanzables.
- [ ] **Duplicación de lógica** que ya existe en otro módulo.
- [ ] **Anidamiento** que obliga a leer cuatro niveles para entender una condición.
- [ ] **Función que hace más de una cosa**, o con más de tres parámetros sin justificar.
- [ ] **Error del dominio representado como string** suelto en lugar de un tipo.
- [ ] **Estado mutable compartido** entre hilos, peticiones o componentes sin control.
- [ ] **Consulta dentro de un bucle** (N+1) o trabajo pesado en el camino de una petición.
- [ ] **Recurso sin cerrar**: fichero, conexión, transacción.

## Mejora — no bloquea, pero se agradece

- [ ] **Nombres** que no describen: `data`, `temp`, `res`, `manager`, `util`.
- [ ] **Comentarios** que repiten el código, decorativos o desactualizados.
- [ ] **Números y cadenas mágicas** sin constante con nombre.
- [ ] **Abstracción con una sola implementación** que nadie pidió.
- [ ] **Dependencia nueva** para algo que la librería estándar cubre.
- [ ] **Formato inconsistente** con el resto del archivo.
- [ ] **Reformateo de líneas no relacionadas**, que infla el diff.

## Preguntas que conviene hacerse en cualquier cambio

- ¿Qué pasa si esto se ejecuta dos veces? ¿Es idempotente donde debe serlo?
- ¿Qué pasa si llega una lista vacía? ¿Y si llega `null`?
- ¿Qué se registra si falla, y basta para diagnosticarlo mañana?
- ¿El test nuevo falla si quito el arreglo? Si no falla, no prueba nada.
- ¿Hay algún caso que solo funciona en mi máquina?
- ¿El cambio toca algo que otro equipo consume sin avisar?

## Formato de la respuesta

Ordena por gravedad y sé concreto. Un comentario útil dice qué, dónde y por qué:

> `pedidos.ts:42` — el `catch` vacío se come el error de `guardarPedido`. Si falla la escritura, la API responde 200 igual y el pedido se pierde. Registrar con contexto y relanzar.

Un comentario inútil dice qué sin por qué:

> `pedidos.ts:42` — mejora el manejo de errores.

Cierra con un veredicto explícito: aprobado, aprobado con cambios menores, o cambios requeridos. Quien lee la revisión debe saber si puede fusionar sin adivinar.
