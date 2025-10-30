Quiero construir un **Habit Tracker minimalista** enfocado en simplicidad y trazabilidad efectiva de hábitos mediante visualizaciones gráficas claras.

## Usuarios
**Usuario único (sin roles diferenciados):**
- Persona dedicada al seguimiento de sus hábitos personales
- Gestiona aproximadamente 5 hábitos simultáneos de forma activa
- Requiere compromiso diario con el registro
- Busca simplicidad y evita funcionalidades innecesarias

**Capacidades del usuario:**
- Crear, editar y eliminar hábitos ilimitados
- Registrar cumplimiento diario (solo día actual)
- Categorizar hábitos con tags predefinidos o personalizados
- Visualizar históricos y estadísticas detalladas
- Comparar períodos de tiempo
- Ver progreso individual por hábito

## Características Clave

### 1. Gestión de Hábitos
- **Creación de hábitos** con los siguientes campos:
  - Nombre (obligatorio)
  - Descripción (opcional)
  - Frecuencia: Diaria, Semanal o Mensual (obligatorio)
  - Meta específica (opcional, ej: "30 minutos", "10 páginas")
  - Tag/Categoría (obligatorio, uno por hábito)
- **Edición y eliminación** de hábitos en cualquier momento
- **Sin límite** de hábitos activos

### 2. Registro Diario (Funcionalidad Core)
- **Vista principal:** Lista de hábitos aplicables para el día actual
- **Interfaz de registro:** Botones en línea por cada hábito con 4 estados:
  - Sin registrar (estado inicial)
  - ✅ Cumplido (100%)
  - ◐ Parcial (cumplido parcialmente)
  - ❌ No cumplido
- **Notas opcionales:** Campo de texto por hábito por día
- **Restricción temporal:** Solo se puede registrar el día actual (no pasado ni futuro)
- **Reset automático:** Medianoche (00:00)

### 3. Sistema de Categorización (Tags)
- **Tags predefinidos:** Salud, Ejercicio, Productividad, Aprendizaje, Social, Finanzas, Bienestar Mental, Hobbies
- **Tags personalizados:** Usuario puede crear ilimitados
- **Un tag por hábito** (no múltiples)
- **Colores asociados:** Cada tag tiene un color (predefinido pero personalizable)
- **Usos:**
  - Organización visual
  - Filtrado de hábitos
  - Estadísticas agrupadas por categoría

### 4. Históricos y Visualizaciones
- **Calendario estilo GitHub:**
  - Cajitas con escala de saturación de color
  - Más saturado = más hábitos cumplidos ese día
  - Menos saturado/vacío = pocos o ningún hábito cumplido
  
- **Sistema de Rachas:**
  - Racha actual (días consecutivos)
  - Mejor racha histórica
  
- **Métricas de cumplimiento:**
  - Porcentaje de cumplimiento global
  - Semanas con mayor % cumplimiento
  - Semanas con menor % cumplimiento
  
- **Gráficas de tendencia:**
  - Líneas mostrando evolución temporal
  - Vista de todos los hábitos o los más importantes
  
- **Estadísticas por categoría:**
  - Agrupación y análisis por tags
  
- **Períodos visualizables:**
  - Últimos 7 días
  - Últimos 30 días
  - Último año
  - Todo el histórico
  - Rango personalizado (fechas seleccionadas por usuario)
  
- **Vista por hábito individual:**
  - Histórico detallado de un hábito específico
  - Todas las métricas aplicadas a ese hábito

### 5. Comparaciones Temporales
- Mes actual vs mes anterior
- Semana actual vs semana anterior
- Rangos personalizados seleccionables por el usuario

## Reglas de Negocio

### Reglas de Frecuencia
1. **Hábitos Diarios:**
   - Aparecen en la lista TODOS los días
   - Deben registrarse cada día
   - Si no se registran, se marcan automáticamente como "no cumplido"

2. **Hábitos Semanales:**
   - Aparecen en la lista TODOS los días de la semana
   - Usuario puede marcarlos CUALQUIER día de la semana
   - Solo necesitan cumplirse UNA vez durante la semana
   - Si la semana termina (domingo 23:59) sin registro, se marca como "no cumplido" para esa semana
   - Nueva semana = nueva oportunidad

