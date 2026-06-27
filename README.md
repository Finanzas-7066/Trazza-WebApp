# 📊 Explicación del Código: Trazza WebApp

Este documento detalla la estructura interna de Trazza WebApp, desglosando la responsabilidad de cada capa de la Arquitectura N-Tier y sus algoritmos.

---

## 1. 🖥️ Explicando el Código: El Frontend (React + Next.js)

La carpeta `frontend/` aloja la interfaz de usuario interactiva, diseñada para consumir los endpoints de la API del backend y representar visualmente las decisiones de la Inteligencia Artificial. Está construida sobre **React** y el framework **Next.js**. 

A continuación, se detalla la responsabilidad de los archivos clave en el código fuente (`src/`):

### 📂 Carpeta `src/app/` (Enrutamiento y Vistas)
*   **`layout.tsx`**: Define la estructura global (*Root Layout*) de la aplicación Next.js. Aquí se inyectan las fuentes globales, metadatos y el esqueleto HTML base.
*   **`globals.css`**: Archivo de estilos globales. Utiliza utilidades de TailwindCSS para definir esquemas de colores (como los fondos oscuros), animaciones personalizadas (destellos de carga) y el soporte tipográfico general.
*   **`page.tsx`**: Es la **Vista Principal (Dashboard del Operador)**. Su función es dual:
    *   **Controlador Lógico**: Maneja el estado (Hooks de React) para gestionar las fases de la simulación (`config` -> `ia_loading` -> `ia_done`), procesa los errores matemáticos y orquesta las llamadas HTTP (`fetch`) hacia los endpoints de Python.
    *   **Diseño (UI)**: Renderiza el panel flotante interactivo (estilo *Glassmorphism*). Despliega métricas avanzadas como la barra de "Ocupación de Flota", el balance de ingresos (S/.) y la tarjeta de fallos si la IA declara un Flete Muerto.

### 📂 Carpeta `src/components/` (Componentes Reutilizables)
*   **`MapComponent.tsx`**: Es el núcleo visual del proyecto. 
    *   Emplea la librería **React-Leaflet** (una adaptación de *Leaflet.js* para React) para pintar el mapa cartográfico oscuro de la ciudad.
    *   Cuenta con una factoría de íconos (`createNumberedIcon`) que dibuja dinámicamente pines de colores con numeración secuencial.
    *   Aloja la lógica de renderizado espacial, como el dibujo simultáneo de polilíneas (Cyan para la ruta de Ida; paleta neón para el Backhaul) y aplica los *despliegues tácticos* (empujando pines hacia abajo) cuando detecta colisiones geométricas entre recojos y metas.

### ⚙️ Archivos de Configuración (Raíz del Frontend)
*   **`next.config.ts`**: Configuraciones críticas del compilador de Next.js.
*   **`tailwind.config.ts` / `postcss.config.mjs`**: Motores de estilos para interpretar las clases de utilidades estéticas y compilar el CSS.
*   **`package.json`**: Define el ecosistema de dependencias (React, Next, Lucide-React para iconos, Leaflet) y expone los comandos de ejecución como `npm run dev`.

---

## 2. 🗄️ Explicando el Código: El Backend (Capa de Datos)

La carpeta `backend/datos/` representa la **Capa de Persistencia** en nuestra Arquitectura N-Tier. Esta capa es completamente agnóstica; es decir, no sabe nada sobre algoritmos de ruteo, FastAPI o el frontend. Su única misión es comunicarse con el disco duro para guardar y recuperar la información del negocio.

A continuación, se detalla la función de cada elemento presente en el directorio:

### 📂 Elementos de la Capa de Datos
*   **`__pycache__/`**: Una carpeta generada automáticamente por Python. Contiene el *bytecode* compilado de los scripts para acelerar los tiempos de carga en futuras ejecuciones. (No se sube al repositorio, es de uso local).
*   **`__init__.py`**: Un archivo especial (usualmente vacío) que le indica a Python que la carpeta `datos` debe ser tratada como un "Módulo" importable. Esto permite que otras capas puedan hacer `from datos.models import...`.
*   **`database.py`**: Es el **Motor de Conexión**. Aquí se configura SQLAlchemy, se establece la cadena de conexión (Connection String) hacia SQLite y se instancian los creadores de sesiones (`sessionmaker`) para garantizar transacciones seguras (ACID).
*   **`models.py`**: Contiene el **Mapeo Objeto-Relacional (ORM)**. Convierte las tablas relacionales en Clases de Python. Aquí se define la arquitectura *Single Table Inheritance*, alojando entidades polimórficas (`EmpresaB2B`, `ProveedorMaterial`, `PlantaReciclaje`), `FlotaTransporte` y los `LoteCarga`.
*   **`seed.py`**: Es el **Script de Poblamiento Masivo (Seed)**. Se encarga de inyectar las 27 cargas sintéticas pesadas, calcular sus nodos geográficos exactos usando OSMnx y guardarlos en la base de datos para que el motor matemático tenga escenarios complejos que resolver.
*   **`trazza_datos.db`**: Es el **Archivo Físico de SQLite**. Toda la base de datos relacional, con sus tablas, columnas y registros, vive encapsulada dentro de este único archivo portable.

