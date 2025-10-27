# 🔧 Micro:bit Reader - Go

## 📋 Descripción
Lector de datos seriales desde micro:bit implementado en **Go**. Recibe datos JSON en tiempo real de sensores y detecta alertas automáticamente.

---

## 🏗️ Estructura del Proyecto Go
```
microbit-go/
├── 📄 main.go             # Código principal
├── 📄 go.mod              # Módulo y dependencias
├── 📄 go.sum              # Checksums de dependencias
└── 📄 README.md           # Esta documentación
```

---

## 📦 Dependencias Requeridas

### Módulos Go:
- **go.bug.st/serial** v1.6.0 - Comunicación serial

---

## 🟢 Código Completo - `main.go`

```go
package main

import (
	"bufio"
	"encoding/json"
	"fmt"
	"log"
	"math"
	"time"

	"go.bug.st/serial"
)

// 🏷️ Estructura para los datos del micro:bit
type MicrobitData struct {
	ID    string  `json:"id"`
	TS    float64 `json:"ts"`
	TempC float64 `json:"tempC"`
	Ax    float64 `json:"ax"`
	Ay    float64 `json:"ay"`
	Az    float64 `json:"az"`
	Light int     `json:"light"`
	Bat   float64 `json:"bat"`
}

func main() {
	// ⚙️ CONFIGURACIÓN - AJUSTAR SEGÚN SISTEMA
	portName := "COM3" // Windows: COM3, COM4 | Linux: /dev/ttyACM0 | Mac: /dev/cu.usbmodem14102
	baudRate := 115200

	fmt.Println("🚀 Iniciando Lector micro:bit en Go")
	fmt.Printf("📡 Puerto: %s\n", portName)
	fmt.Printf("⚡ Baud Rate: %d\n", baudRate)
	fmt.Println("⏳ Esperando datos...\n")

	// 🔌 Configurar modo del puerto serial
	mode := &serial.Mode{
		BaudRate: baudRate,
		DataBits: 8,
		Parity:   serial.NoParity,
		StopBits: serial.OneStopBit,
	}

	// Abrir puerto serial
	port, err := serial.Open(portName, mode)
	if err != nil {
		log.Fatalf("❌ Error abriendo puerto: %v", err)
	}
	defer port.Close()

	// Configurar timeout de lectura
	port.SetReadTimeout(1 * time.Second)

	fmt.Println("✅ Puerto serial abierto correctamente")
	fmt.Println("📊 Leyendo datos del micro:bit...")
	fmt.Println("💡 Presiona Ctrl + C para detener\n")

	// Crear scanner para leer líneas
	scanner := bufio.NewScanner(port)
	
	// 🔄 Bucle principal de lectura
	for scanner.Scan() {
		line := scanner.Text()
		if line != "" {
			processData(line)
		}
	}

	// Manejar error del scanner
	if err := scanner.Err(); err != nil {
		log.Printf("❌ Error leyendo datos: %v", err)
	}
}

// 📊 Procesar datos recibidos del micro:bit
func processData(jsonData string) {
	var data MicrobitData
	err := json.Unmarshal([]byte(jsonData), &data)
	if err != nil {
		fmt.Printf("❌ JSON inválido: %s\n", jsonData)
		return
	}

	// Validar datos mínimos
	if data.ID == "" {
		fmt.Printf("❌ Datos incompletos: %s\n", jsonData)
		return
	}

	// 🖥️ Mostrar datos en consola
	fmt.Printf("📱 Dispositivo: %s\n", data.ID)
	fmt.Printf("⏰ Timestamp: %.0f\n", data.TS)
	fmt.Printf("🌡️ Temperatura: %.1f°C\n", data.TempC)
	fmt.Printf("📊 Acelerómetro: X=%.3f Y=%.3f Z=%.3f\n", data.Ax, data.Ay, data.Az)
	fmt.Printf("💡 Luz: %d\n", data.Light)
	fmt.Printf("🔋 Batería: %.2fV\n", data.Bat)

	// 📈 Calcular y mostrar magnitud de aceleración
	magnitude := calculateMagnitude(data.Ax, data.Ay, data.Az)
	fmt.Printf("📈 Magnitud aceleración: %.3fg\n", magnitude)

	// 🚨 Verificar y mostrar alertas
	checkAlerts(data, magnitude)
	
	fmt.Println("──────────────────────────────────────────")
}

// 📐 Función para calcular magnitud de aceleración
func calculateMagnitude(ax, ay, az float64) float64 {
	return math.Sqrt(ax*ax + ay*ay + az*az)
}

// 🚨 Función para verificar alertas
func checkAlerts(data MicrobitData, magnitude float64) {
	alerts := []string{}

	if magnitude > 1.5 {
		alerts = append(alerts, "🚨 MOVIMIENTO BRUSCO")
	}
	if data.TempC > 30 {
		alerts = append(alerts, "🌡️ ALTA TEMPERATURA")
	}
	if data.Light < 20 {
		alerts = append(alerts, "🌑 BAJA LUZ")
	}
	if data.Bat < 3.0 {
		alerts = append(alerts, "🔋 BATERÍA BAJA")
	}

	if len(alerts) > 0 {
		fmt.Println("🚨 ALERTAS DETECTADAS:")
		for _, alert := range alerts {
			fmt.Printf("   %s\n", alert)
		}
	}
}
```

