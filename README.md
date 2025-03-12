# Guía de Conceptos Avanzados de JavaScript. Las 30 Cuestiones Más Difíciles de JavaScript:

Recurso didáctico completo que explora los 30 temas más desafiantes de JavaScript. Esta guía incluye explicaciones claras y ejemplos de código prácticos sobre hoisting, closures, promesas, async/await, prototipos y más. 

Ideal para estudiantes, autodidactas y profesionales que buscan fortalecer sus fundamentos y superar las confusiones comunes del lenguaje.

## Contenido

- **Fundamentos Avanzados**: Hoisting, Closures, Scope, TDZ
- **Contexto y Funciones**: This, Arrow Functions, Bind/Call/Apply
- **Asincronía**: Promesas, Async/Await, Event Loop
- **Patrones y Paradigmas**: Prototypes, Clases, Módulos
- **Optimización**: Memory Management, Web APIs, Service Workers

## Cómo usar este recurso

Esta guía está diseñada para ser estudiada progresivamente, aunque cada tema es autocontenido. Recomendamos:

1. Leer la explicación teórica
2. Analizar los ejemplos de código
3. Experimentar con las variaciones en tu entorno
4. Revisar los casos comunes de error

¡Bienvenidos a esta guía especializada! Como profesor de bootcamp especializado en JavaScript y manejo asíncrono, he identificado los 30 conceptos que más suelen confundir a desarrolladores en crecimiento. Para cada tema, incluyo explicaciones claras y ejemplos de código prácticos.

## 1. Hoisting

El hoisting es el comportamiento de JavaScript que mueve las declaraciones al principio de su ámbito actual antes de la ejecución del código.

```javascript
console.log(x); // undefined (no error)
var x = 5;

// Esto ocurre porque JavaScript internamente lo interpreta así:
// var x;
// console.log(x);
// x = 5;

// Con let y const
console.log(y); // ReferenceError: Cannot access 'y' before initialization
let y = 10;
```

Las declaraciones de funciones se elevan completamente:

```javascript
saludar(); // "Hola" (funciona correctamente)

function saludar() {
  console.log("Hola");
}

// Pero las expresiones de función no:
despedir(); // TypeError: despedir is not a function

var despedir = function() {
  console.log("Adiós");
};
```

## 2. Closures (Cierres)

Un closure se forma cuando una función interna tiene acceso al ámbito léxico de una función externa incluso después de que ésta haya terminado.

```javascript
function crearContador() {
  let contador = 0;
  
  return function() {
    contador++;
    return contador;
  };
}

const miContador = crearContador();
console.log(miContador()); // 1
console.log(miContador()); // 2
console.log(miContador()); // 3

// La variable contador persiste entre llamadas aunque está "protegida" del exterior
```

Los closures son fundamentales para la privacidad de datos y la creación de funciones especializadas.

## 3. This en Diferentes Contextos

El valor de `this` cambia según cómo se invoque la función, lo que confunde a muchos desarrolladores.

```javascript
// En método de objeto
const persona = {
  nombre: "Ana",
  saludar: function() {
    console.log(`Hola, soy ${this.nombre}`);
  }
};

persona.saludar(); // "Hola, soy Ana"

// En función standalone
function quienSoy() {
  console.log(this);
}
quienSoy(); // Window (en navegador) o global (en Node.js)

// En event handler
const boton = document.getElementById("miBoton");
boton.addEventListener("click", function() {
  console.log(this); // El elemento HTML del botón
});

// Con call, apply, bind
function presentar(saludo) {
  console.log(`${saludo}, soy ${this.nombre}`);
}

const carlos = { nombre: "Carlos" };
presentar.call(carlos, "Hola"); // "Hola, soy Carlos"
presentar.apply(carlos, ["Buen día"]); // "Buen día, soy Carlos"
const presentarCarlos = presentar.bind(carlos);
presentarCarlos("Saludos"); // "Saludos, soy Carlos"
```

## 4. Arrow Functions y Lexical This

Las funciones flecha capturan el valor de `this` del contexto circundante.

```javascript
const persona = {
  nombre: "Laura",
  // Con función tradicional
  saludarTradicional: function() {
    setTimeout(function() {
      console.log(`Hola, soy ${this.nombre}`); // this.nombre es undefined
    }, 100);
  },
  // Con arrow function
  saludarArrow: function() {
    setTimeout(() => {
      console.log(`Hola, soy ${this.nombre}`); // "Hola, soy Laura"
    }, 100);
  }
};

persona.saludarTradicional(); // "Hola, soy undefined"
persona.saludarArrow(); // "Hola, soy Laura"
```

