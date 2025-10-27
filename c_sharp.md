# 🔧 Micro:bit Reader - C# (Visual Studio)

## 📋 Descripción
Lector de datos seriales desde micro:bit implementado en **C#** con Visual Studio 2022. Recibe datos JSON en tiempo real de sensores y detecta alertas automáticamente.

---

## 🏗️ Estructura del Proyecto Visual Studio
```
microbit-csharp/
├── 📄 Program.cs          # Código principal
├── 📄 microbit.csproj     # Archivo del proyecto
├── 📄 packages.config     # Paquetes NuGet (opcional)
├── 📄 README.md           # Esta documentación
└── 📁 bin/
    └── 📁 Debug/
        └── microbit.exe   # Ejecutable compilado
```

---

## 📦 Dependencias Requeridas

### Paquetes NuGet:
- **System.IO.Ports** >= 7.0.0 - Comunicación serial
- **System.Text.Json** >= 7.0.0 - Parseo JSON (incluido en .NET)

---

## 💻 Código Completo - `Program.cs`

```csharp
using System;
using System.IO.Ports;
using System.Text.Json;

namespace MicrobitReader
{
    class Program
    {
        static void Main(string[] args)
        {
            // ⚙️ CONFIGURACIÓN - AJUSTAR SEGÚN SISTEMA
            string portName = "COM3"; // Windows: COM3, COM4, COM5, etc.
            int baudRate = 115200;
            
            Console.WriteLine("🚀 Iniciando Lector micro:bit en C#");
            Console.WriteLine($"📡 Puerto: {portName}");
            Console.WriteLine($"⚡ Baud Rate: {baudRate}");
            Console.WriteLine("⏳ Esperando datos...\n");

            using (SerialPort port = new SerialPort(portName, baudRate))
            {
                port.NewLine = "\n"; // Importante para micro:bit
                
                // 🔌 Configurar evento de datos recibidos
                port.DataReceived += (sender, e) =>
                {
                    try
                    {
                        string line = port.ReadLine();
                        ProcessData(line);
                    }
                    catch (Exception ex)
                    {
                        Console.WriteLine($"❌ Error leyendo datos: {ex.Message}");
                    }
                };

                try
                {
                    // Abrir puerto serial
                    port.Open();
                    Console.WriteLine("✅ Puerto serial abierto correctamente");
                    Console.WriteLine("📊 Leyendo datos del micro:bit...");
                    Console.WriteLine("💡 Presiona Enter para salir...\n");
                    
                    // Mantener el programa ejecutándose
                    Console.ReadLine();
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"❌ Error abriendo puerto: {ex.Message}");
                    Console.WriteLine("\n🔧 Solución de problemas:");
                    Console.WriteLine("   1. Verifica que el micro:bit esté conectado por USB");
                    Console.WriteLine("   2. Confirma el nombre del puerto (COM3, COM4, etc.)");
                    Console.WriteLine("   3. Asegúrate de que el puerto no esté en uso");
                    Console.WriteLine("   4. Revisa que el micro:bit esté ejecutando el código correcto");
                }
            }
        }

        // 📊 Procesar datos recibidos del micro:bit
        static void ProcessData(string jsonData)
        {
            try
            {
                var data = JsonSerializer.Deserialize<MicrobitData>(jsonData);
                
                if (data == null || string.IsNullOrEmpty(data.Id))
                {
                    Console.WriteLine($"❌ Datos incompletos: {jsonData}");
                    return;
                }

                // 🖥️ Mostrar datos en consola
                Console.WriteLine($"📱 Dispositivo: {data.Id}");
                Console.WriteLine($"⏰ Timestamp: {data.Ts}");
                Console.WriteLine($"🌡️ Temperatura: {data.TempC:F1}°C");
                Console.WriteLine($"📊 Acelerómetro: X={data.Ax:F3} Y={data.Ay:F3} Z={data.Az:F3}");
                Console.WriteLine($"💡 Luz: {data.Light}");
                Console.WriteLine($"🔋 Batería: {data.Bat:F2}V");

                // 📈 Calcular y mostrar magnitud de aceleración
                double magnitude = CalculateMagnitude(data.Ax, data.Ay, data.Az);
                Console.WriteLine($"📈 Magnitud aceleración: {magnitude:F3}g");

                // 🚨 Verificar y mostrar alertas
                CheckAlerts(data, magnitude);
                
                Console.WriteLine(new string('─', 50));
            }
            catch (JsonException)
            {
                Console.WriteLine($"❌ Error parseando JSON: {jsonData.Trim()}");
            }
            catch (Exception ex)
            {
                Console.WriteLine($"❌ Error procesando datos: {ex.Message}");
            }
        }

        // 📐 Función para calcular magnitud de aceleración
        static double CalculateMagnitude(double x, double y, double z)
        {
            return Math.Sqrt(x * x + y * y + z * z);
        }

        // 🚨 Función para verificar alertas
        static void CheckAlerts(MicrobitData data, double magnitude)
        {
            bool hasAlerts = false;

            if (magnitude > 1.5)
            {
                Console.WriteLine("🚨 MOVIMIENTO BRUSCO");
                hasAlerts = true;
            }
            if (data.TempC > 30)
            {
                Console.WriteLine("🌡️ ALTA TEMPERATURA");
                hasAlerts = true;
            }
            if (data.Light < 20)
            {
                Console.WriteLine("🌑 BAJA LUZ");
                hasAlerts = true;
            }
            if (data.Bat < 3.0)
            {
                Console.WriteLine("🔋 BATERÍA BAJA");
                hasAlerts = true;
            }

            if (hasAlerts)
            {
                Console.WriteLine("🚨🚨🚨 ALERTAS DETECTADAS");
            }
        }
    }

    // 🏷️ Clase para deserializar JSON del micro:bit
    public class MicrobitData
    {
        public string Id { get; set; } = string.Empty;
        public double Ts { get; set; }
        public double TempC { get; set; }
        public double Ax { get; set; }
        public double Ay { get; set; }
        public double Az { get; set; }
        public int Light { get; set; }
        public double Bat { get; set; }
    }
}
```

