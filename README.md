# Picross y sus Variaciones — README

## Resumen
Este proyecto modela y resuelve el **Picross** (también conocido como Nonograma) y sus variaciones avanzadas (**Color Picross** y **Mega Picross**) como **Problemas de Satisfacción de Restricciones (CSP)** usando Google OR-Tools en Python. El Picross es un puzzle lógico donde se deben pintar celdas en una cuadrícula siguiendo pistas numéricas para revelar una imagen oculta. El objetivo es implementar solvers eficientes que encuentren la configuración única que satisface todas las restricciones del puzzle.

## Objetivo general
Modelar y resolver diferentes variaciones del Picross mediante Programación por Restricciones (CP) con OR-Tools, demostrando cómo técnicas de CSP pueden abordar puzzles lógicos de complejidad creciente.

## Objetivos específicos
- Comprender el Picross como un Problema de Satisfacción de Restricciones y sus diferencias con problemas de optimización.
- Conocer el rol de los solvers de CP (propagación de restricciones, búsqueda y heurísticas) en la resolución de puzzles lógicos.
- Implementar modelos eficientes en OR-Tools que capturen las restricciones específicas de cada variación.
- Comparar la complejidad computacional entre Picross regular, Color Picross y Mega Picross.
- Generar visualizaciones gráficas de las soluciones encontradas.

## Descripción del problema

### Picross Regular
- **Cuadrícula:** n×m celdas (binarias: pintada o vacía)
- **Pistas:** Números en filas y columnas que indican grupos consecutivos de celdas pintadas
- **Restricciones:**
  - Cada grupo de celdas consecutivas debe estar separado por al menos una celda vacía
  - Todas las pistas de filas y columnas deben satisfacerse simultáneamente
- **Ejemplo resuelto:** Tablero 15×20 con 314 celdas pintadas

### Color Picross
- **Extensión:** Múltiples colores además de pintado/vacío
- **Pistas:** Incluyen tanto el número como el color de cada grupo
- **Complejidad adicional:** 
  - Grupos de diferentes colores deben estar separados por al menos una celda vacía
  - Aumenta exponencialmente el espacio de búsqueda
- **Ejemplo resuelto:** Tablero 10×10 con 3 colores

### Mega Picross
- **Innovación:** Pistas "mega" que abarcan **dos líneas adyacentes simultáneamente**
- **Restricción especial:** Los grupos mega deben formar **regiones conectadas** que pueden moverse entre ambas líneas (horizontal y verticalmente)
- **Complejidad:** Los patrones pueden formar formas no lineales (L, T, zigzag)
- **Ejemplo resuelto:** Tablero 5×5 con restricciones mega en filas 2-3 y columnas 0-1

## Metodología

### 1. Modelado como CSP
- **Variables:** Una variable por celda representando su estado (vacía/pintada o color)
- **Dominio:** 
  - Picross regular: {0, 1}
  - Color Picross: {0, 1, 2, ..., k} donde k = número de colores
  - Mega Picross: {0, 1} con restricciones adicionales de conectividad
- **Restricciones:** Expresadas mediante patrones válidos (`AddAllowedAssignments`)

### 2. Generación de patrones válidos
Cada variación implementa su propia lógica:

#### Picross Regular (`generate_line_patterns`)
- **Algoritmo:** Backtracking para generar todas las configuraciones válidas
- **Entrada:** Longitud de línea y secuencia de pistas
- **Salida:** Lista de tuplas representando patrones válidos
- **Ejemplo:** Para longitud 5 y pistas [1,1] → 6 patrones posibles

#### Color Picross (`generate_color_line_patterns`)
- **Extensión:** Similar al regular pero considerando colores
- **Restricción adicional:** Grupos de diferentes colores separados por vacíos
- **Complejidad:** O(n^k) donde k = número de grupos

#### Mega Picross (`generate_mega_patterns`)
- **Innovación clave:** Verifica conectividad de celdas entre dos líneas
- **Algoritmo:**
  1. Generar todas las combinaciones de k celdas en 2n posiciones
  2. Verificar conectividad mediante DFS (Depth-First Search)
  3. Filtrar solo patrones donde celdas forman componente conexo
- **Conectividad:** Las celdas pueden conectarse horizontal o verticalmente
- **Complejidad:** Exponencial - para 5 celdas entre 2 líneas de longitud 5 → 44 patrones válidos

### 3. Solver CP-SAT de OR-Tools
- **Propagación de restricciones:** Reduce el espacio de búsqueda eliminando valores inconsistentes
- **Búsqueda inteligente:** Emplea heurísticas para explorar el espacio de soluciones eficientemente
- **AddAllowedAssignments:** Restricción que fuerza variables a tomar valores de un conjunto predefinido de tuplas

### 4. Visualización
- **Matplotlib:** Generación de gráficos del tablero con:
  - Celdas pintadas/coloreadas según la solución
  - Pistas mostradas en los márgenes
  - Bordes y separaciones claras

## Resultados principales