## 5. Promesas y Manejo de Errores

Las promesas facilitan operaciones asíncronas, pero su manejo de errores puede ser confuso.

```javascript
function obtenerDatos(id) {
  return new Promise((resolve, reject) => {
    // Simulación de petición a API
    setTimeout(() => {
      if (id > 0) {
        resolve({ id: id, nombre: "Producto " + id });
      } else {
        reject("ID no válido");
      }
    }, 1000);
  });
}

// Error común: olvidar manejar rechazos
obtenerDatos(1)
  .then(data => console.log(data));
  // Si hay error, quedará como "Unhandled promise rejection"

// Correcto con catch
obtenerDatos(-1)
  .then(data => console.log(data))
  .catch(error => console.error("Error:", error));
  // "Error: ID no válido"

// Correcto con segundo parámetro de then
obtenerDatos(-1)
  .then(
    data => console.log(data),
    error => console.error("Error en then:", error)
  );
  // "Error en then: ID no válido"
```

## 6. Async/Await y Manejo de Excepciones

Async/await simplifica el código asíncrono pero requiere otro enfoque para los errores.

```javascript
async function procesarDatos(id) {
  try {
    const datos = await obtenerDatos(id);
    console.log("Datos procesados:", datos);
    return datos;
  } catch (error) {
    console.error("Error en procesamiento:", error);
    // Podemos relanzar, transformar o manejar el error
    throw new Error(`Procesamiento fallido: ${error}`);
  }
}

// Uso
procesarDatos(1)
  .then(resultado => console.log("Éxito:", resultado))
  .catch(err => console.error("Error capturado:", err.message));

// Errores comunes:
// 1. Olvidar await (la función sigue siendo asíncrona pero no espera)
// 2. No usar try/catch y perder errores
// 3. Olvidar que el return de una función async siempre es una Promise
```

## 7. Ejecución Asíncrona vs. Síncrona

Entender el modelo de ejecución asíncrona es fundamental en JavaScript.

```javascript
console.log("Inicio");

setTimeout(() => {
  console.log("Timeout");
}, 0);

Promise.resolve().then(() => {
  console.log("Promise");
});

console.log("Fin");

// El orden de salida será:
// "Inicio"
// "Fin"
// "Promise"
// "Timeout"
```

Esto ocurre por las colas de tareas y microtareas del Event Loop:
1. El código síncrono se ejecuta primero
2. Las microtareas (promesas) tienen prioridad sobre tareas (setTimeout)

## 8. Event Loop y Call Stack

El funcionamiento interno del Event Loop es crucial para entender JavaScript asíncrono.

```javascript
function a() {
  console.log("Función a");
  b();
}

function b() {
  console.log("Función b");
  setTimeout(() => {
    console.log("Timeout en b");
  }, 0);
}

console.log("Inicio del programa");
a();
console.log("Fin del programa");

// Salida:
// "Inicio del programa"
// "Función a"
// "Función b"
// "Fin del programa"
// "Timeout en b"
```

El Call Stack mantiene el contexto de ejecución actual, mientras que el Event Loop:
1. Ejecuta todo el código síncrono
2. Verifica las colas de microtareas (promises)
3. Verifica la cola de tareas (timers, eventos)
4. Repite el proceso

## 9. Scope de Variables (var, let, const)

El ámbito de las variables difiere según cómo se declaren.

```javascript
// Scope con var (función)
function ejemploVar() {
  if (true) {
    var x = 10;
  }
  console.log(x); // 10 (var no respeta bloques)
}

// Scope con let y const (bloque)
function ejemploLet() {
  if (true) {
    let y = 20;
    const z = 30;
  }
  console.log(y); // ReferenceError: y is not defined
  console.log(z); // ReferenceError: z is not defined
}

// Scope global vs scope de función
var global = "soy global";
let globalBlock = "soy global de bloque";

function scopeTest() {
  var funcVar = "var de función";
  let funcLet = "let de función";
  
  console.log(global); // "soy global"
  console.log(globalBlock); // "soy global de bloque"
}

console.log(funcVar); // ReferenceError: funcVar is not defined
```

## 10. Temporal Dead Zone (TDZ)

Con `let` y `const`, las variables existen en el bloque pero no se pueden utilizar antes de su declaración.

