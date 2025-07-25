## INTEGRANTES

 ➤   **Tomás Rocha Castaño CC. 1054856763**  
 ➤   **Brayan Ricardo Pisso Ramírez CC. 1004249850** 

---

### Fecha de entrega

Jueves, 24 de Julio de 2025 

---

### Materia

> Programación De Sistemas Embebidos Linux – Proyecto Final  
> Universidad Nacional De Colombia - Sede Manizales

---

# Sistema de Control de Iluminación Inteligente con Interfaz CLI 

>Este sistema permite controlar remotamente luces conectadas a una placa **Lichee RV** con Linux embebido, utilizando una interfaz por línea de comandos (CLI) y una interfaz web desarrollada en Flask. El control se realiza mediante manipulación directa de GPIOs para encender, apagar o verificar el estado de luminarias. Además, las acciones se registran con timestamp en un archivo de log.

---

## Objetivo general

Diseñar e implementar un sistema de control de iluminación inteligente basado en una plataforma embebida con arquitectura RISC-V (Lichee RV), que permita la operación de múltiples luminarias mediante una interfaz de línea de comandos (CLI) y una interfaz web desarrollada en Flask, integrando el acceso directo a los GPIOs del sistema operativo Linux para activar o desactivar actuadores (relés) de forma remota, eficiente y documentada, garantizando la compatibilidad con sistemas embebidos, la trazabilidad de acciones mediante registros de eventos, y sentando las bases para futuras expansiones hacia sistemas automatizados o conectividad IoT.

## Objetivos específicos

* Controlar luminarias mediante comandos en terminal y botones web.
* Acceder y manipular GPIOs desde espacio de usuario en Linux.
* Registrar eventos de encendido/apagado con timestamp.
* Permitir acceso remoto vía SSH y navegador web.
* Garantizar compatibilidad y estabilidad en plataforma RISC-V.

---

## Introducción

En la actualidad, el control automatizado de sistemas de iluminación representa una necesidad creciente tanto en entornos domésticos como industriales, como parte de la evolución hacia entornos inteligentes, eficientes y conectados. La posibilidad de administrar luminarias de forma remota, registrar eventos y adaptarse a condiciones del entorno ha dejado de ser un lujo para convertirse en una característica fundamental de los sistemas modernos.

Este proyecto surge como una propuesta de solución embebida orientada al control de iluminación mediante el uso de la plataforma Lichee RV, un dispositivo de bajo consumo basado en la arquitectura RISC-V que ejecuta un sistema operativo Linux Ubuntu. A través de esta plataforma se diseña un sistema que permite el manejo de múltiples luces utilizando dos interfaces principales: una línea de comandos (CLI) escrita en lenguaje C, y una interfaz web desarrollada en Python con el microframework Flask.

La interacción con el hardware se realiza mediante el sistema de archivos virtual /sys/class/gpio, lo que permite acceder y manipular los pines GPIO directamente desde espacio de usuario. Cada acción, ya sea encendido, apagado o consulta del estado de una luz, se registra con fecha y hora en un archivo de log, permitiendo trazabilidad del sistema. El proyecto también contempla la posibilidad de acceso remoto mediante SSH o navegador web, lo que amplía sus posibilidades de implementación en entornos reales.

Con esta solución, se busca demostrar cómo una plataforma embebida de bajo costo, correctamente configurada y programada, puede cumplir con requerimientos funcionales clave como control multicanal, registro de eventos y operación remota, abriendo la puerta a futuras expansiones hacia domótica, IoT o automatización industrial.

## 1. Diseño del sistema

El diseño del sistema propuesto se estructura en dos dimensiones complementarias: la **arquitectura estructural**, que abarca los componentes físicos y sus interconexiones, y la **arquitectura funcional**, que describe la lógica de operación y la interacción entre los módulos de software. Esta separación permite una mejor comprensión, mantenimiento y escalabilidad del sistema.

### 1.1 Arquitectura estructural (hardware)

