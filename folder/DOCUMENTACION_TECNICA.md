# Autos Colombia — documentación de ingeniería

## 1. Objetivo y alcance
Sistema web para administrar usuarios/clientes, vehículos, celdas, entradas y salidas, novedades y mensualidades. La aplicación registra movimientos históricos y muestra la ocupación actual de las celdas.

## 2. Requisitos funcionales
- RF01: crear y consultar usuarios, con documento único y datos de contacto.
- RF02: registrar vehículos y asociarlos con su propietario (`owner_id`).
- RF03: crear celdas, activar/desactivar y visualizar su estado.
- RF04: registrar entrada; validar vehículo, impedir sesiones duplicadas y asignar celda libre o seleccionada.
- RF05: registrar salida; calcular permanencia, liquidar tarifa y liberar celda.
- RF06: conservar historial de sesiones y emitir número de comprobante.
- RF07: registrar novedades asociadas a vehículo y opcionalmente al usuario que reporta.
- RF08: registrar mensualidades por usuario, vehículo y periodo; controlar pendiente/pagado.
- RF09: consultar indicadores de ocupación y novedades abiertas desde el panel.

## 3. Historias de usuario y criterios de aceptación

### HU01 — Administración de usuarios
Como administrador, quiero registrar usuarios para identificar propietarios y responsables.
- Dado un documento no registrado, cuando guardo nombre y documento válidos, entonces se crea el usuario.
- Si el documento ya existe, la API rechaza el duplicado y conserva los datos previos.
- El usuario queda disponible para asociarlo con vehículos y mensualidades.

### HU02 — Asociación de vehículo y propietario
Como operador, quiero registrar vehículos vinculados a sus propietarios para consultar su responsable.
- La placa es obligatoria y única.
- Si se selecciona propietario, este debe existir.
- La consulta de vehículos muestra placa, tipo, marca y propietario asociado.

### HU03 — Gestión de celdas
Como operador, quiero administrar celdas para conocer disponibilidad.
- Una celda nueva inicia activa y libre.
- Una celda ocupada no puede asignarse a otro vehículo.
- El mapa distingue libre, ocupada e inactiva.
- Una celda solo se libera al completar la salida de su sesión activa.

### HU04 — Entrada y asignación
Como operador, quiero registrar el ingreso de un vehículo y asignarle una celda.
- Solo se acepta un vehículo registrado.
- No se permite una segunda sesión activa del mismo vehículo.
- Se asigna una celda libre automáticamente o se respeta la seleccionada si está disponible.
- La celda pasa a ocupada y la sesión queda en el historial.

### HU05 — Salida, liquidación y comprobante
Como operador, quiero cerrar la sesión con cálculo automático para cobrar de forma consistente.
- La salida se rechaza si la sesión no existe o ya está cerrada.
- El sistema calcula minutos transcurridos desde la entrada.
- La tarifa base es configurable; por defecto COP $3.000 por hora iniciada, con mínimo de una hora.
- El total es horas facturables × tarifa por hora.
- Se guarda fecha/hora de salida, duración, valor y número único de comprobante.
- La celda queda libre y el registro permanece en el historial.

### HU06 — Novedades y mensualidades
Como administrador, quiero documentar incidentes y controlar planes mensuales.
- Cada novedad exige vehículo, categoría y descripción; puede indicar usuario reportante.
- Las novedades se consultan por fecha/estado y se cuentan las abiertas en el panel.
- La mensualidad exige usuario, vehículo, periodo válido y valor positivo.
- Una mensualidad puede pasar de pendiente a pagado y se registra la fecha del pago.

## 4. Reglas de negocio
1. Placa, documento y código de celda son identificadores únicos.
2. Un vehículo no puede tener más de una sesión sin salida.
3. Una celda solo admite una sesión activa.
4. La salida se calcula en backend; el navegador no decide el importe.
5. La facturación por hora iniciada redondea hacia arriba; fracción de hora cuenta como hora completa.
6. La tarifa se define mediante variable de entorno `HOURLY_RATE_COP`.
7. El cobro mensual se registra como cuota/periodo fijo independiente del cobro por hora. El valor mensual lo ingresa el administrador según el contrato o tarifa vigente.
8. El sistema registra pagos administrativos; no procesa transacciones bancarias.