---

## 3. 🧠 Explicando el Código: El Backend (Capa de Aplicación)

La carpeta `backend/aplicacion/` constituye el **Core (Cerebro Lógico)** de nuestra arquitectura N-Tier. Esta capa es la más compleja y crítica del sistema, ya que aquí reside el motor de Inteligencia Logística y las reglas de negocio, siendo totalmente independiente de la base de datos (persistencia) y de las peticiones HTTP (presentación).

A continuación, se detalla la función de cada elemento presente en el directorio:

### 📂 Elementos de la Capa de Aplicación
*   **`__pycache__/`**: Carpeta de uso interno de Python que contiene los binarios pre-compilados de la lógica de negocio, optimizando la velocidad de ejecución y de importación en el servidor.
*   **`algoritmos/` (Directorio)**: Es la biblioteca matemática de Trazza. Aquí viven de forma aislada e independiente (cumpliendo el principio *SOLID* de Responsabilidad Única) todos los motores pesados del sistema:
    *   *Mochila (Knapsack Problem)* para optimización de pesos y ganancias monetarias.
    *   *Held-Karp (TSP)* para ordenar la secuencia topológica del agente viajero.
    *   *UFDS y BFS* para validación de clústeres zonales y accesibilidad física.
    *   *Restricción Espacial* para evitar el "Flete Muerto".
*   **`__init__.py`**: Archivo inicializador que convierte a la carpeta `aplicacion` en un paquete oficial de Python, habilitando la inyección de dependencias seguras entre las demás capas.
*   **`servicios_logistica.py`**: Es el **Director de Orquesta (Patrón de Diseño Facade / Fachada)**. En lugar de exponer la complejidad matemática hacia el exterior, este archivo centraliza la lógica maestra:
    1.  Recibe la petición del usuario.
    2.  Extrae los datos limpios solicitándolos a la Capa de Datos (SQLite).
    3.  Pasa la información por los filtros matemáticos en este orden estricto de cascada: `UFDS -> Filtro Espacial -> Dijkstra/Voraz -> Knapsack (Mochila) -> TSP (Held-Karp)`.
    4.  Aplica nuestra lógica patentada de **Fusión Espacial Continua** (agrupando cargas idénticas para evitar superposiciones en el mapa).
    5.  Intercepta los errores matemáticos para devolver mensajes humanos precisos (Ej: "Restricción Base abortada").
    6.  Devuelve la solución JSON perfectamente empaquetada lista para que el Frontend la dibuje.

### 🧠 3.1. El Ecosistema Algorítmico de Trazza (Internos y Externos)

El poder de decisión de Trazza no se basa en un solo cálculo matemático, sino en un *Pipeline* (Tubería) de algoritmos encadenados. Algunos han sido aislados en su propia clase por su complejidad computacional, mientras que otros han sido integrados tácticamente en caliente dentro del Orquestador.

#### A) Algoritmos Modulares (Directorio `algoritmos/`)
Son aquellos lo suficientemente extensos como para requerir su propio archivo físico, respetando el Principio de Responsabilidad Única.

1.  **`ufds.py` (Union-Find Disjoint Sets)**: Se encarga de particionar el mapa. Si hay 27 cargas dispersas, el UFDS encuentra conexiones y agrupa las cargas en "zonas logísticas" (Clústeres) para que el camión no intente evaluar viajes entre zonas geográficamente aisladas.
2.  **`restriccion_base.py` (Filtro Topológico Espacial)**: Es el escudo financiero. Analiza si las cargas de un clúster acercan al camión de vuelta a su Base de Origen. Si ninguna carga apunta a la Base, el algoritmo aborta la búsqueda y decreta un "Flete Muerto" inevitable para no incurrir en gastos de combustible adicionales.
3.  **`voraz.py` (Algoritmo Greedy)**: Es un motor de ordenamiento rápido. Toma las cargas sobrevivientes y las ordena priorizando a los proveedores que estén físicamente más cerca de la ubicación actual del camión, buscando ganancias a corto plazo.
4.  **`backtracking.py` (Problema de la Mochila 0/1)**: El núcleo del *Backhaul*. Evalúa todas las combinaciones posibles de cargas (Explosión Combinatoria) buscando la que maximice los ingresos (S/.) pero podando inmediatamente los árboles de búsqueda que superen la capacidad física del camión (Ej. 25 Toneladas).
5.  **`tsp.py` (Agente Viajero - Held-Karp)**: Programación Dinámica pura. Recibe los orígenes y destinos seleccionados por la Mochila y calcula matemáticamente cuál es el orden exacto en el que el camión debe visitarlos para que la distancia total manejada sea la mínima posible. Resuelve el problema NP-Hard en tiempo $O(n^2 2^n)$.
6.  **`bfs.py` (Búsqueda en Anchura)**: Un validador de topología. Escanea nivel por nivel los nodos de las calles para asegurar que existe un camino de asfalto real entre dos puntos antes de que el camión intente ir.
7.  **`grafos.py`**: Intermediario traductor. Convierte la data extraída de OpenStreetMap (OSMnx) en el lenguaje matemático de Grafos entendible por NetworkX.

