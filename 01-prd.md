# 01 — Documento de Requerimientos del Producto (PRD)
> Yavoy v1 — Ride-hailing para Rubio, Táchira, Venezuela

---

## Visión

Yavoy es una app de transporte bajo demanda (ride-hailing) enfocada en Rubio y zonas aledañas del estado Táchira, Venezuela. Conecta pasajeros con mototaxistas y taxistas locales de forma rápida, segura y confiable. La app no procesa pagos — el dinero se maneja directamente entre pasajero y conductor (efectivo o pago móvil), en cumplimiento de la regulación financiera venezolana (Sudeban).

---

## Alcance v1

- Servicios: **mototaxi** y **taxi** únicamente.
- Zona de operación: Rubio y áreas aledañas (Táchira).
- Sin delivery, sin favores, sin envío de paquetes (fases posteriores).

---

## Roles de usuario

Yavoy v1 tiene **dos roles**. No hay panel de administración en la app — la administración se hace manualmente desde la consola de Firebase.

### Pasajero

| Funcionalidad | Descripción |
|---|---|
| Registro / login | Teléfono + SMS único, sesión persistente |
| Pedir viaje | Elegir origen (GPS + pin ajustable + referencia escrita), destino, tipo de servicio |
| Ver precio antes de confirmar | Desglose visible: precio base + recargo + total |
| Confirmar viaje | La app busca conductor disponible |
| Ver datos del conductor asignado | Nombre, foto, tipo/color/placa del vehículo, teléfono |
| Cancelar viaje | Solo si estado de pago es NO_PAGADO |
| Ver historial de viajes | Viajes pasados con detalle |
| Calificar al conductor | 1–5 estrellas, anónimo, comentario opcional si < 5 |
| Direcciones guardadas | Casa y trabajo |
| Soporte | Formulario o link a WhatsApp |
| Pedir para otra persona | Toggle "para otra persona" + teléfono obligatorio + punto de recogida del tercero |

### Conductor

| Funcionalidad | Descripción |
|---|---|
| Registro / aplicar | Subir documentos — queda en estado "pendiente" hasta aprobación manual |
| Estado de disponibilidad | Disponible / ocupado / desconectado |
| Recibir solicitud de viaje | Origen, destino, precio — ventana de 15 segundos para aceptar |
| Aceptar o rechazar | Si no responde en 15s, pasa al siguiente conductor disponible |
| Ver datos del pasajero | Nombre, teléfono, pin del punto de recogida, referencia escrita |
| Navegar al pasajero | Deep link a Google Maps o Waze (conductor elige), pin estático del punto de recogida |
| Confirmar pago recibido | Botón "Recibí el pago" — congela el viaje (no se puede cancelar) |
| Marcar viaje como completado | Cambia estado a COMPLETADO |
| Calificar al pasajero | 1–5 estrellas, anónimo |
| Ver historial | Viajes completados, calificaciones |
| Soporte | Formulario o link a WhatsApp |

---

## Modelo de precio

- **Tipo:** Matriz origen–destino por zonas (par de zonas).
- **Zonas:** Modeladas como polígonos geográficos (se definen en la fase de implementación, transcritos de las tarifas reales de Rubio).
- **Tablas separadas:** mototaxi y taxi tienen tablas independientes.
- **Moneda:** Pesos colombianos (COP). Tasa de cambio a bolívares controlada manualmente desde la consola de Firebase.
- **Recargo nocturno:** Automático, por franjas horarias (Opción B — varias franjas). Recargos independientes por tipo de servicio. Valores exactos se definen en implementación.
- **Desglose visible al pasajero:** la app siempre muestra precio base + recargo + total. Nunca el total pelado.
- **El precio se congela al momento de solicitar el viaje** — no cambia si la franja horaria avanza durante la espera.
- **Regla de arquitectura:** el cálculo de precio lo hace una Cloud Function en el servidor, nunca el teléfono. El cliente nunca envía el precio calculado — siempre se recalcula en el servidor.

---

## Pago y cancelaciones

