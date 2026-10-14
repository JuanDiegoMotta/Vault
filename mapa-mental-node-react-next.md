# Mapa mental: Node, React y Next

> En los nodos de un `mindmap` de Mermaid no pueden ir `( )`, `[ ]` ni `{ }` (son sintaxis de forma). Para ampliar un diagrama: `Ctrl` + rueda, o el plugin de comunidad **Diagram Zoom Drag**.
>
> Los colores vienen de la línea `%%{init: ...}%%` al principio de cada bloque: `cScaleN` es el fondo de la rama N y `cScaleLabelN` su texto; `git0`/`gitBranchLabel0` son el nodo raíz. Cambia los códigos hex a tu gusto.

## Vista general

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'fontFamily': 'Inter, Segoe UI, sans-serif', 'fontSize': '15px','git0': '#E0E7FF', 'gitBranchLabel0': '#1E1B4B','cScale0': '#1D4ED8', 'cScaleLabel0': '#FFFFFF','cScale1': '#15803D', 'cScaleLabel1': '#FFFFFF','cScale2': '#B91C1C', 'cScaleLabel2': '#FFFFFF','cScale3': '#7E22CE', 'cScaleLabel3': '#FFFFFF','cScale4': '#C2410C', 'cScaleLabel4': '#FFFFFF','cScale5': '#0F766E', 'cScaleLabel5': '#FFFFFF','cScale6': '#BE185D', 'cScaleLabel6': '#FFFFFF','cScale7': '#4338CA', 'cScaleLabel7': '#FFFFFF','cScale8': '#A16207', 'cScaleLabel8': '#FFFFFF','cScale9': '#0E7490', 'cScaleLabel9': '#FFFFFF','cScale10': '#6D28D9', 'cScaleLabel10': '#FFFFFF','cScale11': '#047857', 'cScaleLabel11': '#FFFFFF'}}}%%
mindmap
  root((JS en servidor y navegador))
    Node.js
      Event loop: I/O vs CPU
      npm y SemVer
      Módulos
      Express y middleware
    Event Driven
      Eventos y broker
      Outbox e idempotencia
    React
      Componentes y JSX
      Props y estado
      Eventos
      Hooks
      Datos: TanStack y Zustand
    Next.js
      Dónde se genera el HTML
      Server vs Client Components
      Route Handlers y Server Actions
      Despliegue
      Arquitectura en capas
```

## Node.js

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'fontFamily': 'Inter, Segoe UI, sans-serif', 'fontSize': '15px','git0': '#E0E7FF', 'gitBranchLabel0': '#1E1B4B','cScale0': '#1D4ED8', 'cScaleLabel0': '#FFFFFF','cScale1': '#15803D', 'cScaleLabel1': '#FFFFFF','cScale2': '#B91C1C', 'cScaleLabel2': '#FFFFFF','cScale3': '#7E22CE', 'cScaleLabel3': '#FFFFFF','cScale4': '#C2410C', 'cScaleLabel4': '#FFFFFF','cScale5': '#0F766E', 'cScaleLabel5': '#FFFFFF','cScale6': '#BE185D', 'cScaleLabel6': '#FFFFFF','cScale7': '#4338CA', 'cScaleLabel7': '#FFFFFF','cScale8': '#A16207', 'cScaleLabel8': '#FFFFFF','cScale9': '#0E7490', 'cScaleLabel9': '#FFFFFF','cScale10': '#6D28D9', 'cScaleLabel10': '#FFFFFF','cScale11': '#047857', 'cScaleLabel11': '#FFFFFF'}}}%%
mindmap
  root((Node.js))
    Runtime: V8 fuera del navegador
    Un hilo + event loop
      I/O-bound: espera, hilo libre
      CPU-bound: calcula, bloquea
      Nunca bloquear el event loop
      Soluciones CPU
        Worker threads: mismo proceso, otro hilo, mensajes
        Otro servicio con cola: convierte CPU en I/O
      Proceso vs hilo
    npm
      package.json: deps, devDeps, version, scripts
      package-lock y npm ci
      Rangos: caret y tilde
      npm run X, npm start, argumentos tras --
    SemVer
      MAJOR.MINOR.PATCH
      Añadir es minor, quitar o cambiar es breaking
      APIs: v1 y v2, expand/contract, feature flags
    Módulos
      export named vs default
      import con llaves vs sin llaves
      ESM vs CJS, type module, .mts
      Módulo se ejecuta una vez: singleton
    Express
      Rutas = controladores
      Middleware en cadena
        req, res, next
        Sin next la petición se cuelga
        Error handler de 4 params, el último
```

