# 🔧 Micro:bit Reader - C++

## 📋 Descripción
Lector de datos seriales desde micro:bit implementado en **C++**. Recibe datos JSON en tiempo real de sensores y detecta alertas automáticamente. Compilado con **MinGW-w64** en Windows.

---

## 🏗️ Estructura del Proyecto C++
```
microbit-cpp/
├── 📄 main.cpp            # Código principal
├── 📄 serial_reader.h     # Header para comunicación serial
├── 📄 serial_reader.cpp   # Implementación serial
├── 📄 Makefile           # Script de compilación
├── 📄 microbit-reader.exe # Ejecutable compilado
└── 📄 README.md          # Esta documentación
```

---

## 📦 Dependencias Requeridas

### Librerías C++:
- **Windows API** - Comunicación serial nativa
- **nlohmann/json** v3.11.2 - Parseo JSON (single header)

---

## ⚙️ serial_reader.h

```cpp
#ifndef SERIAL_READER_H
#define SERIAL_READER_H

#include <string>
#include <windows.h>

class SerialReader {
public:
    SerialReader();
    ~SerialReader();
    
    bool open(const std::string& portName, int baudRate = 115200);
    bool isOpen() const;
    void close();
    std::string readLine();
    
private:
    HANDLE hSerial;
    bool connected;
    
    void cleanup();
};

#endif // SERIAL_READER_H
```

---

## ⚙️ serial_reader.cpp

```cpp
#include "serial_reader.h"
#include <iostream>
#include <sstream>

SerialReader::SerialReader() : hSerial(INVALID_HANDLE_VALUE), connected(false) {}

SerialReader::~SerialReader() {
    cleanup();
}

bool SerialReader::open(const std::string& portName, int baudRate) {
    // Formatear nombre del puerto para Windows
    std::string fullPortName = "\\\\.\\" + portName;
    
    hSerial = CreateFileA(
        fullPortName.c_str(),
        GENERIC_READ,
        0,
        NULL,
        OPEN_EXISTING,
        FILE_ATTRIBUTE_NORMAL,
        NULL
    );
    
    if (hSerial == INVALID_HANDLE_VALUE) {
        std::cerr << "❌ Error: No se pudo abrir el puerto " << portName << std::endl;
        return false;
    }
    
    // Configurar parámetros del puerto
    DCB dcbSerialParams = {0};
    dcbSerialParams.DCBlength = sizeof(dcbSerialParams);
    
    if (!GetCommState(hSerial, &dcbSerialParams)) {
        std::cerr << "❌ Error obteniendo estado del puerto" << std::endl;
        cleanup();
        return false;
    }
    
    dcbSerialParams.BaudRate = baudRate;
    dcbSerialParams.ByteSize = 8;
    dcbSerialParams.StopBits = ONESTOPBIT;
    dcbSerialParams.Parity = NOPARITY;
    
    if (!SetCommState(hSerial, &dcbSerialParams)) {
        std::cerr << "❌ Error configurando puerto serial" << std::endl;
        cleanup();
        return false;
    }
    
    // Configurar timeouts
    COMMTIMEOUTS timeouts = {0};
    timeouts.ReadIntervalTimeout = 50;
    timeouts.ReadTotalTimeoutConstant = 50;
    timeouts.ReadTotalTimeoutMultiplier = 10;
    timeouts.WriteTotalTimeoutConstant = 50;
    timeouts.WriteTotalTimeoutMultiplier = 10;
    
    if (!SetCommTimeouts(hSerial, &timeouts)) {
        std::cerr << "❌ Error configurando timeouts" << std::endl;
        cleanup();
        return false;
    }
    
    connected = true;
    return true;
}

bool SerialReader::isOpen() const {
    return connected;
}

void SerialReader::close() {
    cleanup();
}

std::string SerialReader::readLine() {
    if (!connected) {
        return "";
    }
    
    std::stringstream ss;
    char buffer[1];
    DWORD bytesRead;
    
    while (true) {
        if (ReadFile(hSerial, buffer, 1, &bytesRead, NULL) && bytesRead > 0) {
            if (buffer[0] == '\n') {
                break;
            }
            if (buffer[0] != '\r') {
                ss << buffer[0];
            }
        } else {
            break;
        }
    }
    
    return ss.str();
}

void SerialReader::cleanup() {
    if (hSerial != INVALID_HANDLE_VALUE) {
        CloseHandle(hSerial);
        hSerial = INVALID_HANDLE_VALUE;
    }
    connected = false;
}
```

---

## 🧩 main.cpp

