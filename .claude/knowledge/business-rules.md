# Business Rules - Habit Tracker

**Target: ~2000 tokens | Optimized for AI agents**

Esta es la fuente de verdad para todas las reglas de negocio de la aplicación. Cualquier implementación debe cumplir estas reglas al 100%.

---

## 1. Hábitos (Habit Entity)

### 1.1 Definición
Un hábito es una acción que el usuario desea realizar de forma recurrente para mejorar su vida.

### 1.2 Campos Obligatorios
- **nombre** (string, 1-100 caracteres)
  - No puede estar vacío ni contener solo espacios
  - Debe ser único por usuario (case-insensitive)
  - Ejemplos: "Hacer ejercicio", "Meditar 10 minutos", "Beber 2L de agua"

- **frecuencia** (enum: 'daily' | 'weekly' | 'monthly')
  - Define el patrón de repetición del hábito
  - No se puede cambiar después de crear el hábito (solo eliminando y recreando)

- **tag** (obligatorio, relación con Tag entity)
  - Cada hábito debe tener exactamente UN tag
  - El tag debe existir en la base de datos antes de asignar
  - Ejemplos: "Salud", "Productividad", "Desarrollo Personal", "Finanzas"

### 1.3 Campos Opcionales
- **descripción** (string, 0-500 caracteres)
  - Puede ser null o vacío
  - Sirve para aclarar el propósito o contexto del hábito
  - Ejemplo: "Sesión de 30 minutos en el gimnasio o rutina en casa"

- **meta** (string, 0-200 caracteres)
  - Puede ser null o vacío
  - Define criterios de cumplimiento específicos
  - Ejemplos: "Correr 5km", "Leer 20 páginas", "Ahorrar $50"

### 1.4 Restricciones
- ✅ Solo hábitos positivos (cosas que QUIERES hacer)
- ❌ NO hábitos negativos (cosas que NO quieres hacer)
  - Razón: enfoque psicológico en acciones constructivas
  - Alternativa: reformular en positivo ("Meditar 5 min" en vez de "No revisar redes sociales")

- ✅ Sin límite de cantidad de hábitos por usuario
- ✅ Usuario puede crear, editar (nombre, descripción, meta, tag), o eliminar hábitos en cualquier momento
- ⚠️ Editar un hábito NO afecta registros históricos (los registros mantienen snapshot del estado al momento de registro)

### 1.5 Estados del Hábito
- **activo**: aparece en la lista diaria, se puede registrar
- **archivado** (futuro): no aparece en lista diaria, pero mantiene historial
  - MVP: no se implementa archivado, solo eliminación

---

## 2. Registro Diario (DailyRecord Entity)

### 2.1 Concepto
Cada día, el usuario marca el estado de cumplimiento de cada hábito activo según su frecuencia.

### 2.2 Restricción Temporal CRÍTICA
**REGLA DE ORO: Solo se puede registrar el día ACTUAL (hoy)**

- ✅ Registrar hoy: permitido
- ❌ Registrar días pasados: NO permitido
- ❌ Registrar días futuros: NO permitido

**Razones:**
- Fomenta compromiso diario sin procrastinación
- Evita "rellenar" historial artificialmente
- Diseño minimalista sin complejidad de edición histórica

**Excepciones:** Ninguna. Esta regla NO tiene excepciones en MVP.

### 2.3 Estados de Cumplimiento

Cada registro tiene uno de 3 estados:

1. **completed** (Cumplido)
   - El hábito se completó según la definición del usuario
   - Color: verde (GitHub-style heatmap)
   - Puntos para rachas: +1

2. **partial** (Parcial)
   - El hábito se intentó pero no se completó al 100%
   - Ejemplo: "Correr 5km" → corrió 2km
   - Color: amarillo/naranja (GitHub-style heatmap)
   - Puntos para rachas: NO suma (+0)
   - Nota: "parcial" NO rompe rachas, pero tampoco las incrementa

