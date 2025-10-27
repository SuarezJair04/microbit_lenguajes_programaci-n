# 🔧 Micro:bit Reader - Rust

## 📋 Descripción
Lector de datos seriales desde micro:bit implementado en **Rust**. Recibe datos JSON en tiempo real de sensores y detecta alertas automáticamente. Compilado con toolchain **stable-gnu**.

---

## 🏗️ Estructura del Proyecto Rust
```
microbit-rust/
├── 📄 Cargo.toml          # Configuración y dependencias
├── 📁 src/
│   └── 📄 main.rs         # Código principal
├── 📁 target/             # Archivos compilados
└── 📄 README.md           # Esta documentación
```

---

## 📦 Dependencias Requeridas

### Crates Rust:
- **serialport** = "4.2" - Comunicación serial
- **serde** = { version = "1.0", features = ["derive"] } - Serialización
- **serde_json** = "1.0" - Parseo JSON

---

## 🦀 Código Completo - `src/main.rs`

```rust
use serialport::{self, SerialPort};
use std::io::{BufRead, BufReader};
use std::time::Duration;
use serde::Deserialize;
use std::process;

#[derive(Debug, Deserialize)]
struct MicrobitData {
    id: String,
    ts: f64,
    #[serde(rename = "tempC")]
    temp_c: f64,
    ax: f64,
    ay: f64,
    az: f64,
    light: u32,
    bat: f64,
}

fn main() {
    // ⚙️ CONFIGURACIÓN - AJUSTAR SEGÚN SISTEMA
    let port_name = "COM3"; // Windows: COM3, COM4 | Linux: /dev/ttyACM0 | Mac: /dev/cu.usbmodem14102
    let baud_rate = 115200;

    println!("🚀 Iniciando Lector micro:bit en Rust");
    println!("📡 Puerto: {}", port_name);
    println!("⚡ Baud Rate: {}", baud_rate);
    println!("⏳ Esperando datos...\n");

    // 🔌 Configurar y abrir puerto serial
    let port = match serialport::new(port_name, baud_rate)
        .data_bits(serialport::DataBits::Eight)
        .stop_bits(serialport::StopBits::One)
        .parity(serialport::Parity::None)
        .timeout(Duration::from_millis(1000))
        .open() {
            Ok(port) => port,
            Err(e) => {
                eprintln!("❌ Error abriendo puerto: {}", e);
                eprintln!("🔧 Solución:");
                eprintln!("   1. Verifica que el micro:bit esté conectado por USB");
                eprintln!("   2. Ajusta el nombre del puerto (COM3, COM4, etc.)");
                eprintln!("   3. Asegúrate que el puerto no esté en uso");
                process::exit(1);
            }
        };

    println!("✅ Puerto serial abierto correctamente");
    println!("📊 Leyendo datos del micro:bit...");
    println!("💡 Presiona Ctrl + C para detener\n");

    let reader = BufReader::new(port);

    // 🔄 Bucle principal de lectura
    for line in reader.lines() {
        match line {
            Ok(line) => {
                let trimmed_line = line.trim();
                if !trimmed_line.is_empty() {
                    process_data(trimmed_line);
                }
            }
            Err(e) => {
                eprintln!("❌ Error leyendo datos: {}", e);
            }
        }
    }
}

// 📊 Procesar datos recibidos del micro:bit
fn process_data(json_data: &str) {
    match serde_json::from_str::<MicrobitData>(json_data) {
        Ok(data) => {
            // 🖥️ Mostrar datos en consola
            println!("📱 Dispositivo: {}", data.id);
            println!("⏰ Timestamp: {:.0}", data.ts);
            println!("🌡️ Temperatura: {:.1}°C", data.temp_c);
            println!("📊 Acelerómetro:");
            println!("   X: {:.3} g", data.ax);
            println!("   Y: {:.3} g", data.ay);
            println!("   Z: {:.3} g", data.az);
            println!("💡 Nivel de luz: {}", data.light);
            println!("🔋 Batería: {:.2}V", data.bat);

            // 📈 Calcular y mostrar magnitud de aceleración
            let magnitude = (data.ax.powi(2) + data.ay.powi(2) + data.az.powi(2)).sqrt();
            println!("📈 Magnitud aceleración: {:.3}g", magnitude);

            // 🚨 Verificar y mostrar alertas
            check_alerts(&data, magnitude);
            
            println!("{}", "─".repeat(50));
        }
        Err(e) => {
            println!("❌ Error parseando JSON: {}", e);
            println!("📝 Dato recibido: {}", json_data);
        }
    }
}

// 🚨 Función para verificar alertas
fn check_alerts(data: &MicrobitData, magnitude: f64) {
    let mut alerts = Vec::new();

    if magnitude > 1.5 {
        alerts.push("🚨 MOVIMIENTO BRUSCO");
    }
    if data.temp_c > 30.0 {
        alerts.push("🌡️ ALTA TEMPERATURA");
    }
    if data.light < 20 {
        alerts.push("🌑 BAJA LUZ");
    }
    if data.bat < 3.0 {
        alerts.push("🔋 BATERÍA BAJA");
    }

    if !alerts.is_empty() {
        println!("🚨 ALERTAS DETECTADAS:");
        for alert in alerts {
            println!("   {}", alert);
        }
    }
}
```

