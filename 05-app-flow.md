# 05 — App Flow
> Yavoy v1 — Flujo de navegación, pantallas e interacciones
> ⚠️ DOCUMENTO EN PROGRESO — se completa en sesiones posteriores

---

## Estructura de navegación

### Tipo de app
- **Una sola app, dos modos:** pasajero y conductor.
- **Switch de modo:** Patrón A — pantalla de selección explícita.
- **La app recuerda el último modo usado.** La pantalla de selección aparece solo cuando el usuario cambia de modo intencionalmente — no en cada apertura.
- **Tab bar** como navegación principal.

### Tabs por modo

#### Modo pasajero
| Tab | Ícono | Contenido |
|---|---|---|
| Inicio | `ti-home` | Pantalla principal, pedir viaje, viaje en curso |
| Actividad | `ti-clock` | Historial de viajes pasados |
| Cuenta | `ti-user` | Perfil, direcciones guardadas, soporte, cambiar modo |

#### Modo conductor
| Tab | Ícono | Contenido |
|---|---|---|
| Mapa | `ti-map` | Toggle conectado/desconectado, solicitudes, viaje activo |
| Actividad | `ti-clock` | Historial de viajes completados, calificaciones |
| Cuenta | `ti-user` | Perfil, documentos, soporte, cambiar modo |

---

## Flujo 1 — Onboarding y registro

### Pantalla: Splash / Bienvenida
- Muestra el logo de Yavoy.
- Transición automática a → Selección de modo inicial.

### Pantalla: Selección de modo inicial
- "¿Cómo querés usar Yavoy?"
- Opción A: "Soy pasajero" → Registro pasajero.
- Opción B: "Soy mototaxista o taxista" → Registro conductor.

---

### Flujo 1A — Registro pasajero

Flujo de 4 pasos. La app no guarda progreso automático (es muy corto).

#### Paso 1: Teléfono
- Campo de número de teléfono + botón "Enviar código".
- ✓ Éxito → Paso 2.
- ✗ Número inválido → "Número no válido, revísalo" (error inline).
- ✗ SMS no llega → "¿No recibiste el código? Reenviar" (con límite de reenvíos).

#### Paso 2: Verificación SMS
- Campo de 6 dígitos.
- ✓ Código correcto → Paso 3.
- ✗ Código incorrecto → "Código incorrecto, intenta de nuevo" (máximo 3 intentos → bloqueo temporal 10 min).
- ✗ Código vencido → "El código expiró, solicita uno nuevo".

#### Paso 3: Datos personales
- Campos: Nombre + Apellido (separados).
- ✓ Llenos → Paso 4.
- ✗ Campo vacío → "Este campo es obligatorio" (inline).
- ✗ Nombre muy corto (1 letra) → "Ingresa tu nombre completo".

#### Paso 4: Términos y condiciones
- Checkbox de aceptar.
- ✓ Aceptado → Pantalla Principal Pasajero (Tab Inicio).
- ✗ Sin aceptar → "Debes aceptar los términos para continuar".

#### Error global (todos los pasos)
- Sin internet → reintento silencioso (2–3 intentos, ~6–8 seg) → "Sin conexión. Verifica tu señal e intenta de nuevo" + botón reintentar.
- Error de servidor → "Algo salió mal de nuestro lado. Intenta en unos minutos."

---

### Flujo 1B — Registro conductor

Flujo de 9 pasos. **La app guarda el progreso automáticamente en Firestore** al completar cada paso. Si el conductor cierra la app a mitad, retoma desde donde dejó.

#### Paso 1: Teléfono
- Idéntico al Paso 1 del registro pasajero.

#### Paso 2: Verificación SMS
- Idéntico al Paso 2 del registro pasajero.

#### Paso 3: Datos personales
- Campo: Nombre completo.
- ✓ Lleno → Paso 4.
- ✗ Campo vacío → "Este campo es obligatorio".

#### Paso 4: Tipo de servicio
- Elige: Mototaxi O Taxi (no ambos en v1).
- ✓ Seleccionado → Paso 5.
- ✗ Sin selección → "Debes elegir un tipo de servicio para continuar".

