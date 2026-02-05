# Caso de negocio – Agenda Salón de Belleza (MVP)

## 1. Contexto del negocio

En el mercado peruano, la mayoría de salones de belleza pequeños y medianos gestionan sus citas mediante:

    - WhatsApp
    - llamadas telefónicas
    - agendas físicas 
    - Mensajes en redes sociales 

Este modelo genera fricción tanto para el negocio como para el cliente, especialmente cuando el volumen de citas aumenta o cuando existen varios servicios con duraciones distintas.
La reserva digital autogestionada ha demostrado reducir fricción, mejorar la experiencia del cliente y optimizar la ocupación del negocio. Sin embargo, muchas soluciones disponibles en el mercado suelen ser complejas, costosas o sobredimensionadas para salones pequeños.

## 2. Problema de negocio

#### Desde la perspectiva del salón las respuestas manuales generan:

    - Alta carga operativa para personal administrativo
    - Altos tiempos de respuestas  para cliente
    - Errores causando doble reserva 
    - Tiempos muertos durante el dia 


#### Desde la perspectiva del cliente

    - Necesidad de esperar respuesta para confirmar una cita.
    - Poca visibilidad de horarios disponibles y personal para atencion
    - Mala experiencia en reprogramaciones o cancelaciones.

Esto impacta directamente en ventas perdidas, satisfacción del cliente y eficiencia operativa.

## 3. Oportunidad

Existe una oportunidad clara de ofrecer una agenda digital web que permita:

    - Mostrar disponibilidad real por fecha y franja horaria.
    - Permitir que el cliente reserve sin intermediación.
    - Centralizar la agenda del salón en un solo sistema.


## 4. Propuesta de solución (visión de negocio)

Desarrollar una aplicacion web llamada Agenda Salón que permita a usuarios(rol cliente) ejecutar el proceso de agendar una cita en un salon (rol negocio)
La funcionalidad principal le permite al cliente seleccionar 1 de los servicios ofrecidos por el salon; luego elegir fecha, horario y personal (no obligatorio) 
para la atencion según disponibilidad proporcionada por el salon(negocio). La applicacion le permitira al cliente ver sus reservas activas con la posibilidad de editarlas y cancelarlas
Por el lado del salon, debe poder determinar su horario y dias de atencion, asi tambien tener visibilidad de la agenda con las citas activas.

### Roles y Permisos

####  Cliente

**Capacidades:**

- Registrarse y autenticarse
- Ver detalles del negocio (servicios, horarios, ubicación)
- Revisar disponibilidad en tiempo real
- Agendar citas
- Ver historial de citas (próximas y pasadas)
- Cancelar/editar citas

####  Negocio (Business Owner)

**Capacidades:**

- Gestionar servicios (nombre, duración, precio)
- Configurar horarios de atención por día de la semana
- Definir excepciones (feriados, vacaciones)
- Ver todas las citas (filtros por fecha, estado)

### Flujo de reserva para el cliente

1. El cliente se registra y define su login con email and password
2. El cliente ingresa al landing de la aplicacion web y visualiza el catálogo de servicios del salón (ej. corte de pelo, tinte, manicure). Cada servicio debe mostrar su costo y duracion referencial
3. El cliente selecciona un servicio de los visto en paso 2 y un espacio de tiempo mediante filtros predeterminados: 1 semana, 2 semanas , 1 mes. 
4. El sistema muestra la pantalla de agenda donde se visualiza un calendario con fechas disponibles
5. El cliente selecciona un personal para la atencion. Si lo deja en blanco se le asigna un personal disponible  
6. El sistema  muestra los horarios disponibles (slots) para el servicio seleccionado, agrupados por franja horaria (mañana / mediodía / tarde). Se mostraran semana a semana y se tendra una fecha para avanzar
a la siguiente semana en caso elija un filtro mayor a 1 semana. 
7. El cliente selecciona una fecha y hora.
8. El cliente debe ingresar sus datos de contacto telefono, email, nombre.
9. El sistema muestra un mensaje de confirmacion de la cita con la siguiente informacion: servicio seleccionado, duración, fecha y hora, profesional 
10. El cliente confirma la reserva.

### Flujo de cancelacion de citas  para el cliente

1. El cliente logueado aterriza en el landing, selecciona la seccion citas
2. El cliente busca entre sus citas activas y selecciona una cita 
3. El cliente visualiza los detalles de la cita y puede elergir entre editarla o cancelarla
4. Al elegir cancelar el cliente debera confirmar la cancelacion 
5. Al elegir editar el cliente volvera al flujo de eleccion del slot de tiempo para elegir un nuevo espacio. Elegido el espacio puede elegir el personal
disponible o dejarlo en blanco en cuyo caso se le asignara un personal disponible.

### Funcionalidades clave para el salón

1. Definir y gestionar el catálogo de servicios del salon definira duracion, precio, personal .
2. Configurar horarios de atención: los horarios podria ser variables de acuerdo a la temporada
3. Visualizar la agenda del salon se le aplicaran filtros por tiempo definido: hoy, manana, semana, 2 semanas. 
4. Bloquear horarios no disponibles y dias de no atencion.

Este enfoque asegura que la disponibilidad mostrada al cliente esté directamente condicionada por el servicio seleccionado, evitando reservas inviables por duración o solapamiento.
Por qué este flujo es correcto para el negocio
Reduce errores, ya que el sistema calcula disponibilidad según la duración del servicio.
Guía al cliente paso a paso durante el proceso de reserva.
Mejora la experiencia del cliente y la eficiencia operativa del salón.
Facilita la escalabilidad futura (más servicios, profesionales o reglas).


## 5. Segmento objetivo

Salones de belleza pequeños y medianos.
Clientes recurrentes que valoran rapidez y claridad al reservar.

## 6. Propuesta de valor

Para el salón
    Reducción de errores en la agenda.
    Menor tiempo invertido en coordinación manual.
    Mayor control y visibilidad de la operación diaria.
    Mayor ocupación efectiva (menos huecos por mala coordinación).
Para el cliente
    Reserva inmediata, sin llamadas ni esperas.
    Claridad de horarios disponibles.
    Experiencia consistente y confiable.
    Menos cancelaciones por confusión.

## 7. Éxito del MVP 
El MVP se considera exitoso si:
    Un cliente puede reservar en menos de 2 minutos.
    El sistema evita la doble reserva para un mismo horario.
    El admin puede ver la agenda del día y bloquear horarios.

## 8. Alcance del negocio (MVP)
Incluye:
    Gestión de servicios
    Gestión de disponibilidad
    Reserva de citas
    Agenda visual para el salón
    Confirmación básica
No incluye 
    Pagos en línea
    Integración con Yape/Plin
    Integración con calendarios externos
    Programas de fidelización
    Notificaciones avanzadas (WhatsApp/SMS)