## Event Driven

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'fontFamily': 'Inter, Segoe UI, sans-serif', 'fontSize': '15px','git0': '#E0E7FF', 'gitBranchLabel0': '#1E1B4B','cScale0': '#1D4ED8', 'cScaleLabel0': '#FFFFFF','cScale1': '#15803D', 'cScaleLabel1': '#FFFFFF','cScale2': '#B91C1C', 'cScaleLabel2': '#FFFFFF','cScale3': '#7E22CE', 'cScaleLabel3': '#FFFFFF','cScale4': '#C2410C', 'cScaleLabel4': '#FFFFFF','cScale5': '#0F766E', 'cScaleLabel5': '#FFFFFF','cScale6': '#BE185D', 'cScaleLabel6': '#FFFFFF','cScale7': '#4338CA', 'cScaleLabel7': '#FFFFFF','cScale8': '#A16207', 'cScaleLabel8': '#FFFFFF','cScale9': '#0E7490', 'cScaleLabel9': '#FFFFFF','cScale10': '#6D28D9', 'cScaleLabel10': '#FFFFFF','cScale11': '#047857', 'cScaleLabel11': '#FFFFFF'}}}%%
mindmap
  root((Event Driven))
    Petición-respuesta vs eventos
    Evento = hecho pasado, inmutable
      OperacionAprobada, UsuarioRegistrado
    Broker: Kafka, RabbitMQ, SQS
    Ganas
      Desacoplo: añadir canal = añadir consumidor
      Resiliencia: los eventos esperan
      Respuesta inmediata
      Escalado por consumidor
    Pagas
      Consistencia eventual
      At-least-once: consumidores idempotentes
      Doble escritura: patrón outbox
      Trazabilidad: ids de correlación
    Historia notificaciones
      Fue integración directa con APIs del banco
      Cómo lo haría hoy con eventos
```

## React

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'fontFamily': 'Inter, Segoe UI, sans-serif', 'fontSize': '15px','git0': '#E0E7FF', 'gitBranchLabel0': '#1E1B4B','cScale0': '#1D4ED8', 'cScaleLabel0': '#FFFFFF','cScale1': '#15803D', 'cScaleLabel1': '#FFFFFF','cScale2': '#B91C1C', 'cScaleLabel2': '#FFFFFF','cScale3': '#7E22CE', 'cScaleLabel3': '#FFFFFF','cScale4': '#C2410C', 'cScaleLabel4': '#FFFFFF','cScale5': '#0F766E', 'cScaleLabel5': '#FFFFFF','cScale6': '#BE185D', 'cScaleLabel6': '#FFFFFF','cScale7': '#4338CA', 'cScaleLabel7': '#FFFFFF','cScale8': '#A16207', 'cScaleLabel8': '#FFFFFF','cScale9': '#0E7490', 'cScaleLabel9': '#FFFFFF','cScale10': '#6D28D9', 'cScaleLabel10': '#FFFFFF','cScale11': '#047857', 'cScaleLabel11': '#FFFFFF'}}}%%
mindmap
  root((React))
    Declarativo: UI es función del estado
      Virtual DOM: compara y aplica diferencias
    Componente = función de props que devuelve JSX
      Mayúscula, se usa como etiqueta, no se llama
      JSX: solo expresiones entre llaves
        ternario, and lógico, map
        className, htmlFor, fragmento
        trampa del cero con and
    Props
      Un objeto, desestructuración
      Solo lectura
      key: única y estable
    Estado: useState
      setter provoca re-render
      Inmutable: triple igual compara referencias
        spread, map, filter
        copia superficial
      Derivado no es estado
      Setter funcional
    Eventos
      Pasar función, no llamarla
      e.target.value, preventDefault
      Input controlado: value + onChange
    Padre e hijo
      Datos abajo por props
      Eventos arriba por callbacks, closure
      Lifting state up
      Prop drilling: Context o Zustand
    Hooks
      useEffect: sincronizar con lo externo
        deps: lista, vacío, ninguno
        limpieza: timers, listeners
        no async, no estado derivado
      useRef: persiste, no repinta
      useMemo: resultado caro
      useCallback: misma referencia
    Datos
      fetch con then = await
      TanStack Query: servidor
        queryKey, queryFn, enabled
        useMutation + invalidate
      Zustand: global cliente
        store con set y get
        selectores
      Local, global, servidor
    Bundle: el JS que viaja al usuario
```

