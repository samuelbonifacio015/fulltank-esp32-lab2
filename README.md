# FullTank — Lab 2: ESP32 → ThingSpeak

Curso **1ASI0572 Desarrollo de Soluciones IoT (UPC)**. Laboratorio 2: conectividad del dispositivo IoT con la nube.

**Proyecto:** Tanques de combustible (sensor HC-SR04)
**Ruta:** 1, simulación controlada (obligatoria). Las Rutas 2 y 3 son opcionales y requieren el sensor.
**Grupo:** G## (pendiente)
**Canal ThingSpeak:** `TANQUE-G##`

> **Logro:** el ESP32 se conecta a una red Wi-Fi y envía datos del sensor a ThingSpeak. Luego se analizan los datos recibidos y se documenta el proceso con evidencias en GitHub y en el aula virtual.

---

## Estructura del repositorio

```
fulltank-esp32-lab2/
├── sketch/
│   ├── sketch.ino            # Parte C (base) + Parte D2 Ruta 1 al final
│   ├── secrets.example.h     # valores de ejemplo (sí se sube)
│   └── secrets.h             # credenciales reales (NO se sube)
├── .gitignore                # incluye secrets.h
└── evidencias/sesionXX/
    ├── README.md             # informe (sección F4)
    ├── G##_E1_entorno.png ... G##_E9_repo.png
    ├── G##_E7_datos.csv
    └── G##_E8_video.mov      # < 100 MB, solo GitHub
```

---

## Checklist

### Parte A: canal en ThingSpeak (0–10 min)

