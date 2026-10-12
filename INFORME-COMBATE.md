# Informe: cómo combatir los 4 puntos críticos

> Nota: este repo (`haceronline/hola`) solo contiene un README "hello-world". No tiene `src/`, `NOW.md` ni los ADRs citados, así que este informe se basa en tu diagnóstico, no en una lectura del código. Para auditar con datos reales, añade el repo de la app a la sesión.

## Diagnóstico de fondo

Los 4 puntos son un solo problema: **optimizas lo que controlas (docs, CSS, esquema) y evitas lo que te expone (usuario real, Postgres real, decisiones irreversibles)**. La cura no es "trabajar más" sino **poner contacto con la realidad en el camino crítico**.

Regla única para las próximas 3 semanas: **nada se hace si no acerca al piloto a usar la app en producción.**

---

## 1. Más burocracia que producto

**Acción (1 tarde):**
- Congela todo: ningún doc, agente, skill o prompt nuevo hasta que el piloto esté en producción.
- Fusiona `WORKFLOW.md`, `CLOUD-BUDGET.md`, `ROADMAP-APP.md`, `MIGRATION.md`, `CAPABILITIES.md` en **un solo `NOW.md`** de máximo 1 pantalla: objetivo, próximas 3 tareas, bloqueos con fecha.
- Mueve auditoría, ADRs y prompts no usados a `docs/archive/`. No los borres; sácalos de la vista.
- Agentes/skills: quédate con **1 agente y las skills que usaste en los últimos 7 días**. El resto, archivo.

**Métrica:** líneas de docs nuevas por semana ≈ 0; commits a `src/` > commits a docs.

## 2. CSS pesa más que el backend

**Acción:**
- Congelación de UI: cero commits de estilo salvo bugs que bloqueen al piloto.
- Tema oscuro: déjalo como está, no lo mantengas. Si algo se rompe, se quita.
- No refactorices `styles.css` ahora (sería otra forma de procrastinar). Solo una regla: **CSS nuevo = 0** hasta producción.
- Los textos de botones ("Nueva semana") se deciden con el piloto, no en commits.

**Métrica:** diff de `styles.css` = 0 líneas hasta el go-live.

## 3. Búnker de papel (multi-tenant sin Postgres real)

Este es el riesgo **técnico** más grande: RLS forzada + FKs compuestas + triggers jamás probados en Supabase. PGlite no garantiza el mismo comportamiento (roles, `auth.uid()`, políticas, extensiones, pooler).

**Acción (prioridad #1 de código, 1–2 días):**
1. Crear proyecto Supabase (plan free alcanza) y aplicar migraciones tal cual. Anotar cada error.
2. Test de humo con **dos tenants reales**: usuario A no puede leer/escribir datos de B (select, insert, update, delete). Esto es lo único que importa de RLS.
3. Probar los triggers de inmutabilidad y las FKs compuestas con un flujo real del piloto, no con fixtures.
4. Meter esos tests en CI contra Postgres real (contenedor `postgres` en GitHub Actions), no solo PGlite.
5. Producción para **1 tenant** (tu piloto). Los otros 3 tenants no existen hasta que alguien pague o pida.

**Si algo del blindaje estorba para salir, simplifícalo** (p. ej. quitar un trigger) y anótalo como deuda. Salir > ser perfecto.

**Métrica:** fecha del primer login del piloto en producción.

## 4. Tú eres el cuello de botella

**Acción hoy, timebox de 30 minutos, sin documentar más:**
| Bloqueo | Decisión por defecto (cámbiala solo si tienes razón concreta) |
|---|---|
| D2 subdominio | `app.<tudominio>` — decidido. |
| D3 correos | Correo transaccional de Supabase Auth / proveedor del plan free; plantillas por defecto. |
| D5 costos | Tope mensual fijo (p. ej. USD 25–50) con alerta. Dentro del tope no hay aprobación. |
| Aprobar gasto por sesión | **Eliminado.** Presupuesto mensual pre-aprobado reemplaza la aprobación por sesión. |
| Puntos abiertos del ADR | Se aceptan como están. Se revisan solo si producción los contradice. |

**Reglas para no volver a bloquear:**
- **Decisiones reversibles** (nombres, textos, colores, proveedores baratos): las toma el agente/quien implementa y te avisa. Tú solo vetas.
- **Decisiones irreversibles** (borrar datos, contratos, gasto fuera de tope): tú, con plazo máximo de 24 h; si no respondes, aplica el default.
- Un bloqueo en `NOW.md` sin fecha de resolución no es válido.

---

## Plan de 3 semanas

| Semana | Objetivo | Hecho cuando |
|---|---|---|
| 1 | Decisiones D2/D3/D5 cerradas; docs consolidados; Supabase real con migraciones aplicadas y test de aislamiento entre 2 tenants pasando | Test verde contra Postgres real |
| 2 | Deploy a producción en `app.<dominio>` para el piloto; correo de login funcionando | El piloto usa la app un día real |
| 3 | Solo arreglar lo que el piloto reporte | Lista de feedback del piloto atendida |

## Lo que NO hacer
- No escribir otro ADR, auditoría ni roadmap.
- No añadir agentes/skills para "acelerar".
- No pulir UI.
- No preparar tenants 2–4.

**Indicador único de éxito:** ¿el piloto usó la app en producción esta semana? Sí/No.
