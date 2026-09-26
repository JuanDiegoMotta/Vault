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

**Desestruturación**

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

**Funciones en TS/JS**

*Arrow functions*

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

*Cuerpo de función vs objeto*

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

*Void vs undefined*
Void es un tipo de TS, undefined es lo que devuelve en tiempo de ejecución
