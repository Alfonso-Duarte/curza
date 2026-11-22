# PRD · <curza>

> Documento de requisitos del producto (Product Requirements Document).
> Complétalo en el Sprint 0 (historia H0.3) y pide la aprobación del profesor en un Issue.
> Cada requisito funcional (RF) se convierte en un Issue con la plantilla "Historia de usuario".

| Campo | Valor |
| --- | --- |
| Alumno | Alfonso Duarte |
| Proyecto n.º | 42 |
| Sitio de referencia | Platzi |
| Compañero revisor | |
| URL desplegada | (se completa en el Sprint 5) |

## 1. Problema

Los estudiantes autodidactas que consumen cursos en línea suelen enfrentarse a catálogos planos y desordenados, lo que genera frustración al intentar tomar cursos avanzados sin dominar primero los fundamentos necesarios ni tener visibilidad clara de cuánto les falta para completar un perfil profesional.

## 2. Usuarios

- **Estudiante:** Explora rutas de aprendizaje, visualiza el grafo de dependencias entre cursos, identifica qué cursos tiene habilitados o bloqueados, marca cursos como completados y monitorea su porcentaje de progreso por ruta.
- **Administrador / Gestor académico:** Crea cursos, arma rutas de aprendizaje y define relaciones de prerrequisito entre cursos garantizando que no existan dependencias circulares.

## 3. Objetivo y métrica de éxito

- **Objetivo:** Guiar al estudiante mediante rutas estructuradas con dependencias explícitas para que siempre sepa qué curso puede estudiar a continuación y cuánto ha avanzado hacia su meta profesional.
- **Métricas de éxito:**
  - Un estudiante identifica sus cursos disponibles y registra la aprobación de un curso (actualizando su porcentaje de progreso y desbloqueando los siguientes nodos de la ruta) en **menos de 5 segundos**.
  - El sistema valida y rechaza el **100 %** de los intentos de crear prerrequisitos circulares antes de persistirlos en PostgreSQL.
  - El cálculo del orden topológico y el renderizado de rutas con hasta **10.000 registros** responde en **menos de 300 ms** sin bloquear la interfaz.

## 4. Alcance del MVP

1. **Gestión de rutas de aprendizaje (`paths`):** Creación, listado y consulta detallada de rutas temáticas (estilo escuelas/carreras de Platzi) organizadas por niveles topológicos.
2. **Gestión de cursos (`courses`):** Administración del catálogo de cursos (título, descripción, duración estimada y asignación a una o más rutas).
3. **Grafo de prerrequisitos y orden topológico (`prerequisites`):** Definición de dependencias dirigidas entre cursos con detección de ciclos (DAG) y ordenamiento topológico por niveles mediante el algoritmo de Kahn.
4. **Inscripción y progreso por ruta (`enrollments`):** Inscripción de estudiantes en rutas de aprendizaje y cálculo en tiempo real del porcentaje de avance según los cursos completados.
5. **Registro de cursos aprobados y desbloqueo (`completions`):** Marcado de cursos como completados (solo cuando sus prerrequisitos directos estén aprobados) y desbloqueo automático de los nodos dependientes.
6. **Gestión de usuarios y roles (`users`):** Diferenciación de permisos entre el perfil de estudiante (consulta y avance) y el perfil de gestor académico (configuración de rutas, cursos y prerrequisitos).

### Funcionalidades → carpetas de `src/` (screaming architecture)

Cada funcionalidad del alcance será una carpeta en `src/`. Nómbralas en inglés y en plural.

| Funcionalidad | Carpeta |
| --- | --- |
| Rutas de aprendizaje | `src/paths` |
| Catálogo y gestión de cursos | `src/courses` |
| Grafo de prerrequisitos y orden topológico | `src/prerequisites` |
| Inscripciones y progreso por ruta | `src/enrollments` |
| Cursos completados y desbloqueo de nodos | `src/completions` |
| Usuarios y roles | `src/users` |

## 5. Fuera de alcance

- Reproductor de video en streaming propio o alojamiento de archivos multimedia pesados.
- Pasarela de pagos, planes de suscripción o facturación.
- Emisión de certificados PDF con firma criptográfica.
- Foros de discusión en tiempo real, mensajería interna o sistema de calificaciones por exámenes.
- Autenticación externa mediante proveedores OAuth de terceros (Google/GitHub).

## 6. Requisitos funcionales

| ID | Requisito | Prioridad (alta, media, baja) |
| --- | --- | --- |
| RF-01 | Listar todas las rutas de aprendizaje disponibles mostrando cantidad de cursos y progreso del estudiante inscrito. | alta |
| RF-02 | Visualizar el detalle de una ruta ordenando sus cursos por niveles mediante ordenamiento topológico. | alta |
| RF-03 | Calcular y mostrar el estado de cada curso para el estudiante: `Completado`, `Disponible` o `Bloqueado` según sus prerrequisitos. | alta |
| RF-04 | Permitir al estudiante marcar un curso como completado únicamente si todos sus prerrequisitos directos están previamente completados. | alta |
| RF-05 | Recalcular automáticamente el porcentaje de progreso de la ruta y habilitar los cursos dependientes al aprobar un curso. | alta |
| RF-06 | Permitir al administrador crear y editar cursos y asociarlos a una o más rutas de aprendizaje. | media |
| RF-07 | Permitir al administrador agregar o eliminar relaciones de prerrequisito entre cursos, rechazando cualquier arista que genere un ciclo en el grafo. | alta |
| RF-08 | Filtrar y buscar cursos o rutas por nombre o estado de avance sin degradar el rendimiento con grandes volúmenes de datos. | media |
| RF-09 | Permitir deshacer o revertir el marcado reciente de un curso completado siempre que no tenga cursos posteriores ya aprobados. | baja |

