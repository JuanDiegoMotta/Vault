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

## INFERENCIA DE TIPOS

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

## ANY

La keywork any, más que significar que esa variable pueda tomar cualquier tipo, realmente lo que quiere decir es que *ignore* el tipado de typescript para esa variable. Esto hará que las sugerencias de autocompletado y los errores que podría haber detectado typescript ya no aparezcan.

```TypeScript
let anyType : any = "Hola";

anyType.aaa;
```

## La variable const
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

## Desestructuración

Guardar los valores de un objeto o array en variables en una sola linea.

```TypeScript
//Ejemplo simple sin desestructuración
const persona = {
	nombre: "Ana",
	edad: 25,
	sexo: "F"
}

const nombre = persona.nombre;
const edad = persona.edad;
const sexo = persona.sexo;

console.log(nombre) //-> "Ana"
console.log(edad) // -> 25
console.log(sexo) //"F"

//Ejemplo simple de desestructuración
const {nombre, edad, sexo} = persona
console.log(nombre) //-> "Ana"
console.log(edad) // -> 25
console.log(sexo) //"F"

```

Algunos comportamientos de la desestructuración
```TypeScript
//Ejemplo con arrays
const myArray = [1, 2, 3, 4, 5];

//rest: toma los valores restantes del array como un subarray
const [one, two, three, ...rest] = myArray;
console.log(one); //-> 1
console.log(two); //-> 2
console.log(three); //-> 3
console.log(rest); //-> [4,5]

//salto de posiciones
const[,,number] = myArray;
console.log(number) //-> 3

//Valores por defecto y undefined
const myArray2 = [1,2];
const[one,two,three=5,four, five] = myArray2;
console.log(one) //-> 1
console.log(two) //-> 2
console.log(three) //-> 5 (valor por defecto cuando usamos =, solo si es undefined)
console.log(four) //-> undefined (si intentamos acceder a una posición no definida)
console.log(five) //-> undefined

//Ejemplo con objetos
const myObject = {
	prop1 : 1,
	prop2 : "Hello",
	prop3 : {
		inProp: 34
	}
}

//Cambio de nombre de variable
const {prop1: variable} = myObject;
console.log(variable); //-> 1 

//rest: toma los valores restantes del objeto (no importa el orden, los de antes y dsps)
const {prop2..rest} = myObject;
console.log(prop2); //-> "Hello"
console.log(rest); //-> {prop1: 1, prop3: {inProp: 34}}

//desestructuración anidada 
const{prop3: {inProp}} = myObject;
console.log(inProp); //-> 34
```

## Funciones en TS/JS

#### *Arrow functions*

```TypeScript
//Una función normal 
function myFunction(a: number, b:number): number{
	return a + b;
};

//Arrow function con retorno explícito
const myFunction2 = (a: number, b: number) => {
	return a + b;
};

//Arrow function con retorno implícito (sin llaves ni return)
const myFunction3 = (a: number, b: number) => a + b;
```

Las arrows function no tienen su propio this. Usan el this del lugar en el que se escribieron.

#### *Cuerpo de función vs objeto*

