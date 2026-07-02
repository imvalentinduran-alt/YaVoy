# Yavoy — Documentos de Planificación

App de ride-hailing para Rubio, Táchira, Venezuela.
Founder: solo. Stack: React Native + Expo + Firebase. Vibecoding con Claude Code.

---

## Estado de los documentos

| # | Documento | Estado |
|---|---|---|
| 01 | PRD — Requerimientos del Producto | ✅ Completo |
| 02 | Glosario / Modelo de Dominio | ✅ Completo |
| 03 | TRD — Requerimientos Técnicos | ✅ Completo |
| 04 | UI/UX — Diseño e identidad visual | ✅ Completo |
| 05 | App Flow — Pantallas y navegación | 🔄 En progreso (Flujos 1 y 2 completos) |
| 06 | Esquema del Backend | ⏳ Pendiente |
| 07 | Estructura y Convenciones | ⏳ Pendiente |
| 08 | Config / Ciberseguridad | ⏳ Pendiente |
| 09 | Plan de Implementación | ⏳ Pendiente |
| 00 | CLAUDE.md (índice maestro) | ⏳ Pendiente (se escribe al final) |

---

## Reglas de máxima prioridad (leer siempre)

1. **NUNCA exponer las API keys** — ni en el código, ni en commits, ni en logs.
2. **VALIDAR todo lo que escribe el usuario. NUNCA confiar en el cliente.**
3. **USAR rate limits** — contra scripts, bots y abuso de volumen.
4. **PROTEGER la base de datos** — cada usuario ve solo lo que le corresponde.
5. **Las pantallas NUNCA llaman a Firebase directamente** — siempre a través de los servicios (`*Service.ts`).
6. **El precio SIEMPRE se calcula en el servidor** (Cloud Function) — nunca en el teléfono.
7. **El código en inglés, la interfaz en español** — sin excepción.
8. **Estructura por feature** — nunca carpetas `screens/` o `components/` sueltas.

---

## Cómo usar estos documentos

Antes de empezar cualquier sesión de vibecoding, leer en orden:

1. Este README.
2. `02-glosario.md` — para los términos exactos del código.
3. `03-trd.md` — para las reglas de arquitectura y seguridad.
4. El documento específico del feature que se va a construir.

---

*Yavoy — v1 en construcción.*