---

## 📄 microbit.csproj

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <ApplicationIcon />
    <StartupObject />
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="System.IO.Ports" Version="7.0.0" />
  </ItemGroup>

</Project>
```

---

## 🚀 Guía de Ejecución Paso a Paso - VISUAL STUDIO 2022

### 🔧 Paso 1: Crear Nuevo Proyecto en Visual Studio
1. **Abrir Visual Studio 2022**
2. **Crear nuevo proyecto**: `File → New → Project`
3. **Seleccionar template**: `Console App (.NET)`
4. **Configurar proyecto**:
   - Project name: `microbit-csharp`
   - Location: `F:\iot-projects\csharp\`
   - Framework: `.NET 8.0`

### 📦 Paso 2: Agregar Dependencia NuGet
1. **Solution Explorer** → **clic derecho en Dependencies** → **Manage NuGet Packages**
2. **Buscar**: `System.IO.Ports`
3. **Instalar**: `System.IO.Ports` (versión 7.0.0 o superior)
4. **Verificar** en References que aparece `System.IO.Ports`

### 💻 Paso 3: Reemplazar Código en Program.cs
1. **Abrir** `Program.cs`
2. **Eliminar** todo el contenido
3. **Pegar** el código C# completo proporcionado
4. **Guardar** (`Ctrl + S`)

### ⚙️ Paso 4: Configurar Puerto COM
1. **Encontrar tu puerto COM**:
   - Abrir `Administrador de dispositivos`
   - `Puertos (COM y LPT)`
   - Anotar el puerto del micro:bit (ej: COM4)

2. **Editar** `Program.cs` línea 12:
```csharp
string portName = "COM3"; // ← Cambiar por tu puerto (COM4, COM5, etc.)
```

### 🎯 Paso 5: Compilar y Ejecutar
1. **Compilar proyecto**: `Build → Build Solution` (`Ctrl + Shift + B`)
2. **Ejecutar**: `Debug → Start Without Debugging` (`Ctrl + F5`)

---

## 🖥️ Comandos Alternativos - Terminal

### Compilar y ejecutar desde Terminal/CMD:
```cmd
:: Navegar al proyecto
cd F:\iot-projects\csharp\microbit-csharp

:: Compilar
dotnet build

:: Ejecutar
dotnet run

:: Publicar ejecutable independiente
dotnet publish -c Release -r win-x64 --self-contained
```

---

## 📊 Salida Esperada en Consola

```
🚀 Iniciando Lector micro:bit en C#
📡 Puerto: COM3
⚡ Baud Rate: 115200
⏳ Esperando datos...

