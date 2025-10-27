# 🔧 Micro:bit Reader - Python

## 📋 Descripción
Lector de datos seriales desde micro:bit implementado en **Python**. Recibe datos JSON en tiempo real de sensores (temperatura, acelerómetro, luz) y detecta alertas automáticamente.

---

## 🏗️ Estructura de Archivos
```
python/
├── 📄 reader.py          # Código principal
├── 📄 requirements.txt   # Dependencias
└── 📄 README.md          # Esta documentación
```

---

## 📦 Dependencias Requeridas

### Paquetes Python:
- **pyserial** >= 4.0 - Comunicación serial
- **json** - Incluido en Python estándar
- **math** - Incluido en Python estándar
- **datetime** - Incluido en Python estándar

---

## 🐍 Código Completo - `reader.py`

```python
import serial
import json
import math
import time
from datetime import datetime

# ⚙️ CONFIGURACIÓN - AJUSTAR SEGÚN SISTEMA
PORT_NAME = 'COM3'  # Windows: COM3, COM4 | Linux: /dev/ttyACM0 | Mac: /dev/cu.usbmodem14102
BAUD_RATE = 115200

print("🚀 Iniciando Lector Serial micro:bit con Python")
print(f"📡 Puerto: {PORT_NAME}")
print(f"⚡ Baud Rate: {BAUD_RATE}")
print("⏳ Esperando datos...\n")

# 📐 Función para calcular magnitud de aceleración
def calcular_magnitud_aceleracion(ax, ay, az):
    return math.sqrt(ax**2 + ay**2 + az**2)

# 🚨 Función para verificar alertas
def verificar_alertas(datos):
    magnitud = calcular_magnitud_aceleracion(datos['ax'], datos['ay'], datos['az'])
    alertas = []

    if magnitud > 1.5:
        alertas.append("🚨 MOVIMIENTO BRUSCO")
    if datos['tempC'] > 30:
        alertas.append("🌡️ ALTA TEMPERATURA")
    if datos['light'] < 20:
        alertas.append("🌑 BAJA LUZ")
    if datos['bat'] < 3.0:
        alertas.append("🔋 BATERIA BAJA")

    return alertas

# 🕐 Función para formatear timestamp
def formatear_timestamp(timestamp):
    return datetime.fromtimestamp(timestamp).strftime('%H:%M:%S')

try:
    # 🔌 Configurar y abrir puerto serial
    ser = serial.Serial(
        port=PORT_NAME,
        baudrate=BAUD_RATE,
        bytesize=serial.EIGHTBITS,
        parity=serial.PARITY_NONE,
        stopbits=serial.STOPBITS_ONE,
        timeout=1  # Timeout de 1 segundo
    )

    print("✅ PUERTO SERIAL ABIERTO CORRECTAMENTE")
    print("📊 LEYENDO DATOS DEL MICRO:BIT...")
    print("💡 Presiona Ctrl + C para detener\n")

    # 📁 Crear archivo de log
    with open('microbit-python.log', 'a', encoding='utf-8') as log_file:
        log_file.write(f"=== Inicio de sesion: {datetime.now().isoformat()} ===\n")

        # 🔄 Bucle principal de lectura
        while True:
            try:
                if ser.in_waiting > 0:
                    # Leer línea del puerto serial
                    linea = ser.readline().decode('utf-8').strip()
                    
                    # Ignorar líneas vacías
                    if not linea:
                        continue
                    
                    # Intentar parsear JSON
                    try:
                        datos = json.loads(linea)
                        
                        # Validar datos mínimos
                        if 'id' not in datos or 'tempC' not in datos:
                            print(f"❌ DATOS INCOMPLETOS: {linea}")
                            continue

                        # 🖥️ Mostrar datos en consola
                        print(f"📱 DISPOSITIVO: {datos['id']}")
                        print(f"🕐 FECHA/HORA: {formatear_timestamp(datos['ts'])}")
                        print(f"🌡️ TEMPERATURA: {datos['tempC']:.1f}°C")
                        print("📊 ACELEROMETRO:")
                        print(f"   X: {datos['ax']:.3f} g")
                        print(f"   Y: {datos['ay']:.3f} g")
                        print(f"   Z: {datos['az']:.3f} g")
                        print(f"💡 NIVEL DE LUZ: {datos['light']}")
                        print(f"🔋 VOLTAJE BATERIA: {datos['bat']:.2f}V")

                        # 📈 Calcular y mostrar magnitud
                        magnitud = calcular_magnitud_aceleracion(datos['ax'], datos['ay'], datos['az'])
                        print(f"📈 MAGNITUD ACELERACION: {magnitud:.3f} g")

                        # 🚨 Verificar y mostrar alertas
                        alertas = verificar_alertas(datos)
                        if alertas:
                            print("🚨🚨🚨 ALERTAS DETECTADAS:")
                            for alerta in alertas:
                                print(f"   {alerta}")

                        print("─" * 60)

                        # 💾 Guardar en archivo de log
                        log_file.write(f"{datetime.now().isoformat()} - {linea}\n")
                        if alertas:
                            log_file.write(f"ALERTAS: {', '.join(alertas)}\n")
                        log_file.flush()

                    except json.JSONDecodeError as e:
                        print(f"❌ ERROR PARSEANDO JSON: {e}")
                        print(f"📝 LINEA RECIBIDA: {linea}")

            except KeyboardInterrupt:
                print("\n👋 CERRANDO PROGRAMA...")
                break
            except Exception as e:
                print(f"❌ ERROR INESPERADO: {e}")
                continue

except serial.SerialException as e:
    print(f"❌ ERROR DEL PUERTO SERIAL: {e}")
    print("\n🔧 SOLUCION DE PROBLEMAS:")
    print("   1. Verifica que el micro:bit este conectado por USB")
    print("   2. Confirma el nombre del puerto (COM3, COM4, etc.)")
    print("   3. Asegurate de que el puerto no este en uso")
    print("   4. Revisa que el micro:bit este ejecutando el codigo correcto")

finally:
    # Cerrar puerto serial si está abierto
    if 'ser' in locals() and ser.is_open:
        ser.close()
        print("🔌 PUERTO SERIAL CERRADO")
```