```javascript
// Con var
console.log(x); // undefined (hoisted)
var x = 5;

// Con let/const
console.log(y); // ReferenceError: Cannot access 'y' before initialization
let y = 10;

// Ejemplo más complejo
let a = 1;
{
  console.log(a); // ReferenceError: Cannot access 'a' before initialization
  let a = 2; // La variable local 'a' está en TDZ hasta esta línea
}
```

La TDZ ayuda a evitar errores al impedir el uso de variables antes de su inicialización.

## 11. Prototypes y Herencia Prototípica

JavaScript usa herencia prototípica en lugar de clases tradicionales.

```javascript
// Construir objetos con prototipos
function Animal(nombre) {
  this.nombre = nombre;
}

Animal.prototype.saludar = function() {
  return `Hola, soy ${this.nombre}`;
};

function Perro(nombre, raza) {
  Animal.call(this, nombre);
  this.raza = raza;
}

// Establecer herencia
Perro.prototype = Object.create(Animal.prototype);
Perro.prototype.constructor = Perro;

// Añadir métodos al prototipo hijo
Perro.prototype.ladrar = function() {
  return "¡Guau!";
};

const miPerro = new Perro("Max", "Labrador");
console.log(miPerro.saludar()); // "Hola, soy Max"
console.log(miPerro.ladrar()); // "¡Guau!"

// Verificar prototipos
console.log(miPerro instanceof Perro); // true
console.log(miPerro instanceof Animal); // true
```

## 12. Clases en JavaScript (ES6+)

Las clases son "azúcar sintáctico" sobre los prototipos, pero tienen particularidades.

```javascript
class Animal {
  constructor(nombre) {
    this.nombre = nombre;
  }
  
  saludar() {
    return `Hola, soy ${this.nombre}`;
  }
  
  // Métodos estáticos
  static esAnimal(obj) {
    return obj instanceof Animal;
  }
}

class Perro extends Animal {
  constructor(nombre, raza) {
    super(nombre); // Llamada obligatoria al constructor padre
    this.raza = raza;
  }
  
  saludar() {
    return `${super.saludar()} y soy un ${this.raza}`;
  }
  
  ladrar() {
    return "¡Guau!";
  }
}

const miPerro = new Perro("Rex", "Pastor Alemán");
console.log(miPerro.saludar()); // "Hola, soy Rex y soy un Pastor Alemán"
console.log(Animal.esAnimal(miPerro)); // true

// Error común: olvidar super() en constructor
class ErrorClase extends Animal {
  constructor(nombre) {
    // ReferenceError: Must call super constructor before accessing 'this'
    this.nombre = nombre;
  }
}
```

## 13. Bind, Call y Apply

Métodos para controlar explícitamente el valor de `this`.

```javascript
function saludar(saludo, puntuacion) {
  return `${saludo}, soy ${this.nombre}${puntuacion}`;
}

const persona = { nombre: "Diego" };

// call: pasar argumentos individualmente
console.log(saludar.call(persona, "Hola", "!")); // "Hola, soy Diego!"

// apply: pasar argumentos como array
console.log(saludar.apply(persona, ["Saludos", "..."])); // "Saludos, soy Diego..."

// bind: crear nueva función con this enlazado
const saludarDiego = saludar.bind(persona);
console.log(saludarDiego("Buenos días", ".")); // "Buenos días, soy Diego."

// bind con parámetros parciales
const holaDiego = saludar.bind(persona, "Hola");
console.log(holaDiego("!!")); // "Hola, soy Diego!!"
```

## 14. Callbacks Hell y su Solución

El anidamiento excesivo de callbacks dificulta la legibilidad y mantenimiento.