---

## 📄 go.mod

```mod
module microbit-reader

go 1.21

require go.bug.st/serial v1.6.0

require (
	github.com/creack/goselect v0.1.2 // indirect
	golang.org/x/sys v0.0.0-20220829200755-d48e67d00261 // indirect
)
```

---

## 🚀 Guía de Ejecución Paso a Paso - WINDOWS/CMD

### 📋 Prerrequisitos
- ✅ Go instalado (versión 1.21 o superior)
- ✅ GOPATH configurado (opcional con módules)
- ✅ micro:bit con código MicroPython cargado

### 🔧 Paso 1: Crear Carpeta del Proyecto
```cmd
:: Navegar al disco E:
E:

:: Crear carpeta del proyecto
mkdir microbit-go
cd microbit-go
```

### 📦 Paso 2: Inicializar Módulo Go
```cmd
:: Inicializar módulo Go
go mod init microbit-reader

:: Verificar que se creó go.mod
dir
```

### 💻 Paso 3: Crear Archivo Principal
```cmd
:: Crear main.go con Notepad
notepad main.go
```
**Pegar el código Go completo y Guardar**

### 🔍 Paso 4: Instalar Dependencias
```cmd
:: Descargar e instalar dependencias
go get go.bug.st/serial

:: O instalar todas las dependencias del módulo
go mod tidy

:: Verificar dependencias instaladas
go list -m all
```

### 🔍 Paso 5: Encontrar Puerto COM
```cmd
:: Usar PowerShell desde CMD
powershell "[System.IO.Ports.SerialPort]::getportnames()"
```

### ⚙️ Paso 6: Configurar Puerto (Si es necesario)
```cmd
:: Editar main.go línea 23
notepad main.go
```
**Buscar y cambiar:**
```go
portName := "COM3" // ← Cambiar por tu puerto (COM4, COM5, etc.)
```

### 🎯 Paso 7: Compilar y Ejecutar
```cmd
:: Ejecutar directamente (compila y ejecuta)
go run main.go

:: O compilar primero y luego ejecutar
go build
.\microbit-reader.exe

:: Compilar para distribución
go build -ldflags="-s -w"
```

---

## 🖥️ Comandos Rápidos de CMD para Go

```cmd
:: Verificar instalación Go
go version
where go

:: Comandos útiles del proyecto
go run main.go
go build
go clean
go mod tidy

:: Ver información del módulo
go list -m all
go mod graph

:: Ejecutar el binario compilado
.\microbit-reader.exe
```

---

## 📊 Salida Esperada en CMD

```
🚀 Iniciando Lector micro:bit en Go
📡 Puerto: COM3
⚡ Baud Rate: 115200
⏳ Esperando datos...

✅ Puerto serial abierto correctamente
📊 Leyendo datos del micro:bit...
💡 Presiona Ctrl + C para detener

📱 Dispositivo: M1
⏰ Timestamp: 1699999999
🌡️ Temperatura: 27.1°C
📊 Acelerómetro: X=-0.030 Y=0.980 Z=0.050
💡 Luz: 123
🔋 Batería: 3.01V
📈 Magnitud aceleración: 0.982g
──────────────────────────────────────────
```

---

## 🔧 Solución de Problemas - GO/WINDOWS

### ❌ Error: "go: cannot find main module"
```cmd
:: Solución: Inicializar módulo
go mod init microbit-reader
```

### ❌ Error: "port not found"
```cmd
:: Ver puertos disponibles
powershell "[System.IO.Ports.SerialPort]::getportnames()"

:: Cambiar puerto en main.go
notepad main.go
```

