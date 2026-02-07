# IcedID-like Detection Lab (LAB)

## Objetivo
Simular de forma **controlada y no maliciosa** una cadena de ataque tipo **IcedID (BokBot)** para **validar detecciones por correlación** en Microsoft Defender / Sentinel (Advanced Hunting), especialmente aquellas basadas en:
- Creación de DLL en rutas sospechosas (AppData/Temp/etc.)
- Carga inmediata de la DLL en memoria
- Ejecución mediante LOLBINs (rundll32/regsvr32/powershell)

## Descripción del escenario
Este laboratorio recrea el **comportamiento observable** (telemetría) típico de campañas IcedID/droppers:
1) Un proceso inicial (simulando macro/LNK/HTA) lanza PowerShell  
2) PowerShell descarga una DLL “loader” desde **localhost**  
3) La DLL se ejecuta con **rundll32.exe** (DllRegisterServer)  
4) Se generan artefactos auxiliares (p.ej. `license.dat` y un log)  
5) Se genera tráfico HTTP hacia **127.0.0.1** (simulación de C2 sin riesgo)  
6) Se añade ruido benigno (procesos/archivos normales) para realismo

> No hay C2 real, no hay exfiltración y no hay payload malicioso.

## Flujo del escenario
1. `powershell.exe` realiza `Invoke-WebRequest` hacia `http://127.0.0.1:8080/`  
2. Se crea `update.dll` en una ruta user-writable (p.ej. `%AppData%\Microsoft\Templates\`)  
3. Se crea `license.dat` en `%AppData%`  
4. `rundll32.exe` carga `update.dll` en memoria y ejecuta `DllRegisterServer`  
5. La DLL dummy escribe un log local (`icedid_lab.log`)  
6. Se generan eventos benignos de fondo (p.ej. `invoice.docx`, `notepad++.exe`, etc.)

## Tablas implicadas
- **DeviceFileEvents**: creación de DLL/artefactos y ruido benigno  
- **DeviceImageLoadEvents**: carga de la DLL en memoria  
- **DeviceProcessEvents**: procesos y relaciones padre-hijo (PowerShell → rundll32)  
- **DeviceNetworkEvents**: conexiones HTTP a localhost para simular descarga/C2

## Archivos incluidos
- `lab_device_file_events.csv`  
  Eventos de archivos (FileCreated y ruido)
- `lab_device_image_events.csv`  
  Eventos de carga de imágenes/DLLs en memoria
- `lab_device_process_events.csv`  
  Eventos de procesos (ProcessCreated, parent-child, command lines)
- `lab_device_network_events.csv`  
  Eventos de red hacia `127.0.0.1:8080`

## Uso
1. Importa los CSV a tu entorno de pruebas (Sentinel/ADX/Log Analytics u otro)  
2. Ejecuta tu query de correlación (DeviceFileEvents + DeviceImageLoadEvents + DeviceProcessEvents)  
3. Verifica que aparecen coincidencias:  
   - `FileCreated` de `*.dll` en `AppData/Temp/...`  
   - `ImageLoaded` de esa misma DLL  
   - `rundll32.exe`/`regsvr32.exe` como iniciador o en la cadena  
4. Ajusta filtros/ventanas de tiempo para reducir falsos positivos

## Indicadores esperados (IOCs de laboratorio)
- Archivo: `update.dll` en ruta user-writable  
- Proceso: `rundll32.exe ... ,DllRegisterServer`  
- Red: `http://127.0.0.1:8080/payload.dll` y `http://127.0.0.1:8080/license.dat`  
- Artefacto: `license.dat` y `icedid_lab.log`

## Consideraciones y seguridad
- **No contiene malware real** ni técnicas ofensivas activas (solo simulación de trazas)  
- Tráfico únicamente a **localhost**  
- Diseñado para **laboratorio/entorno aislado**, no producción  
- Útil para tuning de detecciones basadas en comportamiento (no hashes)

## Licencia
Uso interno / laboratorio (ajusta esta sección según tu repo o cliente).