```javascript
// Callback hell: peticiones anidadas
obtenerUsuario(userId, function(usuario) {
  obtenerPermisos(usuario.id, function(permisos) {
    obtenerRoles(permisos.nivel, function(roles) {
      obtenerAccesos(roles, function(accesos) {
        console.log(accesos);
        // Más anidamiento...
      }, handleError);
    }, handleError);
  }, handleError);
}, handleError);

// Solución 1: Funciones nombradas
function procesarAccesos(accesos) {
  console.log(accesos);
}

function obtenerAccesosConRoles(roles) {
  obtenerAccesos(roles, procesarAccesos, handleError);
}

function obtenerRolesConPermisos(permisos) {
  obtenerRoles(permisos.nivel, obtenerAccesosConRoles, handleError);
}

function obtenerPermisosConUsuario(usuario) {
  obtenerPermisos(usuario.id, obtenerRolesConPermisos, handleError);
}

obtenerUsuario(userId, obtenerPermisosConUsuario, handleError);

// Solución 2: Promesas
obtenerUsuarioPromise(userId)
  .then(usuario => obtenerPermisosPromise(usuario.id))
  .then(permisos => obtenerRolesPromise(permisos.nivel))
  .then(roles => obtenerAccesosPromise(roles))
  .then(accesos => console.log(accesos))
  .catch(handleError);

// Solución 3: Async/await
async function obtenerAccesosDeUsuario(userId) {
  try {
    const usuario = await obtenerUsuarioPromise(userId);
    const permisos = await obtenerPermisosPromise(usuario.id);
    const roles = await obtenerRolesPromise(permisos.nivel);
    const accesos = await obtenerAccesosPromise(roles);
    console.log(accesos);
    return accesos;
  } catch (error) {
    handleError(error);
  }
}
```

## 15. Promise Chaining y Composición

Encadenar promesas correctamente es clave para código asíncrono efectivo.

```javascript
// Composición básica
fetchUsuario(id)
  .then(usuario => fetchPosts(usuario.id))
  .then(posts => {
    console.log(posts);
    return posts;
  })
  .catch(error => console.error(error));

// Error común: Romper la cadena
fetchUsuario(id)
  .then(usuario => {
    // Se rompe la cadena al no devolver la promesa
    fetchPosts(usuario.id);
  })
  .then(posts => {
    // posts será undefined
    console.log(posts);
  });

// Correcto: Mantener la cadena
fetchUsuario(id)
  .then(usuario => {
    return fetchPosts(usuario.id);
  })
  .then(posts => {
    console.log(posts);
  });

// Ejecución en paralelo
Promise.all([
  fetchUsuario(id),
  fetchConfig(),
  fetchPreferencias(id)
])
.then(([usuario, config, preferencias]) => {
  console.log(usuario, config, preferencias);
})
.catch(error => {
  // Un solo error detiene todo
  console.error("Algo falló:", error);
});

// First settled con Promise.race
Promise.race([
  fetchLocal(),
  fetchRemote()
])
.then(resultado => console.log("El más rápido:", resultado));
```

## 16. Async/await en Ciclos y Operaciones Paralelas

Manejar múltiples operaciones asíncronas puede ser confuso.

```javascript
const ids = [1, 2, 3, 4, 5];

// ❌ INCORRECTO: map con async no funciona como se espera
async function procesarTodosIncorrecto() {
  // Promise<undefined[]> - no espera a que se resuelvan las promesas
  const resultados = ids.map(async (id) => {
    const data = await fetchData(id);
    return data;
  });
  
  console.log(resultados); // Array de promesas, no de resultados
}

// ✅ CORRECTO: Promise.all con map
async function procesarTodosParalelo() {
  const promesas = ids.map(id => fetchData(id));
  const resultados = await Promise.all(promesas);
  console.log(resultados); // Array con todos los resultados
  return resultados;
}

// ✅ CORRECTO: Procesamiento secuencial
async function procesarTodosSecuencial() {
  const resultados = [];
  
  for (const id of ids) {
    // Espera a cada uno antes de continuar
    const data = await fetchData(id);
    resultados.push(data);
  }
  
  console.log(resultados);
  return resultados;
}

// forEach con async (cuidado: no espera)
async function procesoForEach() {
  let contador = 0;
  
  ids.forEach(async (id) => {
    await fetchData(id);
    contador++;
    console.log(`Procesado: ${contador}`);
  });
  
  console.log(`Total procesado: ${contador}`); // Siempre 0 porque no espera
}
```

## 17. Manejo Avanzado de Promesas

Técnicas para escenarios complejos con promesas.

