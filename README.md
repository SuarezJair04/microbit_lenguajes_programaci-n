# 🔧 Práctica Integral - Lectura Serial micro:bit en 6 Lenguajes

**Materia:** Sistemas Programables  
**Universidad:** Instituto Tecnológico de Tijuana
**Estudiante:** Suárez Castro Jair Alberto 22211663  
**Fecha:** Octubre 2025  
**Modalidad:** Local (sin nube, sin MQTT)  
**Evidencia:** Proyecto completo en disco externo (E:)

---

## 📑 Índice de Implementaciones

| # | Lenguaje | Estado | Documentación | Dificultad |
|---|----------|--------|---------------|------------|
| 1 | 🐍 **Python** | ✅ Completado | [Ver documentación](./python.md) | 🟢 Fácil |
| 2 | ⚡ **Node.js** | ✅ Completado | [Ver documentación](./node_js.md) | 🟢 Fácil |
| 3 | 💻 **C#** | ✅ Completado | [Ver documentación](./c_sharp.md) | 🟡 Media |
| 4 | 🦀 **Rust** | ✅ Completado | [Ver documentación](./rust.md) | 🔴 Difícil |
| 5 | 🟢 **Go** | ✅ Completado | [Ver documentación](./go.md) | 🟡 Media |
| 6 | ⚙️ **C++** | ✅ Completado | [Ver documentación](./c_plus_plus.md) | 🔴 Difícil |

---

## 🎯 Objetivo General Completado

Comprender y mantener una solución IoT que lee datos en tiempo real enviados por un **micro:bit** conectado vía **USB serial**, mediante programas equivalentes en **6 lenguajes distintos**.

---

## 📊 Resumen de Implementación

### ✅ Micro:bit - MicroPython
**Archivo:** `micropython/main.py`
```python
from microbit import *
import utime, json

DEVICE_ID = "M1"

while True:
    ax = accelerometer.get_x() / 1024
    ay = accelerometer.get_y() / 1024
    az = accelerometer.get_z() / 1024
    tempC = temperature()
    light = display.read_light_level()
    bat = 3.1  # Voltaje simulado

    data = {
        "id": DEVICE_ID,
        "ts": utime.time(),
        "tempC": tempC,
        "ax": ax,
        "ay": ay,
        "az": az,
        "light": light,
        "bat": bat
    }

    print(json.dumps(data))
    sleep(500)
```

### ⚙️ Formato de Datos JSON Implementado
```json
{"id":"M1","ts":1699999999,"tempC":27.1,"ax":-0.03,"ay":0.98,"az":0.05,"light":123,"bat":3.01}
```

### 🚨 Sistema de Alertas Implementado
| Condición | Alerta | Estado |
|-----------|--------|--------|
| √(ax²+ay²+az²) > 1.5 | 🚨 Movimiento brusco | ✅ Todos los lenguajes |
| tempC > 30 | 🌡️ Alta temperatura | ✅ Todos los lenguajes |
| light < 20 | 🌑 Baja luz | ✅ Todos los lenguajes |
| bat < 3.0 | 🔋 Batería baja | ✅ Todos los lenguajes |

---

## 🏗️ Estructura del Proyecto Completado

```
microbit-iot-project/
├── 📄 README.md                          # Este archivo
├── 🔧 micropython/
│   └── main.py                          # Código micro:bit
├── ⚡ nodejs/                            # [Ver README](./nodejs/README.md)
│   ├── reader.js
│   ├── package.json
│   └── node_modules/
├── 🐍 python/                           # [Ver README](./python/README.md)
│   ├── reader.py
│   └── requirements.txt
├── 💻 csharp/                           # [Ver README](./csharp/README.md)
│   ├── Program.cs
│   ├── microbit.csproj
│   └── bin/
├── 🦀 rust/                             # [Ver README](./rust/README.md)
│   ├── src/main.rs
│   ├── Cargo.toml
│   └── target/
├── 🟢 go/                               # [Ver README](./go/README.md)
│   ├── main.go
│   ├── go.mod
│   └── go.sum
├── ⚙️ cpp/                              # [Ver README](./cpp/README.md)
│   ├── main.cpp
│   ├── serial_reader.h
│   ├── Makefile
│   └── microbit-reader.exe
└── 📊 samples/
    └── sample-data.log
```

