# Node.js, React y Next.js

## 1. NODE.JS

### Qué es

Entorno para ejecutar JavaScript **fuera del navegador**. Coge el motor V8 de Chrome y le añade lo que un servidor necesita: ficheros, red, procesos. Es un **runtime**: Express, Next y NestJS son frameworks que corren encima.

### El modelo de ejecución: un solo hilo + event loop

Node ejecuta tu código en **un único hilo**. Cuando una operación es lenta (BD, HTTP, disco), Node **la lanza y no espera**: el hilo queda libre para atender otras peticiones, y cuando llega la respuesta, el event loop retoma el código donde se quedó. El `await` es el punto donde el hilo se libera.

```
petición 1 ──┐ lanza BD
petición 2 ──┤ lanza BD        (mismo hilo, no espera)
petición 3 ──┤ lanza BD
             └─► van llegando respuestas y se atienden
```

**I/O (Input/Output):** cualquier operación en la que tu código pide algo "fuera" y espera: consulta a BD, llamada HTTP, leer fichero, red. Durante la espera tu código no hace nada.

| | I/O-bound | CPU-bound |
|---|---|---|
| El tiempo se lo lleva... | **esperar** a sistemas externos | **tu propio código calculando** |
| Ejemplo | Endpoint que consulta BD y llama a otra API | Procesar imágenes, cifrar, motor de cálculo pesado |
| ¿Se libera el hilo? | Sí (en el `await`) | **No**: no hay nada a lo que esperar |
| Node es... | Muy eficiente (muchas conexiones con poca memoria) | Malo: **bloquea el event loop** |

```TypeScript
app.get("/usuarios", async (req, res) => {
  const usuarios = await db.query("...");  // I/O: aquí el hilo se libera
  res.json(usuarios);
});

app.get("/informe", (req, res) => {
  const r = calcularInforme(datos);        // CPU: 3 s de bucles → nadie más es atendido
  res.json(r);
});
```

**Regla nº 1 de Node: nunca bloquees el event loop.** Un bucle pesado, un JSON gigante parseado de golpe o criptografía síncrona tumban el servidor entero.

Analogía: un camarero (el hilo). Un plato (I/O) lo pasa a cocina y atiende otras mesas. Un sudoku (CPU) lo tiene que resolver él y nadie más es atendido.

### Proceso vs hilo

- **Proceso**: un programa en ejecución con su propia memoria aislada (`node app.js` crea uno). Dos procesos no comparten memoria; se comunican por red, colas, ficheros.
- **Hilo**: línea de ejecución **dentro** de un proceso. Los hilos de un proceso comparten memoria.

### Soluciones para CPU-bound

| | Worker threads | Otro servicio (con cola) |
|---|---|---|
| Dónde corre | Mismo proceso, otro hilo | Otro proceso / máquina |
| Memoria | No comparten variables (se pasan mensajes), sí el presupuesto de RAM del proceso | Totalmente separada |
| Comunicación | Mensajes internos | Red: HTTP o cola |
| Si el cálculo peta | Puede tumbar el proceso | La API ni se entera |
| Cuándo | Cálculos puntuales, medianos | Pesados, frecuentes, críticos |

```TypeScript
const worker = new Worker("./calcular.js", { workerData: datos });
worker.on("message", (resultado) => res.json(resultado)); // el hilo principal sigue libre
```

**Idea clave:** sacar el cálculo a un worker o a otro servicio **convierte una tarea CPU-bound en I/O-bound** para tu API: tu hilo ya no calcula, **pide y espera**, que es lo que Node hace bien.

**Streams:** para ficheros grandes, procesar a trozos según llegan (`fs.createReadStream`) en vez de cargar 2 GB en memoria y parsearlos (CPU).

### npm y package.json

```json
{
  "name": "mi-app",
  "version": "1.4.2",
  "type": "module",
  "scripts": { "dev": "next dev", "build": "next build", "start": "next start", "test": "vitest", "lint": "eslint ." },
  "dependencies": { "next": "^16.0.0" },
  "devDependencies": { "typescript": "^5.6.0" }
}
```

- **`dependencies`**: lo que hace falta en producción. **`devDependencies`** (`npm install -D x`): solo para desarrollar (TS, tests, linter).
- **`version`**: la de tu propio artefacto. Importa mucho si publicas una librería; en una aplicación es informativa.
- **`node_modules`**: donde se descargan los paquetes. Nunca se sube a Git.
- **`package-lock.json`**: fija las versiones exactas resueltas. **Sí se sube a Git**. En CI se usa `npm ci` (instala lo que dice el lock, sin resolver).

**Rangos de versión:**

| Sintaxis | Acepta | Con `5.6.0` |
|---|---|---|
| `5.6.0` | Exacta | solo `5.6.0` |
| `^5.6.0` (defecto de npm) | Misma major: minor y patch nuevos | `5.6.1`, `5.9.0`, no `6.0.0` |
| `~5.6.0` | Misma minor: solo patch | `5.6.3`, no `5.7.0` |

### Versionado semántico (SemVer)

`MAJOR.MINOR.PATCH`. Cada número **comunica el tipo de cambio** a quien te consume:

| Subes | Cuando | Promesa |
|---|---|---|
| PATCH | Arreglas un bug sin cambiar la API | "Actualiza sin mirar" |
| MINOR | Añades funcionalidad **sin romper** | "Hay cosas nuevas, lo tuyo sigue funcionando" |
| MAJOR | Cambio **no retrocompatible** (*breaking change*) | "Puede romperte: lee la guía de migración" |

