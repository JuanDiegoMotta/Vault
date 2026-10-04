Lo que hace TypeScript es JavaScript con una capa añadida de tipos.

Para entenderlo es bueno entender primero lo que quiere decir que JS sea dinámico:

```TypeScript
let a = 6;
a = "six"; //NO DA ERROR
```

Esto te lo permite JavaScript porque las variables son dinámicas y permite guardar diferentes tipos de datos en la misma variable.

En cambio en TypeScript cada variable tiene un tipo fijo, ya sea porque lo escribes tú o porque TS lo infiere (ver más abajo):

```TypeScript
let a: number = 6;
a = "six"; //DARÁ ERROR
```

La idea es que sirva como una red de seguridad para que los fallos por tipos no sucedan en tiempo de ejecución sino que se descubran durante el desarrollo.

**TypeScript no se ejecuta**

El compilador de TypeScript (tsc) se encarga de "transpilar" (convertir un lenguaje de alto nivel a otro) el código TS a JS. Lo que se termina ejecutando es JS en el motor de JavaScript del navegador (V8 en Chrome) o en Node.js (en servidor). Por tanto TS existe en tiempo de compilación pero no existe en tiempo de ejecución.

## INFERENCIA DE TIPOS

TypeScript también es capaz de deducir los tipos sin que los escribas explícitamente. Por ejemplo, puede dar error en compilación en cuanto estás desarrollando si intentas acceder a una propiedad que no esté definida o si intentas guardar un valor en una propiedad que no es del tipo correcto:

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

La keyword any, más que significar que esa variable pueda tomar cualquier tipo, realmente lo que quiere decir es que _ignore_ el tipado de TypeScript para esa variable. Esto hará que las sugerencias de autocompletado y los errores que podría haber detectado TypeScript ya no aparezcan.

```TypeScript
let anyType : any = "Hola";

anyType.aaa; // TS no se queja, aunque la propiedad no existe
```

## La variable const

Const es una keyword que existe en JS. Cuando una variable guarda un objeto, lo que guarda realmente es la referencia a ese objeto. Const impide reasignar la variable, así que si intentamos reasignarla nos dará error:

```TypeScript
const myVariable = {
	prop1 : "Hello",
	prop2 : 123
}

myVariable = {prop1 : "Bye", prop2 : 321} // -> ERROR: Cannot assign to 'myVariable' because it is a constant
```

El objeto en sí, sus propiedades, sí que pueden ser modificadas:

```TypeScript
const myVariable = {
	prop1 : "Hello",
	prop2 : 123
}

myVariable.prop1 = "Bye" // -> No dará ERROR, se está cambiando el valor de la propiedad, no la referencia al objeto
```

Con los tipos primitivos, la variable guarda el valor en sí, no una referencia, y al ser inmutables la única forma de guardar otro valor en esa variable es reasignándola por completo. Por tanto, si se ha declarado con const, esa variable no puede cambiar de ninguna forma.

```TypeScript
const myVariable = 2;
myVariable = 3 //-> ERROR: Cannot assign to 'myVariable' because it is a constant

let myVariable2 = 2;
myVariable2 = 5 //-> Se ha reasignado el valor por completo
```

## Desestructuración

Guardar los valores de un objeto o array en variables en una sola línea.

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
//Ejemplo con arrays (se extrae por POSICIÓN)
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
console.log(four) //-> undefined (si intentamos acceder a una posición no definida, no da error)
console.log(five) //-> undefined

//Ejemplo con objetos (se extrae por NOMBRE)
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
const {prop2, ...rest} = myObject;
console.log(prop2); //-> "Hello"
console.log(rest); //-> {prop1: 1, prop3: {inProp: 34}}

//desestructuración anidada (prop3 es solo el camino, NO se crea una variable prop3)
const{prop3: {inProp}} = myObject;
console.log(inProp); //-> 34
```

## Funciones en TS/JS

#### _Arrow functions_

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

Las arrow functions no tienen su propio this. Usan el this del lugar en el que se escribieron.

#### _Cuerpo de función vs objeto_

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

#### _Void vs undefined_

Una función sin return tiene un tipo de retorno void, pero en tiempo de ejecución el valor que retorna es undefined. Ojo, undefined es tanto tipo como valor (void es solo un tipo). Y además, al usarlo como tipo de retorno, es estricto y restrictivo, lo que quiere decir que si la función devuelve algo distinto a undefined, dará error en compilación. En cambio, si el tipo de retorno es void, se ignorará el valor devuelto (y TS no te dejará usarlo), siempre y cuando se haga en una asignación de una función a un tipo de función.

Excepción: si **declaras** tú la función con `: void`, sí da error devolver algo:

```TypeScript
function f(): void { return 5; } //ERROR
const g: () => void = () => 5;   //OK (tipo de función: flexible)
```

```TypeScript
const noReturnFunction = (contador: {n: number}) => {
	contador.n += 1
} //Función sin retorno

