# 03 — Documento de Requerimientos Técnicos (TRD)
> Yavoy v1 — Decisiones técnicas, stack, herramientas y arquitectura

---

## Stack tecnológico

| Capa | Tecnología | Motivo |
|---|---|---|
| Frontend | React Native + Expo | App móvil multiplataforma (Android/iOS) |
| Base de datos | Firebase Firestore | Serverless, escalado horizontal automático, tiempo real |
| Autenticación | Firebase Auth (teléfono + SMS) | Integrado con el resto del stack Firebase |
| Almacenamiento de archivos | Firebase Storage | Fotos de documentos, perfiles, vehículos |
| Notificaciones push | Firebase Cloud Messaging (FCM) | Gratis e ilimitado, integrado con Firebase |
| Lógica de servidor | Firebase Cloud Functions | Cálculos sensibles fuera del teléfono |
| Mapas | Google Maps Platform SDK móvil | Gratuito e ilimitado para apps nativas |
| Navegación del conductor | Deep link a Google Maps + Waze | Sin Directions API propia en v1 |
| Estado global | Zustand | Estado compartido entre pantallas |
| Estado local | useState (React) | Estado de una sola pantalla |
| Testing | Jest | Solo para cálculo de precio y lógica de asignación |
| Repositorio | Monorepo en GitHub | Un solo repo: app + functions + reglas de Firestore |

---

## Arquitectura general

### Principio: capa de abstracción obligatoria

**Las pantallas NUNCA llaman a Firebase directamente.**

Toda comunicación con Firebase pasa por funciones intermedias organizadas en archivos de servicio (`*Service.ts`). Si se necesita migrar de Firebase en el futuro, solo se reescribe la capa de servicios — no las pantallas.

```
Pantalla → Service → Firebase
          (nunca)
Pantalla → Firebase
```

Esta regla se aplica en TODA sesión de vibecoding, sin excepción. La IA nunca debe escribir llamadas directas a Firestore, Firebase Auth o Firebase Storage desde un componente de pantalla.

### Estructura por feature (módulo de negocio)

Todo lo relacionado a una misma función del negocio vive junto en una carpeta:

```
app/
├── trip/           (viaje: service, pantallas, componentes)
├── driver/         (conductor: perfil, documentos, disponibilidad)
├── passenger/      (pasajero: perfil, direcciones)
├── auth/           (login por teléfono, SMS)
├── rating/         (calificaciones)
├── zones/          (zonas y precios)
├── shared/         (componentes y utilidades reutilizables)
functions/          (Cloud Functions)
firestore.rules     (reglas de seguridad)
```

**Regla de organización:**
- Si algo pertenece a UNA entidad → va en su carpeta.
- Si lo usan DOS O MÁS módulos → va en `shared/`.
- Nunca crear carpetas sueltas tipo `screens/` o `components/` fuera de este patrón.

### Estructura de servicios

Cada módulo tiene su propio archivo de servicio que concentra toda la comunicación con Firebase para esa entidad:

```
trip/tripService.ts       → createTrip(), cancelTrip(), confirmPayment()...
driver/driverService.ts   → getAvailableDrivers(), updateStatus()...
passenger/passengerService.ts → updateProfile(), getSavedAddresses()...
auth/authService.ts       → sendCode(), verifyCode(), signOut()...
zones/zoneService.ts      → getZoneForPoint(), calculateFare()...
```

---

## Estrategia de lectura de datos (Firebase)

**Listener en vivo** solo donde el tiempo real es imprescindible:
- Pantalla del pasajero esperando que un conductor acepte.
- Pantalla del viaje en curso (ambos lados, mientras el estado cambia).

**Fetch único** (lectura por demanda) para todo lo demás:
- Historial de viajes
- Perfil del usuario
- Direcciones guardadas
- Listas estáticas

**Regla general:** usar listener SOLO donde el tiempo real es imprescindible para que la app funcione. Listeners innecesarios son la causa #1 de costos elevados en Firebase.

---

## Gestión de estado (Zustand)

