# AI Real Estate Assistant

Agente inmobiliario conversacional desarrollado con **n8n + OpenAI**, diseñado para automatizar la atención de clientes, resolver consultas sobre propiedades y gestionar citas comerciales.

El sistema puede conversar con potenciales clientes, proporcionar información sobre el desarrollo inmobiliario, gestionar visitas en oficina o en el desarrollo, consultar disponibilidad en Google Calendar y registrar información de los contactos.

## Características principales

* Asistente inmobiliario con IA
* Atención conversacional automatizada
* Respuestas sobre propiedades, precios, formas de pago y horarios
* Agendamiento y cancelación de citas
* Validación de disponibilidad mediante Google Calendar
* Envío de emails de confirmación de citas
* Registro y actualización de clientes en Google Sheets
* Memoria conversacional mediante Redis
* Integración con WhatsApp/Chatwoot
* Arquitectura modular mediante subworkflows de n8n

---

## Arquitectura

El proyecto está dividido en tres workflows principales:

```text
                    +---------------------+
                    |       Cliente       |
                    | WhatsApp / Chatwoot |
                    +----------+----------+
                               |
                               v
                 +-------------------------+
                 |   AI Agent Principal    |
                 |                         |
                 |  Comprende intención    |
                 |  Mantiene contexto      |
                 |  Decide qué herramienta |
                 |  utilizar               |
                 +-----------+-------------+
                             |
              +--------------+--------------+
              |                             |
              v                             v
 +-------------------------+   +-------------------------+
 |  Asesor Informativo     |   |    Gestión de Citas     |
 |                         |   |                         |
 | - Propiedades           |   | - Agendar               |
 | - Precios               |   | - Cancelar              |
 | - MSI                   |   | - Disponibilidad        |
 | - Crédito               |   | - Google Calendar       |
 | - Preguntas frecuentes  |   |                         |
 +-------------------------+   +------------+------------+
                                            |
                                            v
                                  +---------------------+
                                  | Email de Confirmación|
                                  |                     |
                                  | - Fecha/hora        |
                                  | - Ubicación         |
                                  | - Datos del cliente |
                                  +---------------------+

                    +---------------------+
                    |    Google Sheets    |
                    |                     |
                    | - Contactos         |
                    | - Teléfono          |
                    | - Email             |
                    | - Última consulta   |
                    | - Datos de cita     |
                    +---------------------+
```

---

# Workflows

## 1. `Flujo Prueba Aru`

Es el **workflow principal** y funciona como orquestador del agente.

Recibe los mensajes entrantes mediante un webhook y procesa la conversación antes de decidir qué acción ejecutar.

Entre sus responsabilidades se encuentran:

* Procesar mensajes entrantes.
* Identificar al contacto.
* Registrar nuevos contactos.
* Mantener contexto de conversación.
* Determinar la intención del usuario.
* Delegar consultas inmobiliarias al asesor informativo.
* Delegar operaciones de calendario al sistema de citas.
* Gestionar conversaciones relacionadas con agendamientos y cancelaciones.

El agente utiliza **OpenAI** como modelo de lenguaje y **Redis Chat Memory** para mantener memoria asociada a la sesión del usuario.

También existe una herramienta específica para consultas inmobiliarias, evitando que el agente principal responda información comercial por cuenta propia.

### Gestión de citas

La herramienta de citas acepta dos acciones:

```text
agendar
cancelar
```

Para agendar una cita se utilizan datos como:

```text
accion
fecha_hora
detalles_cliente
tipo_cita
email
telefono
```

El workflow distingue entre citas de **Oficina** y **Desarrollo**.

Antes de crear una cita, el sistema consulta la disponibilidad del calendario y valida las condiciones necesarias para el horario solicitado.

---

## 2. `Sub_asesor_informativo`

Este workflow funciona como el **asesor inmobiliario especializado**.

Recibe la información enviada por el agente principal y genera una respuesta utilizando un prompt especializado para el proyecto inmobiliario.

El agente está configurado para proporcionar información sobre:

* Desarrollo inmobiliario.
* Ubicación.
* Lotes.
* Formas de pago.
* MSI.
* Crédito propio.
* Información ejidal.
* Preguntas frecuentes.
* Proceso de compra.
* Objeciones comerciales.
* Horarios de atención.

Utiliza un modelo de **OpenAI GPT-4o-mini** para generar las respuestas.

Una de las características importantes es que el agente sigue reglas específicas para mantener respuestas consistentes, incluyendo límites de longitud, cantidad de preguntas y uso de emojis.

---

## 3. `Sub_enviar_email`

Este workflow se encarga de generar y enviar el **correo de confirmación de una cita**.

Recibe:

```text
email
nombre
fecha_hora
tipo_cita
Telefono
```

Posteriormente:

1. Procesa los datos recibidos.
2. Formatea la fecha y hora.
3. Determina la ubicación de la cita.
4. Genera el contenido del email.
5. Actualiza la información del cliente en Google Sheets.
6. Envía el correo mediante Gmail.

El email incluye información como:

```text
Fecha y hora
Lugar de la cita
Ubicación
Nombre del cliente
```