#### B) Algoritmos Integrados (Directamente en `servicios_logistica.py`)
Son rutinas de alta eficiencia que operan estructuralmente dentro del Orquestador sin necesidad de un archivo separado.

8.  **Dijkstra (`nx.shortest_path_length`)**: Implementado a través de la librería NetworkX. El Orquestador lo invoca en tiempo real para calcular la distancia más corta (respetando el tráfico y sentido de las calles) entre el camión y cualquier cliente, alimentando los pesos para el Algoritmo Voraz y el TSP.
9.  **Algoritmo de Agrupación (Hashing y Frecuencias)**: Diseñado para mitigar sobrecargas en el TSP. Si el camión recogerá dos cargas distintas en la *misma fábrica exacta*, este algoritmo usa diccionarios (llaves HASH) para agruparlas en un solo vértice, sumando sus tonelajes. Convierte 4 paradas redundantes en solo 2, evitando el colapso exponencial del TSP y la superposición de pines en React.
10. **Fusión Espacial Continua (Tolerancia Geométrica)**: Evaluador matemático final de colisiones (radio de 0.005 grados). Permite dictaminar, por ejemplo, que dos entregas finales que llegan a la Base de Origen se fusionen numéricamente en un solo evento consolidado (Ej. `5. Entrega / Retorno Base`), brindando limpieza absoluta al renderizador visual y garantizando un conteo cronológico ininterrumpido.

---

## 4. 🌐 Explicando el Código: El Backend (Capa de Presentación)

La carpeta `backend/presentacion/` conforma la última pieza del modelo N-Tier (Capa Exterior). Su único propósito es servir como puente de comunicación (API RESTful) entre el Frontend interactivo (React) y el cerebro matemático (Capa de Aplicación). Fiel al principio de separación de responsabilidades, esta capa no calcula rutas ni accede a la base de datos; solo recibe solicitudes HTTP, las delega de forma asíncrona y devuelve las respuestas.

A continuación, se detalla la función de cada elemento presente en el directorio:

### 📂 Elementos de la Capa de Presentación
*   **`__pycache__/`**: Carpeta de uso interno generada por Python para almacenar los binarios pre-compilados de los endpoints, agilizando enormemente los tiempos de arranque del servidor web (FastAPI).
*   **`__init__.py`**: Archivo declarador que define esta carpeta como un paquete estructurado de Python, permitiendo importar submódulos sin colisiones de dependencias.
*   **`main.py`**: Es el archivo principal del servidor y el corazón de la API REST construida con el framework de alto rendimiento **FastAPI**. Sus responsabilidades clave son:
    1.  **Levantar el Servidor (Uvicorn):** Inicializa el entorno asíncrono ASGI, permitiendo que Trazza soporte múltiples peticiones logísticas concurrentes sin bloquearse.
    2.  **CORS (Cross-Origin Resource Sharing):** Configura los permisos de seguridad (Middlewares) para autorizar explícitamente al Frontend de Next.js (que corre en un puerto distinto, `localhost:3000`) a consumir los servicios de Python.
    3.  **Definición de Endpoints:** Expone las rutas web (ej. `@app.post("/rutas/match-retorno")`). Recibe el Payload JSON enviado por el frontend (coordenadas GPS, peso máximo del camión), lo desempaqueta y se lo inyecta limpiamente al Orquestador (`servicios_logistica.py`).
    4.  **Inyección del Grafo (Memoria Viva):** Durante su ciclo de vida de arranque (*On Startup*), le ordena a la Capa de Aplicación que descargue y consolide el Grafo Topológico de OSMnx directamente en la memoria RAM, manteniéndolo persistente para que todas las peticiones posteriores se procesen en fracciones de segundo.
