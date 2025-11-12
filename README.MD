## Punto 1
## Elevator pitch (3--5 lineas)

Una pequeña tienda necesita un sistema confiable para **gestionar su
inventario** sin errores de duplicados, precios mal ingresados ni
cantidades negativas. Nuestra app web permite **registrar, consultar,
editar y eliminar productos** con **validaciones claras** y alertas de
error (p.ej., producto duplicado). Esta pensada para el
**administrador** (gestiona el inventario) y el **empleado** (consulta
existencias) de forma **rapida y accesible** desde el navegador.

------------------------------------------------------------------------

## Usuarios/segmentos y principales casos de uso

-   **Administrador de la tienda**
    -   Registrar un producto con **nombre, categoria, precio,
        cantidad**.
    -   Editar **precio** o **cantidad** de productos existentes.
    -   Eliminar productos.
-   **Empleado**
    -   Buscar productos por **nombre** para verificar
        disponibilidad y stock.

**Reglas y criterios clave (resumen de historias de usuario):** -
**Duplicados**: si el nombre ya existe -> **HTTP 409**. - **Busqueda**:
siempre retorna **lista paginada** (puede ser vacia). - **Cantidad**:
nunca puede ser **negativa**.

------------------------------------------------------------------------

## Objetivos &  No-objetivos

**Objetivos** 1. CRUD de productos con validaciones de negocio
(duplicados, rangos validos, cantidades >= 0). 2. Busqueda/filtrado por
nombre y categoria con **paginacion**. 3. UI simple y responsiva,
enfocada en velocidad de captura y consulta. 4. API documentada
automaticamente (**Swagger UI** en FastAPI).

**No-objetivos (explicitamente fuera de alcance por ahora)** - Gestion
de usuarios/roles avanzada (mas alla de admin/empleado sin autenticacion
robusta). - Integracion con codigos de barras/lectores fisicos. -
Reportes avanzados (PDF/Excel), analitica o BI. - Integracion con
sistemas contables/ERP o facturacion. - Multi-sucursal o control de
movimientos/bitacora.

------------------------------------------------------------------------

## Metricas/KPIs de exito

-   **Exactitud de datos**: % de operaciones rechazadas por validacion
    (esperado: >= 95% de registros validos a la primera tras 1 semana de
    uso).
-   **Tiempo de operacion**: mediana de tiempo para **registrar** un
    producto (objetivo: \< 15 s) y para **buscar** uno (\< 3 s).
-   **Errores criticos**: 0 caidas en ambiente local durante sesiones de
    \> 1 hora.
-   **Cobertura funcional**: CRUD y busqueda con paginacion disponibles
    y probadas (checklist de QA >= 90% cumplida).
-   **Satisfaccion del usuario**: NPS interno de admin/empleado >= 8/10
    tras prueba piloto.

## Casos de uso (flujo minimo)

1.  **Registrar** -> Validar -> Crear -> Responder `201` con el producto.
2.  **Buscar** -> Filtros `name`, `category`, paginacion `page`, `size` ->
    Lista (puede ser vacia) y `total`.
3.  **Editar** -> Validar no negativo en cantidad -> Actualizar `price` o
    `quantity`.
4.  **Eliminar** -> Borrado por id/nombre.

## Convenciones de validacion (implementacion esperada)

-   **Nombre**: requerido, string no vacio, unico (case_insensitive
    sugerido).
-   **Categoria**: requerida, string no vacia.
-   **Precio**: numero \> 0 con hasta 2 decimales (rechazar valores no
    numericos).
-   **Cantidad**: entero >= 0 (rechazar negativos o no enteros).
-   **Codigos HTTP**:
    -   `201` creacion exitosa, `200` lectura/edicion/eliminacion
        exitosa.
    -   `400/422` validacion de esquema/valores.
    -   `409` conflicto por **nombre duplicado**.


## Must have (MVP inicial)

Registrar producto
Como administrador, quiero registrar un producto con nombre, precio y cantidad, para llevar control del inventario.

## Criterios de aceptación:

Si el nombre ya existe, el sistema devuelve error 409.

No se permite precio < 1 ni cantidad < 1.

El sistema responde con 201 y los datos del producto registrado.

Listar todos los productos
Como empleado, quiero listar todos los productos, para ver el inventario actual de la tienda.

