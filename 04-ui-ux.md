# 04 — Diseño UI/UX
> Yavoy v1 — Identidad visual, sistema de componentes y reglas de diseño

---

## Identidad de marca

- **Nombre:** Yavoy
- **Logo:** La letra Y forma un signo de exclamación — energía, urgencia positiva.
- **Personalidad:** Energía de Rappi + calidez local de Rubio. Moderno sin ser frío. Cercano sin ser infantil.
- **Modo:** Solo modo oscuro en v1. El modo claro es una mejora para v2 si los usuarios lo piden.

---

## Paleta de colores

| Nombre | Hex | Rol |
|---|---|---|
| Carbón | `#15151C` | Fondo principal de pantalla, fondo de tarjetas de información clave |
| Hueso | `#ECEBE2` | Texto principal, elementos sobre fondo Carbón |
| Verde Avanza | `#C5F23E` | Acción principal (botones de confirmar/aceptar), íconos de acento |
| Azul Eléctrico | `#3D3BF0` | Acentos secundarios, información, links |
| Carbón Medio | `#1E1E27` | Fondo de tarjetas secundarias, inputs |

### Jerarquía de color (obligatoria)

- **Verde Avanza** → botón de acción principal únicamente ("Confirmar viaje", "Aceptar", "Pedir mototaxi").
- **Azul Eléctrico** → acentos secundarios, textos de información, links. Nunca en botones primarios.
- **Carbón** → fondo general de pantalla y tarjetas de información clave.
- **Hueso** → texto principal sobre fondos oscuros.

Estos roles NO son intercambiables. Verde y Azul no se usan como si fueran lo mismo.

### Psicología del color

El verde-lima (#C5F23E) sobre fondo Carbón (#15151C) cumple un doble rol psicológico: evoca "luz verde" / "avanzar" (señal universal de acción) y tiene el mayor impacto visual de la paleta. Se usa intencionalmente en los botones de confirmación para incentivar la acción del usuario. El modo oscuro reduce la fatiga visual en conductores que usan la app durante horas seguidas.

---

## Tipografía

- **Familia:** Sora (Google Fonts — gratuita, integrada en Expo)
- **Una sola familia** para toda la app — sin mezclar fuentes.

| Peso | Uso |
|---|---|
| 500 (Medium) | Texto de cuerpo, etiquetas, campos |
| 700 (Bold) | Títulos, precios, botones, datos clave |

**Por qué Sora:** geométrica con personalidad propia, menos sobreusada que Poppins o Inter, buena legibilidad en pantallas chicas con luz solar directa (el caso real de mototaxistas en Rubio).

---

## Sistema de componentes

### Lenguaje de formas: muy redondeado (estilo burbujeante)

Coherente con la personalidad cálida y enérgica de la marca. El peso visual es sólido y visible — botones y tarjetas se ven con confianza, no frágiles.

### Radios de esquina (corner radius)

| Elemento | Radio |
|---|---|
| Botones | 28px (forma píldora completa) |
| Tarjetas grandes / contenedores de pantalla | 18–20px |
| Inputs y tarjetas pequeñas | 16px |
| Íconos en círculo | 50% (círculo perfecto) |

### Espaciado — escala de 4px

Todos los márgenes y separaciones son múltiplos de 4. Nunca números arbitrarios (13px, 27px, etc.).

| Token | Valor | Uso |
|---|---|---|
| xs | 4px | Separación mínima entre elementos dentro de un componente |
| sm | 8px | Separación interna chica |
| md | 12–16px | Separación estándar entre secciones |
| lg | 24px | Separación entre bloques |
| xl | 32px | Separación entre secciones grandes |

- **Margen exterior de pantalla:** 16px (aire entre contenido y borde del teléfono).
- **Separación entre tarjetas:** 16–24px.
- **Padding interno de botones:** 14px vertical, 22px horizontal.

### Botones

- Forma: píldora (border-radius 28px).
- Peso de texto: Sora 700.
- Botón principal: fondo Verde Avanza (`#C5F23E`) + texto Carbón (`#15151C`).
- Botón secundario: fondo Carbón Medio (`#1E1E27`) + texto Hueso (`#ECEBE2`).

### Inputs / campos de texto

- Border-radius: 16px.
- Fondo: `#1E1E27` (Carbón Medio).
- Texto placeholder: Hueso con 40% de opacidad.
- Ícono izquierdo (cuando aplica): en color Verde Avanza o Azul Eléctrico según contexto.

### Tarjetas

- Tarjeta de información clave (precio, viaje activo): fondo Carbón (`#15151C`) con borde sutil.
- Tarjeta secundaria: fondo `#1E1E27`.
- Ambas con border-radius 18–20px.

---

## Sistema de iconografía

- **Librería:** Tabler Icons (outline).
- **Tratamiento:** Estilo B — el ícono se monta sobre un círculo sólido de fondo.

| Variante | Fondo del círculo | Color del ícono | Cuándo usar |
|---|---|---|---|
| Default | Carbón (`#15151C`) | Hueso (`#ECEBE2`) | La mayoría de los íconos |
| Destacado | Verde Avanza (`#C5F23E`) | Carbón (`#15151C`) | Íconos de acción crítica: seguridad, confirmación, alerta importante |

- **Tamaño de ícono:** 24px.
- **Tamaño del círculo:** 48px de diámetro.
- **Razón:** el círculo sólido tiene el mismo peso visual que los botones píldora — la iconografía es coherente con el sistema de componentes. Legible de un vistazo al sol.

---

## Diferenciación mototaxi / taxi

Misma identidad visual para toda la app. No hay colores de marca separados por tipo de servicio.

| Elemento | Mototaxi | Taxi |
|---|---|---|
| Ícono | `ti-motorbike` sobre círculo | `ti-car` sobre círculo |
| Etiqueta | "Mototaxi" | "Taxi" |
| Estado seleccionado | Círculo en Verde Avanza | Círculo en Verde Avanza |
| Estado no seleccionado | Círculo en Carbón, ícono con opacidad reducida | Igual |
| Color de marca | Verde Avanza (el mismo para ambos) | Verde Avanza (el mismo) |

---

## Flujo de trabajo de diseño

1. Definir cada pantalla en el App Flow (doc 05) con descripción textual y especificaciones.
2. Preparar un brief en markdown con los requisitos de la pantalla.
3. Ejecutar el diseño en **Claude Design**, que tiene el sistema de marca cargado.
4. Cuando el diseño está listo → Claude Design genera el paquete de traspaso.
5. Pasar el paquete a **Claude Code** para construir la pantalla real en React Native.

---

## Reglas visuales para la IA

La IA debe seguir estas reglas en toda sesión de diseño o vibecoding:

1. **Modo oscuro siempre** — fondo Carbón, no blanco.
2. **Sora únicamente** — nunca Poppins, Inter, Roboto u otra fuente.
3. **Verde Avanza solo para acciones principales** — nunca para decoración o información.
4. **Escala de 4px para espaciado** — nunca números arbitrarios.
5. **Íconos Tabler sobre círculo sólido** — nunca íconos sueltos sin el tratamiento de círculo.
6. **Bordes muy redondeados** — nunca esquinas cuadradas en botones o tarjetas.
7. **Una sola identidad visual** — no crear variantes de color por tipo de servicio.

---

*Documento generado de la sesión de planificación de Yavoy. Revisión: v1.0*
