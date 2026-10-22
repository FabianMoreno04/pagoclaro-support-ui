# PagoClaro Support UI

Interfaz mínima para soporte N3: diagnosticar y conciliar un pago por su referencia externa usando la API de `pagoclaro-reconciliation`.

Es una sola página estática (`index.html`): HTML, CSS y JavaScript sin dependencias ni paso de compilación.

## Requisitos
- Backend `pagoclaro-reconciliation` en ejecución en `http://localhost:8080` (con Oracle y los scripts cargados, ver el README del backend).
- Node.js, solo para servir el archivo.

## Ejecutar


```bash
npx serve -l 5173 .             # alternativa con Node.js
```

Abrir http://localhost:5173

**Debe ser el puerto 5173.** El backend solo permite CORS desde `http://localhost:5173` y `http://127.0.0.1:5173`; con otro puerto el navegador bloquea las peticiones y la página muestra "No se pudo contactar el backend".

## Cómo se usa

1. Escribir la referencia externa del pago (por ejemplo `PAY-BX17`). La página la convierte a mayúsculas.
2. **Diagnosticar (sin cambios)**: llama a `GET /api/support/payments/{ref}/diagnosis`. Es una simulación: no escribe nada. Los valores "después" son proyectados.
3. **Conciliar**: pide confirmación y llama a `POST /api/support/payments/{ref}/reconcile`, que aplica la corrección. Conviene diagnosticar antes.

## Qué muestra

La tarjeta de resultado trae el estado antes y después, el débito neto antes y después, y los reversos creados.

| Color | Resultado | Significado |
|---|---|---|
| Verde | `CONSISTENT` | El pago ya es coherente; no hay efectos. |
| Azul | `RECONCILED` | Había inconsistencia con evidencia suficiente y se corrigió (o se corregiría, si es simulación). |
| Naranja | `MANUAL_REVIEW` | Evidencia contradictoria o insuficiente. Muestra "Detenerse: requiere revisión manual"; no se modifica nada. |

Errores que puede mostrar (mensajes genéricos, sin detalles internos):

| Mensaje | Causa |
|---|---|
| `PAYMENT_NOT_FOUND` | La referencia no existe. |
| `INVALID_REFERENCE` | Formato inválido: de 3 a 40 caracteres entre letras mayúsculas, dígitos y guion. |
| `CONCURRENT_MODIFICATION_RETRY` | Otra solicitud modificó el pago al mismo tiempo; revisar el estado antes de reintentar. |
| `INTERNAL_ERROR` | Fallo inesperado; el detalle queda solo en los logs del backend. |
| No se pudo contactar el backend | El backend no está corriendo o el puerto de la página no es el 5173. |

## Limitaciones
- La página muestra el resultado de la operación, no el contenido del ledger ni de la auditoría. Para verificar los movimientos y los registros de `SUPPORT_AUDIT` hay que consultar la base de datos.
- La URL del backend está fija en `index.html` (constante `API`). Para apuntar a otro servidor hay que editarla y permitir ese origen en el CORS del backend.
- No hay autenticación; es una herramienta de demostración para entorno local.

## Seguridad
El contenido devuelto por la API se inserta con `textContent`, nunca como HTML, para evitar inyección de código en la página.