#### Paso 5: Datos del vehículo
- Campos: placa, marca, modelo, color.
- ✓ Completos → Paso 6.
- ✗ Campo vacío → error inline por campo faltante.
- ✗ Placa en formato inválido → "Formato de placa no válido".

#### Paso 6: Foto del vehículo
- Abre cámara o galería. Sube foto.
- La app comprime automáticamente si la imagen es muy pesada (sin avisarle al usuario).
- ✓ Foto subida → Paso 7.
- ✗ Falla la subida → "No se pudo subir la foto. Verifica tu conexión e intenta de nuevo" + botón reintentar.

#### Paso 7: Documentos (4 fotos, una por sub-paso)
- Sub-paso 7a: Cédula de identidad.
- Sub-paso 7b: Licencia de conducir (2do grado si eligió moto / 3er grado si eligió taxi — la app adapta el texto según el tipo elegido en el Paso 4).
- Sub-paso 7c: Carnet de circulación / Título de propiedad.
- Sub-paso 7d: Certificado médico (acorde al tipo de vehículo).
- ✓ Cada foto subida → avanza al siguiente sub-paso.
- ✗ Falla subida → mismo patrón: error claro + reintentar.

#### Paso 8: Selfie / Foto de perfil
- Abre cámara (selfie), toma foto.
- ✓ Subida → Paso 9.
- ✗ Sin permiso de cámara → "Yavoy necesita acceso a la cámara para tomar tu foto. Actívalo en Configuración" + botón que abre la configuración del teléfono.

#### Paso 9: Términos y condiciones
- Checkbox de aceptar.
- ✓ Aceptado → Pantalla "Cuenta en revisión".
- ✗ Sin aceptar → "Debes aceptar los términos para continuar".

#### Pantalla: Cuenta en revisión
- Mensaje: "¡Listo! Recibimos tus datos. Revisaremos tus documentos y te avisaremos por notificación cuando estés aprobado para empezar."
- Sin acceso a recibir viajes.
- Puede cambiar a modo pasajero mientras espera (no queda bloqueado).
- Al ser aprobado → notificación push → puede empezar a trabajar.

---

## Flujo 2 — Pedir un viaje (modo pasajero)

### Pantalla [1]: Inicio (Tab 1 del pasajero)

**Composición visual:**
- Mapa a pantalla completa con pin de ubicación actual del usuario (Azul Eléctrico).
- Header flotante superior: logo Yavoy + botón de notificaciones.
- Botón de centrar mapa (esquina inferior derecha del mapa).
- Tarjeta flotante inferior (el elemento principal de la pantalla).

**Tarjeta flotante inferior contiene:**
1. Saludo: "Buenas [momento del día], [Nombre]".
2. Campo "¿A dónde vas?" → toca para abrir pantalla de destino.
3. Accesos rápidos: Casa / Trabajo (si están guardados).
4. Selector de tipo de servicio: Mototaxi / Taxi (con precio base visible).
5. Toggle "Pedir para otra persona".

**Comportamiento del selector de servicio:**
- El tipo seleccionado: círculo en Verde Avanza.
- El no seleccionado: círculo en Carbón con opacidad reducida.

---

### Pantalla [2]: Selección de destino (pantalla separada)

Se abre al tocar el campo "¿A dónde vas?".

**Contiene:**
- Buscador de texto con teclado abierto al entrar.
- Campo de referencia escrita ("portón azul, frente a la cancha") — campo clave para Rubio.
- Sugerencias de direcciones recientes.
- Direcciones guardadas (Casa / Trabajo).
- Mapa chico con pin ajustable para confirmar el punto exacto.

**Errores:**
- Sin resultados → "No encontramos esa dirección. Mueve el pin en el mapa o escribe una referencia".

**Caso "Pedir para otra persona" (toggle activo):**
- Campo adicional obligatorio: teléfono del pasajero real.
- El punto de recogida es el del tercero, no el de quien pide.
- ✗ Sin teléfono → "El teléfono es obligatorio cuando pides para otra persona. El conductor necesita contactarla."

---

### Pantalla [3]: Confirmación de precio

Se muestra en la tarjeta del Inicio después de seleccionar destino.