---

## 📄 Cargo.toml

```toml
[package]
name = "microbit-rust"
version = "1.0.0"
edition = "2021"

[dependencies]
serialport = "4.2"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

---

## 🚀 Guía de Ejecución Paso a Paso - WINDOWS/CMD

### 📋 Prerrequisitos
- ✅ Rust instalado con `stable-gnu` toolchain
- ✅ Cargo disponible en PATH
- ✅ micro:bit con código MicroPython cargado

### 🔧 Paso 1: Crear Proyecto con Cargo
```cmd
:: Navegar al disco E:
E:

:: Crear proyecto Rust
cargo new microbit-rust
cd microbit-rust

:: Verificar estructura creada
dir
dir src
```

### 📦 Paso 2: Configurar Dependencias
```cmd
:: Editar Cargo.toml
notepad Cargo.toml
```
**Reemplazar contenido con:**
```toml
[package]
name = "microbit-rust"
version = "1.0.0"
edition = "2021"

[dependencies]
serialport = "4.2"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

### 💻 Paso 3: Crear Código Principal
```cmd
:: Editar main.rs
notepad src\main.rs
```
**Pegar el código Rust completo y Guardar**

### 🔍 Paso 4: Encontrar Puerto COM
```cmd
:: Usar PowerShell desde CMD
powershell "[System.IO.Ports.SerialPort]::getportnames()"
```

### ⚙️ Paso 5: Configurar Puerto (Si es necesario)
```cmd
:: Editar src/main.rs línea 16
notepad src\main.rs
```
**Buscar y cambiar:**
```rust
let port_name = "COM3"; // ← Cambiar por tu puerto (COM4, COM5, etc.)
```

### 🎯 Paso 6: Compilar y Ejecutar
```cmd
:: Compilar y ejecutar (descarga dependencias automáticamente)
cargo run

:: O solo compilar
cargo build

:: Ejecutar el binario compilado
.\target\debug\microbit-rust.exe
```

---

## 🖥️ Comandos Rápidos de CMD para Rust

```cmd
:: Verificar instalación Rust
rustc --version
cargo --version
rustup show

:: Comandos útiles del proyecto
cargo clean
cargo update
cargo run --release

:: Ver binario generado
dir target\debug\
.\target\debug\microbit-rust.exe
```

---

## 📊 Salida Esperada en CMD

```
🚀 Iniciando Lector micro:bit en Rust
📡 Puerto: COM3
⚡ Baud Rate: 115200
⏳ Esperando datos...

✅ Puerto serial abierto correctamente
📊 Leyendo datos del micro:bit...
💡 Presiona Ctrl + C para detener

📱 Dispositivo: M1
⏰ Timestamp: 1699999999
🌡️ Temperatura: 27.1°C
📊 Acelerómetro:
   X: -0.030 g
   Y: 0.980 g
   Z: 0.050 g
💡 Nivel de luz: 123
🔋 Batería: 3.01V
📈 Magnitud aceleración: 0.982g
──────────────────────────────────
```

---

## 🔧 Solución de Problemas - RUST/WINDOWS

### ❌ Error: "linker `link.exe` not found"
```cmd
:: Solución: Usar toolchain GNU
rustup default stable-gnu
rustup show

:: Verificar
rustc --version
```

### ❌ Error: "Port not found"
```cmd
:: Ver puertos disponibles
powershell "[System.IO.Ports.SerialPort]::getportnames()"

:: Cambiar puerto en main.rs
notepad src\main.rs
```