3. **not_completed** (No cumplido)
   - El hábito no se realizó
   - Color: gris claro (GitHub-style heatmap)
   - Puntos para rachas: rompe racha

### 2.4 Registro Según Frecuencia

#### Frecuencia DIARIA
- **Comportamiento:** Aparece TODOS los días
- **Registro:** Debe marcarse cada día
- **Deadline:** Medianoche (00:00) - si no se marca, se considera "not_completed"
- **Ejemplos:** "Meditar", "Hacer ejercicio", "Beber agua"

#### Frecuencia SEMANAL
- **Comportamiento:** Aparece toda la semana (Lunes a Domingo)
- **Registro:** Usuario puede marcar CUALQUIER día de la semana
- **Deadline:** Domingo 23:59 - si la semana termina sin marcarse, se considera "not_completed"
- **Reset:** Nuevo ciclo comienza el Lunes
- **Ejemplo:** "Limpiar casa" → usuario puede hacerlo Miércoles, Jueves o Domingo
- **Flexibilidad:** Usuario elige qué día de la semana completarlo

**Regla especial:**
- Si se marca "completed" el Martes, puede:
  - Dejar el registro así (cuenta como 1 cumplimiento semanal)
  - Cambiar estado antes de que termine la semana
- Si se marca "completed" múltiples veces en la misma semana:
  - Solo se cuenta como 1 cumplimiento
  - No hay "bonus" por hacerlo más veces

#### Frecuencia MENSUAL
- **Comportamiento:** Aparece todo el mes (día 1 al último día)
- **Registro:** Usuario puede marcar CUALQUIER día del mes
- **Deadline:** Último día del mes 23:59 - si el mes termina sin marcarse, se considera "not_completed"
- **Reset:** Nuevo ciclo comienza el día 1 del siguiente mes
- **Ejemplo:** "Revisar finanzas personales" → usuario puede hacerlo el día 10, 20 o 30
- **Flexibilidad:** Usuario elige qué día del mes completarlo

**Regla especial:** Igual que semanal (solo cuenta 1 vez por mes)

### 2.5 Notas Opcionales
- Cada registro puede tener una **nota** (string, 0-1000 caracteres)
- Opcional, puede ser null o vacío
- Útil para contexto: "Corrí solo 2km porque llovía", "Completado en la mañana"
- Las notas se muestran en el detalle del día/hábito

### 2.6 Cambios de Estado
- ✅ Usuario puede cambiar el estado de un registro SOLO durante el día actual
- ✅ Ejemplo: marcar "completed" a las 8am, cambiar a "partial" a las 6pm
- ❌ Una vez que pasa medianoche, el registro es inmutable
- ❌ No se pueden editar registros de días anteriores (consistente con regla 2.2)

---

## 3. Frecuencias - Detalles de Implementación

### 3.1 Cálculo de "Día Actual"
- **Timezone:** Usar timezone local del usuario (browser timezone)
- **Medianoche:** Reset ocurre a las 00:00:00 local time
- **Consideración:** Si usuario viaja entre timezones, usar el timezone donde se encuentra actualmente

### 3.2 Lógica de Hábitos Visibles Hoy

**Pseudo-código:**
```
function getHabitsForToday(allHabits, today):
  visible = []

  for habit in allHabits:
    if habit.frequency == 'daily':
      visible.push(habit)

    elif habit.frequency == 'weekly':
      # Mostrar de Lunes a Domingo
      visible.push(habit)

    elif habit.frequency == 'monthly':
      # Mostrar del día 1 al último día del mes
      visible.push(habit)

  return visible
```

### 3.3 Estado Automático de "No Cumplido"

**Para hábitos diarios:**
- Si a medianoche no hay registro → crear DailyRecord con estado "not_completed"

**Para hábitos semanales:**
- Si llega Lunes 00:00 y la semana anterior no tiene registro → crear DailyRecord con estado "not_completed" para el Domingo

