# 02 — Glosario / Modelo de Dominio
> Yavoy v1 — Diccionario oficial del proyecto

---

## Principio rector

**El código se escribe en inglés. La interfaz de usuario se muestra en español.**

Esta distinción es absoluta. Ningún nombre de variable, función, colección o componente debe estar en español en el código. La IA debe respetar esta regla en cada sesión, sin excepción.

---

## Tabla de términos

### Entidades principales

| Código (inglés) | Interfaz (español) | Qué es |
|---|---|---|
| `passenger` | Pasajero | Quien solicita el viaje |
| `driver` | Conductor (genérico) | Quien presta el servicio de transporte |
| `trip` | Viaje | Una carrera de punto A a punto B |
| `zone` | Zona | Sector de Rubio definido como polígono geográfico |
| `fare` | Tarifa | Precio de un viaje según la tabla de zonas |
| `rating` | Calificación | Puntuación de 1 a 5 estrellas |

### Vocabulario de interfaz según tipo de conductor

El código usa siempre `driver`. La interfaz en español usa el término según el tipo de servicio:

| `serviceType` | Código | Interfaz (específico) | Interfaz (genérico) |
|---|---|---|---|
| Moto | `"moto"` | Mototaxista | Conductor |
| Taxi | `"taxi"` | Taxista | Conductor |

- **Mototaxista** → cuando el servicio activo es moto.
- **Taxista** → cuando el servicio activo es taxi.
- **Conductor** → en textos genéricos donde no importa el tipo ("Conviértete en conductor", "Califica a tu conductor").

### Tipos de servicio

| Código | Interfaz |
|---|---|
| `serviceType: "moto"` | Mototaxi |
| `serviceType: "taxi"` | Taxi |

---

## Estados del viaje

| Código (`tripStatus`) | Interfaz (español) | Descripción |
|---|---|---|
| `requested` | Solicitado | El pasajero pidió, buscando conductor |
| `assigned` | Asignado | Conductor aceptó, va en camino al pasajero |
| `arrived` | En recogida | Conductor llegó al punto, espera al pasajero |
| `inProgress` | En curso | Pasajero a bordo, viaje hacia el destino |
| `completed` | Completado | Viaje terminado |
| `cancelled` | Cancelado | Alguien canceló (solo posible si `unpaid`) |
| `noDriver` | Sin conductor | Nadie aceptó / no había conductores disponibles |

## Estado de pago

| Código (`paymentStatus`) | Interfaz (español) | Descripción |
|---|---|---|
| `unpaid` | No pagado | Estado inicial — se puede cancelar |
| `paid` | Pagado | Conductor confirmó cobro — viaje congelado |

## Estados de disponibilidad del conductor

| Código (`driverStatus`) | Interfaz (español) |
|---|---|
| `available` | Disponible |
| `busy` | Ocupado |
| `offline` | Desconectado |

## Estados de cuenta del conductor

| Código (`accountStatus`) | Interfaz (español) |
|---|---|
| `pending` | Pendiente de revisión |
| `approved` | Aprobado |
| `rejected` | Rechazado |

---

## Reglas de nomenclatura

Estas reglas aplican a **cualquier nombre nuevo** que no esté en la tabla anterior. La IA debe seguirlas en toda sesión de vibecoding, sin excepción.

| Contexto | Convención | Ejemplo |
|---|---|---|
| Variables y funciones | `camelCase` | `pickupPoint`, `calculateFare()` |
| Componentes (pantallas, UI) | `PascalCase` | `TripScreen`, `DriverProfile` |
| Constantes fijas | `UPPER_SNAKE_CASE` | `MAX_WAIT_SECONDS` |
| Colecciones en Firestore | plural y minúscula | `trips`, `drivers`, `zones` |
| Campos de fecha/hora | terminan en `At` | `createdAt`, `completedAt` |
| Booleanos | empiezan con `is` o `has` | `isAvailable`, `hasPaid` |
| Todo el código | inglés | nunca `viaje`, `conductor` en el código |

---

## Notas adicionales

- `trip` es el término central del dominio. Toda la lógica de negocio gira en torno al objeto `trip`.
- Un `driver` tiene exactamente un `serviceType` — nunca los dos a la vez en v1.
- Las `zone`s no son etiquetas de texto — son polígonos geográficos con coordenadas. El precio se calcula a partir del par `(origin zone, destination zone)`.
- El `fare` nunca viene del cliente — siempre se calcula en el servidor (Cloud Function).

---

*Documento generado de la sesión de planificación de Yavoy. Revisión: v1.0*