Criterios de aceptación:

Devuelve lista completa de productos registrados.

Si no hay productos, retorna lista vacía.

El sistema responde con 200.

## Should have (valores añadidos al MVP)

Validaciones de precio y cantidad
Como administrador, quiero que el sistema valide el precio (mayor o igual a 1) y cantidad (entero ≥ 1), para evitar errores en el inventario.

Criterios de aceptación:

Valores inválidos → 422 Unprocessable Entity.

Precio y cantidad se guardan con formato correcto.

UI con alertas de error
Como usuario (admin/empleado), quiero recibir mensajes claros de error en la interfaz, para entender qué corregir.

Criterios de aceptación:

Mensajes visibles cuando el producto ya existe (409) o cuando hay valores inválidos (422).

Alertas visibles en pantalla sin romper el flujo de uso.

## Could have (no implementado todavía)

Editar producto (precio o cantidad).

Búsqueda por nombre o categoría.

Filtros avanzados de búsqueda (rango de precios, stock).

Exportar inventario a CSV.

## Won’t have (fuera de alcance en esta fase)

Gestión de usuarios y roles avanzados.
Se pospone para fases futuras.

## Mapa de versiones

MVP (Semana 11)

Historias 1 y 2 (Registrar y listar productos).

Historia 3 (Validaciones críticas).

Historia 4 (UI con alertas de error).

Versión final (Semana 16)

CRUD completo (editar, eliminar).

Búsqueda y filtros avanzados.

Exportar inventario a CSV.

Mejoras de UI/UX y pruebas extendidas.

## 5) Arquitectura y decisiones técnicas

### 5.1 Diagrama general
El sistema se compone de dos capas principales:


- **Frontend (React + Vite):**
  - SPA que consume la API mediante `fetch` o `axios`.
  - Manejo de estado con hooks (`useState`, `useEffect`).
  - Componente principal: listado y creación de productos.
  - Configuración de CORS mediante variable `VITE_API_URL`.

- **Backend (FastAPI):**
  - Exposición de endpoints REST para **Productos** y **Categorías**.
  - Validaciones con **Pydantic** (unicidad, precios y cantidades > 0).
  - Manejo de errores con respuestas controladas (422, 404, 409).
  - Persistencia en base de datos relacional (SQLite para desarrollo, PostgreSQL para producción).

- **CORS:**
  - Habilitado en FastAPI para aceptar solicitudes desde `http://localhost:5173` (Vite).

---

### 5.2 Librerías clave

#### Backend
- **FastAPI** → framework para definir endpoints REST.
- **Pydantic** → validación de datos (reglas de negocio).
- **SQLAlchemy** → ORM para mapear entidades (Producto, Categoría).
- **Uvicorn** → servidor ASGI para correr la API.
- **pytest + httpx** → pruebas automáticas de endpoints.

#### Frontend
- **React + Vite** → desarrollo rápido de SPA.
- **axios** → consumo de la API (más cómodo que `fetch`).
- **React Router (opcional)** → navegación entre vistas.
- **TailwindCSS (opcional)** → estilos rápidos y consistentes.

---

### 5.3 Estrategia de estado en frontend
- **Estado local con hooks** (`useState`, `useEffect`).
- No se requiere Redux ni librerías externas dado el alcance del MVP.
- Ejemplo:
  - `productos` → lista obtenida desde el backend.
  - `loading` y `error` → control de estados al hacer fetch.

---

### 5.4 Manejo de errores
- **422 Unprocessable Entity** → datos inválidos (precio < 1, cantidad < 1).
- **404 Not Found** → recurso inexistente (producto o categoría no encontrada).
- **409 Conflict** → duplicados (nombre de producto o categoría ya existente).
- **200/201/204** → operaciones exitosas (lectura, creación, eliminación).

Frontend:
- Mostrar mensajes claros al usuario (ej. “El producto ya existe” o “Cantidad inválida”).
- Estados de **loading/error** visibles en pantalla.

---