**Para hábitos mensuales:**
- Si llega día 1 00:00 del nuevo mes y el mes anterior no tiene registro → crear DailyRecord con estado "not_completed" para el último día del mes anterior

**Implementación:** Background job/cron que corre a medianoche.

---

## 4. Sistema de Rachas (Streaks)

### 4.1 Definición
Una racha es la cantidad de períodos consecutivos donde el hábito fue marcado como "completed".

### 4.2 Cálculo por Frecuencia

**Diaria:**
- Racha actual: días consecutivos con "completed"
- Se rompe con: "not_completed"
- "partial" NO rompe racha, pero NO incrementa contador

**Semanal:**
- Racha actual: semanas consecutivas con al menos 1 "completed"
- Se rompe si: semana termina sin ningún "completed"

**Mensual:**
- Racha actual: meses consecutivos con al menos 1 "completed"
- Se rompe si: mes termina sin ningún "completed"

### 4.3 Métricas
- **Racha actual** (current streak): contador en tiempo real
- **Mejor racha** (best streak): máximo histórico
- Ambas se calculan on-demand (no se almacenan, se derivan de DailyRecords)

### 4.4 Ejemplo Racha Diaria
```
Día 1: completed → racha = 1
Día 2: completed → racha = 2
Día 3: partial → racha = 2 (NO incrementa, NO rompe)
Día 4: completed → racha = 3
Día 5: not_completed → racha = 0 (se rompe)
Día 6: completed → racha = 1 (inicia nueva racha)
```

---

## 5. Visualizaciones y Estadísticas

### 5.1 Calendario Estilo GitHub (Heatmap)

**Descripción:**
- Grid de cuadrados donde cada cuadrado = 1 día
- Muestra últimos 365 días (o desde creación del hábito)
- Color basado en estado:
  - Verde oscuro: completed
  - Verde claro: completed con menor intensidad (si hay escala de intensidad)
  - Amarillo/Naranja: partial
  - Gris claro: not_completed
  - Blanco/Sin color: día sin registro (futuro o antes de crear hábito)

**Interacción:**
- Hover: muestra tooltip con fecha, estado, y nota
- Click: abre detalle del día

### 5.2 Porcentaje de Cumplimiento

**Fórmula:**
```
completion_rate = (total_completed / total_expected) * 100

donde:
  total_completed = cantidad de registros "completed"
  total_expected = cantidad de días desde creación del hábito hasta hoy
```

**Consideraciones:**
- Solo cuenta "completed" como éxito
- "partial" NO cuenta como éxito (para mantener estándar alto)
- Se calcula para diferentes períodos: última semana, último mes, total

### 5.3 Gráficas de Tendencia

**Tipo:** Línea de tiempo
**Eje X:** Tiempo (días/semanas/meses)
**Eje Y:** Porcentaje de cumplimiento

**Períodos disponibles:**
- Últimos 7 días
- Últimos 30 días
- Últimos 90 días
- Todo el tiempo

### 5.4 Estadísticas por Tag

**Agrupación:** Mostrar métricas agregadas por tag/categoría
- Total de hábitos por tag
- Porcentaje de cumplimiento promedio por tag
- Tags con mejor desempeño
- Tags que necesitan atención

**Ejemplo:**
```
Tag: Salud
  - 3 hábitos activos
  - 85% cumplimiento promedio
  - Mejor racha: 45 días (Meditar)

Tag: Productividad
  - 2 hábitos activos
  - 60% cumplimiento promedio
  - Mejor racha: 12 días (Planificar día)
```

---

## 6. Tags/Categorías

### 6.1 Propósito
Organizar hábitos en grupos temáticos para análisis y visualización.

### 6.2 Características
- **Campos:**
  - nombre (string, 1-50 caracteres, único por usuario)
  - color (hex code, para UI diferenciación)
  - icono (opcional, emoji o icon name)

- **Predefinidos:** App puede sugerir tags comunes:
  - Salud, Productividad, Desarrollo Personal, Finanzas, Relaciones, Creatividad, Educación

