# Mapa mental: Node, React y Next

> En los nodos de un `mindmap` de Mermaid no pueden ir `( )`, `[ ]` ni `{ }`, porque son sintaxis de forma. Por eso aquí están escritos con palabras.

```mermaid
mindmap
  root((JS en servidor y navegador))
    Node.js
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
    Event Driven
      Evento = hecho pasado, inmutable
      Broker: Kafka, RabbitMQ, SQS
      Ganas: desacoplo, resiliencia, escalado
      Pagas
        Consistencia eventual
        At-least-once: idempotencia
        Doble escritura: outbox
    React
      Declarativo: UI es función del estado
      Componente = función de props que devuelve JSX
        Mayúscula, se usa como etiqueta, no se llama
        JSX: solo expresiones entre llaves
          ternario, and lógico, map
          className, htmlFor, fragmento
      Props
        Un objeto, desestructuración
        Solo lectura, key
      Estado: useState
        setter provoca re-render
        Inmutable: triple igual compara referencias
          spread, map, filter
        Derivado no es estado
      Eventos
        Pasar función, no llamarla
        e.target.value, preventDefault
        Input controlado: value + onChange
      Padre e hijo
        Datos abajo por props
        Eventos arriba por callbacks, closure
        Lifting state up, prop drilling
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
        TanStack Query: servidor, queryKey, useMutation, invalidate
        Zustand: global cliente, store, selectores
        Local, global, servidor
      Bundle: todo el JS que viaja al usuario
    Next.js
      Framework de React sobre Node
      App Router: carpetas = rutas, layout, slug
      Dónde se genera el HTML
        CSR: navegador
        SSR: cada petición
        SSG: build
        ISR: build + revalidate
        Se elige por página
      Dónde corre el código
        Server Component: solo servidor, sin JS al cliente
        Client Component: use client, se hidrata
        Frontera en las hojas, props serializables
      Ciclo: build, start, petición
      Backend
        Controlador = frontera HTTP
        Route Handler: API REST, webhooks
        Server Action: mutaciones desde la UI
        Leer con Server Components, escribir con actions
      Despliegue
        Proceso tradicional
        Serverless: cold start, sin memoria compartida
        Edge: cerca del usuario, runtime reducido
        CDN: copias de estáticos
      Arquitectura en capas
        Controladores, casos de uso, repositorios
        Dependencia solo hacia abajo
        dominio, aplicacion, infraestructura
    Pendiente
      Caché
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