**Qué es breaking:** código que funcionaba puede dejar de funcionar.

| Cambio | ¿Breaking? |
|---|---|
| Añadir campo a la respuesta / endpoint nuevo / parámetro opcional | No (minor) |
| Corregir un cálculo | No (patch) |
| Renombrar o eliminar un campo | **Sí** |
| Hacer obligatorio un parámetro opcional | **Sí** |
| Cambiar el tipo de un campo (`"12"` → `12`) | **Sí** |
| Cambiar el significado (euros → céntimos) | **Sí**, el peor: falla en silencio |

**Regla:** añadir es seguro; quitar o cambiar es breaking. Todo el ecosistema npm (con `^`) se sostiene sobre esa promesa: un breaking en una minor rompe a cualquiera al reinstalar.

**Aplicado a APIs REST:**
1. Añadir nunca rompe.
2. Si hay que romper: subir major y **convivir** `/v1` y `/v2`, con deprecación anunciada y fecha de retirada.
3. **Expand/contract** si no quieres dos versiones: expandir (añadir el campo nuevo junto al viejo) → migrar consumidores → contraer (eliminar el viejo). Cada paso es retrocompatible. Nota: no puedes "devolver ambos" en la misma clave; el nuevo necesita clave nueva.
4. **Feature flags** para cambios de comportamiento: se despliega apagado, se activa por cliente/entorno/porcentaje, se apaga sin desplegar.
5. El contrato OpenAPI lleva su `version` y comparando contratos se detectan breaking changes en CI.

**Historia para la entrevista:** "En mi equipo se colaban cambios no retrocompatibles en minors, así que el número no era fiable. Yo aplicaría SemVer estricto: añadir es minor; quitar o renombrar es major con v1/v2 conviviendo, o expand/contract; feature flags para comportamiento."

### Scripts de npm

`scripts` es un diccionario **nombre corto → comando de terminal**. `npm run dev` = "npm, ejecuta el script llamado dev" → lanza `next dev`.

Por qué existen: (1) documentan cómo se usa el proyecto; (2) los ejecutables de `node_modules/.bin/` (`next`, `tsc`, `eslint`) **no están en el PATH del sistema**; dentro de un script npm los encuentra (igual que `npx`).

- `npm start`, `npm test`, `npm stop`, `npm restart` no necesitan `run`.
- Argumentos extra: `npm run test -- --watch`.

| Script | Hace |
|---|---|
| `dev` | Arranca en desarrollo con recarga en caliente (transpila en memoria, no genera artefacto) |
| `build` | Compila para producción (genera `.next/` o `dist/`) |
| `start` | Arranca lo ya construido |
| `test` / `lint` / `format` / `typecheck` | Tests / ESLint / Prettier / `tsc --noEmit` |

Flujo: local `npm install` + `npm run dev`; CI `npm ci` → lint → test → build; producción `npm start`. Nunca se despliega con `dev`.

### Módulos (import / export)

Cada fichero es un módulo. **Todo es privado por defecto**: solo se ve lo que lleva `export`.

```TypeScript
// usuario.ts
export type Usuario = { id: number; nombre: string };     // named exports: tantos como quieras
export const crearUsuario = (n: string): Usuario => ({ id: Date.now(), nombre: n });
export default { timeout: 3000 };                          // default: UNO por fichero, puede ser una expresión
const interno = 42;                                        // sin export: invisible

// app.ts
import config, { crearUsuario } from "./usuario";          // default SIN llaves (nombre libre), named CON llaves (mismo nombre)
import type { Usuario } from "./usuario";                  // solo tipos: se borra al compilar
import { crearUsuario as crear } from "./usuario";         // renombrar con "as", no con ":"
import * as utils from "./usuario";                        // todo bajo un nombre
import express from "express";                             // sin ./ → paquete de node_modules
```

Las llaves del `import` **no son desestructuración**: es sintaxis propia. Si al importar sale `undefined`, casi siempre son llaves donde no tocan o al revés. `import { timeout }` de un default export **falla**: `timeout` es una propiedad del objeto default, no un named export.

| | CommonJS (CJS) | ES Modules (ESM) |
|---|---|---|
| Exportar / importar | `module.exports` / `require()` | `export` / `import` |
| Top-level `await` | No | Sí |
| Hoy | Legado | **Estándar** |

Node decide: `.mts`/`.mjs` → ESM; `.cts`/`.cjs` → CJS; `.ts`/`.js` → ESM solo si `package.json` tiene `"type": "module"` (si no, CJS). Por eso un `await` suelto en un `.ts` sin `type: module` falla y en `.mts` funciona.

- **Un módulo se ejecuta una sola vez** y se cachea: `export const db = crearConexion()` es un **singleton** gratis.
- Los imports son referencias vivas, no copias.
- Buenas prácticas: preferir named exports (salvo componentes/páginas donde la convención es default); `import type`; `index.ts` barrel con moderación; evitar imports circulares.

### Express

Framework HTTP **minimalista**: rutas, middlewares y poco más. La estructura la pones tú (ver arquitectura en capas). NestJS es la alternativa con estructura impuesta (decoradores, inyección de dependencias).

