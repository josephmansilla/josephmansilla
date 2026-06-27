# 📘 Cheatsheet — React Hooks, Efectos y Accesibilidad

> Guía de estudio para el parcial de Desarrollo de Software.
> Los ejemplos usan código real del TP **Sweet Medical**.

---

## Tabla de Contenidos

1. [useState](#1-usestate)
2. [useEffect](#2-useeffect)
3. [useMemo](#3-usememo)
4. [useCallback](#4-usecallback)
5. [useRef](#5-useref)
6. [setInterval y setTimeout](#6-setinterval-y-settimeout)
7. [Event Listeners (addEventListener)](#7-event-listeners-addeventlistener)
8. [Cancelación de Fetch (AbortController)](#8-cancelación-de-fetch-abortcontroller)
9. [Accesibilidad (a11y) — Atributos ARIA](#9-accesibilidad-a11y--atributos-aria)
10. [Resumen rápido: Ciclo de vida vs Hooks](#10-resumen-rápido-ciclo-de-vida-vs-hooks)
11. [Errores comunes de examen](#11-errores-comunes-de-examen)

---

## 1. `useState`

### Definición
Hook que permite a un **componente funcional** tener estado interno. Devuelve un par `[valor, setValor]`. Cuando se llama al setter, React **re-renderiza** el componente con el nuevo valor.

### Sintaxis
```jsx
const [estado, setEstado] = useState(valorInicial);
```

### Reglas clave
| Regla | Detalle |
|-------|---------|
| Valor inicial | Puede ser cualquier tipo: `string`, `number`, `boolean`, `array`, `object`, `null`, o una **función** (lazy initializer). |
| Inmutabilidad | **Nunca mutar** el estado directamente. Siempre crear una copia nueva (spread, `.map()`, `.filter()`). |
| Batching | React agrupa múltiples `setState` del mismo evento en **un solo re-render** (batching). |
| Actualizaciones basadas en estado previo | Usar la forma funcional: `setCount(prev => prev + 1)` cuando el nuevo valor depende del anterior. |

### Ejemplo del TP (PatientLayout.jsx)
```jsx
// Estado simple: booleano
const [loading, setLoading] = useState(true);

// Estado simple: string para errores
const [error, setError] = useState("");

// Estado complejo: objeto con múltiples propiedades
const [data, setData] = useState({
    medicos: [],
    sedes: [],
    servicios: [],
    turnos: [],
    turnosDisponibles: [],
    paciente: null,
});
```

### Lazy Initializer (valor inicial costoso)
```jsx
// ❌ Se ejecuta en CADA render (costoso)
const [user, setUser] = useState(JSON.parse(localStorage.getItem("user")));

// ✅ Se ejecuta SOLO en el primer render
const [user, setUser] = useState(() => JSON.parse(localStorage.getItem("user")));
```
> **En el TP:** `App.js` usa esto: `useState(getStoredUser)` — pasa la función *sin invocarla* para que solo se ejecute la primera vez.

---

## 2. `useEffect`

### Definición
Hook que permite ejecutar **efectos secundarios** (side effects) en componentes funcionales: llamadas a APIs, suscripciones, timers, manipulación del DOM, etc. Es el reemplazo de `componentDidMount`, `componentDidUpdate` y `componentWillUnmount`.

### Sintaxis
```jsx
useEffect(() => {
    // Código del efecto (se ejecuta DESPUÉS del render)

    return () => {
        // Función de limpieza (cleanup) — OPCIONAL
        // Se ejecuta ANTES de que el efecto se vuelva a ejecutar
        // o cuando el componente se desmonta.
    };
}, [dependencia1, dependencia2]); // Array de dependencias
```

### Comportamiento según el array de dependencias

| Array de dependencias | ¿Cuándo se ejecuta? | Equivalente en clases |
|-----------------------|----------------------|----------------------|
| `[]` (vacío) | **Solo al montar** (1 vez) | `componentDidMount` |
| `[a, b]` (con valores) | Al montar **y cada vez que `a` o `b` cambien** | `componentDidMount` + `componentDidUpdate` (condicional) |
| Sin array (omitido) | **En cada render** ⚠️ | `componentDidMount` + `componentDidUpdate` (siempre) |

### ⚠️ Peligro: sin array de dependencias
```jsx
// ❌ PELIGRO: bucle infinito si llamas a setState adentro
useEffect(() => {
    fetchData().then(data => setData(data));
    // setData cambia estado → re-render → useEffect se ejecuta → setData → re-render → ...
});

// ✅ Correcto: con array vacío, se ejecuta solo una vez
useEffect(() => {
    fetchData().then(data => setData(data));
}, []);
```

### Ejemplo del TP (PatientLayout.jsx)
```jsx
// Se ejecuta al montar y cada vez que cambie "user"
useEffect(() => {
    loadData(); // Llama a la API para traer médicos, sedes, turnos, etc.
}, [user]);
```

### Función de limpieza (Cleanup)
Se ejecuta en dos momentos:
1. **Antes de re-ejecutar** el efecto (cuando cambian las dependencias).
2. **Cuando el componente se desmonta** (se remueve del DOM).

```jsx
useEffect(() => {
    const intervalId = setInterval(() => {
        verificarEstado();
    }, 5000);

    // CLEANUP: limpiar el timer cuando el componente se desmonte
    return () => {
        clearInterval(intervalId);
    };
}, [turnoId]);
```

---

## 3. `useMemo`

### Definición
Hook que **memoriza el resultado** de un cálculo costoso. Solo lo recalcula cuando cambian sus dependencias. Evita recálculos innecesarios en cada render.

### Sintaxis
```jsx
const valorMemorizado = useMemo(() => {
    return calculoCostoso(a, b);
}, [a, b]);
```

### Ejemplo del TP (MisTurnos.jsx)
```jsx
// Solo recalcula la separación próximos/historial cuando cambia "turnos"
const { proximos, historial } = useMemo(() => {
    return turnos.reduce(
        (groups, turno) => {
            if (isUpcoming(turno)) {
                groups.proximos.push(turno);
            } else {
                groups.historial.push(turno);
            }
            return groups;
        },
        { proximos: [], historial: [] },
    );
}, [turnos]);
```

### Ejemplo del TP (PatientLayout.jsx)
```jsx
// Solo re-filtra los turnos del paciente cuando cambian los datos
const turnosPaciente = useMemo(() => {
    const pacienteId = getEntityId(data.paciente);
    if (!pacienteId) return [];
    return data.turnos.filter((turno) => turno.pacienteId === pacienteId);
}, [data.paciente, data.turnos]);
```

### ¿Cuándo usarlo?
| Usar | No usar |
|------|---------|
| Filtros/ordenamientos sobre listas grandes | Cálculos triviales (sumar dos números) |
| Crear objetos derivados que se pasan como props | Cuando el componente ya no tiene problemas de rendimiento |
| Evitar renders innecesarios en componentes hijos | Por defecto "por las dudas" (premature optimization) |

---

## 4. `useCallback`

### Definición
Hook que **memoriza una función**. Devuelve la misma referencia de función entre renders, a menos que cambien las dependencias. Es similar a `useMemo` pero específico para funciones.

### Sintaxis
```jsx
const funcionMemorizada = useCallback(() => {
    // lógica de la función
}, [dependencia1, dependencia2]);
```

### ¿Cuándo usarlo?
- Cuando pasas una función como **prop** a un componente hijo que usa `React.memo()`.
- Cuando la función se usa como **dependencia** de un `useEffect`.

```jsx
// Sin useCallback: handleClick es una función NUEVA en cada render
const handleClick = () => { setCount(c => c + 1); };

// Con useCallback: handleClick mantiene la misma referencia
const handleClick = useCallback(() => {
    setCount(c => c + 1);
}, []); // [] porque no depende de nada externo
```

### Relación con useMemo
```jsx
// Estas dos líneas son EQUIVALENTES:
const fn = useCallback(() => doSomething(a, b), [a, b]);
const fn = useMemo(() => () => doSomething(a, b), [a, b]);
```

---

## 5. `useRef`

### Definición
Hook que crea una **referencia mutable** que persiste durante toda la vida del componente. A diferencia de `useState`, cambiar un `ref` **NO provoca re-render**.

### Sintaxis
```jsx
const miRef = useRef(valorInicial);
// Acceder: miRef.current
// Modificar: miRef.current = nuevoValor (no causa re-render)
```

### Usos principales
| Uso | Ejemplo |
|-----|---------|
| Acceder a elementos del DOM | `<input ref={inputRef} />` → `inputRef.current.focus()` |
| Guardar valores entre renders sin causar re-render | Timer IDs, valores previos, contadores internos |
| Guardar la instancia de un AbortController | Para cancelar fetch requests |

```jsx
const inputRef = useRef(null);

const handleClick = () => {
    inputRef.current.focus(); // Enfoca el input sin re-render
};

return <input ref={inputRef} type="text" />;
```

---

## 6. `setInterval` y `setTimeout`

### Definiciones

| Función | Qué hace | Retorna |
|---------|----------|---------|
| `setTimeout(fn, ms)` | Ejecuta `fn` **una sola vez** después de `ms` milisegundos | Un ID numérico |
| `setInterval(fn, ms)` | Ejecuta `fn` **repetidamente** cada `ms` milisegundos | Un ID numérico |
| `clearTimeout(id)` | **Cancela** un `setTimeout` pendiente | `undefined` |
| `clearInterval(id)` | **Cancela** un `setInterval` activo | `undefined` |

### ⚠️ Regla fundamental: SIEMPRE limpiar en el cleanup del useEffect

```jsx
// ✅ Correcto: limpiar setInterval
useEffect(() => {
    const id = setInterval(() => {
        console.log("tick");
    }, 1000);

    return () => clearInterval(id); // CLEANUP
}, []);

// ✅ Correcto: limpiar setTimeout
useEffect(() => {
    const id = setTimeout(() => {
        setMostrarMensaje(true);
    }, 3000);

    return () => clearTimeout(id); // CLEANUP
}, []);
```

### ❌ Error clásico de examen: memory leak
```jsx
// ❌ INCORRECTO: no limpiar el intervalo → memory leak
useEffect(() => {
    setInterval(() => {
        fetchTurnos(); // Sigue ejecutándose incluso después de desmontar el componente
    }, 5000);
}, []);
// El intervalo NUNCA se cancela → el componente "fantasma" sigue haciendo requests

// ✅ CORRECTO
useEffect(() => {
    const id = setInterval(() => {
        fetchTurnos();
    }, 5000);
    return () => clearInterval(id);
}, []);
```

---

## 7. Event Listeners (`addEventListener`)

### Definición
Método nativo del DOM que **registra una función** que se ejecutará cuando ocurra un evento específico en un elemento.

### Sintaxis nativa
```javascript
element.addEventListener(tipoEvento, funcionHandler);
element.removeEventListener(tipoEvento, funcionHandler);
```

### Uso dentro de useEffect
```jsx
useEffect(() => {
    const handleResize = () => {
        setWindowWidth(window.innerWidth);
    };

    // SUSCRIBIR al evento
    window.addEventListener("resize", handleResize);

    // CLEANUP: DESUSCRIBIR al desmontar
    return () => {
        window.removeEventListener("resize", handleResize);
    };
}, []);
```

### ⚠️ Error clásico: función anónima en removeEventListener
```jsx
// ❌ INCORRECTO: no se puede remover porque es una función diferente
useEffect(() => {
    window.addEventListener("resize", () => { setWidth(window.innerWidth); });
    return () => {
        window.removeEventListener("resize", () => { setWidth(window.innerWidth); });
        // ↑ Esta es una NUEVA función, no la misma que se registró
    };
}, []);

// ✅ CORRECTO: misma referencia de función
useEffect(() => {
    const handler = () => { setWidth(window.innerWidth); };
    window.addEventListener("resize", handler);
    return () => window.removeEventListener("resize", handler);
}, []);
```

### Eventos comunes
| Evento | Dispara cuando... |
|--------|-------------------|
| `click` | El usuario hace clic |
| `keydown` / `keyup` | Se presiona/suelta una tecla |
| `resize` | Cambia el tamaño de la ventana |
| `scroll` | El usuario hace scroll |
| `focus` / `blur` | Un elemento obtiene/pierde el foco |
| `submit` | Se envía un formulario |
| `change` | Cambia el valor de un input/select |

### En React (eventos sintéticos)
React provee su propio sistema de eventos (SyntheticEvent). Se usan directamente en JSX:
```jsx
<button onClick={handleClick}>Clic</button>
<input onChange={(e) => setQuery(e.target.value)} />
<form onSubmit={handleSubmit}>...</form>
```
> **Diferencia clave:** Los eventos de React se auto-limpian, no necesitan `removeEventListener`. Solo necesitás cleanup cuando usás `addEventListener` nativo (ej: `window`, `document`).

---

## 8. Cancelación de Fetch (`AbortController`)

### Definición
`AbortController` es una API nativa del navegador que permite **cancelar requests HTTP** (fetch) que ya están en curso. Es vital para evitar actualizar el estado de un componente que ya se desmontó.

### Sintaxis
```javascript
const controller = new AbortController();
const signal = controller.signal;

fetch(url, { signal })
    .then(res => res.json())
    .then(data => setData(data))
    .catch(err => {
        if (err.name === "AbortError") {
            console.log("Fetch cancelado");
        }
    });

// Para cancelar:
controller.abort();
```

### Uso correcto dentro de useEffect
```jsx
useEffect(() => {
    const controller = new AbortController();

    const fetchTurnos = async () => {
        try {
            const response = await fetch("/api/turnos", {
                signal: controller.signal,
            });
            const data = await response.json();
            setTurnos(data);
        } catch (error) {
            // Ignorar errores de cancelación
            if (error.name !== "AbortError") {
                setError(error.message);
            }
        }
    };

    fetchTurnos();

    // CLEANUP: cancelar el fetch si el componente se desmonta
    // o si las dependencias cambian antes de que termine
    return () => {
        controller.abort();
    };
}, [medicoId]); // Se cancela y re-ejecuta si cambia medicoId
```

### Con Axios (como en el TP)
```jsx
useEffect(() => {
    const controller = new AbortController();

    const loadData = async () => {
        try {
            const response = await axios.get("/turnos", {
                signal: controller.signal,
            });
            setTurnos(response.data);
        } catch (error) {
            if (!axios.isCancel(error)) {
                setError(error.message);
            }
        }
    };

    loadData();
    return () => controller.abort();
}, []);
```

### ¿Por qué es necesario?
Sin cancelación, si el usuario navega a otra pantalla antes de que termine el fetch:
1. El fetch termina en background.
2. Intenta llamar a `setState` en un componente que ya no existe.
3. React muestra un **warning**: "Can't perform a React state update on an unmounted component."
4. Potencial **memory leak**.

---

## 9. Accesibilidad (a11y) — Atributos ARIA

### Definición
**ARIA** (Accessible Rich Internet Applications) es un conjunto de atributos HTML que mejoran la accesibilidad de aplicaciones web para personas con discapacidades (lectores de pantalla, navegación por teclado, etc.).

### Atributos ARIA más importantes

| Atributo | Propósito | Ejemplo |
|----------|-----------|---------|
| `role` | Define el **rol semántico** del elemento | `role="button"`, `role="status"`, `role="alert"` |
| `aria-label` | Etiqueta textual para elementos sin texto visible | `aria-label="Cerrar modal"` |
| `aria-labelledby` | Referencia al `id` de otro elemento que lo describe | `aria-labelledby="titulo-seccion"` |
| `aria-describedby` | Referencia a una descripción adicional | `aria-describedby="instrucciones"` |
| `aria-live` | Anuncia cambios dinámicos al lector de pantalla | `aria-live="polite"`, `aria-live="assertive"` |
| `aria-hidden` | Oculta el elemento del lector de pantalla | `aria-hidden="true"` (ej: íconos decorativos) |
| `aria-expanded` | Indica si un elemento colapsable está abierto | `aria-expanded="true"` |
| `aria-disabled` | Indica que un elemento está deshabilitado | `aria-disabled="true"` |
| `aria-required` | Indica que un campo es obligatorio | `aria-required="true"` |
| `aria-invalid` | Indica que un campo tiene un error de validación | `aria-invalid="true"` |
| `aria-busy` | Indica que una región está cargando contenido | `aria-busy="true"` |

### `aria-live` (regiones vivas) — Muy importante para parcial

| Valor | Comportamiento |
|-------|---------------|
| `"off"` | No anuncia cambios (default). |
| `"polite"` | Anuncia cambios **cuando el usuario está inactivo** (no interrumpe). |
| `"assertive"` | Anuncia cambios **inmediatamente**, interrumpiendo lo que el lector esté leyendo. |

```jsx
{/* Cuando el texto cambie, el lector de pantalla lo anuncia */}
<div aria-live="polite">
    {loading ? "Cargando turnos..." : `${turnos.length} turnos encontrados`}
</div>

{/* Para alertas urgentes de error */}
<div role="alert" aria-live="assertive">
    {error && `Error: ${error}`}
</div>
```

### Elementos semánticos de HTML5 (preferir sobre ARIA)
> **Regla de oro:** Si existe un elemento HTML nativo que exprese la semántica, **usarlo en vez de ARIA**.

| En vez de esto... | Usar esto |
|--------------------|-----------|
| `<div role="button">` | `<button>` |
| `<div role="navigation">` | `<nav>` |
| `<div role="main">` | `<main>` |
| `<div role="heading">` | `<h1>` a `<h6>` |
| `<span role="link">` | `<a>` |

### Navegación por teclado
```jsx
<button
    onClick={handleReservar}
    onKeyDown={(e) => {
        if (e.key === "Enter" || e.key === " ") {
            handleReservar();
        }
    }}
    tabIndex={0} // Hace el elemento enfocable con Tab
    aria-label="Reservar turno con Dr. Pérez"
>
    Reservar
</button>
```

---

## 10. Resumen rápido: Ciclo de vida vs Hooks

```
┌─────────────────────────────┐
│    CICLO DE VIDA (Clases)   │     HOOKS (Funcional)
├─────────────────────────────┤
│ constructor()               │  → useState(initialValue)
│ componentDidMount()         │  → useEffect(() => {...}, [])
│ componentDidUpdate()        │  → useEffect(() => {...}, [dep])
│ componentWillUnmount()      │  → useEffect(() => { return () => cleanup }, [])
│ shouldComponentUpdate()     │  → React.memo() + useMemo()
│ this.state / this.setState  │  → useState / setter
│ createRef()                 │  → useRef()
└─────────────────────────────┘
```

---

## 11. Errores comunes de examen

### 1. Dependencias faltantes en useEffect
```jsx
// ❌ Bug: el efecto usa "userId" pero no está en las dependencias
useEffect(() => {
    fetch(`/api/turnos?user=${userId}`);
}, []); // userId podría cambiar y el efecto no se re-ejecuta

// ✅ Correcto
useEffect(() => {
    fetch(`/api/turnos?user=${userId}`);
}, [userId]);
```

### 2. Mutar estado directamente
```jsx
// ❌ Mutación directa (React NO detecta el cambio → no re-renderiza)
const agregarTurno = (nuevoTurno) => {
    turnos.push(nuevoTurno);
    setTurnos(turnos); // Misma referencia de array → React ignora el cambio
};

// ✅ Crear nueva referencia con spread
const agregarTurno = (nuevoTurno) => {
    setTurnos([...turnos, nuevoTurno]);
};

// ✅ O con forma funcional (más seguro)
const agregarTurno = (nuevoTurno) => {
    setTurnos(prev => [...prev, nuevoTurno]);
};
```

### 3. Olvidar la limpieza de efectos
```jsx
// ❌ Memory leak: el timer sigue vivo después de desmontar
useEffect(() => {
    setInterval(fetchData, 5000);
}, []);

// ❌ Memory leak: el listener sigue vivo
useEffect(() => {
    window.addEventListener("resize", handleResize);
}, []);

// ❌ Warning: actualizar estado de componente desmontado
useEffect(() => {
    fetch("/api/data").then(r => r.json()).then(setData);
}, []);
```

### 4. Llamar hooks condicionalmente
```jsx
// ❌ PROHIBIDO: los hooks deben llamarse siempre en el mismo orden
if (user) {
    const [nombre, setNombre] = useState(user.nombre);
}

// ❌ PROHIBIDO: hooks dentro de loops
for (const item of items) {
    useEffect(() => { ... }, [item]);
}

// ✅ La condición va DENTRO del hook, no afuera
const [nombre, setNombre] = useState("");
useEffect(() => {
    if (user) {
        setNombre(user.nombre);
    }
}, [user]);
```

### 5. Comparación de objetos/arrays en dependencias
```jsx
// ❌ Bug sutil: el objeto se recrea en cada render → efecto se ejecuta infinitamente
const filtros = { estado: "DISPONIBLE" }; // Nueva referencia cada render
useEffect(() => {
    fetchTurnos(filtros);
}, [filtros]); // Siempre es "diferente" → loop infinito

// ✅ Usar useMemo o primitivos individuales
const filtros = useMemo(() => ({ estado: "DISPONIBLE" }), []);
useEffect(() => {
    fetchTurnos(filtros);
}, [filtros]);

// ✅ O desestructurar en primitivos
useEffect(() => {
    fetchTurnos({ estado });
}, [estado]); // string primitivo → comparación correcta
```

---

> [!TIP]
> **Para el parcial:** Ante la duda, siempre mencioná la **función de limpieza (cleanup)** del `useEffect`. Es el concepto que más puntos otorga y el que más alumnos olvidan.