```javascript
// Timeout para promesas
function conTimeout(promesa, ms) {
  const timeout = new Promise((_, reject) => {
    setTimeout(() => reject(new Error('Timeout excedido')), ms);
  });
  
  return Promise.race([promesa, timeout]);
}

// Retry para promesas
async function conReintentos(fn, maxIntentos = 3, delay = 1000) {
  let error;
  
  for (let intento = 1; intento <= maxIntentos; intento++) {
    try {
      return await fn();
    } catch (err) {
      console.log(`Intento ${intento} fallido`);
      error = err;
      
      if (intento < maxIntentos) {
        await new Promise(r => setTimeout(r, delay));
      }
    }
  }
  
  throw error;
}

// Uso
conReintentos(() => fetchData(5))
  .then(data => console.log("Datos:", data))
  .catch(err => console.error("Todos los intentos fallaron:", err));

// Cancelación de promesas con AbortController
function fetchCancelable(url) {
  const controller = new AbortController();
  const signal = controller.signal;
  
  const promise = fetch(url, { signal })
    .then(response => response.json());
  
  return {
    promise,
    cancel: () => controller.abort()
  };
}

const { promise, cancel } = fetchCancelable('/api/data');

// Cancelar después de 2 segundos
setTimeout(() => {
  cancel();
  console.log("Petición cancelada");
}, 2000);
```

## 18. Módulos en JavaScript

Los sistemas de módulos tienen diferencias importantes.

```javascript
// CommonJS (Node.js)
// archivo: matematica.js
function sumar(a, b) {
  return a + b;
}

function restar(a, b) {
  return a - b;
}

module.exports = {
  sumar,
  restar
};

// archivo: app.js
const matematica = require('./matematica');
console.log(matematica.sumar(5, 3)); // 8

// ES Modules (ECMAScript)
// archivo: matematica.js
export function sumar(a, b) {
  return a + b;
}

export function restar(a, b) {
  return a - b;
}

export default function multiplicar(a, b) {
  return a * b;
}

// archivo: app.js
import multiplicar, { sumar, restar } from './matematica.js';
console.log(sumar(5, 3)); // 8
console.log(multiplicar(4, 2)); // 8

// Importación dinámica
async function cargarModulo() {
  if (condicion) {
    const { default: fn } = await import('./modulo1.js');
    return fn();
  } else {
    const { default: fn } = await import('./modulo2.js');
    return fn();
  }
}
```

## 19. Garbage Collection y Memory Leaks

Evitar fugas de memoria es crucial para aplicaciones de larga ejecución.

```javascript
// Ejemplo de memory leak con closures
function crearTareas() {
  const tareas = [];
  const datos = Array(10000).fill('datos grandes');
  
  for (let i = 0; i < 1000; i++) {
    tareas.push(() => {
      console.log('Tarea', i);
      // Cada función retiene una referencia a todo el array 'datos'
      return datos[i % 100];
    });
  }
  
  return tareas;
}

const tareas = crearTareas();
// 'datos' nunca se libera porque todas las tareas lo referencian

// Corrección
function crearTareasSinLeak() {
  const tareas = [];
  
  for (let i = 0; i < 1000; i++) {
    // Capturar solo lo necesario
    const indice = i % 100;
    tareas.push(() => {
      console.log('Tarea', i);
      // Buscar datos cuando sea necesario
      const datos = obtenerDatos();
      return datos[indice];
    });
  }
  
  return tareas;
}

// Memory leak en event listeners
function setupEvents() {
  const datos = cargarDatosGrandes();
  
  document.getElementById('boton').addEventListener('click', function() {
    console.log(datos.length);
  });
}

// Si setupEvents se llama múltiples veces sin remover los listeners,
// cada referencia a 'datos' permanece en memoria

// Corrección
function setupEventsSinLeak() {
  const datos = cargarDatosGrandes();
  const boton = document.getElementById('boton');
  
  const handler = function() {
    console.log(datos.length);
  };
  
  boton.addEventListener('click', handler);
  
  // Devolver función para limpiar
  return function limpiar() {
    boton.removeEventListener('click', handler);
  };
}

const limpiar = setupEventsSinLeak();
// Cuando ya no necesitemos el evento:
limpiar();
```

## 20. Destructuring y Spread Operator

Estas características pueden ser confusas en aplicaciones avanzadas.

