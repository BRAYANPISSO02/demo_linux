# Sistema de Control de Iluminación Inteligente con Interfaz CLI 

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

## Resumen del Proyecto

>Este sistema permite controlar remotamente luces conectadas a una placa **Lichee RV** con Linux embebido, utilizando una interfaz por línea de comandos (CLI) y una interfaz web desarrollada en Flask. El control se realiza mediante manipulación directa de GPIOs para encender, apagar o verificar el estado de luminarias. Además, las acciones se registran con timestamp en un archivo de log.

---

## Objetivos

### Objetivo general

Diseñar e implementar un sistema embebido para el control de iluminación mediante interfaz CLI y web, usando la placa Lichee RV y GPIOs.

### Objetivos específicos

* Controlar luminarias mediante comandos en terminal y botones web.
* Acceder y manipular GPIOs desde espacio de usuario en Linux.
* Registrar eventos de encendido/apagado con timestamp.
* Permitir acceso remoto vía SSH y navegador web.
* Garantizar compatibilidad y estabilidad en plataforma RISC-V.

---

## Recursos Utilizados

### Hardware

* Lichee RV Dock (SoC RISC-V con Linux embebido)
* Módulo de relés de 4 canales
* Bombillos LED o indicadores de estado
* Fuente de alimentación externa (5V o 12V)

### Software

* Sistema operativo Linux (Debian RISC-V o Tina Linux)
* Lenguaje C (`light.c`) para CLI
* Python + Flask (`app.py`) para interfaz web
* Acceso a `/sys/class/gpio/`

---

## Arquitectura del Sistema

### Arquitectura estructural

* Lichee RV controla GPIOs que activan relés.
* Los relés conmutan luces o cargas externas.
* El usuario interactúa mediante CLI o web.

### Arquitectura funcional

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

## Instalación

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

## Comandos Disponibles (CLI)

```bash
./light {sala|cuarto|baño|cocina} {on|off|status}
```

Ejemplos:

* `./light sala on`
* `./light baño status`
* `./light cocina off`

---

## Interfaz Web

Accede a la interfaz web desde un navegador usando la IP de la Lichee RV:

```
http://<IP_Lichee_RV>:5000/
```

Desde allí puedes:

* Encender y apagar luces por botón.
* Consultar el estado de cada luz.

---

## Registro de eventos

Todas las acciones se almacenan en el archivo `luz.log` con formato:

```text
[2025-07-25 14:30:00] Luz sala encendida
```

---

## Pruebas Realizadas

### 1. Control GPIO

* Verificación de exportación GPIO y escritura en `value`.
* Confirmación de estado mediante multímetro/LED.

### 2. CLI

* `./light sala on` activa el relé.
* `./light sala status` devuelve "ON".
* Comando inválido genera error.

### 3. Web

* Botones ejecutan comandos equivalentes.
* Visualización de estado con mensajes HTML.

### 4. Log

* Se genera entrada por cada acción.
* Persistencia tras reinicio.

---

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