```TypeScript
const function1 = () => 5;

const function2 = () => {5}; // Devuelve undefined

const function3 = () => { return 5 }; // ` => { ` Indica cuerpo de función, no objeto

const function4 = () => ({number: 5}); // ` => ({` Indica objeto

  

console.log(function1()); // 5
 
console.log(function2()); // undefined

console.log(function3()); // 5

console.log(function4()); // {number: 5}
```

#### *Void vs undefined*
Una función sin return, tiene un tipo de retorno void, pero en tiempo de ejecución el valor que retorna es undefinded.
Ojo, undefined es tanto tipo, como valor. Y además al definirlo como tipo, es estricto y restrictivo, lo que quiere decir que si la función devuelve algo distinto a undefined, dará error en compilación.
En cambio, si el tipo de retorno de una función es void, simplemente ignorará el valor devuelto, siempre y cuando se haga en una asignación de una función a un tipo de función.

```TypeScript
const noReturnFunction = (contador: {n: number}) => {
	contador.n += 1
} //Función sin retorno

const myNum = {n: 1};
console.log(noReturnFunction(myNum)); // Devuelve undefined en tiempo de ejecución
console.log(myNum); // 2

//Si definimos el tipo de una funcion y establecemos que su return sea undefined, esto es estricto y no podemos pasar una función a este tipo que no devuelva undefined.

type UndefinedFunction = (nombre: string) => undefined;
const usarConTexto = (texto: string, fn: UndefinedFunction) => fn(texto);
usarConTexto("Hola Juan", (n) => n + ", como estas?"); //DARÁ ERROR
//Type 'string' is not assignable to type 'undefined'.

//En cambio con void, al querer decir que no importa lo que devuelva la funcion, no dará error.
type FunctionVoid = (nombre: string) => void;
const usarConTexto = (texto: string, fn: FunctionVoid) => fn(texto);
console.log(usarConTexto("Hola Juan", (n) => n + ", como estas?")); //OK

```

#### Shorthand property
```TypeScript
//Si la variable que guarda el valor se llama igual que la propiedad, puedes escribir solo la prop una vez
const edad = 25;
const usuario = {nombre: "Juan", edad}; // = {nombre: "Juan, edad: 25}

//Es lo contrario de la desestructuración, si está a la izda, extraes la prop de un objeto en una variable, si está a la derecha, creas la propiedad en un objeto

const persona = {altura: "1,7m", peso: 85}

const {altura} = persona;

console.log(altura); //"1,7m"

  

const nombre = "Julian";

const usuario = {nombre};

console.log(usuario); // {nombre: "Julian"}
```

#### Funciones como *first class citizens*

En JS las funciones son un valor más, por tanto pueden:
1- Ser guardadas en una variable:
```TypeScript
const saludar = (t: string) => "Hola " + t;
console.log(saludar("Juan")); //-> "Hola Juan"
```
2- Guardar en array u objeto
```TypeScript
//Ejemmplo de guardado en un objeto
const myFunction = () => "Hola mundo";
const myObject = {fn: myFunction};
console.log(myObject.fn()); //Hola mundo

//Ejemplo de guardar en un array
const myArray = [() => "Hola mundo", 2, 3];//El tipo del array para TS es (number | (() => string))[]
console.log(myArray[0]()); //ERROR: TS no sabe si esto es un number o un () => string, por eso te lanza error
console.log(myArray[1]); // 2
console.log(myArray[2]); // 3

const myArray2 : [()=> string, number, number] = [() => "Hola mundo", 2, 3]; //Al definir los tipos, lo conviertes en tupla y TS ya sabe el orden

console.log(myArray2[0]()); //Hola mundo
console.log(myArray2[1]); //2
console.log(myArray2[2]); //3

const myArray3 = [() => "Hola mundo", 2, 3] as const; //tupla de solo lectura
console.log(myArray3[0]()); //Hola mundo
console.log(myArray3[1]); //2
console.log(myArray3[2]); //3

//Array vs tupla

const arreglo: number[] = [1,2,3]; // `: tipo[]` define un array

const tupla: [string, number, number] = ["Hola", 1, 2]; //`: [tipo, tipo,...tipo]` define una tupla

const tupla2 = ["Hola", 1, 2] as const; //`as const` hace que sea una tupla de solo lectura
```

3- Pasar como parámetro de una función
```TypeScript
//Definimos función que pueda recibir otra función como parámetro (callback)
const myFunction = (texto: string, fn: (t:string) => string) => console.log(fn(texto));
const concatenarMundo = (texto: string) => texto + " mundo";
//Pasamos concatenarMundo como parámetro
myFunction("Hola", concatenarMundo); // Hola mundo
```

4- Usar como retorno de una función
```TypeScript
const crearSumador = (n: number) => (x: number) => x + n;
const sumar10 = crearSumador(10);
console.log(sumar10(5));
```

#### Tipar funciones

Sintaxis del tipo de una función:
(parametros con sus tipos) => tipo de retorno

Callback -> Función que se pasa a otra para que esta la ejecute.
Función de orden superior -> Función que recibe o devuelve otra función.

Ejemplo:
```TypeScript
//Tipar un callback directamente en el parámetro
const aplicarYmostrar = (n: nubmer, fn: (t:number) => number) => {
	console.log(fn(n));
}
aplicarYmostrar(2, (t) => t + 10); //12
  
//Con alias de tipo
type Sumar = (n: number) => number;
const aplicarYmostrar = (num: number, fn: Sumar) => { console.log(fn(num))};
aplicarYmostrar(7, (t) => t+5);
```

#### Compatibilidad de funciones (tipado estructural)

En TS un tipo de función no es un molde que haya que copiar exactamente. Es un **contrato de uso**. TS se pregunta:

> ¿Puedo usar esta función de forma segura donde se espera ese tipo?

A esto se le llama **tipado estructural**: importa que encaje, no que sea idéntico. Java, en cambio, usa **tipado nominal**.

**Llamar frente a pasar:**

TypeScript

```TypeScript
const doble = (n: number) => n * 2;
doble();  // ❌ LLAMAR con menos argumentos: le falta un dato
```

TypeScript

```TypeScript
type Filtro = (n: number) => boolean;
const f: Filtro = () => true; // ✅ PASAR una función que pide menos: el dato que sobra se ignora
```

**Reglas:**

||Regla|Ejemplo con `type F = (n: number) => boolean`|
|---|---|---|
|Parámetros|Puede **pedir menos**, nunca más|`() => true` ✅ · `(n, extra) => true` ❌|
|Tipo de parámetro|Debe aceptar lo que se le pasa|`(n: string) => true` ❌|
|Retorno|Debe ser **compatible**. Puede **dar más** (propiedades extra)|`(n) => n * 2` ❌ (number, no boolean)|
|Retorno `void` en callback|Puede devolver **cualquier cosa**, se ignora|`(n) => lista.push(n)` ✅ con `forEach`|

- **Parámetros:** a la función no le puede **faltar** nada de lo que necesita.
- **Retorno:** a quien la usa no le puede **faltar** nada de lo que espera.

**Por qué es así:** JS siempre ha permitido pasar más argumentos de los declarados, y los sobrantes se ignoran. Es lo que permite escribir `map((n) => ...)` aunque `map` pase 3 argumentos: `(elemento, índice, array)`.

**Excepción de `void`:** en un **tipo de función** (callback), `void` significa "no voy a usar lo que devuelvas". En una función declarada con `: void`, sí está prohibido devolver algo:

TypeScript

```TypeScript
function f(): void { return 5; } // ❌
```