```javascript
// Destructuring básico
const persona = { nombre: 'Elena', edad: 28, ciudad: 'Madrid' };
const { nombre, edad } = persona;

// Asignación con nuevos nombres
const { nombre: nombreCompleto, edad: años } = persona;
console.log(nombreCompleto); // 'Elena'

// Valores por defecto
const { profesion = 'Desconocida' } = persona;
console.log(profesion); // 'Desconocida'

// Destructuring anidado
const usuario = {
  id: 123,
  perfil: {
    nombre: 'Carlos',
    social: {
      twitter: '@carlos',
      instagram: '@carlosdev'
    }
  }
};

const { perfil: { nombre, social: { twitter } } } = usuario;
console.log(nombre, twitter); // 'Carlos' '@carlos'

// Destructuring de arrays
const colores = ['rojo', 'verde', 'azul'];
const [primario, secundario, terciario] = colores;

// Ignorar elementos
const [primero, , tercero] = colores;

// Rest con destructuring
const [principal, ...resto] = colores;
console.log(resto); // ['verde', 'azul']

// Spread para clonación y fusión
const original = { a: 1, b: 2 };
const copia = { ...original };

// Fusión de objetos
const objeto1 = { a: 1, b: 2 };
const objeto2 = { b: 3, c: 4 }; // b se sobrescribe
const combinado = { ...objeto1, ...objeto2 };
console.log(combinado); // { a: 1, b: 3, c: 4 }

// Error común: copia superficial
const original = { a: 1, b: { c: 2 } };
const copia = { ...original };
copia.b.c = 3;
console.log(original.b.c); // 3 (también modificado)
```

## 21. Web APIs y su Naturaleza Asíncrona

Entender cómo interactuar correctamente con APIs del navegador.

```javascript
// Geo-localización
function obtenerUbicacion() {
  return new Promise((resolve, reject) => {
    navigator.geolocation.getCurrentPosition(
      position => {
        resolve({
          lat: position.coords.latitude,
          lng: position.coords.longitude
        });
      },
      error => {
        reject(`Error de geolocalización: ${error.message}`);
      }
    );
  });
}

// Uso
obtenerUbicacion()
  .then(ubicacion => console.log('Estás en:', ubicacion))
  .catch(error => console.error(error));

// IndexedDB con promesas
function guardarEnDB(datos) {
  return new Promise((resolve, reject) => {
    const request = indexedDB.open('miBaseDatos', 1);
    
    request.onerror = event => reject('Error al abrir DB');
    
    request.onupgradeneeded = event => {
      const db = event.target.result;
      if (!db.objectStoreNames.contains('items')) {
        db.createObjectStore('items', { keyPath: 'id' });
      }
    };
    
    request.onsuccess = event => {
      const db = event.target.result;
      const transaction = db.transaction(['items'], 'readwrite');
      const store = transaction.objectStore('items');
      
      const addRequest = store.add(datos);
      
      addRequest.onsuccess = () => resolve('Datos guardados');
      addRequest.onerror = () => reject('Error al guardar');
    };
  });
}

// Intersection Observer
function observarElementos(elementos, callback) {
  const observer = new IntersectionObserver((entries, observer) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        callback(entry.target);
        observer.unobserve(entry.target);
      }
    });
  });
  
  elementos.forEach(elemento => observer.observe(elemento));
  
  return {
    detener: () => observer.disconnect()
  };
}

// Uso
const imagenes = document.querySelectorAll('.lazy-image');
observarElementos(imagenes, elemento => {
  elemento.src = elemento.dataset.src;
  elemento.classList.add('loaded');
});
```

## 22. Service Workers y Cache API

Fundamental para PWAs y rendimiento offline.

```javascript
// Registrar Service Worker
navigator.serviceWorker.register('/sw.js')
  .then(registration => {
    console.log('SW registrado:', registration.scope);
  })
  .catch(error => {
    console.error('Error al registrar SW:', error);
  });

// sw.js - Service Worker básico
const CACHE_NAME = 'mi-app-v1';
const RECURSOS = [
  '/',
  '/index.html',
  '/estilos.css',
  '/app.js',
  '/offline.html'
];

// Instalación y cacheo inicial
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => {
        console.log('Cache abierto');
        return cache.addAll(RECURSOS);
      })
  );
});

// Estrategia cache-first
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request)
      .then(respuesta => {
        // Devuelve de cache o va a red
        return respuesta || fetch(event.request)
          .then(respuesta => {
            // Guardar en cache la nueva respuesta
            return caches.open(CACHE_NAME)
              .then(cache => {
                cache.put(event.request, respuesta.clone());
                return respuesta;
              });
          });
      })
      .catch(() => {
        // Fallback para navegación offline
        if (event.request.mode === 'navigate') {
          return caches.match('/offline.html');
        }
      })
  );
});

// Limpiar caches antiguas
self.addEventListener('activate', event => {
  event.waitUntil(
    caches.keys().then(nombres => {
      return Promise.all(
        nombres.filter(nombre => {
          return nombre !== CACHE_NAME;
        }).map(nombre => {
          return caches.delete(nombre);
        })
      );
    })
  );
});
```