### Pago
- El pago ocurre **por fuera de la app** (efectivo o pago móvil directo) en cualquier momento: antes, durante o al final del viaje.
- La app **no procesa ni valida dinero** — solo registra una declaración.
- **Estado de pago** (paralelo al estado del viaje): `NO_PAGADO` → `PAGADO`.
- El **conductor** confirma "Recibí el pago" — es quien recibe, quien se compromete.
- Al confirmarse PAGADO: el viaje queda **congelado** — no se puede cancelar, solo "Reportar problema" → soporte (el administrador).
- **Advertencia al pasajero:** "Si vas a pagar por adelantado, hazlo solo cuando tengas conductor asignado."

### Cancelaciones
- Cualquiera puede cancelar mientras el estado de pago es `NO_PAGADO`.
- Al cancelar: el usuario elige un motivo de una lista.
- Sin castigo automático a la calificación por cancelar.
- **Contador de cancelaciones** por usuario (separado de las estrellas).
- Si alguien cancela en exceso → alerta al administrador → decisión manual.
- Se registra: quién canceló, en qué estado del viaje, el motivo.

---

## Estados del viaje

```
REQUESTED   → Solicitado       (buscando conductor)
ASSIGNED    → Asignado         (conductor aceptó, va en camino)
ARRIVED     → En recogida      (conductor llegó al punto)
IN_PROGRESS → En curso         (pasajero a bordo, rumbo al destino)
COMPLETED   → Completado       (viaje terminado)
CANCELLED   → Cancelado        (solo posible si NO_PAGADO)
NO_DRIVER   → Sin conductor    (nadie aceptó / sin conductores disponibles)
```

**Estado de pago (paralelo):**
```
UNPAID → NO_PAGADO
PAID   → PAGADO (lo confirma el conductor)
```

**Mensajes para NO_DRIVER:**
- Sin conductores conectados → "No hay mototaxistas disponibles ahora mismo. Intenta en unos minutos."
- Todos ocupados → "Todos los mototaxistas están ocupados. Intenta de nuevo en un momento."
- Sin reintento automático en v1 — el pasajero reintenta manualmente.

---

## Ubicación y punto de recogida

- El GPS propone la ubicación inicial del pasajero.
- El pasajero puede **mover el pin** en el mapa para ajustar el punto exacto.
- Campo de **referencia escrita** obligatorio para el conductor ("portón azul, frente a la cancha"). Crítico en Rubio donde las direcciones son por referencia.
- Si no hay permiso de GPS → el usuario escribe dirección y referencia a mano.
- El conductor ve: pin del punto de recogida + dirección + referencia + teléfono.
- **Navegación del conductor:** deep link doble (Google Maps + Waze, el conductor elige). Yavoy no dibuja rutas propias ni usa Directions API en v1.
- El mapa de Yavoy solo muestra **pines estáticos** — sin seguimiento en vivo del conductor (v2).

---

## Calificaciones

- Escala 1–5 estrellas, **bidireccional** (pasajero califica conductor, conductor califica pasajero).
- **Anónima** — no se ve quién te calificó individualmente.
- **Comentario opcional** solo si la nota es < 5 estrellas.
- Se muestra el **promedio simple** (de todos los viajes del usuario).
- Se pide calificar al terminar el viaje — **no bloquea** el uso de la app si se omite.
- Conductor nuevo arranca sin promedio hasta acumular calificaciones.
- Si alguien baja de un umbral (ej. 3.5 estrellas) → **alerta al administrador** → decisión manual. Sin desactivación automática en v1.

---

## Autenticación

- **Método:** Teléfono + SMS de 6 dígitos, verificación única al registrarse.
- **Sesión persistente** — no se pide código nuevo en cada login. Un solo SMS por usuario en toda su vida (salvo reinstalación o cierre manual de sesión).
- Protecciones de costo:
  - Máximo de reenvíos de SMS por número (límite de Firebase).
  - reCAPTCHA para prevenir fraude de SMS.
  - Restricción de región: solo Venezuela y Colombia.
- Sin correo ni contraseña en v1 → evaluación en v2.

---

## Perfiles