```TypeScript
import express from "express";
const app = express();
app.use(express.json());                               // middleware: parsea body JSON → req.body

app.get("/usuarios/:id", async (req, res) => {
  const u = await repo.buscar(req.params.id);
  if (!u) return res.status(404).json({ error: "No encontrado" });
  res.json(u);
});
app.listen(3000);
```

**Middleware:** función que se ejecuta **entre** la llegada de la petición y la respuesta, en una **cadena ordenada**. Cada uno puede: (1) hacer algo y pasar el turno con `next()`, (2) responder y cortar la cadena, (3) lanzar error / `next(err)`.

```
petición ──► logger ──► parseJSON ──► auth ──► ruta ──► respuesta
                                       └── sin token ──► 401 (corta)
```

**`(req, res, next)`:**
- `req`: la petición. `req.params.id`, `req.query.page`, `req.body`, `req.headers.authorization`, `req.method`, `req.path`.
- `res`: la respuesta. `res.status(404)` (fija código, no envía), `res.json(...)` / `res.send(...)` / `res.redirect(...)` (envían y cierran).
- `next()`: pasa al siguiente. **Si no respondes ni llamas a `next()`, la petición se queda colgada para siempre** (error nº 1).

```TypeScript
app.use((req, res, next) => { console.log(req.method, req.path); next(); });        // global
app.use("/admin", soloAdmin);                                                        // por prefijo
app.delete("/usuarios/:id", requiereAuth, requiereAdmin, (req, res) => { ... });     // por ruta
app.use((req, res, next) => { req.usuario = verificarToken(req.headers.authorization); next(); }); // pasar datos: req es el mismo objeto en toda la cadena

app.use((err, req, res, next) => {                 // manejador de errores: 4 parámetros, va el ÚLTIMO
  console.error(err);
  res.status(500).json({ error: "Error interno" });
});
```

El orden de `app.use` importa. En Express 5 los errores en rutas `async` se capturan solos; en Express 4 había que hacer `try/catch` + `next(err)`.

### Event Driven Architecture

**Petición-respuesta** (REST): un servicio le **ordena** a otro qué hacer y espera. **Event Driven**: un servicio **anuncia que ha pasado algo** (un evento) y quien quiera reacciona. El que anuncia no sabe quién escucha.

Un evento es **un hecho pasado, inmutable, en pasado verbal**: `OperacionAprobada`, `UsuarioRegistrado`. No es una orden.

```
Servicio operaciones ──► publica OperacionAprobada ──► broker ──┬──► consumidor push
                     └─► responde al instante                 ├──► consumidor email
                                                              └──► consumidor panel
```

El **broker** (Kafka, RabbitMQ, SQS, Redis Streams) guarda los eventos y los entrega a los suscritos.

**Ganas:** desacoplamiento (añadir SMS = añadir un consumidor), resiliencia (si email cae, los eventos esperan en el broker), rendimiento (respuesta inmediata), escalado independiente por consumidor.

**Pagas (lo que distingue a quien lo entiende):**
1. **Consistencia eventual**: el panel se actualiza "poco después", no en el mismo instante.
2. **Entrega at-least-once** → un evento **puede llegar dos veces** → cada consumidor debe ser **idempotente** (guardar id de evento procesado e ignorar repetidos).
3. **Doble escritura** (guardar en BD ✅, publicar 💥): se pierde el evento. Solución: **patrón outbox**: en la misma transacción se inserta el evento en una tabla `outbox`; un proceso aparte lo publica al broker.
4. Más difícil de depurar: ids de correlación, trazabilidad.

Sacar un cálculo pesado a una cola **es** Event Driven (`InformeSolicitado` → servicio de cálculo → `InformeGenerado`).

**Para la entrevista (notificaciones):** "Funcionaba por integración directa con las APIs del banco. Si lo diseñara hoy publicaría eventos de dominio con consumidores independientes por canal; ganaría desacoplamiento y resiliencia a cambio de consistencia eventual; cuidaría outbox e idempotencia porque la entrega es at-least-once."

---

## 2. REACT

### Qué es y por qué es declarativo

Librería para construir interfaces a partir de **componentes**. Solo hace eso (no rutas, no servidor, no datos): por eso existe Next.

- **Imperativo** (antes): describes **los pasos** para cambiar la pantalla (`getElementById`, `innerHTML = ""`, `createElement`, `appendChild`) y **tú** mantienes sincronizados datos y pantalla. Olvidar un "repintar" = bug.
- **Declarativo** (React): describes **cómo se ve la UI para unos datos** y React la actualiza cuando cambian. **`UI = f(estado)`**. Como SQL frente a recorrer tablas a mano, o `map`/`filter` frente a `for`+`if`+`push`.

Por debajo, al cambiar el estado React: (1) ejecuta tu función y obtiene la nueva descripción (Virtual DOM), (2) la compara con la anterior, (3) aplica al DOM real **solo las diferencias**.

### Componentes y JSX

Un componente es **una función que recibe props y devuelve JSX**. Empieza por **mayúscula** (así React distingue `<Boton>` de `<button>`).

```TypeScript
type Props = { nombre: string; edad: number };
function TarjetaUsuario({ nombre, edad }: Props) {
  return <div className="tarjeta"><h2>{nombre}</h2><p>{edad} años</p></div>;
}
// Uso: <TarjetaUsuario nombre="Ana" edad={28} />
```