const myNum = {n: 1};
console.log(noReturnFunction(myNum)); // undefined en tiempo de ejecución
console.log(myNum); // {n: 2}

//Si definimos el tipo de una función y establecemos que su return sea undefined, esto es estricto y no podemos pasar una función a este tipo que no devuelva undefined.

type UndefinedFunction = (nombre: string) => undefined;
const usarConTexto = (texto: string, fn: UndefinedFunction) => fn(texto);
usarConTexto("Hola Juan", (n) => n + ", como estas?"); //DARÁ ERROR
//Type 'string' is not assignable to type 'undefined'.

//En cambio con void, al querer decir que no importa lo que devuelva la función, no dará error.
type FunctionVoid = (nombre: string) => void;
const usarConTexto2 = (texto: string, fn: FunctionVoid) => fn(texto);
console.log(usarConTexto2("Hola Juan", (n) => n + ", como estas?")); //OK
//En ejecución imprime "Hola Juan, como estas?", pero para TS el resultado es void (no lo puedes usar)

```

#### Shorthand property

```TypeScript
//Si la variable que guarda el valor se llama igual que la propiedad, puedes escribir solo la prop una vez
const edad = 25;
const usuario = {nombre: "Juan", edad}; // = {nombre: "Juan", edad: 25}

//Es lo contrario de la desestructuración, si está a la izda del =, extraes la prop de un objeto en una variable, si está a la derecha, creas la propiedad en un objeto

const persona = {altura: "1,7m", peso: 85}

const {altura} = persona;

console.log(altura); //"1,7m"



const nombre = "Julian";

const usuario2 = {nombre};

console.log(usuario2); // {nombre: "Julian"}
```

#### Funciones como _first class citizens_

En JS las funciones son un valor más, por tanto pueden: 1- Ser guardadas en una variable:

```TypeScript
const saludar = (t: string) => "Hola " + t;
console.log(saludar("Juan")); //-> "Hola Juan"
```

2- Guardar en array u objeto

```TypeScript
//Ejemplo de guardado en un objeto
const myFunction = () => "Hola mundo";
const myObject = {fn: myFunction};
console.log(myObject.fn()); //Hola mundo

//Ejemplo de guardar en un array
const myArray = [() => "Hola mundo", 2, 3];//El tipo del array para TS es (number | (() => string))[]
console.log(myArray[0]()); //ERROR: TS no sabe si esto es un number o un () => string, por eso te lanza error
console.log(myArray[1]); // 2
console.log(myArray[2]); // 3

const myArray2 : [()=> string, number, number] = [() => "Hola mundo", 2, 3]; //Al anotarlo con un tipo tupla, TS ya sabe qué hay en cada posición

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

const tupla2 = ["Hola", 1, 2] as const; //`as const` -> readonly ["Hola", 1, 2] (tupla de solo lectura con valores literales)
```

3- Pasar como parámetro de una función

```TypeScript
//Definimos función que pueda recibir otra función como parámetro (callback)
const myFunction = (texto: string, fn: (t:string) => string) => console.log(fn(texto));
const concatenarMundo = (texto: string) => texto + " mundo";
//Pasamos concatenarMundo como parámetro (sin paréntesis: se pasa la función, no se llama)
myFunction("Hola", concatenarMundo); // Hola mundo
```

4- Usar como retorno de una función

```TypeScript
const crearSumador = (n: number) => (x: number) => x + n;
const sumar10 = crearSumador(10);
console.log(sumar10(5)); // 15
```

#### Tipar funciones

Sintaxis del tipo de una función: (parámetros con sus tipos) => tipo de retorno

Callback -> Función que se pasa a otra para que esta la ejecute. Función de orden superior -> Función que recibe o devuelve otra función.

Ejemplo:

```TypeScript
//Tipar un callback directamente en el parámetro
const aplicarYmostrar = (n: number, fn: (t:number) => number) => {
	console.log(fn(n));
}
aplicarYmostrar(2, (t) => t + 10); //12