### 5.5 Decisiones técnicas
- **Base de datos**: se usará SQLite en desarrollo por simplicidad, pero se puede migrar a PostgreSQL sin cambios mayores gracias a SQLAlchemy.
- **Estructura modular**: separar backend en carpetas (`models/`, `routes/`, `core/`) para mantener orden.
- **Validaciones en backend**: se prioriza la lógica de validación en Pydantic para evitar datos inconsistentes en la BD.
- **UI simple pero robusta**: empezar con listado + creación (vertical slice) y luego expandir a edición/eliminación.
- **Pruebas mínimas**: se incluirá al menos 1 prueba automatizada para verificar el CRUD.

---

### DIAGRAMA SPA SUBIDO AL REPOSITORIO

## SECUENCIA TIPICA
## 1. Usuario  
- Completa formulario **"Nuevo producto"** en la aplicación.  

## 2. React SPA  
- Envía `POST /productos` con el body:  

{
  id_categoria,
  nombre_producto,
  precio,
  cantidad,
  fecha_creacion
}

## 3. FastAPI  
- Valida con **Pydantic**:  
  - `nombre_producto` debe ser único → responde **409** si ya existe.  
  - `precio >= 1` y `cantidad >= 1` → responde **422** si falla.  
- Inserta en la base de datos vía **SQLAlchemy**.  

## 4. Base de datos  
- Devuelve **OK** con el `id` generado.  

## 5. FastAPI  
- Responde con **201 Created**.  
- Incluye cabecera `Location: /productos/{id}`.  

## 6. React SPA  
- Llama a `GET /productos?offset=0&limit=10`.  

## 7. FastAPI → DB → FastAPI  
- Devuelve **200 OK** con:  
  - Header: `X-Total-Count`.  
  - Body: `[ ... productos ... ]`.  

## 8. React SPA  
- Actualiza la lista y muestra mensaje de éxito al usuario.  


## PUERTOS Y FLUJO DE RED
[ Navegador ] Origin: http://localhost:5173 -> [ React (Vite dev server)  5173 ]  --CORS-->  [ FastAPI (uvicorn)  8000
]  -->  [ DB ]
         

## Mini-ERD
## Categoria  
- **id (PK)**  
- **nombre***  
- descripcion  

## Producto  
- **id (PK)**  
- id_categoria (FK → Categoria.id)  
- **nombre***  
- precio  
- cantidad  
- fecha_crea  

## Relación  
- Una **Categoria** puede tener **n Productos**.  
- Un **Producto** pertenece a **una Categoria**.  

(*) únicos

## 6) Plan del proyecto

### 6.1 Roadmap

**Semana 6 (Entregable 1 — MVP Alpha)**
- ✅ Definición del caso (gestión de inventario de productos).  
- ✅ Diseño del modelo de datos base (**productos** y **categorías**).  
- ✅ Primeros endpoints CRUD en FastAPI.  
- ✅ Arquitectura inicial (frontend ↔ backend) documentada.  
- 🚧 Primer prototipo funcional del CRUD básico.  

---

**Semana 7–10**
- ✅ Implementación del **CRUD completo** en backend (productos, categorías, usuarios, órdenes).  
- ✅ Incorporación de **autenticación y registro de usuarios** con JWT.  
- ✅ Protección de endpoints con `Depends(get_current_user)`.  
- ✅ Implementación del **Rate Limiter** (5 solicitudes/minuto por IP en `/login` y `/register`).  
- ✅ Validaciones robustas (unicidad, tipos, valores mínimos).  
- ✅ Configuración de **CORS** y variables de entorno (`.env`).  
- ✅ Frontend React funcional con formularios, tabla y alertas de error/success.  
- ✅ Paginación (`offset`/`limit`) y **búsqueda con debounce (300 ms)**.  
- ✅ Pruebas de endpoints con `pytest` y `httpx` (nivel básico).  

---

**Semana 11 (Entregable 2 — MVP completo)**  
- ✅ Backend y frontend totalmente integrados (API consumida correctamente desde React).  
- ✅ CRUD visual de productos con edición (✏), eliminación (🗑) y validaciones.  
- ✅ Mejoras visuales con **CSS moderno (gradientes, sombras, bordes redondeados)**.  
- ✅ Mensajes de estado (`error`, `success`) en la interfaz.  
- ✅ Login y registro con **almacenamiento de token JWT en localStorage**.  
- ✅ Diseño responsivo y coherente con el tema general.  
**Justificación:**  
Estas mejoras permitieron lograr un flujo completo y seguro de trabajo:  
el usuario puede **registrarse**, **autenticarse**, **gestionar su inventario** y **crear órdenes** dentro de una interfaz unificada.  
El sistema ahora es **modular, validado y protegido**, cumpliendo con las metas del proyecto.

