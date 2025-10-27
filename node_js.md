# 🔧 Micro:bit Reader - Node.js - Windows/CMD

## 📋 Descripción
Lector de datos seriales desde micro:bit implementado en Node.js para **Windows/Command Prompt**. Recibe datos JSON en tiempo real de sensores y detecta alertas automáticamente.

---

## 🏗️ Estructura de Archivos
```
nodejs/
├── 📄 reader.js          # Código principal
├── 📄 package.json       # Configuración y dependencias
├── 📄 README.md          # Esta documentación
└── 📁 node_modules/      # Dependencias (se crea automáticamente)
```

---

## 📦 Dependencias Requeridas

### Paquetes npm:
- **serialport** ^12.0.0 - Comunicación serial
- **@serialport/parser-readline** ^12.0.0 - Parser de líneas seriales

---

## 🧩 Código Completo - `reader.js`

```javascript
const SerialPort = require('serialport');
const Readline = require('@serialport/parser-readline');
const fs = require('fs');

// ⚙️ CONFIGURACIÓN - AJUSTAR SEGÚN SISTEMA
const PORT_NAME = 'COM3'; // Windows: COM3, COM4, COM5, etc.
const BAUD_RATE = 115200;

console.log('🚀 Iniciando Lector Serial micro:bit con Node.js');
console.log('📡 Puerto:', PORT_NAME);
console.log('⚡ Baud Rate:', BAUD_RATE);
console.log('⏳ Esperando datos...\n');

// 📁 Crear archivo de log para datos históricos
const logStream = fs.createWriteSync('microbit-data.log', { flags: 'a' });
logStream.writeSync(`=== Inicio de sesion: ${new Date().toISOString()} ===\n`);

// 🔌 Configurar puerto serial
const port = new SerialPort(PORT_NAME, { 
    baudRate: BAUD_RATE,
    dataBits: 8,
    stopBits: 1,
    parity: 'none'
});

const parser = port.pipe(new Readline({ delimiter: '\n' }));

// 📐 Función para calcular magnitud de aceleración
function calculateAccelerationMagnitude(ax, ay, az) {
    return Math.sqrt(ax * ax + ay * ay + az * az);
}

// 🚨 Función para verificar alertas
function checkAlerts(data) {
    const accelMagnitude = calculateAccelerationMagnitude(data.ax, data.ay, data.az);
    const alerts = [];

    if (accelMagnitude > 1.5) alerts.push('🚨 MOVIMIENTO BRUSCO');
    if (data.tempC > 30) alerts.push('🌡️ ALTA TEMPERATURA');
    if (data.light < 20) alerts.push('🌑 BAJA LUZ');
    if (data.bat < 3.0) alerts.push('🔋 BATERIA BAJA');

    return alerts;
}

// 🕐 Función para formatear timestamp
function formatTimestamp(timestamp) {
    return new Date(timestamp * 1000).toLocaleTimeString();
}

// 📊 Procesar datos recibidos del micro:bit
parser.on('data', (line) => {
    try {
        const cleanLine = line.trim();
        
        // Ignorar líneas vacías o que no son JSON
        if (!cleanLine || !cleanLine.startsWith('{')) {
            return;
        }

        const data = JSON.parse(cleanLine);
        
        // Validar datos mínimos requeridos
        if (!data.id || data.tempC === undefined) {
            console.log('❌ DATOS INCOMPLETOS:', cleanLine);
            return;
        }

        // 🖥️ Mostrar datos en consola
        console.log('📱 DISPOSITIVO:', data.id);
        console.log('🕐 FECHA/HORA:', formatTimestamp(data.ts));
        console.log('🌡️ TEMPERATURA:', data.tempC.toFixed(1) + '°C');
        console.log('📊 ACELEROMETRO:');
        console.log('   X:', data.ax.toFixed(3), 'g');
        console.log('   Y:', data.ay.toFixed(3), 'g'); 
        console.log('   Z:', data.az.toFixed(3), 'g');
        console.log('💡 NIVEL DE LUZ:', data.light);
        console.log('🔋 VOLTAJE BATERIA:', data.bat.toFixed(2) + 'V');

        // 📈 Calcular y mostrar magnitud de aceleración
        const accelMagnitude = calculateAccelerationMagnitude(data.ax, data.ay, data.az);
        console.log('📈 MAGNITUD ACELERACION:', accelMagnitude.toFixed(3), 'g');

        // 🚨 Verificar y mostrar alertas
        const alerts = checkAlerts(data);
        if (alerts.length > 0) {
            console.log('🚨🚨🚨 ALERTAS DETECTADAS:');
            alerts.forEach(alert => console.log('   ' + alert));
        }

        console.log('────────────────────────────────────────────────────────────');

        // 💾 Guardar en archivo de log
        logStream.writeSync(`${new Date().toISOString()} - ${cleanLine}\n`);
        if (alerts.length > 0) {
            logStream.writeSync(`ALERTAS: ${alerts.join(', ')}\n`);
        }

    } catch (error) {
        console.log('❌ ERROR PARSEANDO JSON:', error.message);
        console.log('📝 LINEA RECIBIDA:', line.trim());
    }
});

// 🔌 Manejar eventos del puerto serial
port.on('open', () => {
    console.log('✅ PUERTO SERIAL ABIERTO CORRECTAMENTE');
    console.log('📊 LISTO PARA RECIBIR DATOS DEL MICRO:BIT\n');
});

port.on('error', (err) => {
    console.log('❌ ERROR DEL PUERTO SERIAL:');
    console.log('   Mensaje:', err.message);
    console.log('\n🔧 SOLUCION DE PROBLEMAS:');
    console.log('   1. Verifica que el micro:bit este conectado por USB');
    console.log('   2. Confirma el nombre del puerto (COM3, COM4, etc.)');
    console.log('   3. Asegurate de que el puerto no este en uso');
    console.log('   4. Revisa que el micro:bit este ejecutando el codigo correcto');
});

port.on('close', () => {
    console.log('🔌 PUERTO SERIAL CERRADO');
    logStream.end();
});

// 🛑 Manejar cierre graceful del programa
process.on('SIGINT', () => {
    console.log('\n👋 CERRANDO PROGRAMA...');
    port.close();
    process.exit();
});

console.log('💡 Presiona Ctrl + C para detener el programa');
```