### ❌ Error: "Access denied"
- Ejecutar CMD como Administrador
- Cerrar otras aplicaciones usando el puerto
- Reiniciar micro:bit

### ❌ Error: "Could not resolve dependencies"
```cmd
:: Actualizar cargo y limpiar cache
cargo clean
cargo update

:: Forzar descarga de crates
cargo fetch
```

### ❌ Error: "Serial port error"
```rust
// Verificar configuración del puerto en main.rs
let port = serialport::new(port_name, baud_rate)
    .data_bits(serialport::DataBits::Eight)
    .stop_bits(serialport::StopBits::One)
    .parity(serialport::Parity::None)
    .timeout(Duration::from_millis(1000))
```

---

## 🚨 Sistema de Alertas en Rust

El programa detecta automáticamente:

| Condición | Cálculo | Alerta | Umbral |
|-----------|---------|--------|---------|
| Movimiento brusco | `(ax²+ay²+az²).sqrt() > 1.5` | 🚨 MOVIMIENTO BRUSCO | > 1.5g |
| Alta temperatura | `temp_c > 30.0` | 🌡️ ALTA TEMPERATURA | > 30°C |
| Baja luminosidad | `light < 20` | 🌑 BAJA LUZ | < 20 |
| Batería baja | `bat < 3.0` | 🔋 BATERÍA BAJA | < 3.0V |

---

## 💾 Características Técnicas de Rust

- ✅ **Memory safety** - Sin punteros nulos ni memory leaks
- ✅ **Concurrencia segura** - Ownership system previene data races
- ✅ **Performance nativo** - Compilado a código máquina
- ✅ **Manejo de errores** con `Result<>` y `Option<>`
- ✅ **Pattern matching** - `match` statements robustos
- ✅ **Zero-cost abstractions** - Alto nivel sin penalización de performance

---

## 🎯 Ventajas de Rust para este Proyecto

1. **Performance excelente** - Cercano a C/C++
2. **Seguridad de memoria** - Sin segmentation faults
3. **Sistema de tipos poderoso** - Previene errores en compilación
4. **Tooling moderno** - Cargo, rustfmt, clippy
5. **Fearless concurrency** - Perfecto para aplicaciones I/O
6. **Multiplataforma** - Mismo código para Windows/Linux/Mac

---

## 📁 Estructura Final del Proyecto

```
microbit-rust/
├── 📄 Cargo.toml
├── 📄 Cargo.lock          # (Generado automáticamente)
├── 📁 src/
│   └── 📄 main.rs
├── 📁 target/
│   ├── 📁 debug/
│   │   ├── microbit-rust.exe
│   │   └── build/
│   └── 📁 release/
└── 📄 README.md
```

---

## 🔍 Comandos Avanzados de Cargo

```cmd
:: Compilar en modo release (optimizado)
cargo build --release
.\target\release\microbit-rust.exe

:: Ver dependencias del proyecto
cargo tree

:: Formatear código automáticamente
cargo fmt

:: Verificar código sin compilar
cargo check

:: Ejecutar tests (si los hay)
cargo test
```

---

## ⚡ Modo Release vs Debug

| Aspecto | Debug | Release |
|---------|-------|---------|
| **Compilación** | Rápida | Lenta (optimizaciones) |
| **Ejecutable** | ~3-5MB | ~1-2MB |
| **Performance** | Básica | Máxima |
| **Debugging** | Símbolos completos | Símbolos limitados |
| **Uso** | Desarrollo | Producción |

**Para este proyecto:** Usar `cargo run` (debug) es suficiente.

---

## ✅ Verificación Final

```cmd
:: Verificar instalación
rustc --version
cargo --version

:: Verificar proyecto
cd microbit-rust
cargo check
cargo run
```

**Indicadores de éxito:**
- ✅ `cargo run` compila sin errores
- ✅ "Puerto serial abierto correctamente" en consola
- ✅ Datos del micro:bit aparecen en tiempo real
- ✅ Alertas se activan cuando corresponden

---

## 🛠️ Herramientas Útiles del Ecosistema Rust

- **rustup** - Gestor de toolchains
- **cargo** - Gestor de paquetes y builds
- **rustfmt** - Formateador de código
- **clippy** - Linter para mejores prácticas
- **rust-analyzer** - Soporte IDE

---

**¡Si `cargo run` funciona y ves datos del micro:bit, tu proyecto Rust está listo!**