---

## 📄 requirements.txt

```txt
pyserial>=4.0
```

---

## 🚀 Guía de Ejecución Paso a Paso - WINDOWS/CMD

### 📋 Prerrequisitos
- ✅ Python 3.7 o superior instalado
- ✅ micro:bit con código MicroPython cargado
- ✅ Conexión USB del micro:bit a la computadora

### 🔧 Paso 1: Crear Carpeta y Archivos en CMD
```cmd
:: Navegar al disco donde quieres el proyecto
F:
mkdir python-project
cd python-project

:: Crear requirements.txt
echo pyserial>=4.0 > requirements.txt

:: Crear reader.py con Notepad
notepad reader.py
```
**Pegar el código Python completo y Guardar**

### 📦 Paso 2: Instalar Dependencias en CMD
```cmd
:: Instalar pyserial
pip install pyserial

:: O instalar desde requirements.txt
pip install -r requirements.txt

:: Verificar instalación
python --version
pip list | findstr pyserial
```

### 🔍 Paso 3: Encontrar Puerto COM en Windows
```cmd
:: Método con PowerShell desde CMD
powershell "[System.IO.Ports.SerialPort]::getportnames()"

:: Método alternativo - Administrador de dispositivos
:: Windows + R → "devmgmt.msc" → "Puertos (COM y LPT)"
```

### ⚙️ Paso 4: Configurar Puerto (Si es necesario)
```cmd
:: Editar reader.py para cambiar el puerto COM
notepad reader.py
```
**Buscar esta línea y cambiar:**
```python
PORT_NAME = 'COM3'  # ← Cambiar por tu puerto (COM4, COM5, etc.)
```