**`<Comp prop={x} />` no llama a la función: se la entrega a React** (`createElement(Comp, { prop: x })`), que la llamará cuando toque con el objeto de props. Llamarla a mano (`{Comp({...})}`) funciona pero rompe hooks y estado propio. Nunca con paréntesis.

**JSX** es azúcar sintáctico que se compila a `createElement(...)`. Ficheros `.tsx`. Reglas:
- `{ }` admite **solo expresiones** (cosas que producen un valor), no sentencias (`if`, `for`). Una arrow function **es** una expresión (su valor es la función), aunque su cuerpo tenga sentencias.
- `class` → `className`, `for` → `htmlFor` (palabras reservadas de JS). El resto, camelCase: `onClick`, `tabIndex`, `maxLength`.
- **Un solo elemento raíz.** Para varios hermanos sin envoltorio real: fragmento `<>...</>` (no genera etiqueta; imprescindible en `<tr>`/`<td>`).
- `return ( ... )` con paréntesis si el JSX empieza en la línea siguiente (ASI).

| Quiero | Sentencia (no vale en JSX) | Expresión (sí) |
|---|---|---|
| Elegir entre dos | `if/else` | `cond ? A : B` |
| Pintar si se cumple | `if` | `cond && <A />` |
| Recorrer | `for` | `lista.map(...)` |

Lógica compleja: antes del `return`, con sentencias; en el JSX solo el valor.

**Trampa del `&&`:** `a && b` devuelve `a` si es falsy, si no `b`. React ignora `false/null/undefined/true`, pero **pinta `0`**. `{lista.length && <X/>}` con lista vacía pinta "0". Usar `{lista.length > 0 && <X/>}`.

### Props

React **empaqueta todos los atributos en un único objeto** y llama a tu función con él (props por nombre, opcionales, extensibles). Las llaves en la firma son **desestructuración** de ese objeto, no parte obligatoria:

```TypeScript
function Boton(props: BotonProps) { return <button onClick={props.onPulsar}>{props.texto}</button>; }
function Boton({ texto, onPulsar }: BotonProps) { ... }            // equivalente, convención
function Boton({ texto = "Aceptar", onPulsar }: BotonProps) { ... } // valores por defecto
function Input({ etiqueta, ...resto }: Props) { return <label>{etiqueta}<input {...resto} /></label>; } // rest + spread de atributos
```

`({ nombre, edad }: Props)` se lee: "recibo un objeto de tipo `Props` y saco `nombre` y `edad`". El tipo va **después de todo el patrón**. `({ nombre: string })` sería un **renombrado**, no un tipo.

- Las props son **de solo lectura**: un hijo nunca las modifica.
- **`key`**: prop reservada para React, no llega al componente. Identifica cada elemento de una lista entre renders. Debe ser **única entre hermanos y estable** (el `id`, no el índice).

### Estado: `useState`

```TypeScript
const [cuenta, setCuenta] = useState(0);
const [repos, setRepos] = useState<Repo[]>([]);   // genérico cuando el inicial no basta para inferir
```

**Regla central:** al llamar al setter, React **vuelve a ejecutar la función del componente** (re-render) y actualiza solo lo que cambió. Nunca tocas el DOM ni asignas `cuenta = ...`. Ciclo: estado → render → interacción → nuevo estado → render.

El re-render es **barato** (ejecutar una función + diff). Un render por tecla es normal. Solo es problema si **tú** metes algo caro en el cuerpo (→ `useMemo`).

**Forma funcional del setter:** `setCuenta((c) => c + 1)`. Obligatoria cuando el nuevo valor depende del anterior y el callback puede tener un valor viejo en su closure (timers, varios sets seguidos).

**Estado derivado no es estado:** si se puede calcular a partir del estado (`pendientes = tareas.filter(...).length`), se calcula en el render. Ni `useState` extra ni `useEffect`.

### Inmutabilidad del estado

React decide si repintar comparando el nuevo valor con el anterior con **`===`**. Para objetos y arrays, `===` compara **referencias**. Mutar (`push`, `obj.x = 1`) y pasar la misma referencia → "nada ha cambiado" → **no repinta**. Bug silencioso.

| Quiero | Mutable ❌ | Inmutable ✅ |
|---|---|---|
| Añadir | `lista.push(x)` | `[...lista, x]` |
| Quitar | `lista.splice(i,1)` | `lista.filter((e) => e.id !== id)` |
| Modificar uno | `lista[i].hecha = true` | `lista.map((e) => e.id === id ? { ...e, hecha: true } : e)` |
| Ordenar | `lista.sort()` | `[...lista].sort()` |
| Objeto | `obj.edad = 31` | `{ ...obj, edad: 31 }` |
| Anidado | | `{ ...u, direccion: { ...u.direccion, ciudad: "Madrid" } }` |

El spread copia **superficialmente**: el array es nuevo, los objetos de dentro son los mismos. Para modificar uno hay que crear **también** el objeto nuevo (`{ ...e, ... }`). Para anidamientos pesados: Immer.

### Eventos

Se pasa **una función** (no su resultado) a una prop `onXxx`:

```TypeScript
<button onClick={guardar}>            // ✅ referencia
<button onClick={() => guardar(id)}>  // ✅ con argumentos
<button onClick={guardar()}>          // ❌ se ejecuta EN CADA RENDER y pasa undefined
```