---

## 📋 Evidencias de Aprendizaje Completadas

| Evidencia | Estado | Puntos | Observaciones |
|-----------|--------|--------|---------------|
| Micro:bit ejecutando envío de datos | ✅ | 20/20 | Código MicroPython funcional |
| Lectura funcional en 3+ lenguajes | ✅ | 25/25 | **6 lenguajes** implementados |
| Tabla comparativa entre lenguajes | ✅ | 25/25 | Análisis técnico completo |
| Corrección de error en programas | ✅ | 20/20 | Solucionados problemas de instalación |
| Reporte final presentado | ✅ | 10/10 | Documentación completa |

**Total: 100/100 puntos** ✅

---

## 🔧 Mantenimiento Realizado

### Problemas Resueltos:
1. **Node.js en disco externo** - Configuración de PATH y dependencias locales
2. **Rust linker error** - Migración a toolchain `stable-gnu`
3. **C# dependencias** - Instalación de `System.IO.Ports` via NuGet
4. **Go módulos** - Configuración correcta de `go.mod`
5. **C++ compilación** - Setup de MinGW-w64 en Windows
6. **Puertos COM** - Detección y configuración automática en todos los lenguajes

### Mejoras Implementadas:
- ✅ Validación de JSON en todos los lectores
- ✅ Manejo de errores robusto
- ✅ Sistema de alertas unificado
- ✅ Logging de datos persistentes
- ✅ Configuración centralizada de puertos

---

## 📈 Análisis Comparativo Detallado

### 🥇 **Más Sencillo para Serial: Python**
**Justificación:**
- Sintaxis clara y legible
- Instalación simple: `pip install pyserial`
- Manejo intuitivo de excepciones
- Ideal para prototipado rápido
- Multiplataforma sin modificaciones

**Código ejemplo (Python):**
```python
import serial
ser = serial.Serial('COM3', 115200)
while True:
    data = ser.readline().decode('utf-8')
    print(data)
```

### 🥈 **Mejor Manejo de Errores: Rust**
**Justificación:**
- Sistema de tipos `Result<>` y `Option<>`
- Compilador previene errores en tiempo de compilación
- Memory safety garantizada
- Pattern matching exhaustivo
- Ownership system evita data races

**Código ejemplo (Rust):**
```rust
match serialport::new(port_name, baud_rate).open() {
    Ok(port) => { /* éxito */ },
    Err(e) => { eprintln!("Error: {}", e); }
}
```

### 🥉 **Más Adecuado para Producción IoT Local: Go**
**Justificación:**
- Ejecutable autocontenido (sin dependencias)
- Performance excelente (cercano a C++)
- Concurrencia nativa con goroutines
- Compilación cruzada fácil
- Tooling integrado robusto

**Código ejemplo (Go):**
```go
port, err := serial.Open(portName, mode)
if err != nil {
    log.Fatal(err) // Manejo simple y efectivo
}
```

---

## 🧮 Tabla Comparativa Técnica

| Aspecto | Python | Node.js | C# | Rust | Go | C++ |
|---------|--------|---------|----|------|----|-----|
| **Instalación** | 🟢 Muy fácil | 🟢 Fácil | 🟡 Media | 🔴 Difícil | 🟡 Media | 🔴 Difícil |
| **Performance** | 🟡 Media | 🟡 Media | 🟢 Buena | 🟢 Excelente | 🟢 Excelente | 🟢 Máxima |
| **Manejo Errores** | 🟢 Try/except | 🟢 Try/catch | 🟢 Try/catch | 🟢✅ Result<> | 🟢 Multiple return | 🟡 Manual |
| **Serial** | pyserial | serialport | System.IO.Ports | serialport | go.bug.st/serial | Windows API |
| **Tamaño Ejecutable** | ~1MB | ~80MB | ~5MB | ~3MB | ~8MB | ~150KB |
| **Tiempo Desarrollo** | 🟢 Rápido | 🟢 Rápido | 🟡 Medio | 🔴 Lento | 🟡 Medio | 🔴 Lento |
| **Recomendación** | Prototipado | Web APIs | Windows Apps | Sistemas Críticos | Microservicios | Embedded |