### Picross Regular (15×20)
- **Complejidad:** 314 celdas pintadas de 300 totales
- **Tiempo de resolución:** < 1 segundo
- **Patrones generados:** ~1000 patrones válidos en total
- **Imagen revelada:** Castillo pixelado

### Color Picross (10×10)
- **Complejidad:** 3 colores (rojo, verde, azul)
- **Tiempo de resolución:** ~5 segundos
- **Espacio de búsqueda:** 3^100 ≈ 5×10^47 configuraciones teóricas
- **Imagen revelada:** Pingüino con patrones de color pixelado

### Mega Picross (5×5)
- **Restricciones mega:** 2 pares (filas 2-3, columnas 0-1)
- **Patrones mega generados:** 44 para columnas, 55 para filas
- **Tiempo de resolución:** < 1 segundo
- **Verificación:** Grupos conectados correctamente distribuidos entre líneas adyacentes
- **Imagen revelada:** Sombrero pixelado

## Comparación de complejidad

| Variación | Espacio de búsqueda | Patrones típicos/línea | Algoritmo clave | Complejidad temporal |
|-----------|-------------------|----------------------|----------------|---------------------|
| **Picross Regular** | 2^(n×m) | 1-100 | Backtracking | O(n × m × p) |
| **Color Picross** | k^(n×m) | 10-1000 | Backtracking con colores | O(n × m × p × k^g) |
| **Mega Picross** | 2^(n×m) × conectividad | 50-500 | Combinatoria + DFS | O(C(2n,k) × k) |

Donde:
- n, m = dimensiones del tablero
- p = número promedio de patrones por línea
- k = número de colores
- g = número de grupos
- C(2n,k) = combinaciones de k elementos en 2n posiciones

## Técnicas y algoritmos aplicados

1. **Backtracking:** Generación sistemática de patrones válidos
2. **Depth-First Search (DFS):** Verificación de conectividad en Mega Picross
3. **Constraint Propagation:** Reducción del espacio de búsqueda
4. **Branch and Bound:** Exploración inteligente del solver CP-SAT
5. **Pattern Matching:** Uso de `AddAllowedAssignments` para restricciones complejas

## Conclusiones

- **Modelado declarativo:** Expresar el problema como CSP permitió una implementación elegante y mantenible
- **Escalabilidad:** El enfoque funciona para tableros de diferentes tamaños y complejidades
- **Reutilización:** El código base del Picross regular se reutiliza en las variaciones
- **Trade-off complejidad-flexibilidad:** Mayor flexibilidad (colores, mega) implica mayor complejidad computacional
- **Aplicabilidad:** Las técnicas son generalizables a otros puzzles lógicos (Sudoku, Kakuro, etc.)

## Aplicaciones y extensiones futuras

- **Generación automática de puzzles:** Crear Picross con solución única garantizada
- **Resolución incremental:** Mostrar pasos de razonamiento humano
- **Mega Picross extendido:** Soportar pistas que abarcan 3+ líneas
- **Optimización:** Reducir patrones generados mediante heurísticas
- **Interfaz interactiva:** Aplicación web para jugar y resolver

## Código y ejecución

### Requisitos

#### Backend (Python)
```bash
pip install numpy matplotlib ortools
```

#### Frontend (Next.js)
```bash
cd frontend
npm install
```

### Estructura del proyecto
```
📁 20252_cst_tf_g3/
├── 📁 frontend/              # Aplicación web interactiva (Next.js)
│   ├── app/                  # Páginas y componentes de Next.js
│   ├── package.json          # Dependencias del frontend
│   └── .gitignore           # Archivos ignorados por git
├── 📓 picross.ipynb          # Notebook principal con todas las variaciones
└── 📄 README.md              # Este archivo
```

### Ejecución

#### Análisis y Solver (Python)
1. **Notebook:** Abrir `picross.ipynb` en Jupyter/VS Code y ejecutar las celdas secuencialmente

#### Aplicación Web Interactiva (Frontend)
1. Navegar a la carpeta del frontend:
   ```bash
   cd frontend
   ```

2. Instalar dependencias (solo la primera vez):
   ```bash
   npm install
   ```

3. Iniciar el servidor de desarrollo:
   ```bash
   npm run dev
   ```

4. Abrir el navegador en `http://localhost:3000`

**Características del Frontend:**
- 🎮 **Interfaz interactiva** para visualizar Picross
- 🎨 **Visualización en tiempo real** de las soluciones
- 🧩 **Múltiples ejemplos** con diferentes variaciones
- 📊 **Estadísticas** y seguimiento de progreso
- 🚀 **Responsive design** para móviles y desktop

---

## Créditos
| Código | Apellidos y Nombres |
|--------|--------------------|
| U202122430 | Ariana Graciela Quelopana Puppo |
| U20221E167 | Liam Mikael Quino Neff |
| U20211c688 | Leonardo Leoncio Bravo Ricapa |
| U202210644 | Nathaly Eliane Anaya Vadillo |
