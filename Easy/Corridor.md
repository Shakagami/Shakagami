Navegamos en la pagina web nos encontraremos con muchas puertas, si inspeccionamos la pagina, vamos a ver que cada puerta tiene un hash . Al clickear puerta por puerta, obtendremos el hash en la URL :

```ruby
Puerta 1 --> 'c4ca4238a0b923820dcc509a6f75849b' --> 1
Puerta 2 --> 'c81e728d9d4c2f636f067f89cc14862c' --> 2
Puerta 3 --> 'eccbc87e4b5ce2fe28308fd9f2a7baf3' --> 3
Puerta 4 --> 'a87ff679a2f3e71d9181a67b7542122c' --> 4
Puerta 5 --> 'e4da3b7fbbce2345d7772b0674a318d5' --> 5
Puerta 6 --> '1679091c5a880faf6fb5e6087eb1b2dc' --> 6
Puerta 7 --> '8f14e45fceea167a5a36dedd4bea2543' --> 7
Puerta 8 --> 'c51ce410c124a10e0db5e4b97fc2af39' --> 13
Puerta 9 --> 'c20ad4d76fe97759aa27a0c99bff6710' --> 12
Puerta 10 -> '6512bd43d9caa6e02c990b0a82652dca' --> 11
Puerta 11 -> 'd3d9446802a44259755d38e6d163e820' --> 10
Puerta 12 -> '45c48cce2e2d7fbdea1afc51c7c6ad26' --> 9
Puerta 13 -> 'c9f0f895fb98ab9159f51fd0297e236d' --> 8

```

Tendremos que encontrar la flag, cada puerta te lleva a un numero, y al juntarlas es una numeracion del 1 al 13, por lo que si agregamos al final de la URL el numero 0 obtendremos el hash oculto, que al ponerlo en la URL obtendremos la flag.
# Respuestas

Puerta 0 = cfcd208495d565ef66e7dff9f98764da