3. **Hábitos Mensuales:**
   - Aparecen en la lista TODOS los días del mes
   - Usuario puede marcarlos CUALQUIER día del mes
   - Solo necesitan cumplirse UNA vez durante el mes
   - Si el mes termina (último día 23:59) sin registro, se marca como "no cumplido" para ese mes
   - Nuevo mes = nueva oportunidad

### Reglas de Registro
4. **Restricción temporal estricta:**
   - SOLO se puede registrar el día actual
   - NO se permiten registros retroactivos (días pasados)
   - NO se permiten registros futuros
   - Reset automático a las 00:00 (medianoche)

5. **Estados de cumplimiento:**
   - Cada hábito cada día tiene exactamente uno de 4 estados
   - Estado inicial: "Sin registrar"
   - Usuario puede cambiar entre: Cumplido, Parcial, No cumplido
   - Una vez que pasa la medianoche, los "sin registrar" se convierten automáticamente en "no cumplido"

6. **Notas opcionales:**
   - Cada registro puede tener una nota de texto
   - Las notas son completamente opcionales
   - Se almacenan por hábito por día

### Reglas de Tags
7. **Asignación de tags:**
   - Cada hábito DEBE tener exactamente UN tag
   - No se permite crear hábitos sin tag
   - No se permiten múltiples tags por hábito

8. **Gestión de tags personalizados:**
   - Usuario puede crear tags ilimitados
   - Usuario puede editar nombre y color de tags personalizados
   - Usuario puede eliminar tags personalizados
   - **Regla crítica:** Si se elimina un tag, todos los hábitos con ese tag DEBEN reasignarse a otro tag existente (no pueden quedar sin tag)
   - Tags predefinidos NO se pueden eliminar

### Reglas de Cálculo de Estadísticas
9. **Rachas (Streaks):**
   - **Racha actual:** Cuenta días CONSECUTIVOS con todos los hábitos aplicables del día cumplidos (completo o parcial cuenta, solo "no cumplido" rompe racha)
   - **Mejor racha:** Máximo histórico de días consecutivos
   - Racha se rompe si CUALQUIER hábito aplicable del día queda como "no cumplido"
   - Hábitos no aplicables al día (ej: semanal ya cumplido) no afectan la racha

10. **Porcentaje de cumplimiento:**
    - **Global:** (Total hábitos cumplidos + parciales) / (Total oportunidades de cumplimiento) × 100
    - **Por hábito:** (Días cumplidos + parciales del hábito) / (Días aplicables del hábito) × 100
    - **Por categoría:** Promedio de % de cumplimiento de todos los hábitos de esa categoría
    - "Cumplido" y "Parcial" cuentan positivamente
    - "No cumplido" cuenta negativamente
    - "Sin registrar" (antes de medianoche) no cuenta aún

11. **Visualización de calendario GitHub:**
    - Cada cajita representa un día
    - Color base único (ej: verde)
    - Saturación basada en: (Hábitos cumplidos ese día / Total hábitos aplicables ese día)
    - 100% cumplimiento = color más saturado
    - 0% cumplimiento = cajita vacía/gris claro
    - Valores intermedios = saturaciones intermedias

### Reglas de Visualización
12. **Filtrado por tags:**
    - Usuario puede filtrar vista principal para ver solo hábitos de un tag específico
    - Filtrado afecta: lista diaria, históricos, estadísticas
    - Siempre debe existir opción "Ver todos"

13. **Comparaciones temporales:**
    - Solo se pueden comparar períodos completos (semana completa vs semana completa, mes vs mes)
    - Comparación muestra diferencia porcentual
    - Usuario puede seleccionar períodos arbitrarios para comparar

## Stack Tecnológico
**Pendiente de definir** - Solicitar recomendaciones en el documento

## Consideraciones de Diseño
- **Interfaz minimalista:** Evitar elementos innecesarios, enfoque en funcionalidad core
- **Mobile-first:** Optimizado para uso en dispositivos móviles
- **Controles táctiles:** Todos los elementos deben ser fáciles de usar con touch
- **Paleta de colores:** Limpia, con uso intencional de color para tags y visualizaciones
- **Performance:** Carga rápida, ideal para uso diario rápido

## Alcance MVP vs Roadmap
**MVP (Must-have):**
- Todo lo descrito arriba en características clave

**Roadmap (Nice-to-have futuro):**
- Notificaciones/recordatorios
- Sincronización multi-dispositivo
- Exportar datos
- Modo oscuro
- Widgets
- Compartir progreso
- Motivación/quotes
- Respaldo automático en la nube