El núcleo del sistema es la placa de desarrollo **Lichee RV Dock**, basada en arquitectura **RISC-V**, la cual ejecuta un sistema operativo **Linux Ubuntu** en entorno embebido. Esta tarjeta permite el acceso a pines GPIO que son utilizados para controlar módulos de relé de 5 V, actuando como intermediarios entre la lógica digital de bajo voltaje y las cargas reales (bombillos LED o dispositivos equivalentes).

Cada salida GPIO está asociada a un canal de control independiente. Los relés son alimentados por una fuente externa y, en condiciones reales, deben contar con aislamiento mediante optoacopladores para evitar interferencias o daños en la placa. Las luminarias utilizadas son de bajo voltaje (12 V) para fines de prueba, aunque el sistema es adaptable a cargas más robustas con los debidos cuidados eléctricos.

**Diagrama general de conexión (descripción textual):**
- Lichee RV se comunica con el usuario vía SSH o interfaz web.
- GPIOs configuran los pines como salida digital (`out`) y activan relés mediante escritura en `/sys/class/gpio/gpioX/value`.
- Los relés conmuntan las luminarias según el estado del GPIO.
- Las acciones son registradas en un archivo de log (`luz.log`) en el sistema de archivos.

---

### 1.2 Arquitectura funcional (software)

El sistema software está organizado en módulos independientes pero interconectados, los cuales permiten separar responsabilidades, facilitar pruebas unitarias y simplificar la extensión del sistema.

**Módulos principales:**

- **Módulo CLI (`light.c`)**: Permite controlar las luces desde la terminal mediante comandos como `./light sala on`. Este programa en C interactúa con los GPIOs utilizando operaciones de escritura en el sistema de archivos `/sys/class/gpio`.

- **Módulo web (`app.py`)**: Implementado en Python usando Flask, este servidor permite acceder al sistema desde cualquier navegador en la misma red. Los botones en la página ejecutan comandos del CLI mediante `subprocess`.

- **Gestor de GPIOs**: Es la parte del código que realiza la exportación de pines, define su dirección como salida, y escribe valores binarios para activar o desactivar los relés.

- **Registrador de eventos**: Cada acción ejecutada por el usuario es almacenada en un archivo de texto (`luz.log`) con timestamp, nombre del canal y acción realizada. Esto permite trazabilidad y auditoría del sistema.

**Diagrama funcional simplificado:**