//Con alias de tipo
type Sumar = (n: number) => number;
const aplicarYmostrar2 = (num: number, fn: Sumar) => { console.log(fn(num))};
aplicarYmostrar2(7, (t) => t+5); //12
```

#### Compatibilidad de funciones (tipado estructural)

```TypeScript
//Si intentas llamar a una función con menos parámetros de los declarados, fallará:
function saltar(altura: number) {
	return altura * 2;
}

saltar(); //ERROR

//En cambio si creas un tipo de función y luego intentas pasarle una función que pida menos parámetros sí que va a compilar
type Saltar = (altura: number) => number;

const salto7: Saltar = () => 7; //OK
```

Esto es porque en TS los tipos de función no son un contrato que se tenga que cumplir a rajatabla. TS se preguntará: ¿Puedo usar esta función de forma segura donde se espera ese tipo?

```TypeScript
//Una forma de comprobar si compilará o no es imaginarnos que se llama a la función que hemos pasado con todos los argumentos y ver si no hay ningún problema

//En el ejemplo anterior, imaginemos
salto7(10) //-> 7
//Simplemente ignorará el parámetro, no hay problema

const salto10Extra: Saltar = (altura: number, saltos: number) => altura * saltos;
//Ni siquiera va a compilar, salto10Extra(10) dará problemas ya que hay un parámetro que nunca se va a informar y que estamos explicitando que se necesita.
```

#### Optional properties

Se puede declarar una propiedad como opcional:

```TypeScript
type Hero = {
	id: string
	name: string
	age: number
	email?: string //El tipo de la propiedad es (string | undefined)
}

//Al leer el valor de una propiedad opcional, podemos usar el encadenamiento opcional `?.`
const thor: Hero = {id: "1", name: "Thor", age: 25};
const lowerCaseEmail = thor.email?.toLowerCase(); //Si existe -> string, si no -> undefined (tipo: string | undefined)

//Esto funciona en cadena y con funciones
usuario?.direccion?.ciudad //se corta en el primer undefined
callback?.() //llama a la función solo si existe

//Se suele utilizar con nullish coalescing `??` que da un valor por defecto.
thor.email?.toLowerCase() ?? "sin email"; //Ya no puede ser undefined (tipo: string)
```

#### Readonly properties

Una propiedad se puede declarar como readonly:

```TypeScript
type Hero = {
	readonly id: number
	name: string
	age: number
}

const myHero: Hero = {
	id: 123,
	name: "Thor",
	age: 25
}

myHero.id = 234 //Da error al compilar porque se ha declarado id como readonly
```

Hay que tener en cuenta que esto en tiempo de ejecución, en JS, sí se puede modificar sin problema, como cualquier tipo. El error solo nos aparece al compilar.

Por otro lado, es superficial: solo protege la propiedad, no lo que hay dentro si es un objeto

```TypeScript
type Usuario = {
	id: number,
	name: string,
	readonly inventory: {
		torch: number,
		flint: number
	}
}

const myUser: Usuario = {
	id: 123,
	name: "Juan",
	inventory: {
		torch: 3,
		flint: 2
	}
}

myUser.inventory = {torch:8, flint: 2} //ERROR, no se puede cambiar la referencia = reasignar objeto distinto

myUser.inventory.torch = 8 //OK -> El readonly no protege la propiedad interna del objeto
```

#### Object.freeze()

Este método se utiliza para congelar un objeto en tiempo de ejecución. Ojo: es **superficial**, solo congela el primer nivel (los objetos y arrays anidados se pueden seguir modificando).

```TypeScript
type Hero = {
	id: number
	name: string
	age: number
}

const myHero: Hero = {
	id: 123,
	name: "Thor",
	age: 25
}

Object.freeze(myHero);
myHero.name = "Spiderman"; //OJO: No da error en este caso pq TS no sabe que lo has congelado

console.log(myHero.name) //Thor
//En tiempo de ejecución sí que se congela el objeto y no permite el cambio
//(falla en silencio, o lanza TypeError en modo estricto, el habitual en módulos)

//Para que TS sepa que se ha congelado hay que utilizar el objeto que devuelve la llamada
const freezedObj = Object.freeze(myHero);
freezedObj.name = "Spiderman" //ERROR -> Esta vez sí lo sabe TS
```

#### Template literal types

Se puede usar la sintaxis de los template string de JS para crear tipos de string con un patrón.

```TypeScript
type Id = `${string}-${string}-${string}-${string}`;

