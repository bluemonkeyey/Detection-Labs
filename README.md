# Synthetic Detection Dataset for Turla Kazuar v3 Loader

Este repositorio contiene un conjunto de tablas de telemetría **ligeras y sintéticas** que pueden usarse para **probar y validar detecciones** del loader **Kazuar v3** de **Turla**. Los datos **no provienen de un entorno real**; en su lugar, simulan la secuencia de eventos descrita en el post “**COMmand & Evade: Turla's Kazuar v3 Loader**”. Al incluir solo los campos mínimos necesarios para correlación, estas tablas siguen siendo fáciles de ingerir e inspeccionar, pero permiten pruebas end-to-end de hunting queries.

---

## Contents

| File | Description |
|---|---|
| `device_process_events.csv` | Eventos sintéticos `ProcessCreated` que muestran la ejecución del dropper (`wscript.exe`), el instalador firmado de HP (`hpbprndi.exe`) y un proceso benigno `explorer.exe`. Cada fila incluye timestamps, identificadores de proceso, procesos padre y detalles de command-line. |
| `device_file_events.csv` | Eventos sintéticos `FileCreated` de los ficheros soltados por el VBScript. Incluye la creación de `hpbprndi.exe`, la DLL nativa `hpbprndiLOC.dll` y tres payloads cifrados (`jayb.dadk`, `kgjlj.sil`, `pkrfsu.ldy`) bajo un directorio escribible por el usuario en `AppData`. También se añade un evento benigno de creación de documento como contexto. |
| `device_image_load_events.csv` | Eventos sintéticos `ImageLoaded` que indican cuándo se mapean DLLs en procesos. Registra la carga de `hpbprndiLOC.dll` por `hpbprndi.exe`, además de una carga benigna de `kernel32.dll` por `explorer.exe`. |

---

## Core Columns (shared)

Los tres CSV comparten estas columnas base:

- **Timestamp** — Timestamp ISO-8601 en UTC.
- **DeviceName** — Hostname del sistema simulado.
- **DeviceId** — Identificador único del dispositivo.
- **ActionType** — Tipo de evento (p. ej. `ProcessCreated`, `FileCreated`, `ImageLoaded`).
- **FileName / ProcessId / FolderPath / ProcessCommandLine** — Campos relevantes según el tipo de evento.
- **InitiatingProcessFileName / InitiatingProcessId** — Ejecutable y PID responsable de crear un fichero o cargar un módulo.

Campos adicionales como **ParentProcessName**, **IntegrityLevel** y **SHA256** se incluyen cuando aportan valor.

---

## Schema Overview

Las tablas se diseñaron para ser compatibles con hunting queries escritas contra el esquema de **Microsoft 365 Defender Advanced Hunting**. Aunque se omiten muchas columnas opcionales del esquema nativo, los campos incluidos son suficientes para correlacionar creación de ficheros e image-loads con sus procesos origen.

---

## `device_process_events.csv`

| Column | Description |
|---|---|
| `Timestamp` | Cuándo se creó el proceso. |
| `DeviceName` | Nombre del host. |
| `DeviceId` | Identificador único del host. |
| `ActionType` | Fijado a `ProcessCreated`. |
| `ProcessId` | PID del proceso nuevo. |
| `FileName` | Nombre del ejecutable. |
| `ProcessCommandLine` | Command line completa. |
| `InitiatingProcessFileName` | Proceso padre que lanzó el nuevo proceso. |
| `InitiatingProcessId` | PID del proceso padre. |
| `ParentProcessName` | Igual que `InitiatingProcessFileName` (compatibilidad). |
| `ParentProcessId` | Igual que `InitiatingProcessId` (compatibilidad). |
| `IntegrityLevel` | Nivel de integridad (p. ej. `Medium`). |

---

## `device_file_events.csv`

| Column | Description |
|---|---|
| `Timestamp` | Cuándo ocurrió el evento de fichero. |
| `DeviceName` | Nombre del host. |
| `DeviceId` | Identificador único del host. |
| `ActionType` | Fijado a `FileCreated`. |
| `FileName` | Nombre del fichero creado. |
| `FolderPath` | Ruta del directorio donde se escribió. |
| `InitiatingProcessFileName` | Proceso que creó el fichero. |
| `InitiatingProcessId` | PID del proceso creador. |
| `SHA256` | Hash SHA-256 del fichero (valores sintéticos). |

---

## `device_image_load_events.csv`

| Column | Description |
|---|---|
| `Timestamp` | Cuándo se cargó el módulo en memoria. |
| `DeviceName` | Nombre del host. |
| `DeviceId` | Identificador único del host. |
| `ActionType` | Fijado a `ImageLoaded`. |
| `FileName` | Nombre del módulo cargado (DLL). |
| `FolderPath` | Ruta del módulo en disco. |
| `InitiatingProcessFileName` | Proceso en el que se cargó el módulo. |
| `InitiatingProcessId` | PID del proceso “loader”. |
| `SHA256` | Hash SHA-256 del módulo (valores sintéticos). |