- **Custom:** Usuario puede crear tags propios

### 6.3 Restricciones
- ✅ Un hábito debe tener exactamente 1 tag
- ✅ Un tag puede tener múltiples hábitos
- ❌ No se pueden eliminar tags que tienen hábitos asignados
  - Solución: reasignar hábitos a otro tag primero

---

## 7. Usuario (User Entity)

### 7.1 Campos
- **id** (UUID)
- **email** (único)
- **nombre** (display name)
- **timezone** (para cálculos de medianoche)
- **fecha de creación**

### 7.2 Preferencias (futuro)
- Notificaciones (push, email)
- Hora recordatorio (ej: 8pm "¿Ya registraste tus hábitos?")
- Tema (light/dark)
- Idioma (es/en)

---

## 8. Reglas de Validación (Quick Reference)

**Crear Hábito:**
- ✅ nombre: no vacío, único, max 100 chars
- ✅ frecuencia: 'daily' | 'weekly' | 'monthly'
- ✅ tag: debe existir y pertenecer al usuario
- ✅ descripción: opcional, max 500 chars
- ✅ meta: opcional, max 200 chars

**Registrar Cumplimiento:**
- ✅ Solo día actual
- ✅ Estado: 'completed' | 'partial' | 'not_completed'
- ✅ Nota: opcional, max 1000 chars
- ✅ Puede cambiar estado durante el día actual
- ❌ No puede registrar días pasados/futuros

**Eliminar Hábito:**
- ✅ Usuario puede eliminar en cualquier momento
- ⚠️ Eliminar hábito elimina TODOS sus registros históricos
- 🚨 Mostrar confirmación antes de eliminar

---

## 9. Edge Cases y Consideraciones

### 9.1 Hábito Creado a Mitad de Semana/Mes
**Escenario:** Usuario crea hábito semanal el Miércoles.
**Comportamiento:**
- Aparece desde el Miércoles hasta el Domingo
- Si no se marca antes del Domingo 23:59 → "not_completed"
- Primera semana completa comienza el siguiente Lunes

**Escenario:** Usuario crea hábito mensual el día 15.
**Comportamiento:**
- Aparece desde el día 15 hasta fin de mes
- Si no se marca antes del último día 23:59 → "not_completed"
- Primer mes completo comienza el día 1 del siguiente mes

### 9.2 Cambio de Timezone
**Problema:** Usuario viaja de timezone UTC-5 a UTC+2
**Solución:**
- Usar browser timezone actual
- Si cambia timezone durante el día: respetar el timezone donde marcó el registro
- Medianoche se calcula según timezone actual del browser

### 9.3 Múltiples Registros en Mismo Día (hábitos semanales/mensuales)
**Escenario:** Hábito semanal "Limpiar casa" marcado como "completed" el Martes, usuario intenta marcar nuevamente el Jueves.
**Comportamiento:**
- Permitir cambio de estado (sobrescribe registro del Martes)
- Solo guardar 1 registro por hábito por período (semana/mes)

---

## 10. Prioridades para MVP

**MUST HAVE (P0):**
- Crear/editar/eliminar hábitos
- Registro solo de día actual
- 3 estados: completed, partial, not_completed
- Frecuencias: daily, weekly, monthly
- Tags obligatorios (con sugerencias predefinidas)
- Calendario tipo GitHub heatmap
- Rachas (actual + mejor)
- Porcentaje de cumplimiento

**NICE TO HAVE (P1 - post-MVP):**
- Notas en registros
- Gráficas de tendencia
- Estadísticas por tag
- Archivar hábitos (sin eliminar historial)
- Notificaciones/recordatorios

**FUTURE (P2):**
- Exportar datos (CSV/JSON)
- Compartir hábitos con otros usuarios
- Hábitos colaborativos
- Gamificación (badges, logros)
- Integración con wearables

---

**Fin del documento. Token count: ~2000**