✅ Puerto serial abierto correctamente
📊 Leyendo datos del micro:bit...
💡 Presiona Enter para salir...

📱 Dispositivo: M1
⏰ Timestamp: 1699999999
🌡️ Temperatura: 27.1°C
📊 Acelerómetro: X=-0.030 Y=0.980 Z=0.050
💡 Luz: 123
🔋 Batería: 3.01V
📈 Magnitud aceleración: 0.982g
──────────────────────────────────
```

---

## 🔧 Solución de Problemas - VISUAL STUDIO

### ❌ Error: "The type name 'SerialPort' could not be found"
```xml
<!-- Solución: Agregar referencia en .csproj -->
<PackageReference Include="System.IO.Ports" Version="7.0.0" />
```

### ❌ Error: "Port COM3 does not exist"
- Verificar puerto en Administrador de dispositivos
- Cambiar `portName` en `Program.cs`
- Reiniciar micro:bit

### ❌ Error: "Access to the port 'COM3' is denied"
- Cerrar Arduino IDE, Mu Editor, Putty
- Ejecutar Visual Studio como Administrador
- Reiniciar micro:bit

### ❌ Error: "JsonException"
- Verificar que el micro:bit esté enviando JSON válido
- Revisar código MicroPython en el micro:bit

### ❌ Error al compilar
- **Clean Solution**: `Build → Clean Solution`
- **Rebuild**: `Build → Rebuild Solution`
- **Restore NuGet**: `Tools → NuGet Package Manager → Package Manager Console` → `dotnet restore`

---

## 🚨 Sistema de Alertas en C#

El programa detecta automáticamente:

| Condición | Cálculo | Alerta | Umbral |
|-----------|---------|--------|---------|
| Movimiento brusco | `Math.Sqrt(ax²+ay²+az²) > 1.5` | 🚨 MOVIMIENTO BRUSCO | > 1.5g |
| Alta temperatura | `tempC > 30` | 🌡️ ALTA TEMPERATURA | > 30°C |
| Baja luminosidad | `light < 20` | 🌑 BAJA LUZ | < 20 |
| Batería baja | `bat < 3.0` | 🔋 BATERÍA BAJA | < 3.0V |

---

## 💾 Características Técnicas de C#

- ✅ **Programación orientada a objetos** con clases
- ✅ **Manejo de recursos** con `using` statements
- ✅ **Eventos asíncronos** con `DataReceived`
- ✅ **Tipado fuerte** para prevención de errores
- ✅ **Serialización JSON** nativa con `System.Text.Json`
- ✅ **Manejo de excepciones** completo

---

## 🎯 Ventajas de C# para este Proyecto

1. **Performance excelente** - Compilado a código nativo
2. **Tipado fuerte** - Menos errores en tiempo de ejecución
3. **IDE poderosa** - Visual Studio con debugging avanzado
4. **Sistema de eventos** - Perfecto para I/O asíncrono
5. **.NET Ecosystem** - Librerías robustas y mantenidas
6. **Windows nativo** - Integración perfecta con Windows

---

## 📁 Estructura Final del Proyecto

```
microbit-csharp/
├── 📄 Program.cs
├── 📄 microbit.csproj
├── 📄 obj/
├── 📁 bin/
│   └── 📁 Debug/
│       └── net8.0/
│           ├── microbit.exe
│           ├── microbit.dll
│           └── microbit.runtimeconfig.json
└── 📄 README.md
```

---

## 🔍 Debugging en Visual Studio

### Puntos de interrupción útiles:
1. **Línea 22**: `port.Open()` - Para verificar apertura de puerto
2. **Línea 28**: `port.ReadLine()` - Para inspeccionar datos crudos
3. **Línea 40**: `ProcessData(line)` - Para debuggear parseo JSON
4. **Línea 60**: `CheckAlerts()` - Para verificar sistema de alertas

### Ventanas de debug:
- **Watch**: Variables personalizadas
- **Locals**: Variables locales automáticas
- **Immediate**: Ejecutar código on-the-fly

---

## ✅ Verificación Final

1. **Compilación exitosa** sin errores
2. **Consola muestra** "Puerto serial abierto correctamente"
3. **Datos fluyen** desde el micro:bit
4. **Alertas funcionan** cuando se cumplen condiciones
5. **Programa responde** a Enter para salir

---
---

**¿Necesitas ayuda con algún paso específico de Visual Studio o la configuración del proyecto?**