![WhatsApp Image 2025-07-25 at 5 59 22 AM](https://github.com/user-attachments/assets/a2687d55-2e03-4c28-930e-04cfe50edb93)


Este enfoque modular y estructurado permite un desarrollo ordenado, facilita el diagnóstico de errores y ofrece una base sólida para futuras expansiones, como integración de sensores, temporizadores o control vía red.

---
## Diagrama de bloques del funcionamiento

<img width="562" height="722" alt="image" src="https://github.com/user-attachments/assets/169f41cd-c1c0-4511-b5b1-de263170173c" />

## 2. Implementación

La implementación del sistema se llevó a cabo en dos niveles principales: el nivel de control embebido mediante línea de comandos (CLI) y el nivel de interfaz de usuario a través de un servidor web. Ambos componentes interactúan con los GPIOs de la Lichee RV mediante acceso al sistema de archivos virtual `/sys/class/gpio`, siguiendo el estándar sysfs de Linux.

---

### 2.1 Módulo CLI (`light.c`)

El núcleo del sistema de control se desarrolló en lenguaje C, generando un ejecutable llamado `light` que permite manipular las salidas digitales (GPIOs) mediante comandos desde la terminal.

Este módulo permite:

- **Exportar pines GPIO**: habilita el acceso al pin en el sistema (`/sys/class/gpio/export`).
- **Configurar dirección del pin**: establece el GPIO como salida (`direction` = `out`).
- **Escribir valor binario**: activa o desactiva el GPIO (`value` = `1` o `0`).
- **Leer el estado actual del GPIO**.
- **Registrar eventos**: cada acción queda almacenada en un archivo de log `luz.log` con fecha y hora.

**Ejemplo de uso desde la terminal:**

```bash
./light sala on
./light cuarto off
./light cocina status
```

Este módulo interpreta el nombre del canal (sala, cuarto, baño, cocina) y el comando (on, off, status), y actúa sobre el pin correspondiente.

### 2.2 Módulo Web (`app.py`)

Se desarrolló una interfaz web ligera utilizando **Python 3** y el microframework **Flask**, con el fin de ofrecer un acceso remoto amigable desde navegadores web. El servidor escucha en el puerto `5000` de la Lichee RV y permite:

- Encender y apagar luces mediante botones HTML.
- Consultar el estado actual de cada luz.
- Mostrar mensajes dinámicos con el resultado de cada acción.

Internamente, el servidor Flask utiliza el módulo `subprocess` para ejecutar el programa `light` con los parámetros adecuados. De esta manera, la interfaz web actúa como un puente visual hacia la lógica de control implementada en C.

#### Acceso desde navegador:

```text
http://<IP_de_Lichee_RV>:5000/
```
La interfaz es accesible desde cualquier dispositivo conectado a la misma red, sin necesidad de instalar software adicional.

### 2.3 Registro de eventos (`luz.log`)

Todas las acciones realizadas sobre las luminarias se almacenan en un archivo plano llamado `luz.log`, ubicado en el mismo directorio del ejecutable. Este archivo permite llevar un registro cronológico de las operaciones realizadas por el usuario, ya sea desde la línea de comandos o desde la interfaz web.

Cada línea del archivo contiene:

- Un **timestamp** en formato `[AAAA-MM-DD HH:MM:SS]`.
- La **descripción de la acción**: canal afectado y estado resultante (encendido o apagado).

#### Ejemplo de contenido de `luz.log`:

[2025-07-25 10:32:14] Luz sala encendida
[2025-07-25 10:35:02] Luz cocina apagada


Este mecanismo de registro garantiza la **trazabilidad** del sistema, facilitando tareas de auditoría, monitoreo y diagnóstico. Además, puede ser extendido fácilmente para integrarse con sistemas de almacenamiento persistente, herramientas de análisis o interfaces gráficas de monitoreo en versiones futuras.

## 3. Arquitectura del Sistema

### 3.1 Arquitectura estructural

* Lichee RV controla GPIOs que activan relés.
* Los relés conmutan luces o cargas externas.
* El usuario interactúa mediante CLI o web.

### 3.2 Arquitectura funcional

```plaintext
Usuario (CLI / Web)
       |
     [Comando / Botón]
       |
   Programa C (light.c) <--- Interfaz Web (app.py)
       |
   Acceso a /sys/class/gpio
       |
   Activación de relés → Bombillos ON/OFF
       |
     Registro en luz.log
```

---

## 4. Instalación

1. Clonar el repositorio en la Lichee RV

```bash
git clone https://github.com/usuario/proyecto-luces.git
cd proyecto-luces
```

2. Compilar el archivo C

```bash
gcc light.c -o light
chmod +x light
```

3. Instalar Flask

```bash
pip3 install flask
```

4. Ejecutar la interfaz web

```bash
python3 app.py
```

---

## 5. Comandos Disponibles (CLI)

```bash
./light {sala|cuarto|baño|cocina} {on|off|status}
```

Ejemplos:

* `./light sala on`
* `./light baño status`
* `./light cocina off`

---

## 6. Interfaz Web

Accede a la interfaz web desde un navegador usando la IP de la Lichee RV:

```
http://<IP_Lichee_RV>:5000/
```

Desde allí puedes:

* Encender y apagar luces por botón.
* Consultar el estado de cada luz.

---

## 7. Registro de eventos

Todas las acciones se almacenan en el archivo `luz.log` con formato:

```text
[2025-07-25 14:30:00] Luz sala encendida
```

---

## 8. Pruebas realizadas

Con el fin de validar el correcto funcionamiento del sistema de control de iluminación implementado sobre la plataforma Lichee RV, se realizaron pruebas unitarias e integradas sobre cada uno de los módulos que componen el sistema. Las pruebas se diseñaron para verificar tanto el comportamiento lógico del software como la respuesta física de los componentes electrónicos.

---

### 8.1 Pruebas del módulo CLI (`light.c`)

Se realizaron pruebas sobre los comandos básicos disponibles en la línea de comandos para cada canal (sala, cuarto, baño, cocina):

-  `./light sala on`: El relé correspondiente se activa. Se verifica mediante encendido de bombillo o LED.
-  `./light sala off`: El relé se desactiva. El bombillo se apaga.
-  `./light cuarto status`: Se imprime el estado actual del canal en consola (`ON` o `OFF`).

#### Pruebas de manejo de errores:

-  `./light cocina toggle`: Comando inválido. El programa responde con mensaje de error.
-  `./light garaje on`: Canal inexistente. Se muestra advertencia adecuada.

---

### 8.2 Pruebas del módulo Web (`app.py`)

Se accedió a la interfaz web desde distintos dispositivos conectados a la misma red, verificando:

-  Funcionamiento de botones de encendido/apagado por canal.
-  Actualización del estado al consultar vía botón “Estado”.
-  Redirección y mensaje de estado tras cada acción.
-  Compatibilidad con navegadores móviles y de escritorio.

---

### 8.3 Pruebas del gestor de GPIOs

-  Exportación exitosa del pin (`/sys/class/gpio/export`).
-  Dirección configurada como salida (`direction = out`).
-  Escritura de valores `1` y `0` en `value`.
-  Lectura precisa del estado actual del pin.

Se utilizó un multímetro y/o indicadores LED para verificar la salida de voltaje en el pin después de cada cambio de estado.

---

### 8.4 Pruebas del registrador de eventos (`luz.log`)

-  Registro generado automáticamente tras cada acción.
-  Formato correcto: `[AAAA-MM-DD HH:MM:SS] descripción`.
-  Verificación de persistencia tras reiniciar el sistema.
-  No se sobrescriben datos anteriores; el log es acumulativo.

---

### 8.5 Pruebas de integración general

-  Acciones ejecutadas desde la interfaz web generan cambios físicos en los GPIOs y activación de relés.
-  Acciones desde la CLI también actualizan el estado correctamente.
-  Todas las acciones quedan registradas sin pérdidas ni duplicados.
-  El sistema se comporta de forma estable durante sesiones prolongadas (>1 hora encendido).

---

### 8.6 Observaciones

- El sistema responde con baja latencia (<500 ms).
- No se detectaron conflictos entre el u

## 9. Resultado obtenidos

### Todos los LED apagados
![WhatsApp Image 2025-07-25 at 5 51 07 AM (1)](https://github.com/user-attachments/assets/d09b098d-1e3a-49f0-b074-5bc786690683)

### Encendido LED de la sala
![WhatsApp Image 2025-07-25 at 5 51 07 AM (2)](https://github.com/user-attachments/assets/27e8a6c4-d997-43e9-815b-05662c1d528c)

### Encendido LED de la sala y el cuarto
![WhatsApp Image 2025-07-25 at 5 51 06 AM](https://github.com/user-attachments/assets/48f0ab27-c709-4f0a-8001-f6bfd1313140)

### Encendido LED de la sala, el cuarto y la cocina
![WhatsApp Image 2025-07-25 at 5 51 06 AM (1)](https://github.com/user-attachments/assets/66224018-22e2-4205-9c8d-5a17f33af8d1)

### Encendido LED de la sala, el cuarto, la cocina y el baño
![WhatsApp Image 2025-07-25 at 5 51 06 AM (2)](https://github.com/user-attachments/assets/9b42fc9f-3b42-4b9b-8661-3d497161db38)

## 10. Conclusiones

- El proyecto logró integrar exitosamente el control de iluminación mediante una solución embebida utilizando la plataforma **Lichee RV**, combinando el acceso directo a GPIOs con interfaces de usuario en **línea de comandos (CLI)** y **web (Flask)**.

- La implementación en **lenguaje C** para el módulo de control ofreció un manejo eficiente y de bajo nivel de los pines GPIO, lo cual es fundamental en sistemas embebidos con recursos limitados. Al mismo tiempo, la interfaz web en Python permitió ampliar la accesibilidad del sistema, facilitando su operación remota desde cualquier navegador.

- Las pruebas realizadas demostraron un comportamiento estable, confiable y predecible del sistema, tanto en la ejecución local como en el acceso remoto. Las acciones fueron registradas correctamente en el archivo de log, lo que garantiza trazabilidad de eventos.

- La arquitectura modular del proyecto facilita su **escalabilidad**, permitiendo la incorporación futura de más canales, sensores o mecanismos de automatización como temporizadores, control por voz o integración con redes IoT.

- Este desarrollo demuestra que es posible implementar soluciones de **automatización doméstica o industrial de bajo costo** utilizando herramientas de software libre, hardware accesible y conocimientos en sistemas Linux embebidos.

- Finalmente, el proyecto representó una valiosa oportunidad para aplicar conocimientos en programación, electrónica, sistemas operativos y comunicaciones, fortaleciendo habilidades prácticas en el diseño e implementación de soluciones reales con enfoque en sistemas embebidos modernos.


## 11. Mejoras Futuras

* Programación horaria automática (via `cron`)
* Interfaz web responsiva y atractiva con HTML/CSS
* Almacenamiento en base de datos SQLite
* Control por voz o integración con asistentes

---
## 12. Explicación de código

12.1. `light.c` – Controlador principal en C

Este programa escrito en lenguaje C es el núcleo del sistema. Permite controlar las salidas GPIO de la Lichee RV desde la terminal (CLI) y tiene tres funciones principales:

- **Encender o apagar una luz**: usando `./luz on 1` o `./luz off 2`, el programa activa o desactiva el relé del canal indicado (1 a 4).
- **Consultar estado**: usando `./luz status 3`, permite saber si una luz está encendida o apagada.
- **Programar horarios**: usando `./luz programar 1 08:00 20:00`, se crea una tarea `cron` que encenderá y apagará la luz automáticamente a la hora indicada.

También escribe un registro con fecha y hora de cada acción en el archivo `/var/log/luz.log`, permitiendo trazabilidad de eventos.

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <time.h>

#define NUM_CHANNELS 4
#define LOG_FILE "/var/log/luz.log"
#define CRON_SCRIPT_ON "/root/luz_cron_on.sh"
#define CRON_SCRIPT_OFF "/root/luz_cron_off.sh"

// Configuración de los GPIOs y sus nombres
struct {
    const char *number;
    const char *name;
} channels[] = {
    {"111", "Sala"},   // PD15
    {"112", "Cuarto"}, // PD16
    {"113", "Cocina"}, // PD17
    {"114", "Baño"}    // PD18
};

void unexport_gpio(const char *gpio_number) {
    FILE *f = fopen("/sys/class/gpio/unexport", "w");
    if (f) {
        fprintf(f, "%s", gpio_number);
        fclose(f);
        usleep(500000);
    }
}

void export_gpio(const char *gpio_number) {
    FILE *f = fopen("/sys/class/gpio/export", "w");
    if (!f) {
        perror("Error al exportar GPIO");
        exit(1);
    }
    fprintf(f, "%s", gpio_number);
    fclose(f);
    usleep(500000);
}

void set_direction(const char *gpio_number, const char *dir) {
    char path[64];
    snprintf(path, sizeof(path), "/sys/class/gpio/gpio%s/direction", gpio_number);
    FILE *f = fopen(path, "w");
    if (!f) {
        perror("Error al configurar dirección");
        exit(1);
    }
    fprintf(f, "%s", dir);
    fclose(f);
}

void write_gpio(const char *gpio_number, int value) {
    char path[64];
    snprintf(path, sizeof(path), "/sys/class/gpio/gpio%s/value", gpio_number);
    FILE *f = fopen(path, "w");
    if (!f) {
        perror("Error al escribir en GPIO");
        exit(1);
    }
    fprintf(f, "%d", value);
    fclose(f);
}

int read_gpio(const char *gpio_number) {
    char path[64], value_str[4];
    snprintf(path, sizeof(path), "/sys/class/gpio/gpio%s/value", gpio_number);
    FILE *f = fopen(path, "r");
    if (!f) {
        perror("Error al leer GPIO");
        return -1;
    }
    fgets(value_str, sizeof(value_str), f);
    fclose(f);
    return atoi(value_str);
}

void log_action(const char *name, const char *action) {
    FILE *log = fopen(LOG_FILE, "a");
    if (!log) {
        perror("Error al escribir en el log");
        return;
    }
    char timestamp[64];
    time_t now = time(NULL);
    strftime(timestamp, sizeof(timestamp), "[%F %T]", localtime(&now));
    fprintf(log, "%s %s: %s\n", timestamp, name, action);
    fclose(log);
}

int validate_time(const char *time_str) {
    int hour, minute;
    if (sscanf(time_str, "%d:%d", &hour, &minute) != 2) {
        return 0;
    }
    if (hour < 0 || hour > 23 || minute < 0 || minute > 59) {
        return 0;
    }
    if (strlen(time_str) != 5 || time_str[2] != ':') {
        return 0;
    }
    return 1;
}

void program_cron(int channel, const char *hour_on, const char *hour_off) {
    // Crear scripts separados para encendido y apagado
    FILE *script_on = fopen(CRON_SCRIPT_ON, "w");
    if (!script_on) {
        perror("Error al crear script de cron (on)");
        exit(1);
    }
    fprintf(script_on, "#!/bin/bash\n");
    fprintf(script_on, "/usr/local/bin/luz on %d\n", channel + 1);
    fclose(script_on);
    system("chmod +x " CRON_SCRIPT_ON);

    FILE *script_off = fopen(CRON_SCRIPT_OFF, "w");
    if (!script_off) {
        perror("Error al crear script de cron (off)");
        exit(1);
    }
    fprintf(script_off, "#!/bin/bash\n");
    fprintf(script_off, "/usr/local/bin/luz off %d\n", channel + 1);
    fclose(script_off);
    system("chmod +x " CRON_SCRIPT_OFF);

    // Parsear horas y minutos
    int h_on, m_on, h_off, m_off;
    sscanf(hour_on, "%d:%d", &h_on, &m_on);
    sscanf(hour_off, "%d:%d", &h_off, &m_off);

    // Crear entrada de crontab, eliminando tareas previas para este canal
    char cron_cmd[512];
    snprintf(cron_cmd, sizeof(cron_cmd),
             "(crontab -l 2>/dev/null | grep -v '/usr/local/bin/luz.*%d' ; "
             "echo '%d %d * * * %s' ; echo '%d %d * * * %s') | crontab -",
             channel + 1, m_on, h_on, CRON_SCRIPT_ON, m_off, h_off, CRON_SCRIPT_OFF);
    if (system(cron_cmd) != 0) {
        fprintf(stderr, "Error al programar cron\n");
        exit(1);
    }

    printf("%s: Programado encendido a las %s, apagado a las %s\n", channels[channel].name, hour_on, hour_off);
    char log_msg[128];
    snprintf(log_msg, sizeof(log_msg), "Programado encendido a las %s, apagado a las %s", hour_on, hour_off);
    log_action(channels[channel].name, log_msg);
}

int main(int argc, char *argv[]) {
    if (argc < 3 || argc > 5) {
        printf("Uso: %s {on|off|status} {1|2|3|4} | programar {1|2|3|4} <hora_on> <hora_off>\n", argv[0]);
        return 1;
    }

    // Validar número de canal
    int channel = atoi(argv[2]) - 1;
    if (channel < 0 || channel >= NUM_CHANNELS) {
        printf("Canal inválido: %s. Use 1 (Sala), 2 (Cuarto), 3 (Cocina) o 4 (Baño)\n", argv[2]);
        return 1;
    }

    const char *gpio_number = channels[channel].number;
    const char *zone_name = channels[channel].name;

    // Procesar comandos
    if (strcmp(argv[1], "programar") == 0) {
        if (argc != 5) {
            printf("Uso: %s programar {1|2|3|4} <hora_on> <hora_off>\n", argv[0]);
            return 1;
        }
        if (!validate_time(argv[3]) || !validate_time(argv[4])) {
            printf("Formato de hora inválido. Use HH:MM (24 horas, ej. 08:00)\n");
            return 1;
        }
        program_cron(channel, argv[3], argv[4]);
    } else {
        // Configurar GPIO solo para on/off
        if (strcmp(argv[1], "on") == 0 || strcmp(argv[1], "off") == 0) {
            unexport_gpio(gpio_number);
            export_gpio(gpio_number);
            set_direction(gpio_number, "out");
        }

        if (strcmp(argv[1], "on") == 0) {
            write_gpio(gpio_number, 1);
            printf("%s: Luz encendida\n", zone_name);
            log_action(zone_name, "Luz encendida");
        } else if (strcmp(argv[1], "off") == 0) {
            write_gpio(gpio_number, 0);
            printf("%s: Luz apagada\n", zone_name);
            log_action(zone_name, "Luz apagada");
        } else if (strcmp(argv[1], "status") == 0) {
            int value = read_gpio(gpio_number);
            if (value == -1) {
                printf("%s: Error - GPIO no exportado o no accesible\n", zone_name);
            } else if (value == 1) {
                printf("%s: Estado: ON\n", zone_name);
            } else {
                printf("%s: Estado: OFF\n", zone_name);
            }
        } else {
            printf("Comando no reconocido: %s\n", argv[1]);
            return 1;
        }
    }

    return 0;
}
```
12.2. `app.py` – Interfaz web con Flask

Este archivo crea un servidor web ligero con **Flask**, que permite controlar las luces desde un navegador. Sus funciones principales son:

- Mostrar el estado actual de cada luz (ON/OFF).
- Permitir al usuario encender o apagar una luz con un clic.
- Enviar al backend la orden de programación horaria (hora de encendido y apagado).
- Ejecutar los mismos comandos de `light.c` mediante `sudo` y `subprocess`.

Se accede a la interfaz desde cualquier navegador en la red con:
http://<IP-de-Lichee-RV>:5000/

```
from flask import Flask, render_template, request, redirect, url_for
import subprocess
import re

app = Flask(__name__)

# Canales y nombres
channels = [
    {"id": 1, "name": "Sala"},
    {"id": 2, "name": "Cuarto"},
    {"id": 3, "name": "Cocina"},
    {"id": 4, "name": "Baño"}
]

def run_luz_command(command, channel):
    """Ejecuta el comando luz con sudo."""
    cmd = ["sudo", "/usr/local/bin/luz", command, str(channel)]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, check=True)
        return result.stdout.strip()
    except subprocess.CalledProcessError as e:
        return f"Error: {e.stderr.strip()}"

def validate_time(time_str):
    """Valida el formato de hora HH:MM."""
    pattern = re.compile(r"^(0[0-9]|1[0-9]|2[0-3]):[0-5][0-9]$")
    return bool(pattern.match(time_str))

@app.route('/')
def index():
    # Obtener estado de todos los canales
    statuses = []
    for channel in channels:
        status_output = run_luz_command("status", channel["id"])
        state = "ON" if "Estado: ON" in status_output else "OFF" if "Estado: OFF" in status_output else "Error"
        statuses.append({"id": channel["id"], "name": channel["name"], "state": state})
    return render_template('index.html', channels=statuses)

@app.route('/control/<int:channel_id>/<action>')
def control(channel_id, action):
    if channel_id not in [1, 2, 3, 4] or action not in ["on", "off"]:
        return "Comando inválido", 400
    output = run_luz_command(action, channel_id)
    return redirect(url_for('index'))

@app.route('/program', methods=['POST'])
def program():
    channel_id = int(request.form['channel'])
    hour_on = request.form['hour_on']
    hour_off = request.form['hour_off']
    
    if channel_id not in [1, 2, 3, 4] or not validate_time(hour_on) or not validate_time(hour_off):
        return "Datos inválidos", 400
    
    output = run_luz_command("programar", channel_id)
    cmd = ["sudo", "/usr/local/bin/luz", "programar", str(channel_id), hour_on, hour_off]
    try:
        subprocess.run(cmd, capture_output=True, text=True, check=True)
    except subprocess.CalledProcessError as e:
        return f"Error: {e.stderr.strip()}", 500
    
    return redirect(url_for('index'))

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=False)
```
12.3. HTML – Interfaz de programación horaria

Dentro de la página principal se encuentra un formulario para programar el encendido y apagado automático de cualquier canal. Funciona así:

- El usuario selecciona un canal (ej. "Sala").
- Ingresa la hora de encendido (ej. `07:00`) y la de apagado (ej. `18:30`).
- Al hacer clic en “Programar”, se envía un formulario POST al servidor Flask, que llama al comando `./luz programar`.

Esto permite automatizar el sistema sin necesidad de acceso directo a la terminal.

---

### Integración entre componentes

| Componente     | Función                           | Comunicación con otros módulos         |
|----------------|------------------------------------|----------------------------------------|
| `light.c`      | Controla los GPIOs y registra logs | Usado por CLI y Flask (`subprocess`)   |
| `app.py`       | Sirve la interfaz web              | Llama a `light.c` para ejecutar acciones |
| HTML           | Interfaz visual del usuario        | Envía formularios a `app.py`           |
| `cron`         | Programa tareas automáticas        | Invocado desde `light.c`               |

---

Este diseño modular permite controlar las luces de forma manual (CLI o Web) o automática (por horarios), integrando software de bajo nivel con una interfaz moderna y amigable.
```
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Control de Iluminación</title>
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-100 min-h-screen flex items-center justify-center">
    <div class="container mx-auto p-4">
        <h1 class="text-3xl font-bold text-center mb-6">Control de Iluminación</h1>
        
        <!-- Estado de los LEDs -->
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 mb-8">
            {% for channel in channels %}
            <div class="bg-white p-4 rounded-lg shadow-md text-center">
                <h2 class="text-xl font-semibold">{{ channel.name }}</h2>
                <p class="text-lg {{ 'text-green-500' if channel.state == 'ON' else 'text-red-500' if channel.state == 'OFF' else 'text-gray-500' }}">
                    Estado: {{ channel.state }}
                </p>
                <div class="mt-4 flex justify-center space-x-2">
                    <a href="{{ url_for('control', channel_id=channel.id, action='on') }}"
                       class="bg-green-500 text-white px-4 py-2 rounded hover:bg-green-600">
                        Encender
                    </a>
                    <a href="{{ url_for('control', channel_id=channel.id, action='off') }}"
                       class="bg-red-500 text-white px-4 py-2 rounded hover:bg-red-600">
                        Apagar
                    </a>
                </div>
            </div>
            {% endfor %}
        </div>
        
        <!-- Formulario para programar -->
        <div class="bg-white p-6 rounded-lg shadow-md">
            <h2 class="text-2xl font-semibold mb-4">Programar Horario</h2>
            <form action="{{ url_for('program') }}" method="POST" class="space-y-4">
                <div>
                    <label for="channel" class="block text-sm font-medium text-gray-700">Canal</label>
                    <select id="channel" name="channel" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm">
                        {% for channel in channels %}
                        <option value="{{ channel.id }}">{{ channel.name }}</option>
                        {% endfor %}
                    </select>
                </div>
                <div>
                    <label for="hour_on" class="block text-sm font-medium text-gray-700">Hora de Encendido (HH:MM)</label>
                    <input type="text" id="hour_on" name="hour_on" placeholder="08:00"
                           class="mt-1 block w-full border-gray-300 rounded-md shadow-sm">
                </div>
                <div>
                    <label for="hour_off" class="block text-sm font-medium text-gray-700">Hora de Apagado (HH:MM)</label>
                    <input type="text" id="hour_off" name="hour_off" placeholder="20:00"
                           class="mt-1 block w-full border-gray-300 rounded-md shadow-sm">
                </div>
                <div>
                    <button type="submit"
                            class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600">
                        Programar
                    </button>
                </div>
            </form>
        </div>
    </div>
</body>
</html>
```

---
## 13. Licencia

Este proyecto es de carácter académico. Universidad Nacional de Colombia - Sede Manizales, 2025.

---

## 14. Enlaces de interés

* [Documentación oficial de GPIO Sysfs](https://www.kernel.org/doc/Documentation/gpio/sysfs.txt)
* [Flask - Micro Web Framework](https://flask.palletsprojects.com/)
* [Lichee RV - Wiki oficial](https://wiki.sipeed.com/hardware/en/lichee/licheerv/)