La ubicación se determina automáticamente según si la cita corresponde a la oficina o al desarrollo.

El workflow también actualiza los datos del cliente en Google Sheets utilizando el teléfono como identificador.

---

# Flujo de una conversación

Un ejemplo de interacción sería:

```text
Cliente
   |
   | "¿Cuánto cuesta un lote?"
   v
AI Agent
   |
   | Detecta consulta inmobiliaria
   v
Asesor Informativo
   |
   | Genera respuesta
   v
Cliente
   |
   | "Me interesa, quiero visitar"
   v
AI Agent
   |
   | Solicita datos necesarios
   v
Gestión de Citas
   |
   | Verifica disponibilidad
   v
Google Calendar
   |
   | Cita disponible
   v
Crear cita
   |
   v
Solicitar / procesar email
   |
   v
Enviar email
   |
   v
Cliente recibe confirmación
```

---

# Tecnologías utilizadas

| Tecnología          | Uso                                                   |
| ------------------- | ----------------------------------------------------- |
| **n8n**             | Automatización y orquestación de workflows            |
| **OpenAI**          | Modelos de lenguaje para el agente                    |
| **Redis**           | Memoria conversacional                                |
| **Google Calendar** | Gestión y disponibilidad de citas                     |
| **Google Sheets**   | Registro de contactos y datos comerciales             |
| **Gmail**           | Envío de confirmaciones                               |
| **WhatsApp**        | Canal de comunicación                                 |
| **Chatwoot**        | Gestión de conversaciones                             |
| **JavaScript**      | Transformación y procesamiento de datos dentro de n8n |

---

# Estructura del proyecto

```text
.
├── Flujo Prueba Aru.json
├── Sub_asesor_informativo.json
├── Sub_enviar_email.json
├── SECURITY.md
└── README.md
```

Los workflows pueden importarse directamente en una instancia de **n8n**, configurando posteriormente las credenciales correspondientes.

---

# Instalación

## 1. Instalar n8n

Puedes utilizar n8n mediante una instalación local, Docker o una instancia alojada.

## 2. Importar los workflows

Desde n8n:

```text
Workflows
   -> Import from File
   -> Seleccionar archivo JSON
```

Importa los tres workflows:

```text
Flujo Prueba Aru.json
Sub_asesor_informativo.json
Sub_enviar_email.json
```

## 3. Configurar credenciales

El proyecto requiere configurar las credenciales de los servicios utilizados:

* OpenAI
* Google Calendar
* Google Sheets
* Gmail
* Redis
* WhatsApp / Meta
* Chatwoot

Las credenciales **no deben almacenarse directamente en el repositorio**.

---

# Configuración

El agente principal utiliza herramientas especializadas para decidir qué workflow ejecutar.

### Consultas inmobiliarias

Preguntas relacionadas con:

```text
Propiedades
Precios
MSI
Crédito
Pagos
Horarios
Información del desarrollo
```

son delegadas al workflow `Sub_asesor_informativo`.

### Citas

Las operaciones relacionadas con Google Calendar se delegan al sistema de gestión de citas.

```text
agendar
cancelar
```

La herramienta recibe la fecha/hora, información del cliente y tipo de cita.

### Cancelaciones

Para cancelar una cita, el sistema primero identifica el evento correspondiente y posteriormente utiliza su `event_id` para eliminarlo del calendario.

---

# Seguridad

Las credenciales deberían gestionarse mediante:

* n8n Credentials
* Variables de entorno
* Secret Managers
* GitHub Secrets cuando corresponda

---

# Objetivo del proyecto

El objetivo es demostrar cómo construir un **agente inmobiliario autónomo** capaz de pasar de una conversación comercial a una acción real.

En lugar de utilizar un único agente para todas las tareas, el sistema separa responsabilidades:

```text
Conversación
     |
     v
Comprensión de intención
     |
     +-------------------+
     |                   |
     v                   v
Información          Gestión de citas
inmobiliaria
     |                   |
     +---------+---------+
               |
               v
       Servicios externos
               |
               v
    Calendar / Gmail / Sheets
```

Esta arquitectura permite mantener cada componente especializado y facilita su mantenimiento y evolución.

---

# Posibles mejoras

Algunas mejoras futuras podrían incluir:

* [ ] Migrar credenciales y configuraciones sensibles a variables de entorno.
* [ ] Implementar seguimiento automático de leads.
* [ ] Añadir scoring automático de prospectos.
* [ ] Incorporar disponibilidad de múltiples asesores.
* [ ] Añadir recordatorios automáticos antes de las citas.
* [ ] Registrar métricas de conversaciones y conversiones.
* [ ] Añadir dashboards comerciales.
* [ ] Implementar recuperación automática ante errores.
* [ ] Separar configuración del proyecto y lógica del agente.
* [ ] Añadir pruebas automatizadas para los principales escenarios conversacionales.

---

# Estado del proyecto

**Prototype / Proof of Concept**

El proyecto demuestra una arquitectura funcional de automatización inmobiliaria basada en agentes de IA y workflows especializados.

---

# Autor

**Aruama Devaca**

Proyecto desarrollado como demostración de integración entre **IA conversacional, automatización de procesos y herramientas empresariales**.