## 7. Requisitos no funcionales

- Responsive, móvil primero (375 px y 1280 px).
- Lighthouse ≥ 90 en Accesibilidad y Buenas prácticas.
- 0 errores en la consola del navegador.
- Funciona con 10.000 registros sin congelarse.

## 8. Modelo de datos

```mermaid
erDiagram
  USER ||--o{ ENROLLMENT : se_inscribe
  USER ||--o{ COURSE_COMPLETION : aprueba
  LEARNING_PATH ||--o{ ENROLLMENT : tiene
  LEARNING_PATH ||--|{ PATH_COURSE : agrupa
  COURSE ||--o{ PATH_COURSE : pertenece_a
  COURSE ||--o{ COURSE_COMPLETION : es_aprobado
  COURSE ||--o{ PREREQUISITE : requiere
  COURSE ||--o{ PREREQUISITE : habilita

  USER {
    int id PK
    text nombre
    text email UK
    text rol
    timestamptz creado_en
  }
  LEARNING_PATH {
    int id PK
    text titulo
    text slug UK
    text descripcion
    text nivel
    timestamptz creado_en
  }
  COURSE {
    int id PK
    text titulo
    text slug UK
    text descripcion
    int duracion_min
    timestamptz creado_en
  }
  PATH_COURSE {
    int id PK
    int path_id FK
    int course_id FK
    boolean es_obligatorio
  }
  PREREQUISITE {
    int id PK
    int course_id FK
    int prerequisite_course_id FK
  }
  ENROLLMENT {
    int id PK
    int user_id FK
    int path_id FK
    timestamptz inscrito_en
  }
  COURSE_COMPLETION {
    int id PK
    int user_id FK
    int course_id FK
    timestamptz completado_en
  }

## 9. Reto técnico distintivo

**Grafo de prerrequisitos con ordenamiento topológico y progreso por ruta de aprendizaje:**

1. **Representación como DAG (Grafo Dirigido Acíclico):** Las relaciones de `PREREQUISITE` dentro de una ruta se construyen en memoria como una lista de adyacencia acompañada de un mapa de grados de entrada (*in-degree*) por cada curso.
2. **Validación anti-ciclos en tiempo de escritura:** Antes de registrar una nueva dependencia `A → B` (`A` es prerrequisito de `B`), se ejecuta una búsqueda en profundidad o anchura (DFS/BFS) partiendo desde `B`. Si es posible alcanzar `A`, la operación se rechaza con un error de dominio, garantizando matemáticamente que el grafo siempre admita un orden topológico válido.
3. **Ordenamiento topológico por niveles (Algoritmo de Kahn):** Se procesan en una cola los cursos con grado de entrada `0` (Nivel 0: fundamentos sin prerrequisitos) y se reducen iterativamente los grados de entrada de sus nodos adyacentes para agruparlos por niveles (`Nivel 1`, `Nivel 2`, ..., `Nivel N`) en complejidad lineal `O(V + E)`. Esto determina tanto la secuencia válida de estudio como la disposición visual de la ruta.
4. **Cálculo de estado y progreso dinámico:** Los cursos aprobados del estudiante (`COURSE_COMPLETION`) se indexan en una tabla hash / conjunto para consultas en `O(1)`. Cada nodo de la ruta se clasifica automáticamente en `Completado`, `Disponible` (100 % de sus prerrequisitos directos aprobados) o `Bloqueado`, y el avance de la ruta se obtiene mediante:

`Progreso (%) = (Cursos completados de la ruta / Total de cursos de la ruta) × 100`

## 10. Riesgos y preguntas abiertas

- **Riesgo · Edición de grafos con estudiantes activos:** Si un administrador agrega un nuevo prerrequisito a un curso que algunos estudiantes ya habían aprobado, se debe definir si esos estudiantes conservan su estado `Completado` (regla de derechos adquiridos) o si el nodo pasa a requerir el nuevo prerrequisito.
- **Riesgo · Visualización en pantallas móviles (375 px):** Un grafo con múltiples ramificaciones horizontales puede generar desplazamiento incómodo en móvil; se mitigará agrupando los cursos por niveles topológicos colapsables con indicadores claros de conexión.
- **Riesgo · Rendimiento con 10.000 registros:** Calcular el orden topológico y evaluar estados de desbloqueo mediante consultas repetitivas podría ralentizar la respuesta; se mitigará indexando llaves foráneas en PostgreSQL y ejecutando el algoritmo de Kahn en complejidad lineal `O(V + E)` con estructuras de búsqueda en `O(1)`.
- **Pregunta abierta · Prerrequisitos transversales entre rutas:** Si un estudiante completa un curso base (por ejemplo, *Lógica de Programación*) dentro de la ruta *Desarrollo Frontend*, ¿debe reflejarse automáticamente como aprobado al inscribirse en la ruta *Desarrollo Backend*? *(Propuesta inicial: sí, dado que `COURSE_COMPLETION` se asocia al usuario y al curso).*
- **Pregunta abierta · Reversión de progreso:** ¿Debe permitirse desmarcar un curso como completado si el estudiante ya aprobó cursos posteriores que dependían de él, o debe exigirse desmarcar primero los nodos dependientes?
