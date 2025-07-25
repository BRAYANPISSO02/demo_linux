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

## Resultado obtenidos

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


## Mejoras Futuras

* Programación horaria automática (via `cron`)
* Interfaz web responsiva y atractiva con HTML/CSS
* Almacenamiento en base de datos SQLite
* Control por voz o integración con asistentes

---

## Licencia

Este proyecto es de carácter académico. Universidad Nacional de Colombia - Sede Manizales, 2025.

---

## Enlaces de interés

* [Documentación oficial de GPIO Sysfs](https://www.kernel.org/doc/Documentation/gpio/sysfs.txt)
* [Flask - Micro Web Framework](https://flask.palletsprojects.com/)
* [Lichee RV - Wiki oficial](https://wiki.sipeed.com/hardware/en/lichee/licheerv/)