const myString: Id = "abc-def-ghi-jkl" //OK
const myString2: Id = "abc" //ERROR
//Ojo: ${string} acepta cualquier string, incluido el vacío -> "---" también sería válido

//Se utiliza sobre todo para combinar uniones
type Color = "rojo" | "azul";
type Size = "sm" | "lg";
type ClaseBoton = `btn-${Color}-${Size}`;
//= "btn-rojo-sm" | "btn-rojo-lg" | "btn-azul-sm" | "btn-azul-lg"
```

#### Intersection types

Si creas un type como la intersección de otros dos (&) el type resultante tendrá todas las propiedades de ambos.

```TypeScript
type ConId = { id: string };
type User = {name:string, age:number}
type UserConId = ConId & User;

const user: UserConId = { id: "123", name: "Sara", age: 26}

```

No confundir con la unión `|`: `A | B` significa que cumple **al menos uno** (puede cumplir los dos), `A & B` que cumple **los dos** obligatoriamente.

Si intentaras crear un type con intersection pero tienen propiedades incompatibles, no dará error TS, pero esa propiedad sería de tipo "never", es decir nunca podrías crear un objeto.

```TypeScript
type ConId = { id: string };
type User = {id: number}
type UserConId = ConId & User;

const user: UserConId = { id: 123} //Type 'number' is not assignable to type 'never'.
```

#### Type indexing

Obtener un tipo a partir de la propiedad de otro

```TypeScript
//Type indexing

type Hero = {
    name: string
    bag:{
        chips:number
        cola: number
    }
}

const fillBag = (chips: number, cola: number): Hero["bag"] => ({chips, cola});
const indexedNumber: Hero["bag"]["chips"] = 42; //Con corchetes y comillas, nunca con punto
```

#### Typeof

Existen dos:

- El de JS -> Se ejecuta y devuelve un string

```TypeScript
const myNumber = 42;
console.log(typeof myNumber); //"number"
```

- El de TS -> Va en la posición de tipo, devuelve un TIPO y se borra al compilar. Se utiliza para inferir tipos a partir de un valor.

```TypeScript
const myNumber = 42;
type MyOtherNumber = typeof myNumber; // tipo: 42 (al ser const, el tipo es el literal; con let sería number)
```

#### ReturnType

ReturnType devuelve el tipo del valor de retorno de una función.

```TypeScript

type User = {nombre: string, age: number};

const myFunction = ():User => ({nombre: "juan", age: 15});

type OtherUser = ReturnType<typeof myFunction>; // User

```

**Resumen de las herramientas para sacar tipos de otras cosas:**

|Quiero...|Uso|
|---|---|
|Combinar dos tipos|`A & B`|
|El tipo de una propiedad|`Tipo["prop"]`|
|El tipo de un valor|`typeof valor`|
|El tipo de retorno de una función|`ReturnType<typeof fn>`|

#### Tuplas

Son listas con una longitud fija y un tipo específico en cada posición. Solo existen en TS, en ejecución son arrays

```TypeScript
type RGB = [number, number, number];

//El useState de react devuelve una tupla
const [value, setValue] = useState("Value");
```

Array -> Una lista de longitud variable de elementos del mismo tipo: (tipo)[] Tupla -> Grupo fijo donde cada posición tiene un significado (rgb, coordenadas, etc)

Ojo! TS nunca infiere tupla por sí solo. Un array literal sin anotar siempre es un array.

```TypeScript
const a = [1, 3, 5] //-> Array normal: number[]

const b: [number, number] = [1, 2]; //-> tupla (anotando la variable)

const c = [1, 2, 3] as const //-> readonly [1, 2, 3], tupla readonly con valores literales

const d = (): [number, string] => [123, "x"]; //-> Tupla anotando el retorno

```

Una tupla normal sigue teniendo push() y TS no se va a quejar. Para que sea fija del todo, hay que hacerla readonly.

```TypeScript

type RGB = readonly [number, number, number];

const myRgb: RGB = [255, 0, 182];

myRgb[0] = 182; //Cannot assign to '0' because it is a read-only property.
myRgb.push(255); //Property 'push' does not exist on type 'RGB'.


const changedRgb = myRgb.map((n) => n/2); //No da error, .map no cambia la tupla original, devuelve un array nuevo.
```

**Map** y **filter** siempre devolverán un array normal. Aunque el original sea una tupla.

#### CLOSURE

Una función creada dentro de otra recuerda las variables de la función de fuera, incluso cuando la de fuera ha terminado. Estas variables siguen vivas y son privadas.

```TypeScript