```cpp
#include <iostream>
#include <string>
#include <cmath>
#include <vector>
#include "serial_reader.h"

// Incluir librería JSON de un solo archivo
#include "json.hpp"
using json = nlohmann::json;

// 🏷️ Estructura para datos del micro:bit
struct MicrobitData {
    std::string id;
    double ts;
    double tempC;
    double ax;
    double ay;
    double az;
    int light;
    double bat;
};

// 📐 Función para calcular magnitud de aceleración
double calculateMagnitude(double ax, double ay, double az) {
    return std::sqrt(ax * ax + ay * ay + az * az);
}

// 🚨 Función para verificar alertas
void checkAlerts(const MicrobitData& data, double magnitude) {
    std::vector<std::string> alerts;
    
    if (magnitude > 1.5) {
        alerts.push_back("🚨 MOVIMIENTO BRUSCO");
    }
    if (data.tempC > 30) {
        alerts.push_back("🌡️ ALTA TEMPERATURA");
    }
    if (data.light < 20) {
        alerts.push_back("🌑 BAJA LUZ");
    }
    if (data.bat < 3.0) {
        alerts.push_back("🔋 BATERIA BAJA");
    }
    
    if (!alerts.empty()) {
        std::cout << "🚨 ALERTAS DETECTADAS:" << std::endl;
        for (const auto& alert : alerts) {
            std::cout << "   " << alert << std::endl;
        }
    }
}

// 📊 Procesar datos JSON recibidos
void processData(const std::string& jsonData) {
    try {
        auto j = json::parse(jsonData);
        
        MicrobitData data;
        data.id = j.value("id", "");
        data.ts = j.value("ts", 0.0);
        data.tempC = j.value("tempC", 0.0);
        data.ax = j.value("ax", 0.0);
        data.ay = j.value("ay", 0.0);
        data.az = j.value("az", 0.0);
        data.light = j.value("light", 0);
        data.bat = j.value("bat", 0.0);
        
        // Validar datos
        if (data.id.empty()) {
            std::cout << "❌ DATOS INCOMPLETOS: " << jsonData << std::endl;
            return;
        }
        
        // 🖥️ Mostrar datos en consola
        std::cout << "📱 DISPOSITIVO: " << data.id << std::endl;
        std::cout << "⏰ TIMESTAMP: " << data.ts << std::endl;
        std::cout << "🌡️ TEMPERATURA: " << data.tempC << "°C" << std::endl;
        std::cout << "📊 ACELEROMETRO:" << std::endl;
        std::cout << "   X: " << data.ax << " g" << std::endl;
        std::cout << "   Y: " << data.ay << " g" << std::endl;
        std::cout << "   Z: " << data.az << " g" << std::endl;
        std::cout << "💡 LUZ: " << data.light << std::endl;
        std::cout << "🔋 BATERIA: " << data.bat << "V" << std::endl;
        
        // 📈 Calcular y mostrar magnitud
        double magnitude = calculateMagnitude(data.ax, data.ay, data.az);
        std::cout << "📈 MAGNITUD ACELERACION: " << magnitude << "g" << std::endl;
        
        // 🚨 Verificar alertas
        checkAlerts(data, magnitude);
        
        std::cout << "──────────────────────────────────────────" << std::endl;
        
    } catch (const json::parse_error& e) {
        std::cout << "❌ ERROR PARSEANDO JSON: " << e.what() << std::endl;
        std::cout << "📝 DATO RECIBIDO: " << jsonData << std::endl;
    }
}

int main() {
    // ⚙️ CONFIGURACIÓN - AJUSTAR SEGÚN SISTEMA
    std::string portName = "COM3"; // Windows: COM3, COM4, COM5, etc.
    int baudRate = 115200;
    
    std::cout << "🚀 Iniciando Lector micro:bit en C++" << std::endl;
    std::cout << "📡 Puerto: " << portName << std::endl;
    std::cout << "⚡ Baud Rate: " << baudRate << std::endl;
    std::cout << "⏳ Esperando datos..." << std::endl << std::endl;
    
    SerialReader reader;
    
    // 🔌 Abrir puerto serial
    if (!reader.open(portName, baudRate)) {
        std::cerr << "❌ No se pudo abrir el puerto serial" << std::endl;
        std::cerr << "🔧 Solucion:" << std::endl;
        std::cerr << "   1. Verifica que el micro:bit este conectado" << std::endl;
        std::cerr << "   2. Confirma el nombre del puerto (COM3, COM4, etc.)" << std::endl;
        std::cerr << "   3. Asegurate que el puerto no este en uso" << std::endl;
        return 1;
    }
    
    std::cout << "✅ Puerto serial abierto correctamente" << std::endl;
    std::cout << "📊 Leyendo datos del micro:bit..." << std::endl;
    std::cout << "💡 Presiona Ctrl + C para detener" << std::endl << std::endl;
    
    // 🔄 Bucle principal de lectura
    while (reader.isOpen()) {
        std::string line = reader.readLine();
        if (!line.empty()) {
            processData(line);
        }
    }
    
    reader.close();
    return 0;
}
```