| Tipo de estado | Herramienta |
|---|---|
| Estado compartido entre pantallas (viaje activo, sesión, disponibilidad del conductor) | Zustand |
| Estado local de una sola pantalla (formularios, toggles, inputs) | `useState` (React) |

Zustand es especialmente importante para el estado conectado a listeners de Firebase — centraliza las actualizaciones en tiempo real sin re-renders innecesarios.

---

## Cloud Functions — lógica en servidor

Las siguientes operaciones **SIEMPRE** corren en Cloud Functions, nunca en el teléfono:

| Función | Motivo |
|---|---|
| Calcular el precio del viaje (zona + franja horaria + recargo) | Evita que el cliente manipule el precio |
| Asignar viaje al conductor más cercano (15s por conductor) | Coordinación entre múltiples conductores — no puede vivir en un solo teléfono |
| Enviar notificación push al conductor cuando hay nueva solicitud | Se dispara desde el servidor al cambiar el estado del viaje |
| Calcular y actualizar el promedio de calificación | Evita que alguien manipule su propio promedio |

**Regla:** todo lo que involucra confianza o coordinación entre varias personas va al servidor. Lo que es mostrar y manejar los datos propios de un usuario se queda en el teléfono.

---

## Principio de confianza cero (Zero Trust)

> **El servidor NUNCA confía en valores calculados que vienen del cliente.**

Reglas específicas:

1. **El cliente nunca envía el precio.** La Cloud Function calcula el precio ella misma con `(zona_origen, zona_destino, hora_servidor)`. Cualquier precio que el teléfono intente mandar es ignorado.

2. **Cada Cloud Function valida la identidad real de quien llama** contra Firestore, nunca confía en lo que el cliente "dice ser". Ejemplo: cuando el conductor confirma el pago, la función verifica que quien está autenticado es efectivamente el conductor asignado a ese viaje.

3. **App Check obligatorio desde v1.** Bloquea pedidos que no vienen de la app real de Yavoy (anti-scripts, anti-bots).

4. **Rate limiting / cuotas** para prevenir ataques de volumen.

5. **Todo dato que afecte precio, estado, aprobación o reputación se recalcula en el servidor a partir de datos crudos confiables** (hora actual del servidor, coordenadas, identidad verificada). Nunca se acepta un valor "ya calculado" que venga del teléfono.

Esta regla aplica a TODAS las Cloud Functions, presentes y futuras.

---

## Reglas de seguridad de Firestore

### Principio base
Por defecto todo está prohibido. Cada colección tiene reglas explícitas.

### Colección `trips`
- **Leer:** solo el pasajero y el conductor de ESE viaje específico.
- **Crear:** cualquier pasajero autenticado, solo viajes donde él sea el pasajero.
- **Actualizar estado/pago:** SOLO mediante Cloud Function. El teléfono no puede escribir directamente en un documento de viaje.

### Colección `drivers`
- **Leer datos públicos** (nombre, foto, calificación): cualquier usuario autenticado.
- **Leer datos sensibles** (cédula, documentos): solo el propio conductor.
- **Escribir perfil propio:** solo el propio conductor, en campos no-sensibles.
- **El campo `accountStatus`:** el conductor NO puede escribirlo. Solo se modifica desde la consola de Firebase (acceso total) o una Cloud Function restringida. Sin esta regla, cualquiera podría auto-aprobarse.

### Colección `zones` / tabla de precios
- **Leer:** cualquier usuario autenticado.
- **Escribir:** NADIE desde la app. Solo desde la consola de Firebase.

### Colección `ratings`
- **Leer promedio:** público para usuarios autenticados.
- **Crear:** solo el pasajero/conductor de ESE viaje, una única vez por viaje.
- **Editar / borrar:** nadie, nunca. Una calificación puesta no se puede modificar.

### Colección `passengers`
- **Leer datos propios:** solo el propio pasajero.
- **Leer nombre básico en el contexto de un viaje:** solo a través de lo que la Cloud Function expone en el documento del viaje.
- **Escribir:** solo el propio pasajero en sus campos no-sensibles (nombre, foto, direcciones).
- **Calificación y contador de cancelaciones:** el pasajero NO puede editar estos campos propios.