---

**Semana 12–15**
- ✅ Rediseño visual completo de pantallas de **Login** y **Register**.  
- ✅ Título principal 🛒 *Inventario de Productos* resaltado con recuadro blanco y sombra.  
- ✅ Creación y visualización de **órdenes** (relación N:M entre productos y pedidos).  
- ✅ Revisión final de **validaciones y seguridad JWT**.  
- ✅ Documentación técnica mejorada (README + Swagger).  
- ✅ Estructura del repositorio final (frontend/backend separados).  
- ⚙️ Ensayos de demo y ajustes finales previos a sustentación.  

---

**Semana 16 (Entrega final)**  
- 🎬 Demo funcional completa:  
  - Registro/Login con JWT.  
  - CRUD de productos, categorías y órdenes.  
  - Validaciones, mensajes y control de errores (401, 409, 422).  
- 🧩 Evidencia: capturas + video corto (≤ 3 min).  
- 📁 Repositorio final con documentación y entorno reproducible.  
- 🧾 Checklist de entregable cumplido al 100 %.  
- 🌟 Proyecto preparado para sustentación y presentación final.  

---


## 6.2 Kanban En Imagen Subida al repositorio

## 7) Cambios recientes y mejoras

### 🆕 Actualización — Semana 11 a 16 (Versión Final del Proyecto)

El sistema evolucionó significativamente desde el MVP inicial, pasando de un CRUD simple a una **aplicación web completa con autenticación, seguridad y diseño profesional**.

---

### 🖥️ **Frontend (React + Vite)**

- 💄 **Nuevo diseño visual**:  
  - Se mejoraron los estilos con **gradientes, sombras y bordes redondeados** para lograr una interfaz más moderna y atractiva.  
  - Se agregó un **recuadro blanco con sombra al título principal 🛒 “Inventario de Productos”**, para que destaque sobre el fondo degradado.  

- 🔒 **Login y Registro rediseñados**:  
  - Se centraron los formularios verticalmente con un diseño limpio y formal.  
  - Se aplicaron los mismos colores corporativos (`#2575fc` y `#6a11cb`) para mantener coherencia visual.  
  - Se añadieron mensajes dinámicos de error y éxito con estilos (`.message.error`, `.message.success`).  

- 🧭 **Interactividad y usabilidad**:  
  - Se implementó **búsqueda en tiempo real con debounce (300 ms)** para evitar recargar el backend mientras el usuario escribe.  
  - Se añadió **paginación funcional** con botones de *“Anterior”* y *“Siguiente”* y límite de productos por página.  
  - Se mejoró el flujo de edición de productos con el botón (✏) que permite modificar directamente sin recargar toda la lista.  
  - Se agregó confirmación visual al eliminar productos (🗑) mediante mensajes de éxito o error.  

- ⚙️ **Integración total con el backend**:  
  - Manejo del token JWT almacenado en `localStorage`.  
  - Verificación automática del token al recargar la página (`verifyToken()` en `App.jsx`).  
  - Todas las solicitudes (`GET`, `POST`, `PUT`, `DELETE`) incluyen el encabezado `Authorization: Bearer <token>`.  

- 📱 **Mejoras de UX**:  
  - Componentes reutilizables, tipografía moderna (Poppins), y diseño responsivo.  
  - El sistema ahora ofrece una experiencia consistente y atractiva tanto en pantallas grandes como pequeñas.  

---

### ⚙️ **Backend (FastAPI + SQLite)**

- 🔑 **Autenticación segura con JWT**:  
  - Se implementó un sistema de autenticación **JWT hecho a mano** (sin dependencias externas).  
  - Incluye expiración de token, validación de firma HMAC SHA256 y verificación de usuario.  
  - Endpoints protegidos con `Depends(get_current_user)` para operaciones CRUD y creación de órdenes.  

- 🧠 **Rate Limiter incorporado**:  
  - Previene abuso de endpoints críticos (`/login`, `/register`) con un límite de 5 solicitudes por minuto por IP.  
  - Al exceder el límite, el backend responde con **HTTP 429 “Too many requests”**.  