React llama al manejador con el **objeto evento `e`** (no el elemento): `e.target` es el elemento donde ocurrió (`e.target.value`, `e.target.checked`), `e.preventDefault()` cancela lo nativo (un `<form>` recarga la página al enviarse: **siempre `preventDefault` en `onSubmit`**), `e.key` en teclado.

Tipos: `React.ChangeEvent<HTMLInputElement>`, `React.FormEvent<HTMLFormElement>`, `React.MouseEvent<HTMLButtonElement>`, `React.KeyboardEvent<...>`. En línea TS los infiere; aparte hay que escribirlos.

Eventos frecuentes: `onClick`, `onChange` (en cada tecla), `onSubmit`, `onKeyDown`, `onFocus`/`onBlur`, `onMouseEnter`/`onMouseLeave`.

**Input controlado:** `value={texto}` + `onChange={(e) => setTexto(e.target.value)}`. React es la **única fuente de verdad**: el input muestra siempre el estado. Permite validar/transformar en cada tecla, desactivar botones, rellenar desde la API. Sin `onChange` el input se congela. **No controlado:** sin `value`, se lee con `useRef` al enviar (formularios simples, ficheros).

### Padre e hijo: datos abajo, eventos arriba

Si en el JSX de `A` aparece `<B />`, **A es el padre de B**. Forman un árbol.

- **Datos hacia abajo:** el padre pasa valores por props.
- **Eventos hacia arriba:** el padre pasa **una función** por prop; el hijo la llama; como la función se escribió en el padre y toca su estado (closure), el efecto ocurre arriba. El hijo no sabe qué pasa, solo avisa → reutilizable.

```TypeScript
{tareas.map((t) => <Tarea key={t.id} tarea={t} onAlternar={() => alternar(t.id)} />)}
// cada iteración crea una función distinta con SU t dentro (closure). Tarea llama onAlternar() sin argumentos.
```

- **El estado vive en el componente que llama a `useState`**; por eso las operaciones (`añadir`, `borrar`) se definen donde está el estado. Se coloca en el **ancestro común más cercano** de quienes lo necesitan (*lifting state up*). El estado que solo usa un componente (texto de un input) es local.
- **Prop drilling:** pasar datos por 5 niveles que no los usan. Lo resuelven Context o Zustand.

### Hooks

Funciones `use...` que dan capacidades a un componente. **Reglas:** siempre en el nivel superior del componente (no en `if`/bucles/funciones anidadas), siempre en el mismo orden.

#### `useEffect`: sincronizar con algo externo

Para **efectos secundarios** (red, DOM, timers, suscripciones, `localStorage`): cosas que no son pintar. Corre **después** de pintar.

```TypeScript
useEffect(() => {
  // efecto
  return () => { /* limpieza (opcional) */ };
}, [dep1, dep2]);
```

| Array | Corre | Uso |
|---|---|---|
| `[a, b]` | Primer render + cuando `a` o `b` cambien | Reaccionar: buscar al escribir, sincronizar |
| `[]` | Solo el primer render | Carga inicial, suscripciones, timers |
| ninguno | Cada render | Casi nunca |

**Limpieza:** si el efecto arranca algo vivo, devuelve una función que lo pare. React la ejecuta antes de re-ejecutar el efecto y al desmontar.

```TypeScript
useEffect(() => {
  const id = setInterval(() => setHora(new Date()), 1000);  // setInterval: ejecuta cada X ms, devuelve id
  return () => clearInterval(id);
}, []);

useEffect(() => {
  const f = () => setAncho(window.innerWidth);
  window.addEventListener("resize", f);
  return () => window.removeEventListener("resize", f);
}, []);

useEffect(() => {                       // condición de carrera al buscar
  let cancelado = false;
  fetch(`/api?q=${texto}`).then(r => r.json()).then(d => { if (!cancelado) setRes(d); });
  return () => { cancelado = true; };
}, [texto]);
```

- **Regla de dependencias:** todo valor del componente usado dentro va en el array (si no, *stale closure*). El linter `exhaustive-deps` avisa.
- **El callback no puede ser `async`**: usar `.then` o declarar una función async dentro y llamarla.
- **No es para estado derivado** ni para "hacer cosas cuando cambia el estado" en general: solo para lo **externo** a React. Error de principiante más común.
- `document.title`: el título de la pestaña; ejemplo mínimo de efecto (tocar algo fuera de React).

#### `useRef`: una caja que persiste y no repinta

`useRef(inicial)` → `{ current }`. Persiste entre renders; cambiar `.current` **no** provoca re-render.

| | `let x` | `useState` | `useRef` |
|---|---|---|---|
| Sobrevive al render | No | Sí | Sí |
| Cambiarla repinta | No | Sí | No |
| Para | Cálculo del render | Datos que se **ven** | Datos que se **recuerdan** sin pintarse |

Usos: (1) ids de timers/valores de fondo (`idIntervalo.current = setInterval(...)`), (2) elementos del DOM (`<input ref={inputRef} />` → `inputRef.current?.focus()`), (3) valor anterior de algo.

#### `useMemo` y `useCallback`

- `useMemo(() => calculoCaro(a, b), [a, b])`: memoriza el **resultado**; recalcula solo si cambian deps. Para cálculos **caros de verdad** (ordenar 50.000 filas) en componentes que se re-renderizan por otros motivos. Para cálculos baratos cuesta más de lo que ahorra. **Primero sin memo; si mides lentitud, memoriza.**
- `useCallback(fn, deps)`: `useMemo` para funciones; mantiene **la misma referencia** entre renders. Solo aporta en dos casos: (1) la función es prop de un hijo envuelto en `React.memo` (que no repinta si sus props no cambian por `===`), (2) la función es dependencia de un `useEffect`.

