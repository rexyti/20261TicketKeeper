# Feature Specification: [Procesar intento de ingreso]

**Created**: 21-02-2026 

## User Scenarios & Testing *(mandatory)*

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE - meaning if you implement just ONE of them,
  you should still have a viable MVP (Minimum Viable Product) that delivers value.
  
  Assign priorities (P1, P2, P3, etc.) to each story, where P1 is the most critical.
  Think of each story as a standalone slice of functionality that can be:
  - Developed independently
  - Tested independently
  - Deployed independently
  - Demonstrated to users independently
-->

### User Story 1 - Validar y procesar intento de ingreso exitoso (Priority: P1)

Yo como encargado de control de acceso escaneo un ticket y el sistema valida su estado, zona y sesión. Si todo es correcto, el sistema registra el ingreso (check-in) y permite el acceso.

**Why this priority**:

Es la funcionalidad núcleo del sistema de control de acceso. Sin esto, el sistema no cumple su propósito principal.

**Independent Test**: 

Puede probarse escaneando un ticket válido en una puerta autorizada. Si el sistema registra el check-in y devuelve estado "Ingreso autorizado", el MVP es funcional.

**Acceptance Scenarios**:

1. **Scenario**: Ingreso válido autorizado
   - **Given** un ticket existente con estado válido para ingreso
   - **When** el encargado escanea el ticket
   - **Then** el sistema registra el check-in

---

### User Story 2 - Rechazar intento por ticket duplicado (P)riority: P1

Yo como encargado escaneo un ticket que ya fue procesado previamente en otro lector.

**Why this priority**:

Evita fraude y reuso de credenciales. Es crítico para la seguridad del evento.

**Independent Test**:

Escanear dos veces el mismo ticket en lectores distintos. El segundo intento debe generar error “Ticket Duplicado”.

**Acceptance Scenarios**:

1. **Scenario**: Ticket ya procesado
   - **Given** un ticket que ya tiene un registro previo de check-in
   - **When** el encargado escanea nuevamente el ticket
   - **Then** el sistema rechaza el intento

---

### User Story 3 - Rechazar ingreso por zona incorrecta (P)riority: P1

Yo como encargado escaneo un ticket válido, pero en una puerta que no corresponde a su categoría o zona.

**Why this priority**:

Controla segmentación de accesos (VIP, general, staff, etc.).

**Independent Test**:

Escanear un ticket válido en una puerta distinta a la asignada. Debe devolver error “Zona Incorrecta”.

**Acceptance Scenarios**:

1. **Scenario**: Acceso no autorizado por zona
   - **Given** un ticket válido para zona A
   - **When** se escanea el ticket
   - **Then** el sistema rechaza el intento

---

### User Story 4 - Rechazar ingreso por estado inválido (P)riority: P1

Yo como encargado escaneo un ticket cuyo estado no permite ingreso (cancelado, reembolsado, bloqueado, etc.).

**Why this priority**: 

Garantiza que solo tickets activos puedan ingresar.

**Independent Test**:

Escanear un ticket con estado "cancelado". El sistema debe devolver “Estado inválido”.

**Acceptance Scenarios**:

1. **Scenario**: Ticket con estado no válido
   - **Given** un ticket con estado distinto a "activo"
   - **When** se escanea el ticket
   - **Then** el sistema rechaza el intento

---

### User Story 5 - Rechazar ingreso por sesión inválida (P)riority: P1

Yo como encargado escaneo un ticket correspondiente a otro evento o fecha.

**Why this priority**: 

Evita ingresos fuera de la fecha o evento correspondiente.

**Independent Test**:

Escanear ticket de evento pasado o distinto. El sistema debe devolver “Sesión Inválida”.

**Acceptance Scenarios**:

1. **Scenario**: Ticket de evento distinto
   - **Given** un ticket asociado a una sesión diferente a la activa
   - **When** se escanea el ticket
   - **Then** el sistema rechaza el intento

---

### Edge Cases

¿Qué pasa si el ticket no existe en la base de datos?
→ El sistema debe rechazar el intento y devolver error “Ticket no encontrado”, registrando el intento como fallido.

¿Qué pasa si hay pérdida de conexión con la base de datos?
→ El sistema debe devolver error técnico y no permitir el ingreso.

¿Qué pasa si dos lectores procesan el mismo ticket exactamente al mismo tiempo?
→ El sistema debe garantizar atomicidad y evitar doble check-in (control de concurrencia).

¿Qué pasa si el lector no tiene zona configurada?
→ El sistema debe rechazar el procesamiento y registrar error de configuración.

## Requirements *(mandatory)*

<!--
  ACTION REQUIRED: The content in this section represents placeholders.
  Fill them out with the right functional requirements.
-->

### Functional Requirements


-**FR-001**: System MUST validar la existencia del ticket en la base de datos.

-**FR-002**: System MUST validar que el estado del ticket sea válido para ingreso.

-**FR-003**: System MUST validar que la sesión/evento del ticket coincida con la sesión 
activa.

-**FR-004**: System MUST validar que la puerta pertenezca a la zona autorizada para el ticket.

-**FR-005**: System MUST verificar que el ticket no haya sido previamente utilizado.

-**FR-006**: System MUST registrar el intento de ingreso (exitoso o fallido) con timestamp y lector.

-**FR-007**: System MUST devolver un resultado estructurado con estado (aprobado/rechazado) y código de error cuando corresponda.

-**FR-008**: System MUST garantizar control de concurrencia para evitar doble procesamiento.

-**FR-009**: System MUST actualizar el estado del ticket a “ingresado” cuando el intento sea exitoso.

-**FR-010**: System MUST almacenar el motivo de rechazo usando el diccionario de errores definido.

### Key Entities 

**Ticket**:
    Representa la credencial de acceso.
    Atributos clave: id, código único, estado, categoría, zona permitida, sesión/evento, fecha, indicador de usado.
**IntentoIngreso**:
    Representa cada intento de validación de acceso.
    Atributos: id, ticket_id, fecha_hora, lector_id, resultado (aprobado/rechazado), código_error.
**Lector/Puerta**:
    Representa el dispositivo o acceso físico.
    Atributos: id, zona_asignada, estado.
**Sesión/Evento**:
    Representa la instancia temporal del evento.
    Atributos: id, fecha, estado (activa/inactiva).

## Success Criteria *(mandatory)*

<!--
  ACTION REQUIRED: Define measurable success criteria.
  These must be technology-agnostic and measurable.
-->

### Measurable Outcomes

  -**SC-001**: 100% de los intentos de ingreso generan un registro auditable.
  -**SC-002**: El sistema responde a un intento de escaneo en menos de 2 segundos en condiciones normales.
  -**SC-003**: 0 casos de doble ingreso para un mismo ticket bajo condiciones de concurrencia.
  -**SC-004**: 99% de los intentos válidos son autorizados correctamente sin intervención manual.
  -**SC-005**: 100% de los rechazos muestran un código de error correspondiente al diccionario definido.