### Pasajero
- Nombre + Apellido (separados)
- Teléfono (identidad de login)
- Foto de perfil (opcional, se puede agregar/cambiar después del registro)
- Direcciones guardadas (casa / trabajo)
- Calificación promedio + contador de cancelaciones
- Fecha de registro

### Conductor
- Nombre completo
- Teléfono (identidad de login)
- Foto de perfil / selfie (obligatoria)
- Número de cédula + foto del documento
- Foto de licencia de conducir (2do grado para moto / 3er grado para taxi)
- Foto de carnet de circulación / título de propiedad
- Foto de certificado médico (acorde al tipo de vehículo)
- Tipo de servicio: `"moto"` O `"taxi"` (único, elegido al registrarse, ligado al vehículo)
- Datos del vehículo: marca, modelo, placa, color, foto del vehículo
- Estado de cuenta: `pending` / `approved` / `rejected`
- Estado de disponibilidad: `available` / `busy` / `offline`
- Calificación promedio + contador de cancelaciones
- Fecha de registro / fecha de aprobación

### Aprobación de conductores
- Manual por el administrador desde la consola de Firebase.
- El conductor sube documentos → queda en `pending`.
- El administrador revisa fotos a mano → cambia `accountStatus` a `approved` o `rejected`.
- Un conductor con `accountStatus` distinto de `approved` **no puede recibir viajes** (reforzado en reglas de seguridad de Firestore, no solo en la UI).
- No hay verificación de cédula del pasajero en v1.

---

## Registro del conductor — progreso automático

Si el conductor cierra la app a mitad del registro, el progreso se guarda automáticamente en Firestore. Al volver, retoma desde el paso donde quedó.

---

## Notificaciones push

### Pasajero
| Evento | Notificación |
|---|---|
| Conductor aceptó el viaje | Sí (siempre a quien PIDIÓ, sea para sí mismo o para tercero) |
| Conductor llegó al punto de recogida | Sí |
| Sin conductor disponible | Sí |
| Viaje completado | No — la pantalla de calificación aparece sola (vía listener) |

### Conductor
| Evento | Notificación |
|---|---|
| Nueva solicitud de viaje | Sí — sonido propio, fuerte y reconocible (nivel 1: push estándar con audio personalizado) |
| Pasajero canceló (si ya iba en camino) | Sí |
| Cuenta aprobada / rechazada | Sí |

El tercero (carrera para otra persona sin cuenta de Yavoy) se entera por llamada directa del conductor — sin notificación push.

---

## Lo que Yavoy v1 NO tiene

| Exclusión | Motivo |
|---|---|
| ❌ Pagos dentro de la app | Decisión legal (Sudeban) |
| ❌ Seguimiento del conductor en vivo | v2 — el pin estático SÍ va en v1 |
| ❌ Rutas dibujadas dentro de la app / Directions API | Se usa deep link a Google Maps/Waze |
| ❌ Negociación de precio estilo Cuádralo | v2 |
| ❌ Propinas dentro de la app | La app no toca dinero |
| ❌ Delivery de comida | Fase posterior |
| ❌ Favores / Mandaditos / envío de paquetes | Fase posterior |
| ❌ Verificación de identidad del pasajero | v2 |
| ❌ Panel de administración | Se administra por consola de Firebase |
| ❌ Viajes entre ciudades | Solo Rubio y zonas aledañas |
| ❌ Programar viajes con antelación | v2 |

---

## Features planificadas para v2

- Seguimiento del conductor en vivo (mapa en tiempo real)
- Negociación de precio estilo Cuádralo (pasajero propone, conductor acepta)
- Verificación de identidad del pasajero
- "Avísame cuando haya conductor disponible" (notificación de disponibilidad)
- Rutas dibujadas dentro de la app (Directions API)
- Notificación push al tercero (si tiene cuenta de Yavoy)
- Notificación tipo llamada entrante para el conductor (nivel 2)
- Filtro de pasajeros por calificación mínima (para el conductor)

---

*Documento generado de la sesión de planificación de Yavoy. Revisión: v1.0*
