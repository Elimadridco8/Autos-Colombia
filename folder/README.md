# Autos Colombia — sistema de gestión de parqueadero

Entrega académica organizada y corregida. Aplicación web Angular + API FastAPI + SQLAlchemy + SQLite. La estructura incluye historias de usuario, criterios de aceptación, modelo UML, arquitectura, esquema MySQL de referencia, pruebas y guía de publicación.

## Requisitos
- Python 3.11 o superior
- Node.js 20 o superior y npm

## Ejecutar backend (Windows)
```powershell
cd backend
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
$env:HOURLY_RATE_COP="3000"
uvicorn app.main:app --reload
```
Linux/macOS: `source .venv/bin/activate` y `export HOURLY_RATE_COP=3000`.

API: http://127.0.0.1:8000 — Swagger: http://127.0.0.1:8000/docs

## Ejecutar frontend
En otra terminal:
```bash
cd frontend
npm install
npm start
```
Abrir http://localhost:4200.

## Reglas de cobro implementadas
La salida no solicita un valor manual. El backend calcula los minutos de permanencia y cobra por hora iniciada, con mínimo de una hora. Tarifa inicial demostrativa: COP $3.000/hora, modificable con `HOURLY_RATE_COP`. El backend genera comprobante único y libera la celda. La mensualidad se registra aparte por periodo y su importe contractual lo define el administrador. El gestor valida usuarios activos, vehículos existentes, valores positivos y periodos no superpuestos por vehículo; muestra facturación, recaudo y saldo pendiente, permite filtrar el historial y registra la fecha de confirmación del pago.

## Pruebas
```bash
cd backend
# Windows PowerShell:
$env:PYTHONPATH="."
pytest -q
# Linux/macOS:
PYTHONPATH=. pytest -q
```

## Archivos de apoyo
- `DOCUMENTACION_TECNICA.md`: requerimientos, seis historias, criterios, reglas, modelo, arquitectura, API, pruebas y publicación GitHub.
- `diagrams/diagrama_clases.puml`: diagrama UML de clases.
- `diagrams/arquitectura.puml`: arquitectura.
- `database/mysql_schema.sql`: modelo de referencia MySQL 8.
- `frontend/src/app/app.component.html`: mockups implementados como pantallas funcionales.

## Nota sobre bases anteriores
`create_all()` genera tablas faltantes, no modifica columnas en una base SQLite ya creada. Para esta entrega práctica, respalde y retire `backend/autos_colombia.db` antes de iniciar con el nuevo modelo si ya existía una base anterior. Para producción, usar migraciones Alembic.

## GitHub
Los comandos para crear y publicar el repositorio están en `DOCUMENTACION_TECNICA.md`. El enlace remoto se debe completar con el usuario y repositorio reales del estudiante; esta entrega no puede publicar en una cuenta GitHub sin autorización y acceso.