### Fetch: `.then` vs `await`

```TypeScript
fetch(url).then((r) => r.json()).then((data) => setRepos(data.items)).catch(console.error).finally(() => setCargando(false));
// equivale a try { const r = await fetch(url); const data = await r.json(); setRepos(data.items); } catch {...} finally {...}
```

Cada `.then` es un `await`; si el callback devuelve una promesa, el siguiente `.then` recibe su valor (aplanado). `r.json()` devuelve `any`, así que `data.items` encaja en `Repo[]` **sin comprobar nada** (no es tipado estructural, es `any`). Bien: anotar `const data: { items: Repo[] }` (confiado) o validar con Zod (correcto).

### TanStack Query: datos del servidor

Sustituye los tres `useState` + `useEffect` + limpieza por un hook con caché, reintentos, cancelación, revalidación y deduplicación.

```TypeScript
const { data, isPending, isError, error } = useQuery({
  queryKey: ["repos", texto],              // identidad = clave de caché; TODO lo que use queryFn va aquí
  queryFn: () => buscarRepos(texto),       // async; si lanza → isError
  enabled: texto.length >= 3,
});
// data: Repo[] | undefined (inferido del retorno de queryFn)

const crear = useMutation({
  mutationFn: (n: NuevaTarea) => fetch("/api/tareas", { method: "POST", body: JSON.stringify(n) }),
  onSuccess: () => queryClient.invalidateQueries({ queryKey: ["tareas"] }),  // mutar y luego invalidar
});
crear.mutate({ texto });
```

Configuración una vez: `<QueryClientProvider client={new QueryClient()}>`.

### Zustand: estado global de cliente

```TypeScript
export const useCarrito = create<CarritoStore>((set, get) => ({
  items: [],
  añadir: (p) => set((s) => ({ items: [...s.items, p] })),   // inmutable; set fusiona superficialmente
  vaciar: () => set({ items: [] }),
  total: () => get().items.reduce((sum, p) => sum + p.precio, 0),
}));

const añadir = useCarrito((s) => s.añadir);        // selector: solo repinta si cambia lo seleccionado
const cantidad = useCarrito((s) => s.items.length);
```

Sin providers, desde cualquier componente. Extras: `persist` (localStorage), `immer`, `devtools`. Frente a Context: menos ceremonia y repintado selectivo.

| Tipo de estado | Ejemplo | Herramienta |
|---|---|---|
| Local | Texto de input, menú abierto | `useState` |
| Global de cliente | Usuario logueado, tema, carrito | Zustand (o Context) |
| De servidor | Lista de repos, perfil | TanStack Query |

Error típico: meter datos de la API en Zustand (reinventar caché, carga y errores).

### Bundle de JS

Resultado de **juntar todos los ficheros** (los tuyos + `node_modules`) en pocos ficheros JS optimizados para el navegador. Lo hace el **bundler** (Turbopack/Webpack/Vite) en `npm run build`: sigue los `import`, transpila TS/JSX, *tree shaking* (quita lo no usado), minifica, *code splitting* (un fichero por página), hash en el nombre (`main-8f3a2c.js`) para invalidar caché. Todo lo que va en el bundle **lo descarga y ejecuta cada usuario**: cada KB cuesta. Server Components no entran en él.

---

## 3. NEXT.JS

### Qué es

Framework de React para aplicaciones completas: **rutas, renderizado en servidor, backend y despliegue**. **Corre sobre Node**: una parte se ejecuta en el servidor y otra en el navegador.

**App Router:** las carpetas definen las URLs. `app/page.tsx` → `/`; `app/blog/[slug]/page.tsx` → `/blog/mi-post` (`slug` parámetro); `layout.tsx` envuelve las páginas de dentro (cabecera, menú).

### CSR, SSR, SSG, ISR: dónde y cuándo se genera el HTML

| | Quién genera el HTML | Cuándo | Cuándo usarlo |
|---|---|---|---|
| **CSR** (Client-Side Rendering) | El navegador, con JS | Cada vez que un usuario abre la página | Detrás de login, sin SEO, muy interactivo (paneles, CRM, la plataforma de BBVA) |
| **SSR** (Server-Side Rendering) | El servidor (Node) | **Cada petición** | Personalizado o cambia constantemente, y necesita SEO/primera carga rápida (búsqueda `?q=`, feed, producto con stock en vivo) |
| **SSG** (Static Site Generation) | El build | **Una vez**, al desplegar | Igual para todos y cambia poco (landings, "Sobre mí", docs) |
| **ISR** (Incremental Static Regeneration) | El build + regeneración | Pasado `revalidate` o bajo demanda (webhook del CMS) | "Casi estático": blogs con CMS, catálogos (lo más habitual en sitios de contenido) |

- CSR: pantalla en blanco hasta que carga el JS; mal SEO; navegación instantánea después.
- SSR: contenido inmediato; luego el navegador descarga el JS e **hidrata** (lo hace interactivo). Más carga en servidor.
- SSG: ficheros servidos desde CDN; lo más rápido; datos del momento del build.
- **Se decide por página**, no por app. Datos de disponibilidad (huecos de un calendario) nunca SSG/ISR: se piden en cliente o SSR sin caché.