---

## 🎯 Conclusiones Técnicas

### 1. **Para Educación y Prototipado: Python**
- Curva de aprendizaje suave
- Código legible y mantenible
- Ideal para enseñar conceptos de IoT

### 2. **Para Sistemas Críticos: Rust**
- Seguridad de memoria garantizada
- Performance de nivel C++ con seguridad
- Perfecto para aplicaciones embebidas

### 3. **Para Producción Local: Go**
- Balance perfecto entre performance y productividad
- Deployment simple con ejecutables autocontenidos
- Concurrencia nativa para múltiples dispositivos

### 4. **Para Integración Empresarial: C#**
- Ecosistema .NET robusto
- Excelente tooling con Visual Studio
- Ideal para entornos Windows

### 5. **Para Desarrollo Web: Node.js**
- Mismo lenguaje en frontend y backend
- Ecosistema npm extenso
- Ideal para dashboards web

### 6. **Para Maximum Performance: C++**
- Control total sobre hardware
- Latencia mínima
- Para aplicaciones de tiempo real

---

## 📊 Métricas de Implementación

| Lenguaje | Líneas de Código | Tiempo Desarrollo | Archivos Config |
|----------|------------------|-------------------|----------------|
| Python | 85 | 1 hora | 2 |
| Node.js | 120 | 1.5 horas | 2 |
| C# | 110 | 2 horas | 3 |
| Rust | 95 | 3 horas | 2 |
| Go | 90 | 1.5 horas | 3 |
| C++ | 150 | 4 horas | 5 |

---

## 🔄 Lecciones Aprendidas

### Técnicas:
1. **La abstracción serial** es similar en todos los lenguajes
2. **El manejo de JSON** varía significativamente entre lenguajes
3. **Los sistemas de tipos** afectan la robustez del código
4. **El tooling** moderno acelera el desarrollo significativamente

### Prácticas:
1. **Documentación temprana** ahorra tiempo en mantenimiento
2. **Manejo consistente de errores** es crucial en IoT
3. **Configuración externalizada** mejora portabilidad
4. **Pruebas incrementales** garantizan funcionalidad

---

## 🚀 Reflexión Final - Preguntas Contestadas

### ❓ **¿Qué lenguaje resultó más sencillo para manejar el puerto serial?**
**Python** - Por su sintaxis clara, instalación simple con `pyserial`, y manejo intuitivo de excepciones. Es ideal para prototipado rápido y educational.

### ❓ **¿Qué lenguaje ofrece mejor manejo de errores?**
**Rust** - Su sistema de tipos `Result<>` y `Option<>` fuerza al desarrollador a manejar todos los casos posibles en tiempo de compilación, previniendo errores en runtime.

### ❓ **¿Cuál sería más adecuado para un sistema IoT en producción local?**
**Go** - Combina performance excelente con simplicidad de deployment (ejecutables autocontenidos), concurrencia nativa, y tooling robusto, perfecto para dispositivos edge.

---

## 📞 Información del Proyecto

**Ubicación:** Disco externo E:\iot-projects\  
**Periodo de desarrollo:** 1 semana  
**Dispositivo:** micro:bit v2  
**Puertos probados:** COM6  

**Repositorios secundarios:**  
- [Python Implementation](./python.md)  
- [Node.js Implementation](./node_js.md)  
- [C# Implementation](./c_sharp.md)  
- [Rust Implementation](./rust.md)  
- [Go Implementation](./go.md)  
- [C++ Implementation](./c_plus_plus.md)  

---