## Next.js

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'fontFamily': 'Inter, Segoe UI, sans-serif', 'fontSize': '15px','git0': '#E0E7FF', 'gitBranchLabel0': '#1E1B4B','cScale0': '#1D4ED8', 'cScaleLabel0': '#FFFFFF','cScale1': '#15803D', 'cScaleLabel1': '#FFFFFF','cScale2': '#B91C1C', 'cScaleLabel2': '#FFFFFF','cScale3': '#7E22CE', 'cScaleLabel3': '#FFFFFF','cScale4': '#C2410C', 'cScaleLabel4': '#FFFFFF','cScale5': '#0F766E', 'cScaleLabel5': '#FFFFFF','cScale6': '#BE185D', 'cScaleLabel6': '#FFFFFF','cScale7': '#4338CA', 'cScaleLabel7': '#FFFFFF','cScale8': '#A16207', 'cScaleLabel8': '#FFFFFF','cScale9': '#0E7490', 'cScaleLabel9': '#FFFFFF','cScale10': '#6D28D9', 'cScaleLabel10': '#FFFFFF','cScale11': '#047857', 'cScaleLabel11': '#FFFFFF'}}}%%
mindmap
  root((Next.js))
    Framework de React sobre Node
    App Router: carpetas = rutas, layout, slug
    Dónde se genera el HTML
      CSR: navegador
      SSR: cada petición
      SSG: build
      ISR: build + revalidate
      Se elige por página
      Hidratación
    Dónde corre el código
      Server Component: solo servidor, sin JS al cliente
      Client Component: use client, se hidrata
      Frontera en las hojas
      Props serializables
    Ciclo: build, start, petición
      Es una función JS dentro del proceso Node
    Backend
      Controlador = frontera HTTP
      Route Handler: API REST, webhooks, móvil
      Server Action: mutaciones desde la UI
      Leer con Server Components, escribir con actions
      Middleware en el edge
    Despliegue
      Proceso tradicional: WebSockets, trabajos largos
      Serverless: cold start, sin memoria compartida
      Edge: cerca del usuario, runtime reducido
      CDN: copias de estáticos
    Arquitectura en capas
      Controladores, casos de uso, repositorios
      Dependencia solo hacia abajo
      dominio, aplicacion, infraestructura
    Pendiente: caché
```

## El mismo mapa en texto

- **Node.js**: runtime de JS sobre V8
  - **Un hilo + event loop**: I/O-bound libera el hilo (`await`); CPU-bound lo bloquea → workers (mismo proceso, otro hilo) o servicio aparte con cola (convierte CPU en I/O)
  - **npm**: `package.json` (deps, devDeps, version, scripts), lock + `npm ci`, rangos `^`/`~`, `npm run X`
  - **SemVer**: añadir = minor; quitar/cambiar = major → APIs con v1/v2, expand/contract, feature flags
  - **Módulos**: named vs default, llaves en el import, ESM vs CJS, módulo = singleton
  - **Express**: rutas + middlewares `(req, res, next)`; `next()` o se cuelga; error handler de 4 params
- **Event Driven**: evento = hecho pasado → broker → consumidores; ganas desacoplo/resiliencia; pagas consistencia eventual, at-least-once (idempotencia), doble escritura (outbox)
- **React**: `UI = f(estado)`
  - Componente = función(props) → JSX; `<Comp/>`; solo expresiones en `{}`
  - Props: un objeto desestructurado, solo lectura, `key`
  - Estado: `useState` → re-render; inmutable porque `===` compara referencias; derivado se calcula
  - Eventos: pasar función; `e.target.value`; `preventDefault`; input controlado
  - Padre/hijo: datos abajo, eventos arriba (closure); lifting state up; prop drilling
  - Hooks: `useEffect` (externo, deps, limpieza), `useRef` (persiste sin repintar), `useMemo`, `useCallback`
  - Datos: fetch; TanStack Query (servidor); Zustand (global cliente); local/global/servidor
  - Bundle: el JS que viaja al usuario
- **Next.js**: React + rutas + servidor + despliegue, sobre Node
  - HTML: CSR / SSR / SSG / ISR, por página
  - Código: Server Components (sin JS al cliente) vs Client Components (`"use client"`, hidratan)
  - Ciclo: build → start → petición ejecuta la función del componente en Node
  - Backend: controlador = frontera HTTP; Route Handlers (API, webhooks); Server Actions (mutaciones UI)
  - Despliegue: proceso / serverless / edge / CDN; sin estado en memoria en serverless
  - Capas: controladores → casos de uso → repositorios; dependencia hacia abajo
- **Pendiente**: caché