---

## 📄 package.json

```json
{
  "name": "microbit-serial-reader",
  "version": "1.0.0",
  "description": "Lector serial para micro:bit en Node.js - Proyecto IoT SP",
  "main": "reader.js",
  "scripts": {
    "start": "node reader.js",
    "test": "echo \"Ejecuta el codigo en el micro:bit primero\" && exit 1"
  },
  "keywords": [
    "microbit",
    "serial",
    "iot",
    "sensors",
    "nodejs"
  ],
  "author": "Alberto",
  "license": "MIT",
  "dependencies": {
    "serialport": "^12.0.0",
    "@serialport/parser-readline": "^12.0.0"
  }
}
```

---

## 🚀 Guía de Ejecución Paso a Paso - WINDOWS/CMD

### 📋 Prerrequisitos
- ✅ Node.js instalado en `F:\nodejs\`
- ✅ micro:bit con código MicroPython cargado
- ✅ Conexión USB del micro:bit a la computadora

### 🔧 Paso 1: Crear Carpeta y Archivos en CMD
```cmd
:: Navegar al disco F:
F:

:: Crear carpeta del proyecto
mkdir nodejs-project
cd nodejs-project

:: Crear package.json manualmente
echo { > package.json
echo   "name": "microbit-serial-reader", >> package.json
echo   "version": "1.0.0", >> package.json
echo   "description": "Lector serial para micro:bit", >> package.json
echo   "main": "reader.js", >> package.json
echo   "dependencies": { >> package.json
echo     "serialport": "^12.0.0", >> package.json
echo     "@serialport/parser-readline": "^12.0.0" >> package.json
echo   } >> package.json
echo } >> package.json
```

### 🔧 Paso 2: Crear reader.js en CMD
```cmd
:: Crear archivo reader.js con Notepad
notepad reader.js
```
**Pegar el código JavaScript completo y Guardar**

### 📦 Paso 3: Instalar Dependencias en CMD
```cmd
:: Configurar PATH temporal para Node.js
set PATH=F:\nodejs;%PATH%

:: Instalar dependencias
npm install
```

### 🔍 Paso 4: Encontrar Puerto COM en Windows
```cmd
:: Método 1 - PowerShell desde CMD
powershell "[System.IO.Ports.SerialPort]::getportnames()"

:: Método 2 - Administrador de dispositivos
:: Presiona Windows + R, escribe "devmgmt.msc"
:: Ve a "Puertos (COM y LPT)"
```

### ⚙️ Paso 5: Configurar Puerto (Si es necesario)
```cmd
:: Editar reader.js para cambiar el puerto COM
notepad reader.js
```
**Buscar esta línea y cambiar:**
```javascript
const PORT_NAME = 'COM3'; // ← Cambiar por tu puerto (COM4, COM5, etc.)
```

### 🎯 Paso 6: Ejecutar el Programa en CMD
```cmd
:: Ejecutar con Node.js
node reader.js

:: O usando la ruta completa si no está en PATH
F:\nodejs\node.exe reader.js
```

---

## 🖥️ Comandos Rápidos de CMD

```cmd
:: Navegación básica
F:
cd nodejs-project
dir

:: Instalación rápida
set PATH=F:\nodejs;%PATH%
npm install
node reader.js

:: Ver archivos creados
dir
type package.json
```

---

## 📊 Salida Esperada en CMD

```
🚀 Iniciando Lector Serial micro:bit con Node.js
📡 Puerto: COM3
⚡ Baud Rate: 115200
⏳ Esperando datos...

✅ PUERTO SERIAL ABIERTO CORRECTAMENTE
📊 LISTO PARA RECIBIR DATOS DEL MICRO:BIT

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

## 🔧 Solución de Problemas - WINDOWS

### ❌ Error: "npm no se reconoce"
```cmd
:: Solución: Configurar PATH
set PATH=F:\nodejs;%PATH%
```

### ❌ Error: "Puerto COM no encontrado"
```cmd
:: Ver puertos disponibles
powershell "[System.IO.Ports.SerialPort]::getportnames()"

:: Probar diferentes puertos COM
:: Editar reader.js y cambiar COM3 por COM4, COM5, etc.
```

### ❌ Error: "Access denied to COM port"
- Cerrar el Arduino IDE si está abierto
- Cerrar el Mu Editor si está abierto
- Reiniciar el micro:bit

### ❌ Error: "Serialport installation failed"
```cmd
:: Limpiar cache y reinstalar
npm cache clean --force
npm install
```

### ❌ Error: "Node.js no encontrado"
```cmd
:: Usar ruta completa
F:\nodejs\node.exe reader.js
```

---

## 📝 Notas Importantes para Windows

1. **Siempre ejecuta CMD como Administrador** para acceso a puertos seriales
2. **Cierra todas las aplicaciones** que puedan usar el puerto COM (Arduino IDE, Putty, etc.)
3. **Los puertos COM** suelen ser COM3, COM4, COM5 en Windows
4. **Node.js en F:** funciona pero necesita PATH configurado

---

## ✅ Verificación Final en CMD

```cmd
F:
cd nodejs-project
dir
node --version
npm --version
node reader.js
```


---