---

## 📄 Makefile

```makefile
# Configuración del compilador
CXX = g++
CXXFLAGS = -std=c++17 -Wall -Wextra -O2
LDFLAGS = -lwinmm

# Archivos fuente
SOURCES = main.cpp serial_reader.cpp
OBJECTS = $(SOURCES:.cpp=.o)
TARGET = microbit-reader.exe

# Descargar nlohmann/json si no existe
JSON_HEADER = json.hpp

$(JSON_HEADER):
	curl -L -o $(JSON_HEADER) https://github.com/nlohmann/json/releases/download/v3.11.2/json.hpp

# Regla principal
all: $(JSON_HEADER) $(TARGET)

$(TARGET): $(OBJECTS)
	$(CXX) $(OBJECTS) -o $(TARGET) $(LDFLAGS)

%.o: %.cpp
	$(CXX) $(CXXFLAGS) -c $< -o $@

# Limpiar archivos compilados
clean:
	del /Q *.o $(TARGET) 2>nul

# Ejecutar el programa
run: $(TARGET)
	.\$(TARGET)

.PHONY: all clean run
```

---

## 🚀 Guía de Ejecución Paso a Paso - WINDOWS/CMD

### 📋 Prerrequisitos
- ✅ MinGW-w64 instalado en `E:\mingw64\`
- ✅ Git para descargar dependencias (opcional)
- ✅ micro:bit con código MicroPython cargado

### 🔧 Paso 1: Crear Estructura del Proyecto
```cmd
:: Navegar al disco E:
E:

:: Crear carpeta del proyecto
mkdir microbit-cpp
cd microbit-cpp

:: Crear archivos fuente
notepad serial_reader.h
notepad serial_reader.cpp
notepad main.cpp
notepad Makefile
```
**Pegar el contenido correspondiente en cada archivo y Guardar**

### 📦 Paso 2: Descargar Dependencia JSON
```cmd
:: Descargar nlohmann/json (single header)
curl -L -o json.hpp https://github.com/nlohmann/json/releases/download/v3.11.2/json.hpp

:: O manualmente:
:: 1. Ir a: https://github.com/nlohmann/json/releases/download/v3.11.2/json.hpp
:: 2. Descargar y guardar como json.hpp en la carpeta del proyecto
```

### ⚙️ Paso 3: Configurar Compilador
```cmd
:: Configurar PATH para MinGW
set PATH=E:\mingw64\bin;%PATH%

:: Verificar instalación
g++ --version
where g++
```

### 🔍 Paso 4: Encontrar Puerto COM
```cmd
:: Usar PowerShell desde CMD
powershell "[System.IO.Ports.SerialPort]::getportnames()"
```

### ⚙️ Paso 5: Configurar Puerto (Si es necesario)
```cmd
:: Editar main.cpp línea 85
notepad main.cpp
```
**Buscar y cambiar:**
```cpp
std::string portName = "COM3"; // ← Cambiar por tu puerto (COM4, COM5, etc.)
```

### 🎯 Paso 6: Compilar con Make
```cmd
:: Compilar el proyecto
make

:: O compilar manualmente
g++ -std=c++17 -Wall -Wextra -O2 -c main.cpp -o main.o
g++ -std=c++17 -Wall -Wextra -O2 -c serial_reader.cpp -o serial_reader.o
g++ main.o serial_reader.o -o microbit-reader.exe -lwinmm
```

### 🚀 Paso 7: Ejecutar el Programa
```cmd
:: Ejecutar el ejecutable compilado
.\microbit-reader.exe
```

---

## 🖥️ Comandos Rápidos de CMD para C++

```cmd
:: Verificar compilador
g++ --version
where g++

:: Compilación rápida
make clean
make
.\microbit-reader.exe

:: Compilación manual
g++ -std=c++17 -Wall -O2 main.cpp serial_reader.cpp -o microbit-reader.exe -lwinmm

:: Ver archivos generados
dir
```

---

## 📊 Salida Esperada en CMD

```
🚀 Iniciando Lector micro:bit en C++
📡 Puerto: COM3
⚡ Baud Rate: 115200
⏳ Esperando datos...

✅ Puerto serial abierto correctamente
📊 Leyendo datos del micro:bit...
💡 Presiona Ctrl + C para detener

📱 DISPOSITIVO: M1
⏰ TIMESTAMP: 1699999999
🌡️ TEMPERATURA: 27.1°C
📊 ACELEROMETRO:
   X: -0.03 g
   Y: 0.98 g
   Z: 0.05 g