type Estado = [() => number, (n: number) => void];

const crearEstado = (inicial: number): Estado => {
    let valor = inicial;
    const leer = () => valor;
    const escribir = (n: number) => {valor = n};
    return [leer, escribir];
}

const [leer, escribir] = crearEstado(10);
leer(); //-> 10
escribir(5);
leer(); //-> 5; Las dos comparten la misma variable.
```

#### ¿ES O FABRICA?

Antes de poner un alias de tipo de función pregúntate si **es** de ese tipo o si **fabrica** ese tipo de función.

```TypeScript
type Validador = (n: number) => boolean;

const esPositivo: Validador = (n) => n > 0;                        //ES un Validador -> el alias va en la variable
const crearEsMenorQue = (max: number): Validador => (n) => n < max; //FABRICA Validadores -> el alias va en el retorno

esPositivo(2); //-> true
const menosQueCuatro = crearEsMenorQue(4);
menosQueCuatro(6) //-> false

```

#### TRES FORMAS DE DECLARAR FUNCIONES

El `=` solo aparece cuando la función se guarda en una variable (`const nombre =`). Nunca entre `): tipo` y `{`.

```TypeScript

function normalDeTodaLaVida(n:number): number { //Declaración: no lleva `=`
    return n*2;
}

const expresionDeFuncion = function (n:number): number { return n*2;} //Expresión de función: lleva el `=` de la variable

const arrowFunctionReturnExplicito = (n:number): number => { return n*2;}
const arrowFunctionReturnImplicito = (n:number): number => n*2;
```

#### USAR TYPEOF DENTRO DE UN IF

Usar typeof dentro de un if hace que TS estreche el tipo dentro de esa rama (narrowing)

```TypeScript
const myArray = [() => "x", 42, true]; //-> Según TS el tipo es (number | boolean | (() => string))[]

myArray.forEach((elemento) => {
    if(typeof elemento === "function"){
        console.log(elemento()); //Aquí sabe que es () => string
    } else if(typeof elemento === "number"){
        console.log(elemento * 10); //Aquí sabe que es number
    }else{
        console.log("otro") //Aquí sabe que es boolean
    }
})
```

typeof va delante del valor y devuelve un string.

#### As const

As const aplica a todos los niveles:

- Todo pasa a ser readonly (los arrays internos se convierten en tuplas readonly)
- Los valores se fijan como literales

```TypeScript
const config = { tema: "oscuro", zoom: [1, 2] };
// { tema: string; zoom: number[] }

const config2 = { tema: "oscuro", zoom: [1, 2] } as const;
// { readonly tema: "oscuro"; readonly zoom: readonly [1, 2] }
```

#### Parámetros readonly

Si una función **solo lee** un array o una tupla, tipa el parámetro como `readonly`. Así acepta tanto versiones normales como `readonly` (al revés no funciona):

```TypeScript
type RGB = readonly [number, number, number];
const aCss = ([r, g, b]: RGB) => `rgb(${r}, ${g}, ${b})`;

const paleta = { primario: [0, 120, 255] } as const;

aCss([0, 0, 0]);      //OK
aCss(paleta.primario); //OK: readonly [0, 120, 255] encaja con RGB
```

#### Utility types

Tipos que trae TS de serie. Los tipos se pasan con **`< >`**, nunca con `( )`:

```TypeScript
const saludar = (nombre: string, edad: number) => `Hola ${nombre}`;

type R = ReturnType<typeof saludar>;  // string
type P = Parameters<typeof saludar>;  // [nombre: string, edad: number] → SIEMPRE una tupla
type L = Readonly<{ a: number }>;     // { readonly a: number }
```

#### Todo es superficial

`const`, `readonly` y `Object.freeze` **solo protegen el primer nivel**. Los objetos y **arrays** anidados se pueden seguir modificando.

**Los arrays son objetos**: se guardan y se pasan por referencia. Solo los primitivos (`string`, `number`, `boolean`, `null`, `undefined`) van por valor.

```TypeScript
const juego = Object.freeze({ nivel: 1, items: ["espada"] });
juego.nivel = 2;            //ERROR en TS, y en ejecución no cambia (congelado)
juego.items.push("escudo"); //SÍ cambia: el array interno no está congelado
console.log(juego.items);   // ["espada", "escudo"]
```

#### ?? elimina el undefined del tipo

```TypeScript
type Perfil = { alias?: string };
const p: Perfil = {};

const a = p.alias;                // string | undefined
const b = p.alias?.length;        // number | undefined
const c = p.alias ?? "anónimo";   // string → ya no puede ser undefined
```

`?.` y `??` son de **JavaScript**, no de TypeScript. TS solo detecta cuándo hacen falta.

#### QUÉ ES UN ENUM

Un enum es un conjunto de constantes con nombre agrupadas dentro de un tipo. Sirve para representar un grupo cerrado de opciones

```TypeScript
enum Direccion{
	Arriba, //0
	Abajo, //1
	Izda, //2
	Dcha, //3
}
```

Si no le asignas valor a las constantes, tomarán números empezando por 0, como los índices de un array. Si das valor a una, las siguientes continúan desde ahí (`Baja = 1, Media, Alta` → 1, 2, 3). En los enums numéricos puedes acceder al valor de forma bidireccional (mapeo inverso):

```TypeScript
Direccion[0] //-> "Arriba"
Direccion.Arriba //-> 0
```

Lo más recomendable es hacer enums de strings (el valor es legible). Estos no tienen mapeo inverso:

```TypeScript
enum Rol{
	Admin = "ADMIN",
	Editor = "EDITOR",
	Lector = "LECTOR"
}

const usuario = {nombre: "Ana", rol: Rol.Admin};
console.log(usuario.rol)// "ADMIN"
```

Ojo: los enums de string no aceptan el string suelto. Si una función espera un `Rol`, `"ADMIN"` da error: hay que pasar `Rol.Admin`.

Los enums son a la vez un tipo y un valor. Es por eso que existen en ejecución: tsc los transpila y genera código JS (un objeto). A diferencia del resto del código TS, que es "erasable" (borrable) y que sí puede ejecutar node directamente (simplemente quita los tipos y pone espacios en blanco), los enums necesitan transpilarse para generar ese código JS. Por eso `node archivo.ts` da error con ellos, y si queremos ejecutarlo lo podemos hacer con `npx tsx myArchivo.ts` (o compilando con tsc). Ni node ni tsx comprueban los tipos, solo tsc.

Recomendación: Usa unión de literales por defecto y un objeto as const si necesitas los valores en ejecución.

#### ASYNC, AWAIT Y PROMESAS

Al llamar a una API, la respuesta puede tardar en llegar. Como JS solo tiene 1 hilo de ejecución, si se quedase parado esperando esta respuesta, todo el navegador se quedaría bloqueado (no podrías clicar ni navegar en la página). Para eso existen las operaciones asíncronas: JS lanzará la petición, el programa continuará con otras cosas, y se retomará una vez llegue la respuesta.

**PROMISE** Una promesa se utiliza para representar un valor que llegará más adelante, puede tener tres estados

- Pendiente -> todavía no ha llegado el resultado
- Resuelta -> Ya ha llegado el resultado
- Rechazada -> Ha fallado, con un error

**FETCH**

`fetch(url)` lanza una petición a una URL e **inmediatamente devuelve una promesa**, no la respuesta.

```TypeScript
const promise = fetch(API_URL) //promise: Promise<Response>, todavía no hay respuesta
```

**AWAIT** `await` significa "espera hasta que la promesa se resuelva y dame su valor"

```TypeScript
const response = await fetch(API_URL) //response: Response, ya ha llegado la respuesta y tenemos su valor.
```

OJO! Si después quieres acceder al body de esta respuesta, deberás utilizar `await` de nuevo, ya que se descarga aparte y al ser grande puede tardar más

```TypeScript
const response = await fetch(API_URL);  // 1. llegan los datos básicos de la respuesta (estado, cabeceras)
const data = await response.json();     // 2. se descarga y se interpreta el cuerpo como JSON (data: any)
```

**ASYNC** Normalmente, `await` solo se puede utilizar en una función marcada como `async` (fuera de una función solo se permite en módulos ES, p.ej. archivos `.mts`)

```TypeScript
const obtenerRepos = async () => {
	const response = await fetch(API_URL);
	const data = await response.json();
	return data;
}
```

Una función `async` **siempre** devuelve una promesa (aunque dentro devuelvas un valor normal). Por eso, allí donde se llame, también se debe utilizar `await` para obtener el valor:

```TypeScript
const promesa = obtenerRepos()       //promesa: Promise<any>
const repos = await obtenerRepos()   //repos: any (el valor ya resuelto)
```

#### INTERFACES

Una interfaz se utiliza para definir la forma de un objeto. Es muy similar a los tipos, de hecho en el 90% de los casos son intercambiables.

```TypeScript
interface MyObject{ //-> No tiene = como en los tipos, y no se suele poner ; al final
	name: string,
	age: number,
	id: string
}

type MyObject2 = {
	name: string,
	age: number,
	id: string
};
```

**Extends vs &** En las interfaces usamos extends para que una interfaz "extienda" las propiedades de otra

```TypeScript
interface Persona {
	name: string,
	age: number,
	sex: string
}

interface Empleado extends Persona{
	salario: number
}

//En cambio con tipos se haría así:
type Persona2 = {
	nombre: string
};
type Empleado2 = Persona2 & { salario: number };
```

Se pueden extender varias a la vez `interface A extends B,C,D...`.

Ventaja de `extends`: si hay propiedades incompatibles, te avisa al declarar. Con `&` el conflicto da `never` en silencio.

**Fusión de declaraciones** Si declaras dos veces la misma interfaz, a diferencia de los tipos, no te dará error: las dos se fusionan en una

```TypeScript
interface Config { tema: string }
interface Config { zoom: number }

const c: Config = { tema: "oscuro", zoom: 1 }; // ✅ tiene las dos
```

¿Para qué sirve? Sobre todo para ampliar tipos de librerías sin tocar su código, por ejemplo para añadir una propiedad propia al objeto global window:

```TypeScript
interface Window { miApp: { version: string } }
window.miApp.version; // ✅
```

Riesgo: si usas sin querer el nombre de una interfaz que ya existe, se fusionará en silencio.

**Lo que solo puede hacer un `TYPE`**

```TypeScript
type Id = string | number;                        // uniones
type Rol = "ADMIN" | "EDITOR";                    // uniones de literales
type RGB = [number, number, number];              // tuplas
type Validador = (n: number) => boolean;          // tipos de función
type Ruta = `/${string}`;                         // template literal types
type Sesion = ReturnType<typeof crearSesion>;     // tipos derivados
type Claves = keyof typeof paleta;                // keyof / typeof
```

#### TYPE GUARD

Cualquier comprobación que hace que TS estreche el tipo de una variable dentro de un bloque de código.

**typeof** Se usa principalmente para primitivos y funciones

```TypeScript
const formatear = (valor: string | number) => {
	if (typeof valor === "string"){
		return valor.toUpperCase(); //TS sabrá que es string
	}

	return valor.toFixed(2); //TS sabrá que solo puede ser number
}
```

**instanceof** Se utiliza para objetos de una clase. Instancias.

```TypeScript
try {
	// ...
} catch(error){ //error: unknown
	if (error instanceof Error){
		console.log(error.message); //error: Error
	}
}
```

**comprobación de null/undefined**

```TypeScript
const input = document.getElementById("email"); // HTMLElement | null

if (input) {
  input.focus();   // input: HTMLElement (null descartado)
}

if (input !== null) { /* igual */ }
```

Ojo con `if (x)` si x puede ser un número o un string: `0` y `""` también se descartan. En ese caso, mejor `x !== undefined`.

**Igualdad con literales**

```TypeScript
type Tema = "claro" | "oscuro" | "sistema";
const aplicar = (t: Tema) => {
	if (t === "sistema"){
		return; //t -> "sistema"
	}
	t; //-> "claro" | "oscuro"
}
```

**operador in** Sirve para distinguir objetos que no son clases por las propiedades que tienen.

```TypeScript
type Pez = {nadar: () => void};
type Ave = {volar: () => void};

const mover = (animal: Pez | Ave) => {
	if("nadar" in animal){
		animal.nadar(); //animal: Pez
	} else {
		animal.volar(); //animal: Ave
	}
}
```

**uniones discriminadas**

Es el más común: todos los tipos de la unión tienen una propiedad común con un literal distinto. A esta propiedad se la llama el discriminante.

```TypeScript
type Cargando = {
	estado: "Cargando";
}
type Exito = {
	estado: "Exito";
	datos: string[];
}
type Fallo = { //No llamarlo Error: ya existe la clase Error de JS
	estado: "Error";
	mensaje: string;
}

type Peticion = Exito | Fallo | Cargando
const pintar = (p: Peticion) => {
	switch(p.estado){
		case "Exito":
			return p.datos.join(", "); // p: Exito
		case "Error":
			return p.mensaje; //p: Fallo
		case "Cargando":
			return "Cargando...";	//p: Cargando
	}
}
```

**guards personalizados** Si la comprobación es compleja, la puedes poner en una función, pero si solo devuelve boolean TS no estrecha el tipo fuera de ella.

```TypeScript
const esUsuario = (x: unknown): boolean =>
	typeof x === "object" && x !== null && "nombre" in x;

if (esUsuario(dato)){
	dato.nombre; //ERROR: TS no sabe nada, solo vio boolean
}
```

Hay que decirle explícitamente a TS que, si la función devuelve true, el valor es de un tipo en concreto (predicado de tipo `x is Tipo`):

```TypeScript
type Usuario = {
	nombre: string;
};

const esUsuario = (x: unknown): x is Usuario =>
	typeof x === "object" && x !== null && "nombre" in x;

if(esUsuario(dato)){
	dato.nombre; //dato: Usuario
}
```

Ojo: TS confía en ti (como con `as`). No comprueba que tu función verifique realmente lo que dices.

### Resumen

| Type guard              | Para qué                                                  |
| ----------------------- | --------------------------------------------------------- |
| `typeof x === "string"` | Primitivos y funciones                                    |
| `x instanceof Clase`    | Objetos de clases (`Error`, `Date`, elementos del DOM...) |
| `if (x)`, `x !== null`  | Descartar `null`/`undefined` (cuidado con `0` y `""`)     |
| `x === "literal"`       | Uniones de literales                                      |
| `"prop" in x`           | Objetos distinguibles por sus propiedades                 |
| `switch (x.tipo)`       | ⭐ Uniones discriminadas                                   |
| `(x): x is T =>`        | Comprobaciones reutilizables propias                      |
#### GENÉRICOS
Cuando quieres que una función reciba "algo" y devuelva ese mismo "algo" utilizas las variables de tipo (genéricos).

```TypeScript
//Ejemplo: Una función que devuelva el primer elemento de un array de lo que sea
function returnFirst<T>(array: T[]): T {
	return T[0];
}

returnFirst<string>(["a", "b", "c"]); //"a"
returnFirst([1, 2, 3]) //1 -> TS infiere el tipo según el argumento
```

Con las arrow function `<T>` va delante de los paréntesis.

```TypeScript
const returnFirst = <T>(array: T[]): T => T[0];
```

**Alias y interfaces genéricos**
Los alias y tipos también pueden ser genéricos
```TypeScript
type Respuesta<T> {
	datos: T;
	error: string | null;
}

const r1: Respuesta<string[]> = { datos: ["a", "b", "c"], error: null }
const r2: Respuesta<{id: number}> = { datos: {id: 4}, error: null}
```

**Genéricos con extends**
A veces quieres que cumplan algo, para ello puedes usar "extends"
```TypeScript
//Que tenga una prop en específico
const masLargo = <T extends { length: number }>(a: T, b: T): T =>
	a.length > b.length? a : b;

masLargo(4, 5) //-> 5

//Con keyof y indexed access
const obtener = <T, K extends keyof T>(lista: T[], clave: K): T[K] => lista[clave];
const usuario = {nombre: "Juan", edad: 25}

obtener(usuario, "nombre") //-> "Juan"

```

### Resumen

- **`<T>`** es un parámetro de tipo: conecta los tipos de entrada y salida.
- TS lo **infiere** en la mayoría de los casos.
- **`<T extends X>`** restringe qué tipos se aceptan.
- Funciones, `type` e `interface` pueden ser genéricos.
- Como siempre, **se borra al compilar**: los genéricos solo existen en TS.

#### UTILITY TYPES
Genéricos que trae TS por defecto

**Partial`<T>`**
Hace que todas las propiedades del objeto sean opcionales
```TypeScript
type Usuario = {
	id: number;
	name: string;
	age: number;
	email: string;
}

type UsuarioParcial = Partial<Usuario>;
//{ id?: number; nombre?: string; email?: string; edad?: number }
```

Se utilizan sobre todo en funciones de actualización donde solo se pasa alguna propiedad
```TypeScript
const actualizar = (u: Usuario, cambio: Partial<Usuario>): Usuario => ({
	...u,
	...cambio
});

actualizar(ana, {edad: 30}) //Ok
actualizar(juan, {rol: "Admin"}) //ERROR
```

Su opuesto es Required`<T>`. Hace que todas las pro