---

## Mapas y navegación

| Componente | Implementación | Costo |
|---|---|---|
| Mapa en la app (mostrar mapa + pines) | Maps SDK móvil nativo | Gratis, ilimitado |
| Pin estático del punto de recogida | Coordenadas dibujadas en el mapa | Gratis |
| Navegación del conductor | Deep link a Google Maps + Waze | Gratis (navega el usuario en su app, Yavoy no paga) |
| Rutas dibujadas en Yavoy | ❌ No en v1 | — |
| Seguimiento en vivo del conductor | ❌ No en v1 (v2) | — |
| Directions API | ❌ No en v1 | — |

**Regla explícita para la IA:** "Navegación en v1 = deep link a Waze/Google Maps. Yavoy NO dibuja rutas propias ni usa Directions API en v1. El mapa de Yavoy solo muestra pines estáticos."

---

## Autenticación

- Método: teléfono + SMS de 6 dígitos (Firebase Phone Auth).
- Verificación única al registrarse. Sesión persistente — no se repite el SMS en logins posteriores.
- Protecciones:
  - Límite de reenvíos de SMS por número.
  - reCAPTCHA (reduce fraude de SMS 10–30%).
  - Restricción de región: solo Venezuela y Colombia. Sin esta regla, un atacante puede disparar SMS a números internacionales caros.

---

## Notificaciones push

- Servicio: Firebase Cloud Messaging (FCM) — gratis e ilimitado.
- El sonido de nueva solicitud para el conductor es un audio personalizado (fuerte, reconocible) configurado como sonido de la notificación push.
- Las notificaciones se disparan desde Cloud Functions, no desde el teléfono del cliente.

---

## Manejo de errores

### Recuperación automática del viaje activo
Al abrir la app (o reconectar a internet), lo primero que hace es consultar "¿hay un `trip` activo para este usuario?":
- Si sí → salta directo a la pantalla de ese viaje.
- Si no → muestra la pantalla normal.

### Fallas de red
1. La acción falla.
2. La app reintenta en silencio 2–3 veces (~6–8 segundos totales).
3. Si sigue fallando → mensaje claro y humano + botón de reintentar manual.
4. Nunca mostrar códigos de error técnicos al usuario.
5. Siempre proveer una salida clara (reintentar / esperar / soporte).

---

## Testing

- **Pruebas automatizadas (Jest):** SOLO para cálculo de precio (matriz + recargo horario) y lógica de asignación de conductor.
- **Resto de la app:** prueba manual durante el vibecoding.
- Sin testing de UI ni end-to-end en v1.
- Ampliar cobertura en v2+.

---

## Variables de entorno

- Las claves API viven en un archivo `.env` local — **nunca se suben a GitHub**.
- El repo incluye `.env.example` con los nombres de las claves pero sin los valores.
- Cada colaborador (presente o futuro) tiene sus propias claves independientes.
- Las claves son revocables individualmente desde la consola de Google Cloud.
- Un solo set de claves por ahora (dev/prod separados → antes del lanzamiento a producción).

---

## Reglas de seguridad — 4 principios máximos (obligatorios, máxima prioridad)

Estas reglas se repiten aquí, en el doc de ciberseguridad (08), y en CLAUDE.md. La IA debe respetarlas en TODA sesión sin excepción:

1. **NUNCA exponer las API keys** — ni en el código, ni en commits, ni en logs.
2. **VALIDAR todo lo que escribe el usuario. NUNCA confiar en el cliente** — todo dato sensible se recalcula en el servidor.
3. **USAR rate limits** — contra scripts, bots y abuso de volumen.
4. **PROTEGER la base de datos** — cada usuario ve solo lo que le corresponde. Las reglas de Firestore son la última línea de defensa.

**Marco de referencia:** OWASP Mobile Top 10 (ver doc 08 para el checklist completo).

---

*Documento generado de la sesión de planificación de Yavoy. Revisión: v1.0*