- 🗂️ **CRUD extendido y modular**:  
  - Entidades implementadas: **usuarios**, **categorías**, **productos**, **órdenes** y **order_items** (relación N:M).  
  - Soporte para crear productos con `id_categoria` numérico o nombre de categoría (se crea automáticamente si no existe).  
  - Endpoints `/orders` y `/orders/my` agregados para registrar pedidos del usuario autenticado.  

- 🧩 **Validaciones avanzadas**:  
  - Evita duplicados (productos, categorías, usuarios).  
  - Controla rangos válidos de precio y cantidad.  
  - Devuelve errores consistentes (`400`, `401`, `404`, `409`, `422`) con mensajes claros.  

- 🧱 **Persistencia robusta con SQLite**:  
  - Base de datos inicializada automáticamente al ejecutar `main.py`.  
  - Tablas relacionadas con claves foráneas (`ON DELETE SET NULL` y `ON DELETE CASCADE`).  

- 🧪 **Pruebas y documentación**:  
  - Endpoints probados con `pytest` y `httpx`.  
  - Documentación generada automáticamente en Swagger (`/docs`).  
  - Código modular en archivos: `db.py`, `auth.py`, `models.py`, `routes.py`, `main.py`.  

---

### 🎨 **Mejoras globales de experiencia y presentación**

- 🧾 Se añadieron **mensajes informativos y alertas visuales** en todo el flujo (creación, edición, eliminación).  
- 🧠 Se mantuvo consistencia en nombres, estilos y estructura del repositorio.  
- 🪄 Se eliminó código duplicado y se mejoró la legibilidad (nombres descriptivos, comentarios clave).  
- 📘 Se actualizó y amplió el `README.md` con instrucciones detalladas, endpoints y guía de despliegue.  

---

### 🚀 Resultado final

El proyecto pasó de un MVP funcional a un **sistema web completo y seguro**, con:
- **Autenticación JWT**,  
- **CRUD de cuatro entidades con relaciones**,  
- **Validaciones y rate limiting**,  
- **Frontend moderno, interactivo y estéticamente consistente**.

Este conjunto de mejoras cumple con **todos los requisitos del entregable final**, listo para **demo y sustentación profesional**.  


## Como utilizar el programa

Backend (FastAPI)
1. Entrar a la carpeta del backend
```bash
cd backend
```
2. Crear entorno virtual con uv
```bash
uv venv .venv
```
3. Activar entorno virtual
En Windows (PowerShell):
```powershell
.venv\Scripts\Activate.ps1
```
En Linux/Mac:
```bash
source .venv/bin/activate
```
4. Instalar dependencias
```bash
uv pip install -r requirements.txt
```
5. Levantar servidor FastAPI
```bash
uv run uvicorn app.main:app --reload --port 8000
```
👉 El backend queda disponible en:
- API: http://127.0.0.1:8000
- Swagger Docs: http://127.0.0.1:8000/docs
 Frontend (React + Vite)
1. Entrar a la carpeta del frontend
```bash
cd frontend
```
2. Instalar dependencias
```bash
npm install
```
3. Configurar variables de entorno
Crear el archivo `.env.development` dentro de `frontend/` con el contenido:
```env
VITE_API_URL=http://127.0.0.1:8000
```
4. Levantar servidor de desarrollo
```bash
npm run dev
```
👉 El frontend queda disponible en:
- http://localhost:5173
🧪 Flujo de prueba
1. Levantar backend en el puerto 8000.
2. Levantar frontend en el puerto 5173.
3. Abrir http://localhost:5173 en el navegador.
4. Probar:
   - Crear un producto en el formulario.
   - Ver el listado actualizado.
   - Intentar crear un producto duplicado → debe aparecer error 409.

## 7 Estructura del proyecto
```json
project/
├─ backend/
│  ├─ app/
│  │  ├─ api/routes/products.py
│  │  ├─ models/product.py
│  │  ├─ core/config.py
│  │  └─ main.py
│  ├─ requirements.txt
│  └─ tests/test_products.py
└─ frontend/
   ├─ src/
   │  ├─ api.js
   │  ├─ App.jsx
   │  └─ main.jsx
   ├─ .env.development
   ├─ package.json
   └─ vite.config.js

## requirements.txt
fastapi
uvicorn
sqlalchemy
pydantic
pytest
httpx