## 5. Modelo de datos
- Usuario 1 — N Vehículo.
- Vehículo 1 — N SesiónParqueo.
- Celda 1 — N SesiónParqueo (histórico); como máximo una sesión activa por celda.
- Vehículo 1 — N Novedad.
- Usuario 1 — N Novedad reportada (relación opcional).
- Usuario 1 — N Mensualidad y Vehículo 1 — N Mensualidad.

Entidades: Usuario(id, nombre, documento, teléfono, correo, rol, activo); Vehículo(id, placa, tipo, marca, color, owner_id); Celda(id, código, tipo, activa, ocupada); Sesión(id, vehicle_id, cell_id, entrada, salida, duración, importe, comprobante); Novedad(id, vehicle_id, reported_by_user_id, fecha, categoría, descripción, estado); Mensualidad(id, user_id, vehicle_id, inicio, fin, valor, estado, referencia, fecha_pago).

## 6. Arquitectura
Frontend Angular standalone + TypeScript consume API REST FastAPI. FastAPI valida solicitudes con Pydantic y aplica reglas de negocio; SQLAlchemy persiste en SQLite. La separación por capas es presentación (Angular), servicio/API (FastAPI), dominio/modelos (SQLAlchemy) y persistencia (SQLite).

## 7. API principal
- `GET/POST /api/users`; `PATCH/DELETE /api/users/{id}`
- `GET/POST /api/vehicles`; `PATCH/DELETE /api/vehicles/{id}`
- `GET/POST /api/cells`; `PATCH/DELETE /api/cells/{id}`
- `GET /api/parking/active`; `GET /api/parking/history`
- `POST /api/parking/entry`; `POST /api/parking/{id}/exit`
- `GET/POST /api/incidents`
- `GET/POST /api/monthly-payments`; `GET /api/monthly-payments/summary`; `PATCH /api/monthly-payments/{id}/pay`

Documentación interactiva en `http://127.0.0.1:8000/docs`.

## 8. Mockups / pantallas previstas
1. Panel principal: tarjetas de vehículos registrados, vehículos dentro, celdas libres y novedades abiertas; tabla de sesiones activas.
2. Usuarios: formulario de nombre, documento, teléfono, correo y rol; tabla de usuarios.
3. Vehículos: placa, tipo, marca, color y selector de propietario; tabla de vehículos.
4. Celdas: formulario de código/tipo y mapa visual de celdas libres, ocupadas e inactivas.
5. Parqueadero: formulario de entrada y lista de sesiones activas con acción “Calcular y dar salida”.
6. Novedades: formulario con vehículo, categoría y descripción; historial de novedades.
7. Gestión de pagos: formulario de usuario, vehículo, fechas, valor y referencia; indicadores de facturación, recaudo y saldo pendiente; búsqueda por responsable/placa/referencia; filtro por estado; alertas de vencimiento; historial y confirmación de pago.

## 9. Pruebas y validación
Casos mínimos: endpoint raíz; creación de usuario; placa/documento duplicados; vehículo sin propietario válido; entrada con celda libre; entrada con celda ocupada; vehículo con sesión activa; salida y liberación de celda; segunda salida; cálculo de tarifa; novedad asociada a vehículo/usuario; periodo mensual inválido.

## 10. SQL Server/MySQL
El código operativo de esta entrega usa SQLite para facilitar ejecución local. Se incluye `database/mysql_schema.sql` como diseño equivalente MySQL 8. Antes de migrar los datos, ejecutar una migración formal (Alembic) y validar tipos de fecha, restricciones y datos existentes.

## 11. Limitaciones y pendientes para producción
No se incluye autenticación, autorización real por roles, pasarela de pagos, auditoría inmutable ni despliegue público. `Base.metadata.create_all()` crea tablas nuevas pero no altera tablas existentes; si se usa una base anterior, respaldar y recrear la BD de práctica o desarrollar migraciones Alembic. La tarifa predeterminada es demostrativa y debe ser aprobada por el negocio.

## 12. Publicación GitHub
Crear un repositorio vacío en GitHub y ejecutar desde la carpeta `autos_colombia`:
```bash
git init
git add .
git commit -m "Entrega sistema Autos Colombia"
git branch -M main
git remote add origin https://github.com/USUARIO/autos-colombia.git
git push -u origin main
```
Sustituir `USUARIO` por la cuenta del estudiante. El enlace definitivo se obtiene después de crear el repositorio; no se inventa en esta entrega.