💡 LUZ: 123
🔋 BATERIA: 3.01V
📈 MAGNITUD ACELERACION: 0.982g
──────────────────────────────────────────
```

---

## 🔧 Solución de Problemas - C++/WINDOWS

### ❌ Error: "g++ no se reconoce"
```cmd
:: Verificar instalación MinGW
set PATH=E:\mingw64\bin;%PATH%
g++ --version

:: O usar ruta completa
E:\mingw64\bin\g++.exe --version
```

### ❌ Error: "json.hpp no encontrado"
```cmd
:: Descargar manualmente
curl -L -o json.hpp https://github.com/nlohmann/json/releases/download/v3.11.2/json.hpp
```

### ❌ Error: "undefined reference to WinMain"
```cmd
:: Verificar que main.cpp tenga función main()
:: Compilar con -lwinmm
g++ main.cpp serial_reader.cpp -o microbit-reader.exe -lwinmm
```

### ❌ Error: "puerto no encontrado"
```cmd
:: Ver puertos disponibles
powershell "[System.IO.Ports.SerialPort]::getportnames()"

:: Cambiar puerto en main.cpp
notepad main.cpp
```

### ❌ Error: "make no se reconoce"
```cmd
:: Usar compilación manual
g++ -std=c++17 -Wall -O2 main.cpp serial_reader.cpp -o microbit-reader.exe -lwinmm
```

---

## 🚨 Sistema de Alertas en C++

El programa detecta automáticamente:

| Condición | Cálculo | Alerta | Umbral |
|-----------|---------|--------|---------|
| Movimiento brusco | `std::sqrt(ax²+ay²+az²) > 1.5` | 🚨 MOVIMIENTO BRUSCO | > 1.5g |
| Alta temperatura | `tempC > 30` | 🌡️ ALTA TEMPERATURA | > 30°C |
| Baja luminosidad | `light < 20` | 🌑 BAJA LUZ | < 20 |
| Batería baja | `bat < 3.0` | 🔋 BATERÍA BAJA | < 3.0V |

---

## 💾 Características Técnicas de C++

- ✅ **Performance máximo** - Compilado a código máquina nativo
- ✅ **Control total** - Gestión manual de memoria y recursos
- ✅ **Windows API nativa** - Acceso directo al hardware
- ✅ **Ejecutable pequeño** - ~100-200KB sin dependencias externas
- ✅ **Tiempo real** - Ideal para aplicaciones de baja latencia

---

## 🎯 Ventajas de C++ para este Proyecto

1. **Performance máximo** - El más rápido de todos los lenguajes
2. **Control granular** - Gestión exacta de recursos del sistema
3. **Ejecutable pequeño** - No requiere runtime externo
4. **Windows nativo** - Integración perfecta con API de Windows
5. **Tiempo real** - Baja latencia para aplicaciones críticas
6. **Herencia C** - Compatibilidad con librerías de sistema

---

## 📁 Estructura Final del Proyecto

```
microbit-cpp/
├── 📄 main.cpp
├── 📄 serial_reader.h
├── 📄 serial_reader.cpp
├── 📄 json.hpp           # (Descargado de nlohmann/json)
├── 📄 Makefile
├── 📄 microbit-reader.exe # (~150KB)
├── 📄 main.o             # (Objetos temporales)
├── 📄 serial_reader.o    # (Objetos temporales)
└── 📄 README.md
```

---

## 🔍 Comandos Avanzados de Compilación

```cmd
:: Compilar con debugging symbols
g++ -std=c++17 -Wall -g main.cpp serial_reader.cpp -o microbit-reader-debug.exe -lwinmm

:: Compilar con optimizaciones máximas
g++ -std=c++17 -Wall -O3 -s main.cpp serial_reader.cpp -o microbit-reader-opt.exe -lwinmm

:: Compilar estáticamente
g++ -std=c++17 -Wall -O2 -static main.cpp serial_reader.cpp -o microbit-reader-static.exe -lwinmm

:: Ver información del ejecutable
dir microbit-reader.exe
```

---

## ⚡ Estadísticas del Ejecutable

| Configuración | Tamaño | Performance | Uso |
|---------------|--------|-------------|-----|
| **Debug** | ~500KB | Básica | Desarrollo |
| **Release** | ~150KB | Alta | Distribución |
| **Static** | ~1.5MB | Alta | Portable |
| **Optimizado** | ~100KB | Máxima | Producción |

---

## ✅ Verificación Final

```cmd
:: Verificar compilador
g++ --version

:: Verificar proyecto
cd microbit-cpp
make clean
make
.\microbit-reader.exe
```