### 🎯 Paso 5: Ejecutar el Programa en CMD
```cmd
:: Ejecutar el script Python
python reader.py

:: O si tienes múltiples versiones de Python
python3 reader.py

:: O usando la ruta completa si es necesario
C:\Python311\python.exe reader.py
```

---

## 🖥️ Comandos Rápidos de CMD para Python

```cmd
:: Navegación y verificación
F:
cd python-project
dir
python --version

:: Instalación de dependencias
pip install pyserial

:: Ejecución
python reader.py

:: Ver archivo de log generado
type microbit-python.log
```

---

## 📊 Salida Esperada en CMD

```
🚀 Iniciando Lector Serial micro:bit con Python
📡 Puerto: COM3
⚡ Baud Rate: 115200
⏳ Esperando datos...

✅ PUERTO SERIAL ABIERTO CORRECTAMENTE
📊 LEYENDO DATOS DEL MICRO:BIT...
💡 Presiona Ctrl + C para detener

📱 DISPOSITIVO: M1
🕐 FECHA/HORA: 10:30:25
🌡️ TEMPERATURA: 27.1°C
📊 ACELEROMETRO:
   X: -0.030 g
   Y: 0.980 g
   Z: 0.050 g
💡 NIVEL DE LUZ: 123
🔋 VOLTAJE BATERIA: 3.01V
📈 MAGNITUD ACELERACION: 0.982 g
────────────────────────────────────────────────────────────
```

---

## 🔧 Solución de Problemas - WINDOWS/PYTHON

### ❌ Error: "ModuleNotFoundError: No module named 'serial'"
```cmd
:: Solución: Instalar pyserial
pip install pyserial

:: O si tienes múltiples Python
python -m pip install pyserial
```

### ❌ Error: "No such file or directory: 'COM3'"
```cmd
:: Ver puertos disponibles
powershell "[System.IO.Ports.SerialPort]::getportnames()"

:: Cambiar puerto en reader.py
notepad reader.py
```

### ❌ Error: "Access is denied"
- Ejecutar CMD como Administrador
- Cerrar Arduino IDE, Mu Editor, Putty
- Reiniciar micro:bit

### ❌ Error: "UnicodeDecodeError"
```python
# Cambiar la línea de lectura en reader.py:
linea = ser.readline().decode('utf-8', errors='ignore').strip()
```

### ❌ Error: "Python no se reconoce"
```cmd
:: Agregar Python al PATH o usar ruta completa
C:\Python311\python.exe reader.py

:: O verificar instalación
where python
```

---

## 🚨 Sistema de Alertas en Python

El programa detecta automáticamente:

| Condición | Cálculo | Alerta | Umbral |
|-----------|---------|--------|---------|
| Movimiento brusco | `math.sqrt(ax²+ay²+az²) > 1.5` | 🚨 MOVIMIENTO BRUSCO | > 1.5g |
| Alta temperatura | `tempC > 30` | 🌡️ ALTA TEMPERATURA | > 30°C |
| Baja luminosidad | `light < 20` | 🌑 BAJA LUZ | < 20 |
| Batería baja | `bat < 3.0` | 🔋 BATERÍA BAJA | < 3.0V |

---

## 💾 Características Técnicas de Python

- ✅ **Lectura síncrona** simple y directa
- ✅ **Manejo de excepciones** robusto con try-except
- ✅ **Logging automático** a archivo con timestamp
- ✅ **Detección de alertas** en tiempo real
- ✅ **Cierre graceful** con Ctrl+C
- ✅ **Validación de datos** JSON completa

---

## 📈 Ventajas de Python para este Proyecto

1. **Sintaxis clara y legible**
2. **Instalación simple** de dependencias
3. **Manejo nativo de JSON**
4. **Gran comunidad** y documentación
5. **Multiplataforma** (Windows, Linux, Mac)
6. **Ideal para prototipado rápido**

---

## ✅ Verificación Final en CMD

```cmd
F:
cd python-project
dir
python --version
pip show pyserial
python reader.py
```