**Muestra:**
```
El Rosal → Centro ........... $4.000
Recargo nocturno (+20%) ..... +$800    ← solo si aplica
──────────────────────────────────────
Total ........................ $4.800
```

- Desglose siempre visible — nunca solo el total.
- Precio congelado al momento de solicitar — no cambia si la franja horaria avanza.
- Botón "Confirmar viaje" en Verde Avanza.

**Errores:**
- Puntos fuera de zona → "Esta dirección está fuera de nuestra zona de cobertura por ahora".
- Sin internet → reintento silencioso → error claro.

---

### Pantalla [4]: Buscando conductor (espera)

- Animación de búsqueda.
- Texto: "Buscando tu mototaxista..." (o "taxista" según el tipo elegido).
- Botón "Cancelar viaje" visible.
- La Cloud Function ofrece el viaje al conductor más cercano disponible (15 segundos por conductor, luego pasa al siguiente).

**Resultados:**
- ✓ Conductor acepta → Pantalla [5].
- ✗ Nadie disponible → Pantalla [4B].

---

### Pantalla [4B]: Sin conductor disponible

Mensaje específico según el caso:
- Sin conductores conectados → "No hay mototaxistas disponibles ahora mismo. Intenta en unos minutos."
- Todos ocupados → "Todos los mototaxistas están ocupados. Intenta de nuevo en un momento."

Botones:
- "Reintentar" → vuelve a la Pantalla [4].
- "Volver al inicio" → vuelve a la Pantalla [1].

---

### Pantalla [5]: Conductor asignado (viaje activo — esperando recogida)

**Muestra:**
- Foto, nombre y calificación del conductor.
- Tipo, color y placa del vehículo.
- Teléfono del conductor con botón de llamada directa.
- Mapa con pin estático del punto de recogida.
- Tiempo estimado de llegada (rango, no exacto):
  - Distancia < 300m → "Tu mototaxista está muy cerca".
  - Distancia media → "Tu mototaxista está cerca, suele llegar en X–Y min".
  - Distancia > 3km → "Tu mototaxista va en camino" (sin estimación).
- Advertencia si hay posibilidad de pago adelantado: "Si vas a pagar por adelantado, hazlo solo cuando el conductor llegue".
- Botón "Cancelar viaje" (disponible mientras `paymentStatus: unpaid`).

**Caso "para otra persona":**
- Se muestra el teléfono del tercero prominentemente con botón de llamada directa.

---

### Pantalla [6]: Conductor llegó (estado `arrived`)

- Notificación push al pasajero.
- La pantalla actualiza: "Tu mototaxista llegó".
- Mismos datos del conductor.
- Si el conductor confirmó el pago → viaje CONGELADO → botón "Cancelar" desaparece → aparece "Reportar problema".

---

### Pantalla [7]: En curso (estado `inProgress`)

- Texto: "Vas en camino a tu destino".
- Datos del conductor visibles.
- Sin botón de cancelar.
- Botón "Reportar problema" disponible.

---

### Pantalla [8]: Viaje completado — Calificación

La pantalla de calificación aparece automáticamente (vía el listener del estado del viaje) — sin notificación push.

- Texto: "¿Cómo estuvo tu viaje con [nombre del conductor]?"
- Estrellas 1–5.
- Comentario opcional si la nota es < 5 estrellas.
- Botón "Calificar" → vuelve a la Pantalla [1].
- Botón "Ahora no" → vuelve a la Pantalla [1] sin calificar (el viaje queda en historial sin calificación).

---

## Pendiente — Flujos a definir en próximas sesiones

Los siguientes flujos están identificados pero no mapeados todavía:

- **Flujo 3:** Pantalla del conductor (recibir solicitud, aceptar/rechazar, viaje activo, confirmar pago, completar viaje).
- **Flujo 4:** Historial de viajes (pasajero y conductor).
- **Flujo 5:** Perfil y cuenta (editar perfil, direcciones guardadas, cambiar modo, soporte).
- **Flujo 6:** Calificación (desde el lado del conductor).
- **Flujo 7:** Soporte (formulario o link a WhatsApp).
- **Flujo 8:** Pantalla de "Cuenta en revisión" y aprobación del conductor.

---

*Documento en progreso — generado de la sesión de planificación de Yavoy. Revisión: v0.5*