Decisión: (1) ¿igual para todos? No → SSR (o CSR tras login). Sí → (2) ¿cuánto cambia? Casi nunca → SSG; a veces → ISR; constantemente → SSR.

### Server Components vs Client Components (eje distinto a SSR/CSR)

SSR/CSR = **dónde se genera el HTML**. Server/Client Components = **dónde corre el código de cada componente y si viaja al navegador**.

| | Server Component (defecto) | Client Component (`"use client"`) |
|---|---|---|
| Corre en servidor | Siempre | Sí (SSR en primera carga) |
| Corre en navegador | **Nunca** | Sí (hidratación + interacciones) |
| Su JS va en el bundle | **No** | Sí |
| Hooks, eventos | No | Sí |
| BD, secretos, ficheros | Sí | No |

**`"use client"` no significa "solo en el cliente"**: significa "necesita JS en el navegador". Se renderiza en los dos sitios.

```TypeScript
// page.tsx (Server): trae datos, pinta lo pesado, delega lo interactivo
export default async function Post({ params }) {
  const post = await obtenerPost(params.slug);
  return <article><Contenido markdown={post.cuerpo} /><BotonMeGusta postId={post.id} /></article>;
}
// BotonMeGusta.tsx: "use client" + useState
```

Reglas: `"use client"` es una **frontera** (todo lo que importe pasa a cliente → ponerlo en las hojas, no en el layout); un Server puede renderizar un Client, un Client no puede importar un Server (sí recibirlo como `children`); las props servidor→cliente deben ser **serializables** (JSON: no funciones ni instancias). Patrón: servidor por defecto, cliente en las hojas interactivas. Un componente es Server **porque no necesita interactividad**, no porque consulte BD (eso lo hace la página).

### "Se ejecuta en Node": el ciclo de vida

1. **`npm run build`** (una vez por despliegue): genera `.next/` con páginas estáticas, código de servidor transpilado y bundles de cliente.
2. **`npm start`** (producción, una vez): arranca **un proceso Node** que carga el código y se queda escuchando en un puerto. Vive días.
3. **Petición** `/blog/mi-post`: el enrutador elige `page.tsx` → **ejecuta la función del componente** (una llamada normal en el hilo de Node) → `await` a BD (event loop) → el JSX se serializa a HTML → responde con HTML + `<script>` de los bundles → el navegador pinta e hidrata. Si era SSG, se sirve el fichero. Si ISR caducada, sirve el viejo y regenera.

"Se ejecuta en Node" = tu Server Component es **una función JS que corre dentro de ese proceso**, por petición o en el build. No hay `npx` por petición. En Vercel no haces `npm start`: cada página SSR/handler/action se despliega como función serverless.

### Backend dentro de Next: Route Handlers y Server Actions

**Controlador:** la capa que recibe la petición HTTP y devuelve la respuesta. Lee ruta/params/body, valida la forma, llama al caso de uso, convierte el resultado en respuesta (código, JSON). Sin reglas de negocio ni SQL.

**Route Handler** (`app/api/.../route.ts`): endpoint REST explícito, una función por método HTTP, `Request`/`Response` estándar.

```TypeScript
export async function GET() { return Response.json(await tareaRepo.listar()); }
export async function POST(request: Request) {
  const tarea = await crearTarea(await request.json());
  return Response.json(tarea, { status: 201 });
}
// app/api/tareas/[id]/route.ts → DELETE(_: Request, { params })
```

Para: APIs públicas (móvil, otros servicios), **webhooks** (Stripe, Contentful), lecturas desde Client Components.

**Server Action** (`"use server"`): función de servidor que se invoca desde la UI como una función normal; Next hace el HTTP por debajo.

```TypeScript
"use server";
export async function crearTareaAction(formData: FormData) {
  const texto = formData.get("texto") as string;
  if (!texto.trim()) return { error: "Obligatorio" };
  await crearTarea({ texto });
  revalidatePath("/tareas");          // regenera la página con los datos nuevos
}
// <form action={crearTareaAction}> sin onSubmit, fetch ni preventDefault; funciona incluso sin JS
// o desde un Client Component: onClick={() => borrarTareaAction(id)}
```

Para: mutaciones que nacen en tu propia UI (formularios, botones). **Seguridad:** es un endpoint HTTP que cualquiera puede llamar: validar y comprobar permisos como en un Route Handler.

| | Server Component | Route Handler | Server Action |
|---|---|---|---|
| Qué es | Vista renderizada en servidor | Endpoint HTTP | Función de servidor invocable desde la UI |
| Para | **Leer** y pintar | API externa, webhooks | **Escribir** desde la UI propia |
| Devuelve | HTML | `Response` (JSON) | Lo que quieras (serializable) |

Patrón: **leer con Server Components, escribir con Server Actions, Route Handlers solo cuando hace falta una API de verdad** (un móvil nativo necesita JSON, no HTML). Las tres llaman a **la misma lógica de negocio**.

**Middleware de Next** (`middleware.ts`): corre antes de cada petición, en el edge. Comprobar sesión y redirigir, reescribir URLs, cabeceras.

### Despliegue: proceso, serverless, edge, CDN