### ❌ Error: "access denied"
- Ejecutar CMD como Administrador
- Cerrar Arduino IDE, Mu Editor, Putty
- Reiniciar micro:bit

### ❌ Error: "cannot find package"
```cmd
:: Descargar dependencias
go mod tidy
go get go.bug.st/serial

:: Limpiar cache
go clean -modcache
```

### ❌ Error: "Go not found"
```cmd
:: Verificar instalación y PATH
where go
go version

:: Si Go está en F:, configurar PATH temporal
set PATH=F:\go\bin;%PATH%
```

### ❌ Error: "timeout reading serial port"
```go
// Aumentar timeout en main.go línea 43
port.SetReadTimeout(5 * time.Second)
```

---

## 🚨 Sistema de Alertas en Go

El programa detecta automáticamente:

| Condición | Cálculo | Alerta | Umbral |
|-----------|---------|--------|---------|
| Movimiento brusco | `math.Sqrt(ax²+ay²+az²) > 1.5` | 🚨 MOVIMIENTO BRUSCO | > 1.5g |
| Alta temperatura | `TempC > 30` | 🌡️ ALTA TEMPERATURA | > 30°C |
| Baja luminosidad | `Light < 20` | 🌑 BAJA LUZ | < 20 |
| Batería baja | `Bat < 3.0` | 🔋 BATERÍA BAJA | < 3.0V |

---

## 💾 Características Técnicas de Go

- ✅ **Compilación rápida** - Builds en segundos
- ✅ **Ejecutable único** - No necesita runtime externo
- ✅ **Concurrencia nativa** - Goroutines y channels
- ✅ **Recolección de basura** - Memory management automático
- ✅ **Tipado estático** - Detección de errores en compilación
- ✅ **Sintaxis simple** - Fácil de leer y mantener

---

## 🎯 Ventajas de Go para este Proyecto

1. **Compilación rápida** - Ideal para desarrollo iterativo
2. **Ejecutable autocontenido** - Fácil distribución
3. **Concurrencia simple** - Goroutines para I/O asíncrono
4. **Performance excelente** - Cercano a C++ pero más seguro
5. **Tooling integrado** - go fmt, go test, go vet
6. **Cross-compilation** - Fácil compilación para otras plataformas

---

## 📁 Estructura Final del Proyecto

```
microbit-go/
├── 📄 main.go
├── 📄 go.mod
├── 📄 go.sum              # (Generado automáticamente)
├── 📄 microbit-reader.exe # (Generado con go build)
└── 📄 README.md
```

---

## 🔍 Comandos Avanzados de Go

```cmd
:: Compilar para otras plataformas
go build -o microbit-reader-linux linux
go build -o microbit-reader-macos macos

:: Ver información de compilación
go version -m microbit-reader.exe

:: Formatear código automáticamente
go fmt

:: Ejecutar tests (si los hay)
go test

:: Analizar código estáticamente
go vet

:: Ver documentación
go doc go.bug.st/serial
```

---

## ⚡ Características del Binario Compilado

```cmd
:: Compilar con optimizaciones
go build -ldflags="-s -w" -o microbit-reader.exe

:: Tamaño típico del ejecutable
:: ~5-8 MB (dependiendo de las optimizaciones)

:: Ejecutar sin consola (Windows)
go build -ldflags="-s -w -H=windowsgui"
```

**Tamaños aproximados:**
- **Normal:** ~8 MB
- **Optimizado:** ~5 MB
- **Comprimido:** ~2 MB

---

## ✅ Verificación Final

```cmd
:: Verificar instalación Go
go version

:: Verificar proyecto
cd microbit-go
go mod tidy
go run main.go
```

**Indicadores de éxito:**
- ✅ `go run main.go` compila sin errores
- ✅ "Puerto serial abierto correctamente" en consola
- ✅ Datos del micro:bit aparecen en tiempo real
- ✅ Alertas se activan cuando corresponden
- ✅ Programa responde a Ctrl + C

---

## 📝 Notas Importantes para Go

1. **Módulos Go** - Usar `go mod` en lugar de GOPATH para nuevos proyectos
2. **Error handling** - Go no tiene excepciones, usar múltiples returns
3. **Defer statements** - Garantizar cierre de recursos
4. **Convenciones** - `go fmt` formatea automáticamente al estilo estándar
5. **Cross-compilation** - Muy fácil compilar para Linux/Mac desde Windows

---
