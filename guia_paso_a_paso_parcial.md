# 📝 Guía Paso a Paso — Cómo resolver ejercicios de parcial

> Basada en el formato del parcial "Tienda Sol — Wishlist".
> Todos los ejemplos adaptan patrones reales del TP **Sweet Medical**.

---

## Tabla de Contenidos

1. [Estructura general del parcial](#estructura-general-del-parcial)
2. [PASO 1 — UI / Frontend](#paso-1--ui--frontend)
3. [PASO 2 — Backend (Endpoints REST)](#paso-2--backend-endpoints-rest)
4. [PASO 3 — Persistencia (Modelo de datos)](#paso-3--persistencia-modelo-de-datos)
5. [PASO 4 — Sobre la funcionalidad (Preguntas de diseño)](#paso-4--sobre-la-funcionalidad-preguntas-de-diseño)
6. [Ejemplo completo resuelto: Wishlist para Sweet Medical](#ejemplo-completo-resuelto-wishlist-para-sweet-medical)
7. [Checklist final antes de entregar](#checklist-final-antes-de-entregar)

---

## Estructura general del parcial

El parcial típico presenta un **escenario de negocio** (una funcionalidad nueva a agregar) y pide resolver cuatro secciones:

| Sección | Qué evalúa | Peso aprox. |
|---------|-------------|-------------|
| **UI / Frontend** | Croquis, componentes, interacción, accesibilidad | 25% |
| **Backend** | Endpoints REST, lógica de servicios, errores, tests | 35% |
| **Persistencia** | Modelo de datos, embedded vs referencia, esquema | 20% |
| **Funcionalidad** | Decisiones de diseño, justificación, impacto en sistema | 20% |

---

## PASO 1 — UI / Frontend

### 1.1 Croquis / Mockup

**Qué piden:** Dibujar las pantallas nuevas o modificadas que implementan la funcionalidad.

**Checklist:**
- [ ] Identificar **qué pantallas se agregan** (ej: "Mi Wishlist")
- [ ] Identificar **qué pantallas existentes se modifican** (ej: "Detalle de producto" → agregar botón)
- [ ] Dibujar cada pantalla con elementos clave etiquetados
- [ ] Mostrar estados diferentes: vacío, con datos, loading, error

**Plantilla de respuesta:**
> Se necesitan las siguientes pantallas:
>
> **Pantalla 1 — [Nombre]** (NUEVA / MODIFICADA)
> - Descripción: ...
> - Elementos: [lista de inputs, botones, cards]
> - Croquis: [dibujo]
>
> **Pantalla 2 — [Nombre]** (NUEVA / MODIFICADA)
> - ...

### 1.2 Componentes

**Qué piden:** Mencionar qué componentes son nuevos y cuáles existentes se reutilizan o modifican.

**Cómo pensarlo (usando la estructura del TP):**

```
features/
  wishlist/                 ← CARPETA NUEVA (como turnos/)
    Wishlist.jsx            ← Componente página principal
    WishlistItem.jsx        ← Componente para cada item
components/
  cards/
    WishlistCard.jsx        ← NUEVO componente reutilizable
  modals/
    ConfirmarEliminar.jsx   ← EXISTENTE, se reutiliza
```

**Plantilla de respuesta:**
> **Componentes nuevos:**
> - `WishlistPage`: Página principal que muestra la lista completa.
> - `WishlistCard`: Card individual para cada producto en la wishlist.
>
> **Componentes existentes reutilizados:**
> - `ProductoCard`: Se le agrega un botón "Agregar a wishlist" (ícono de corazón).
> - `EmptyState`: Componente genérico ya existente para estado vacío.
> - `ConfirmModal`: Modal de confirmación para eliminar de la wishlist.
>
> **Componentes existentes modificados:**
> - `ProductoDetalle`: Se agrega el botón de wishlist en el header.
> - `Sidebar`: Se agrega un ítem de navegación "Mi Wishlist".

### 1.3 Interacción del usuario

**Qué piden:** Describir paso a paso cómo interactúa el usuario con la funcionalidad.

**Plantilla (flujo "Agregar a wishlist"):**
```
1. El usuario navega a la vista de detalle del producto.
2. Hace clic en el ícono de corazón (❤) en el header del producto.
3. Se dispara un fetch POST al backend.
   - Mientras se procesa: el ícono se muestra en estado "loading" (spinner o pulsación).
   - Si la respuesta es exitosa (201):
       → El ícono pasa de contorno vacío a relleno (visual feedback).
       → Se muestra un toast "Producto agregado a tu wishlist".
   - Si la respuesta es error (409 - ya existe):
       → Se muestra un toast "Este producto ya está en tu wishlist".
   - Si la respuesta es error (500 - servidor):
       → Se muestra un toast de error genérico.
4. El usuario puede navegar a "Mi Wishlist" desde el sidebar para ver todos los productos.
```

### 1.4 Accesibilidad

**Qué piden:** Señalar los aspectos de accesibilidad que tendrías en cuenta.

**Respuesta tipo (adaptable a cualquier funcionalidad):**

| Aspecto | Implementación |
|---------|---------------|
| **Botones con texto descriptivo** | Usar `aria-label` en botones con solo íconos: `<button aria-label="Agregar producto X a wishlist">❤</button>` |
| **Feedback dinámico** | Usar `aria-live="polite"` en la zona de mensajes para que el lector de pantalla anuncie "Producto agregado" sin interrumpir. |
| **Navegación por teclado** | Todos los botones y links deben ser accesibles con `Tab` y activables con `Enter`/`Space`. |
| **Estados del botón** | Usar `aria-pressed="true/false"` en el botón de wishlist para indicar si el producto ya está agregado. |
| **Contraste de colores** | El ícono de corazón relleno (wishlist activa) debe tener relación de contraste ≥ 4.5:1 respecto al fondo. |
| **Cargando** | Mientras se procesa el request, agregar `aria-busy="true"` al botón. |
| **Roles semánticos** | Usar `role="status"` en el contenedor del toast de confirmación. |
| **Formularios accesibles** | Si hay un campo de búsqueda en la wishlist, asociar `<label>` con el `<input>` mediante `htmlFor`. |

---

## PASO 2 — Backend (Endpoints REST)

### 2.1 Enumerar rutas REST

**Qué piden:** Listar endpoints con verbo HTTP, URL, body, query params, path variables.

**Formato recomendado (tabla):**

| Verbo | URL | Body | Query Params | Descripción |
|-------|-----|------|--------------|-------------|
| `GET` | `/usuarios/:userId/wishlist` | — | `?vendedor=id` (opcional) | Obtener la wishlist del usuario, con filtro opcional por vendedor |
| `POST` | `/usuarios/:userId/wishlist` | `{ productoId: "abc123" }` | — | Agregar un producto a la wishlist |
| `DELETE` | `/usuarios/:userId/wishlist/:productoId` | — | — | Eliminar un producto de la wishlist |

**Ejemplo de cómo se vería en el código (al estilo del TP):**
```javascript
// wishlistRoutes.js
import express from "express";
import { WishlistController } from "../controllers/controllerWishlist.js";

const wishlistController = new WishlistController();
const router = express.Router({ mergeParams: true }); // para acceder a :userId

router.route("/")
    .get((req, res, next) => wishlistController.obtenerWishlist(req, res, next))
    .post((req, res, next) => wishlistController.agregarProducto(req, res, next));

router.route("/:productoId")
    .delete((req, res, next) => wishlistController.eliminarProducto(req, res, next));

export default router;
```

```javascript
// router.js — Registrar la ruta
router.use("/usuarios/:userId/wishlist", wishlistRouter);
```

### 2.2 Lógica de la capa de servicios

**Qué piden:** Describir la lógica (orquestaciones, validaciones, servicios que intervienen).

**Plantilla de respuesta:**
```
WishlistService.agregarProducto(userId, productoId):
  1. Validar que userId y productoId sean ObjectIds válidos (BadRequestError si no).
  2. Verificar que el usuario exista (UsuarioService.obtenerPorId) → NotFoundError si no.
  3. Verificar que el producto exista (ProductoService.obtenerPorId) → NotFoundError si no.
  4. Verificar que el producto NO esté ya en la wishlist del usuario → ConflictError (409) si ya está.
  5. Agregar el productoId al array de wishlist del usuario.
  6. Guardar y retornar la wishlist actualizada.

Servicios que intervienen:
  - WishlistService (nuevo): lógica central de la wishlist.
  - UsuarioService (existente): validar existencia del usuario.
  - ProductoService (existente): validar existencia del producto.
```

### 2.3 Gestión de errores

**Qué piden:** Cómo manejarías los errores.

**Usando el patrón del TP (errores custom + middleware):**

| Situación | Error | Status HTTP |
|-----------|-------|-------------|
| userId o productoId inválido | `BadRequestError("Id debe ser un ObjectId válido")` | 400 |
| Usuario no encontrado | `NotFoundError("Usuario no encontrado")` | 404 |
| Producto no encontrado | `NotFoundError("Producto no encontrado")` | 404 |
| Producto ya está en la wishlist | `ConflictError("El producto ya existe en la wishlist")` | 409 |
| Error de base de datos | Se propaga al `errorHandler` middleware | 500 |

```javascript
// Referencia: así manejan errores en el TP (appError.js)
export class ConflictError extends AppError {
    constructor(message) {
        super(message, 409);
    }
}
```

### 2.4 Test de integración

**Qué piden:** Un ejemplo de test de integración.

**Estructura recomendada (siguiendo el patrón del TP con Jest + Supertest):**

```javascript
import { describe, it, expect, beforeEach } from "@jest/globals";
import request from "supertest";
import { crearAppDePrueba } from "./helpers/testApp.js";

describe("POST /sweetmedical/v1/usuarios/:userId/wishlist", () => {
    let app;
    let usuarioRepository;
    let productoRepository;

    beforeEach(() => {
        const deps = crearDependencias();
        usuarioRepository = deps.usuarioRepository;
        productoRepository = deps.productoRepository;
        app = crearAppDePrueba({ wishlistController: deps.wishlistController });
    });

    it("escenario feliz: agrega un producto a la wishlist", async () => {
        const usuario = crearUsuarioPrueba();
        usuarioRepository.guardar(usuario);
        const producto = crearProductoPrueba();
        productoRepository.guardar(producto);

        const res = await request(app)
            .post(`/sweetmedical/v1/usuarios/${usuario.id}/wishlist`)
            .send({ productoId: producto.id });

        expect(res.status).toBe(201);
        expect(res.body.status).toBe("success");
    });

    it("error: producto duplicado devuelve 409", async () => {
        const usuario = crearUsuarioPrueba();
        usuario.wishlist = [producto.id]; // ya tiene el producto
        usuarioRepository.guardar(usuario);

        const res = await request(app)
            .post(`/sweetmedical/v1/usuarios/${usuario.id}/wishlist`)
            .send({ productoId: producto.id });

        expect(res.status).toBe(409);
    });

    it("error: producto inexistente devuelve 404", async () => {
        const usuario = crearUsuarioPrueba();
        usuarioRepository.guardar(usuario);

        const res = await request(app)
            .post(`/sweetmedical/v1/usuarios/${usuario.id}/wishlist`)
            .send({ productoId: "id_inexistente" });

        expect(res.status).toBe(404);
    });

    it("error: body vacío devuelve 400", async () => {
        const usuario = crearUsuarioPrueba();
        usuarioRepository.guardar(usuario);

        const res = await request(app)
            .post(`/sweetmedical/v1/usuarios/${usuario.id}/wishlist`)
            .send({});

        expect(res.status).toBe(400);
    });
});
```

### 2.5 Validación en la capa de servicio (no duplicados)

**Qué piden:** Pseudocódigo de la validación para evitar duplicados.

```javascript
// wishlistService.js
async agregarProducto(userId, productoId) {
    // 1. Validar IDs
    if (!this.isValidId(userId)) {
        throw new BadRequestError("userId debe ser un ObjectId válido");
    }
    if (!this.isValidId(productoId)) {
        throw new BadRequestError("productoId debe ser un ObjectId válido");
    }

    // 2. Verificar existencia
    const usuario = await this.usuarioService.obtenerPorId(userId);
    // obtenerPorId ya lanza NotFoundError si no existe

    const producto = await this.productoService.obtenerPorId(productoId);
    // obtenerPorId ya lanza NotFoundError si no existe

    // 3. Verificar duplicado
    const yaExiste = usuario.wishlist.some(
        (item) => String(item.productoId) === String(productoId)
    );
    if (yaExiste) {
        throw new ConflictError("El producto ya existe en la wishlist del usuario");
    }

    // 4. Agregar y guardar
    usuario.wishlist.push({
        productoId: productoId,
        fechaAgregado: new Date(),
    });

    return await this.usuarioRepository.guardar(usuario);
}
```

---

## PASO 3 — Persistencia (Modelo de datos)

### 3.1 Decisión: ¿Colección nueva o embedded?

**Qué piden:** Elegir entre nueva colección vs embebido, y justificar.

**Opciones:**

#### Opción A — Embedded (array dentro de Usuario)
```javascript
// UsuarioSchema (modificado)
const UsuarioSchema = new mongoose.Schema({
    // ...campos existentes...
    wishlist: [{
        producto: {
            type: mongoose.Schema.Types.ObjectId,
            ref: 'Producto',
            required: true,
        },
        fechaAgregado: {
            type: Date,
            default: Date.now,
        },
    }],
});
```

| ✅ Ventajas | ❌ Desventajas |
|-------------|----------------|
| Una sola consulta trae usuario + wishlist | Si la wishlist crece mucho, el documento se hace pesado |
| Simplicidad: no hay joins ni colecciones extra | Límite de 16MB por documento en MongoDB |
| Operaciones atómicas (todo en un documento) | Si necesitás consultar la wishlist independientemente, es menos eficiente |

#### Opción B — Colección nueva (Wishlist)
```javascript
// WishlistSchema (nuevo)
const WishlistSchema = new mongoose.Schema({
    usuario: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'Usuario',
        required: true,
    },
    producto: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'Producto',
        required: true,
    },
    fechaAgregado: {
        type: Date,
        default: Date.now,
    },
});

// Índice único compuesto: evita duplicados a nivel de BD
WishlistSchema.index({ usuario: 1, producto: 1 }, { unique: true });
```

| ✅ Ventajas | ❌ Desventajas |
|-------------|----------------|
| Escala sin límite de tamaño | Requiere joins (populate) |
| Índice único garantiza no-duplicados a nivel BD | Más complejidad operacional |
| Queries independientes sobre wishlist | Dos consultas para obtener usuario + wishlist |

#### Cómo decidir (criterios)

| Criterio | Embedded | Colección nueva |
|----------|----------|-----------------|
| Tamaño esperado del array | < 100 items ✅ | > 100 items ✅ |
| ¿Se consulta solo? | No ✅ | Sí ✅ |
| ¿Necesita índice único? | Más difícil | Fácil ✅ |
| Consistencia de la respuesta del TP | Seguir el patrón que ya usan | — |

**Recomendación para el parcial:**
> Elegir **Opción A (embedded)** cuando el array es pequeño y acotado (una wishlist rara vez tiene más de 50 items). Justificar que simplifica las consultas, las operaciones son atómicas, y el patrón es consistente con cómo el TP ya embebe arrays (ej: `historialTurnos` en Paciente, `servicios` en Turno).
>
> Mencionar que si escalara mucho, migrar a **Opción B** sería lo correcto.

### 3.2 Ejemplo de modelo Mongoose completo (Opción A)

```javascript
// Agregar al UsuarioSchema existente
const wishlistItemSchema = new mongoose.Schema({
    producto: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'Producto',
        required: true,
    },
    fechaAgregado: {
        type: Date,
        default: Date.now,
    },
}, { _id: false }); // Sin _id propio para cada item

// Dentro de UsuarioSchema:
wishlist: {
    type: [wishlistItemSchema],
    default: [],
    validate: {
        validator: function(items) {
            // Validar que no haya productos duplicados
            const ids = items.map(i => String(i.producto));
            return ids.length === new Set(ids).size;
        },
        message: "No se permiten productos duplicados en la wishlist."
    }
}
```

---

## PASO 4 — Sobre la funcionalidad (Preguntas de diseño)

Estas preguntas evalúan tu **criterio como desarrollador**. No hay una única respuesta correcta, pero sí hay que **justificar** con argumentos técnicos.

### Pregunta tipo: "¿Se debería permitir agregar 2 veces el mismo producto?"

**Respuesta modelo:**
> **No**, no debería permitirse agregar el mismo producto dos veces a la wishlist. La wishlist funciona como un **conjunto** (set) de productos deseados, no como un carrito de compras (donde la cantidad importa). Agregar duplicados:
> - No aporta información útil al usuario.
> - Dificulta la gestión visual (el usuario ve el mismo producto repetido).
> - Complica la lógica de eliminación (¿eliminar uno o todos?).
>
> **Implementación:** Se valida la unicidad tanto en el backend (service → `ConflictError 409`) como opcionalmente en la BD (índice único o validador custom).

### Pregunta tipo: "¿Tiene sentido el mismo producto pero de distinto vendedor?"

**Respuesta modelo:**
> **Sí**, tiene sentido. Un mismo producto (ej: "iPhone 16") puede tener distintos vendedores con diferente precio, tiempo de envío, y reputación. Para el usuario es útil guardar en su wishlist la **oferta específica** de un vendedor.
>
> **Impacto en el modelo:** La clave de unicidad no es solo `productoId`, sino la combinación `(productoId, vendedorId)`. El modelo embedded quedaría:
> ```javascript
> wishlist: [{
>     producto: { type: ObjectId, ref: 'Producto' },
>     vendedor: { type: ObjectId, ref: 'Vendedor' },
>     fechaAgregado: Date,
> }]
> ```
> Y la validación de duplicados debe considerar ambos campos.

### Pregunta tipo: "¿En qué funcionalidades ya desarrolladas impacta la wishlist?"

**Respuesta modelo:**
> La wishlist impacta en:
> 1. **Detalle de producto:** Se debe mostrar si el producto ya está en la wishlist (ícono relleno vs vacío). Requiere consultar la wishlist del usuario al cargar la página.
> 2. **Baja de producto (admin):** Si un admin da de baja un producto que está en wishlists de usuarios, se debe decidir si se elimina automáticamente de las wishlists o se marca como "no disponible".
> 3. **Notificaciones:** Se podría notificar al usuario cuando un producto de su wishlist baje de precio o tenga stock disponible.
> 4. **Perfil del usuario:** Se podría agregar un contador "N productos en tu wishlist" en el sidebar o dashboard.

---

## Ejemplo completo resuelto: Wishlist para Sweet Medical

> Adaptando el ejercicio de "Tienda Sol" al dominio del TP.
> **Escenario:** Agregar una funcionalidad de "Lista de Espera" para turnos. Si un turno con un médico específico no tiene horarios disponibles, el paciente puede anotarse en una lista de espera. Cuando se libere un turno, se le notifica.

### Frontend

**Pantallas modificadas:**
- `BuscarTurno.jsx`: Agregar botón "Anotarme en lista de espera" cuando no hay turnos disponibles para el médico buscado.

**Pantallas nuevas:**
- `MiListaEspera.jsx`: Muestra los médicos/servicios en los que el paciente está anotado.

**Componentes nuevos:**
- `ListaEsperaCard.jsx`: Card que muestra médico, servicio, y posición en la lista.

**Accesibilidad:**
- `aria-live="polite"` en el mensaje de confirmación de inscripción.
- `aria-label="Anotarme en lista de espera para Dr. [nombre]"` en el botón.

### Backend

| Verbo | URL | Body | Descripción |
|-------|-----|------|-------------|
| `GET` | `/pacientes/:pacienteId/lista-espera` | — | Ver mis inscripciones |
| `POST` | `/pacientes/:pacienteId/lista-espera` | `{ medicoId, servicioId }` | Anotarse |
| `DELETE` | `/pacientes/:pacienteId/lista-espera/:inscripcionId` | — | Desanotarse |

**Validación (service):**
```javascript
async anotarse(pacienteId, medicoId, servicioId) {
    const paciente = await this.pacienteService.obtenerPorId(pacienteId);
    const medico = await this.medicoService.obtenerPorId(medicoId);

    const yaAnotado = await this.listaEsperaRepository
        .buscar({ pacienteId, medicoId, servicioId });

    if (yaAnotado) {
        throw new ConflictError("Ya estás en la lista de espera para este médico y servicio");
    }

    return await this.listaEsperaRepository.guardar({
        paciente: pacienteId,
        medico: medicoId,
        servicio: servicioId,
        fechaInscripcion: new Date(),
        estado: "ESPERANDO",
    });
}
```

### Persistencia
**Colección nueva** `ListaEspera` (porque necesitamos queries independientes y el array podría crecer):
```javascript
const ListaEsperaSchema = new mongoose.Schema({
    paciente: { type: ObjectId, ref: 'Paciente', required: true },
    medico: { type: ObjectId, ref: 'Medico', required: true },
    servicio: { type: ObjectId, ref: 'Servicio', required: true },
    fechaInscripcion: { type: Date, default: Date.now },
    estado: { type: String, enum: ["ESPERANDO", "NOTIFICADO", "ASIGNADO"], default: "ESPERANDO" },
});
ListaEsperaSchema.index({ paciente: 1, medico: 1, servicio: 1 }, { unique: true });
```

---

## Checklist final antes de entregar

### UI / Frontend
- [ ] ¿Dibujé las pantallas nuevas Y las modificadas?
- [ ] ¿Mencioné componentes nuevos y reutilizados?
- [ ] ¿Describí la interacción paso a paso (flujo feliz + errores)?
- [ ] ¿Incluí aspectos de accesibilidad? (`aria-live`, `aria-label`, `role`, teclado, contraste)

### Backend
- [ ] ¿Cada endpoint tiene verbo, URL, body, y query params?
- [ ] ¿Las URLs siguen el estilo RESTful del TP? (sustantivos plurales, anidamiento lógico)
- [ ] ¿Describí la lógica del service paso a paso?
- [ ] ¿Identifiqué todos los errores posibles con su status HTTP?
- [ ] ¿El test de integración cubre escenario feliz + al menos 2 errores?
- [ ] ¿La validación anti-duplicados está en el código/pseudocódigo?

### Persistencia
- [ ] ¿Elegí embedded vs colección nueva con justificación?
- [ ] ¿Mencioné la alternativa que NO elegí y por qué la descarté?
- [ ] ¿El schema tiene validaciones relevantes?
- [ ] ¿Definí índices útiles? (especialmente índice único si previene duplicados)

### Funcionalidad
- [ ] ¿Cada respuesta tiene una justificación técnica?
- [ ] ¿Consideré el impacto en funcionalidades existentes?
- [ ] ¿Mi decisión es coherente con la arquitectura del TP?

---

> [!IMPORTANT]
> **Consejo general:** Ante la duda en el parcial, **siempre justificá tu decisión**. Los profesores valoran más una respuesta técnicamente argumentada (aunque no sea la "ideal") que una respuesta correcta sin fundamentación. Usá la terminología de la materia: "capa de servicio", "orquestación", "validación de dominio", "índice compuesto", "consistencia eventual", etc.
