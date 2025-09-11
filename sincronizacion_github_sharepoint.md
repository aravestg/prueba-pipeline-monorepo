### Guía de Automatización de Sincronización entre SharePoint y Repositorio GitHub

#### ✅ Objetivo
Automatizar la sincronización diaria de archivos desde una carpeta de SharePoint hacia un repositorio local sincronizado con GitHub, y realizar un `git push` automático solo si existen cambios detectados.

---

### 1. Estructura de Scripts

#### a. `refrescar_sharepoint.py`
Simula una acción de apertura de carpeta para forzar la sincronización local con SharePoint.

```python
import os
import time

ruta_sharepoint = os.path.expanduser("~/Abc S.A/QA - Componentes-para-QA")
print(f"🌟 Refrescando sincronización de: {ruta_sharepoint}")
try:
    os.system(f"open '{ruta_sharepoint}'")
    time.sleep(3)
    print("✅ Proceso de refresco completado.")
except Exception as e:
    print(f"Error al abrir carpeta: {e}")
```

---

#### b. `sync_sharepoint_to_repo_with_timestamp_check.py`
Sincroniza solo si detecta cambios usando timestamp por subcarpeta.

- Crea archivo `.sync_mod_log.txt` con timestamps.
- Copia carpetas modificadas al repositorio.

---

#### c. `subir_a_github.py`
Ejecuta los comandos:

```bash
git add .
git commit -m "Ambientación QA y Pasos a producción"
git push origin main
git status
```

Con `subprocess.run()` en Python:
```python
import subprocess
import os

os.chdir("/Users/africaravest/RM-GITHUB-PAP/RM-pasos-a-produccion")
subprocess.run(["git", "add", "."])
subprocess.run(["git", "commit", "-m", "Ambientación QA y Pasos a producción"])
subprocess.run(["git", "push", "origin", "main"])
subprocess.run(["git", "status"])
```

---

#### d. `ejecutar_sincronizacion_completa.py`
Orquesta los tres scripts:

```python
import subprocess
print("== INICIO DE EJECUCIÓN DE SINCRONIZACIÓN ==")

# Paso 1: Refrescar SharePoint
subprocess.run(["python3", "refrescar_sharepoint.py"])

# Paso 2: Sincronizar si hay cambios
subprocess.run(["python3", "sync_sharepoint_to_repo_with_timestamp_check.py"])

# Paso 3: Subir a GitHub solo si hubo cambios
if os.path.exists("/tmp/hubo_cambios"):  # Este archivo se crea si hubo cambios
    subprocess.run(["python3", "subir_a_github.py"])
else:
    print("🚫 No se detectaron cambios → GitHub no será actualizado.")
```

---

### 2. Automatización con `launchd` (macOS)

#### a. Crear archivo plist:
```bash
nano ~/Library/LaunchAgents/com.africaravest.syncgithub.plist
```

#### b. Contenido:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
"http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>com.africaravest.syncgithub</string>
    <key>ProgramArguments</key>
    <array>
      <string>/usr/bin/python3</string>
      <string>/Users/africaravest/Library/CloudStorage/OneDrive-abc/ADRetail/RM/scripts-sincronizacion/ejecutar_sincronizacion_completa.py</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
      <key>Hour</key>
      <integer>10</integer>
      <key>Minute</key>
      <integer>0</integer>
    </dict>
    <key>StandardOutPath</key>
    <string>/tmp/sync_to_github.out</string>
    <key>StandardErrorPath</key>
    <string>/tmp/sync_to_github.err</string>
    <key>RunAtLoad</key>
    <true/>
  </dict>
</plist>
```

#### c. Activar tarea:
```bash
launchctl load ~/Library/LaunchAgents/com.africaravest.syncgithub.plist
```

---

### 3. Logs y control de cambios
- Cambios sincronizados se reflejan en `/tmp/sync_to_github.out` y `.log`.
- Si no hay cambios, se evita `git push`.
- Cualquier error se reporta en consola o `/tmp/sync_to_github.err`.

---

### 4. Ubicación de scripts
`/Users/africaravest/Library/CloudStorage/OneDrive-abc/ADRetail/RM/scripts-sincronizacion`

- ejecutar_sincronizacion_completa.py
- refrescar_sharepoint.py
- sync_sharepoint_to_repo_with_timestamp_check.py
- subir_a_github.py