| Modelo | Qué es | Pros | Contras |
|---|---|---|---|
| **Proceso tradicional** | Node siempre encendido (contenedor, PM2, Kubernetes + balanceador) | Control, WebSockets, estado en memoria, trabajos largos | Pagas sin tráfico; escalas tú |
| **Serverless** (Vercel, Lambda) | Una instancia por petición, se apaga después | Pago por uso, escala sola, cero servidores | *Cold start*, **sin estado en memoria**, límite de tiempo, sin conexiones persistentes |
| **Edge** | Serverless en el nodo **más cercano al usuario**, con runtime reducido (APIs web) | Latencia mínima | Sin Node completo, sin ficheros, BD lejos → solo cosas ligeras (middleware, auth, geolocalización) |

**CDN** (Content Delivery Network): red de servidores por el mundo con **copias de tus ficheros estáticos** (HTML generado, bundles, imágenes). Sirve el nodo más cercano sin tocar tu servidor. Por eso SSG es tan rápido. El edge es la CDN pudiendo ejecutar código.

| Pieza de Next en Vercel | Dónde |
|---|---|
| SSG/ISR, bundles, CSS, imágenes | CDN |
| `middleware.ts` | Edge |
| SSR, Route Handlers, Server Actions | Serverless en una región |
| BD (Supabase) | Servidor en una región |

**Estado en memoria en serverless (un `Map` de usuarios en un handler) falla por dos motivos:** (1) la instancia se apaga y el `Map` desaparece; (2) hay **muchas instancias a la vez**, cada una con su `Map`: la petición siguiente cae en otra. Estado compartido → externo (Redis, BD).

### Arquitectura en capas

Express y Next no imponen estructura; la pones tú. Si mezclas HTTP + reglas + SQL + email en un handler: no se reutiliza, no se testea sin BD, cambiar una cosa rompe otra.

```
┌─ RUTAS / CONTROLADORES ─── HTTP: parsear, validar forma, responder (Route Handlers, Server Actions, páginas)
├─ CASOS DE USO / SERVICIOS ─ reglas de negocio; no sabe de HTTP ni de SQL; lanza errores de dominio
└─ REPOSITORIOS / ADAPTADORES ─ BD, APIs externas, email, colas: el "cómo"
```

**Regla de dependencia:** cada capa solo conoce a la de abajo, nunca al revés ni saltándose una.

```
src/
  app/              ← capa 1: page.tsx, api/.../route.ts, actions.ts
  aplicacion/       ← capa 2: crearPedido.ts, cancelarPedido.ts
  dominio/          ← tipos y reglas puras: Pedido, Item, errores de dominio
  infraestructura/  ← capa 3: pedidoRepositorioPostgres.ts, emailResend.ts
```

Ganas: testear el negocio con repositorios falsos; cambiar Postgres o el proveedor de email sin tocar el negocio; añadir puertas de entrada (handler, action, script, consumidor de cola) sin duplicar. Si cambia el proveedor de email → infraestructura; si cambia el límite de negocio → caso de uso; si cambia el formato de entrada (XML) → controlador. Paso siguiente (hexagonal/DDD): el caso de uso depende de una **interfaz** `PedidoRepositorio` inyectada, no de la implementación Postgres.

### Impacto en el ciclo de desarrollo

1. **Un solo lenguaje de punta a punta**: mismos tipos TS en front y back, sin DTOs duplicados.
2. **Feedback rápido**: `npm run dev` en segundos, recarga en caliente.
3. **Menos infraestructura**: Next + Vercel + Supabase, una persona monta producto, backend, auth y despliegue.
4. **Menos estructura por defecto**: la disciplina la pone el equipo (ESLint, Prettier, TS estricto, capas).
5. **Ecosistema rápido**: versiones mayores frecuentes, leer changelogs.
6. **Despliegue distinto**: proceso, serverless o edge según la pieza.

---

## 4. FRASES PARA LA ENTREVISTA

- **Node:** "Un solo hilo con event loop: ideal para I/O-bound, cuidado con CPU-bound (workers o sacarlo a otro servicio con cola). Nunca bloquear el event loop."
- **Express:** "Rutas y middlewares `(req, res, next)` en cadena; la arquitectura en capas la pongo yo: controladores, casos de uso, repositorios."
- **React:** "Componentes declarativos: `UI = f(estado)`; props abajo, eventos arriba; estado inmutable porque React compara referencias; `useEffect` solo para sincronizar con lo externo."
- **Next:** "SSR/SSG/ISR hablan de dónde se genera el HTML; Server/Client Components de dónde corre el código y si viaja al navegador. Servidor por defecto, `'use client'` en las hojas. Leer con Server Components, escribir con Server Actions, Route Handlers para APIs de verdad."
- **Despliegue:** "Lo estático a la CDN, el middleware al edge, SSR y APIs a serverless; para WebSockets o trabajos largos, proceso tradicional. En serverless no hay estado en memoria."
- **Estado:** "Local con `useState`, global de cliente con Zustand, datos de servidor con TanStack Query."
- **Event Driven:** "Eventos de dominio en pasado, broker, consumidores independientes; outbox, idempotencia, at-least-once."
- **APIs:** "SemVer estricto: añadir es minor, romper es major con v1/v2 o expand/contract; feature flags para comportamiento."

## 5. PENDIENTE DE REVISAR

- **Caché**: qué es y cómo se usa (TanStack `queryKey`, revalidación e invalidación; caché HTTP: `Cache-Control`, ETag; caché de Next: `revalidate`, `revalidatePath`; CDN como caché; caché de aplicación: Redis; hash en nombres de bundle).