- [ ] Crear una cuenta en [thingspeak.mathworks.com](https://thingspeak.mathworks.com) (cuenta MathWorks).
- [ ] Ir a Channels → My Channels → New Channel y usar el nombre `TANQUE-G##`.
- [ ] Activar y nombrar los campos:

| Campo   | Tanques       |
|---------|---------------|
| Field 1 | Distancia (cm)|
| Field 2 | Nivel (%)     |
| Field 3 | RSSI (dBm)    |
| Field 4 | Alarma (0/1)  |

- [ ] Pulsar Save Channel.
- [ ] En la pestaña API Keys, copiar la **Write API Key** y el **Channel ID**.
- [ ] En la pestaña Sharing, elegir *Share channel view with everyone*.

### Parte B: Arduino IDE y credenciales (10–25 min)

- [ ] Instalar la librería **ThingSpeak** (de MathWorks) desde el gestor de librerías.
- [ ] En Tools → Board, elegir **DOIT ESP32 DEVKIT V1**.
- [ ] En Tools → Port, elegir el puerto de la placa. En Mac es `/dev/cu.usbserial-XXXX`. En Linux es `/dev/ttyUSB0`, y el usuario debe estar en el grupo `uucp` (Arch/CachyOS) o `dialout` (Debian/Ubuntu).
- [ ] Copiar `sketch/secrets.example.h` a `sketch/secrets.h` y completar los valores reales:

```cpp
#pragma once
#define WIFI_SSID     "nombre_hotspot"
#define WIFI_PASS     "clave_hotspot"
#define TS_CHANNEL    1234567UL          // Channel ID
#define TS_WRITE_KEY  "XXXXXXXXXXXXXXXX" // Write API Key
```

- [ ] Confirmar que `secrets.h` está en `.gitignore` **antes** del primer commit.

> La red debe ser un hotspot de **2.4 GHz** sin portal cautivo (sin página de inicio de sesión). El cable USB no le da internet al ESP32.

### Partes C y D: sketch base + Ruta 1 Tanques (25–55 min)

- [ ] `sketch/sketch.ino` contiene el sketch base de la Parte C. Al final se pegan las tres funciones de la Ruta 1 de Tanques: `configurarSensor()`, `simular()` y `leerDatos()`.
- [ ] Mantener `SEND_MS = 20000`. La cuenta gratuita exige al menos 15 s entre envíos.

Cómo se controla la simulación:

- `N=15` en el monitor serie fija el nivel en %.
- `C=5` en el monitor serie fija el consumo en % por envío (por defecto, 2).
- El botón **BOOT** rellena el tanque al 100 %.
- La alarma (Field 4 y LED en GPIO 2) se activa cuando el nivel baja de 20 %.

### Parte E: subir y verificar (55–80 min)

- [ ] Subir el programa. Si aparece *"Failed to connect... Wrong boot mode"*, mantener presionado BOOT mientras dice *Connecting...*
- [ ] Abrir el monitor serie a **115200** baudios con fin de línea **New Line**.
- [ ] Interpretar el código que devuelve ThingSpeak:

| Código | Significado                    | Acción                      |
|--------|--------------------------------|-----------------------------|
| 200    | Envío correcto                 | Continuar                   |
| 404    | Write API Key incorrecta       | Revisar `secrets.h`         |
| -401   | Se envió antes de los 15 s     | Mantener `SEND_MS` en 20000 |
| -301   | Sin conexión a ThingSpeak      | Revisar el hotspot y el Wi-Fi |

- [ ] En ThingSpeak, abrir **Private View**. Los gráficos se actualizan cada 20 s.
- [ ] **Escenario de prueba:** esperar a que el nivel baje de 20 %, comprobar la alarma, presionar BOOT y ver el rellenado.

> Con el consumo por defecto (2 % por envío), la alarma tarda unos 13 min en llegar. Para acelerar, escribir `N=30` o `C=5` en el monitor serie.

### Parte F: evidencias (80–105 min)

Guardar todo en `evidencias/sesionXX/` con el prefijo `G##_`.

| #  | Evidencia             | Qué debe mostrar                                                    | Dónde se captura                      | Destino              |
|----|-----------------------|---------------------------------------------------------------------|---------------------------------------|----------------------|
| E1 | Entorno               | Placa y puerto seleccionados                                        | Arduino IDE → menú Tools              | Aula virtual + GitHub |
| E2 | Conexión Wi-Fi        | IP, RSSI y MAC                                                      | Monitor serie                         | Aula virtual + GitHub |
| E3 | Canal en la nube      | Channel ID y los 4 campos con nombre                                | ThingSpeak → Channel Settings         | Aula virtual + GitHub |
| E4 | Prototipo             | Foto del ESP32 y los comandos `[SIM]`                               | Celular + monitor serie               | Aula virtual + GitHub |
| E5 | Envío correcto        | Al menos 5 envíos con **código 200**                                | Monitor serie                         | Aula virtual + GitHub |
| E6 | Datos en la nube      | Gráficos con **al menos 15 puntos** (≥ 5 min)                       | ThingSpeak → Private View             | Aula virtual + GitHub |
| E7 | Análisis cuantitativo | Mínimo, máximo, promedio y % de envíos exitosos                     | Data Import/Export → Export recent data (CSV) | Aula virtual + GitHub |
| E8 | Video                 | 30–60 s: un comando o BOOT cambia el dato y el cambio se ve en ThingSpeak | Grabación de pantalla          | **Solo GitHub**      |
| E9 | Repositorio           | Commit con `.gitignore` y **sin** `secrets.h`                       | GitHub, en el navegador               | Aula virtual + GitHub |

- **E7:** % de envíos exitosos = puntos recibidos ÷ puntos esperados. Los puntos esperados son la duración en segundos ÷ 20.
- **E8:** el video debe pesar menos de 100 MB, que es el límite de GitHub por archivo.

**Contenido de `evidencias/sesionXX/README.md` (F4):**

- [ ] Integrantes del grupo
- [ ] Proyecto (Tanques) y ruta lograda (1, 2 o 3)
- [ ] Declaración de datos simulados o reales, con el escenario que se probó
- [ ] Channel ID público
- [ ] Tabla de E7 con su interpretación en 2 o 3 líneas
- [ ] Enlace al video E8 dentro del repositorio

### Entrega (105–120 min)

**1. GitHub** (todas las evidencias, incluido el video):

```bash
git add .
git commit -m "Sesión XX: conectividad ESP32 con ThingSpeak - evidencias"
git push
```

- [ ] Revisar en el navegador que estén las capturas y el video, y que `secrets.h` **no** esté subido.

**2. Aula virtual UPC** (capturas, sin el video):

- [ ] Un integrante sube `G##_SesionXX_Evidencias.zip` con E1–E7, E9, `G##_E7_datos.csv` y `README.md`.
- [ ] En el comentario de la entrega, escribir el link del repositorio y el Channel ID.

---

## Plan B: Wokwi

Usarlo solo si el ESP32 falla.

1. En [wokwi.com](https://wokwi.com), crear un proyecto con ESP32 y HC-SR04.
2. En `secrets.h`, usar `WIFI_SSID "Wokwi-GUEST"` y `WIFI_PASS ""`.
3. Durante la simulación, hacer clic en el sensor para mover el deslizador de distancia.
4. El código de las Rutas 2 y 3 funciona sin cambios.
5. Para E4, usar una captura del circuito en Wokwi y declararlo en el README.

---

## Seguridad

- **Nunca** subir `secrets.h` ni poner keys reales en un archivo versionado.
- Si la Write API Key se filtra, regenerarla en ThingSpeak (API Keys → Generate New Write API Key).
- El HC-SR04 no tiene certificación ATEX para atmósferas inflamables. En la maqueta se usa agua.
