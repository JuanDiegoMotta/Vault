Lo que hace typescript es JavaScript con una capa añadida de tipos.

Para entenderlo es bueno entender primero lo que quiere decir que JS sea dinámico:
```JavaScript
let a = 6;
a = "six"; //NO DA ERROR
```
Esto te lo permite JavaScript porque las variables son dinámicas y permite guardar diferentes tipos de datos en la misma variable.

En cambio en TypeScript se debe primero definir el tipo de dato que irá en esa variable:

```TypeScript
let a: number = 6;
a = "six"; //DARÁ ERROR
```

La idea es que sirva como una red de seguridad para que los fallos por tipos no sucedan en tiempo de ejecución sino que se descubran durante el desarrollo.

**Typescript no se ejecuta**

El compilador de TypeScript (tsc) se encarga de "transpilar" (convertir un lenguaje de alto nivel a otro) el código TS a JS. 
Lo que se termina ejecutando es JS en el motor V8 del navegador o de Node.js (en servidor)
Por tanto TS existe en tiempo de compilación pero no existe en tiempo de ejecución.

**Inferencia de tipos**

Typescript también es capaz de deducir los tipos sin que los escribas explícitamente. Por ejemplo, puede dar error en compilación en cuanto estás desarrollando si intentas acceder a una propiedad que no esté definida o si intentas guardar un valor en una propiedad que no es el tipo correcto:
```TypeScript
const usuario = {
  nombre: "Ana",
  edad: 28,
};
// Tipo inferido: { nombre: string; edad: number }

usuario.nombre = "Luis";  // ✅
usuario.edad = "28";      // ❌ Error: edad es number
usuario.email = "a@b.com"; // ❌ Error: la propiedad email no existe
```

**Any**

La keywork any, más que significar que esa variable pueda tomar cualquier tipo, realmente lo que quiere decir es que *ignore* el tipado de typescript para esa variable. Esto hará que las sugerencias de autocompletado y los errores que podría haber detectado typescript ya no aparezcan.

```TypeScript
let anyType : any = "Hola";

anyType.aaa;
```

**La variable const**
Const es una keyword que existe en JS. Cuando una variable se declara con const, al asignarle un objeto se le está pasando la referencia a ese objeto. Si intentamos reasignar la variable nos dará error:
```TypeScript
const myVariable = {
	prop1 : "Hello",
	prop2 : 123
}

myVariable = {prop1 : "Bye", prop2 : 321} // -> ERROR: Trying to reassign a constant
```

El objeto en sí, sus propiedades, sí que pueden ser modificadas:
```TypeScript
const myVariable = {
	prop1 : "Hello",
	prop2 : 123
}

myVariable.prop1 = "Bye" // -> No dará ERROR, se está cambiando el valor de la propiedad, no la referencia al objeto
```

Con los tipos primitivos, la variable guarda el valor en sí, no una referencia pero al ser inmutable, la única forma de guardar otro valor en esa variable es reescribiendolo por completo.
Aun así, si se ha declarado con const, esa variable, de nuevo no puede ser reescrita.
```TypeScript
const myVariable = 2;
myVariable = 3 //-> ERROR: Trying to reassigna a constant

let myVariable = 2;
myVariable = 5 //-> Se ha reescrito el valor por completo
```
