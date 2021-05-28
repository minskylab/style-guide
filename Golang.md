# Golang Style Guide

https://golangci-lint.run/

https://github.com/mvdan/gofumpt

## Introducción

En la siguiente guía se plantearan algunos patrones y diseños para mantener un estándar en la escritura de código limpio promoviendo siempre la legibilidad, entendimiento, etc.

## Convención de Nombres

### Comentarios

En Go, cada variable y función publica debe de ser comentada, esto a favor de que haya una buena consistencia a la hora de documentar el código. Sin embargo estos a diferencia de los comentarios explicativos, deben de explicar mas un uso general del mismo sin llegar a centrarse tanto en la implementación.

Por lo que, comentarios como el siguiente son una forma en la que no se debería comentar un código:

```go
// iterate over the range 0 to 9 
// and invoke the doSomething function
// for each iteration
for i := 0; i < 10; i++ {
  doSomething(i)
}
```

Estos comentarios son común de verlos en tutoriales a la vez que son de ayuda en los mismos, sin embargo no son útiles cuando se quiere trabajar código de producción ya que al suponerse que todos ya saben cosas tan básicas como un ```for```, estos se hacen innecesarios.

En cambio se puede optar más por un comentario explicativo el cual sobretodo describa lo que se quiere hacer con la siguiente porción de código:

```go
// instatiate 10 threads to handle upcoming work load
for i := 0; i < 10; i++ {
  doSomething(i)
}
```

En este caso se aplican de una mejor manera los comentarios al estar ahí para explicar la lógica del código sin acercarse tanto a describir la implementación.

Finalmente también se pueden evitar los comentarios y proceder con un código que por sus nombres se defina a si mismo:

```go
for workerID := 0; workerID < 10; workerID++ {
  instantiateThread(workerID)
}
```

### Nombramiento de Funciones

Las reglas que se procederán a aplicar a las funciones son las siguientes:

- El nombre de cada función o método deberá de adecuarse al formato Pascal Case(EjemploDePascalCase)

- Se sigue la regla de, conforme mas especifica es la función o método este debe de tener un nombre más especifico, por ejemplo:

  ```go
  func main() {
      configpath := flag.String("config-path", "", "configuration file path")
      flag.Parse()
  
      config, err := configuration.Parse(*configpath)
      
      ...
  }
  ```

  En este caso podemos centrarnos en la función `Parse`, esta muestra claramente su funcionalidad en su nombre y no necesita ser tan especifica, sin embargo si ahora nos centramos en su implementación:

  ```go
  func Parse(filepath string) (Config, error) {
      switch fileExtension(filepath) {
      case "json":
          return ParseJSON(filepath)
      case "yaml":
          return ParseYAML(filepath)
      case "toml":
          return ParseTOML(filepath)
      default:
          return Config{}, ErrUnknownFileExtension
      }
  }
  ```

  Podemos observar como es que el nombre de las funciones pasan a ser algo mas especificas, si en caso se le hubiera colocado otro nombre a `ParseJSON()` como por ejemplo `JSON()`, este no hubiera sido lo suficientemente detallado como para saber si se quiere crear, ejecutar un parse o clasificar un json.

  Así sucesivamente mientras cada función tenga una funcionalidad más particular, esta debe tener un nombre igual de especifico

### Nombramiento de variables

A la hora de nombrar las variables se deben de seguir 2 reglas muy similares a las de las funciones:

- El nombre de cada variable debe de seguir el formato Camel Case(ejemploDeCamelCase)

- A diferencia de las funciones, a la hora de nombrar variables se aplica la regla a la inversa, esto quiere decir, mientras más especifica es la variable, más general su nombre debe de ser. Esto es a favor de mantener los nombres generales en el menor alcance posible de tal forma que mientras menor vida tenga una variable, mas general puede ser su nombre.

  ```go
  func BeerBrandListToBeerList(beerBrands []BeerBrand) []Beer {
      var beerList []Beer
      for _, brand := range beerBrands {
          for _, beer := range brand {
              beerList = append(beerList, beer)
          }
      }
      return beerList
  }
  ```

  En este ejemplo, podemos ver como con cada variable que se crea su nombre se define en base a la duración de su vida o su alcance.

  ```go
  func BeerBrandListToBeerList(b []BeerBrand) []Beer {
      var bl []Beer
      for _, beerBrand := range b {
          for _, beerBrandBeerName := range beerBrand {
              bl = append(bl, beerBrandBeerName)
          }
      }
      return bl
  }
  ```

  Sin embargo en este otro ejemplo podemos ver como a pesar de que con algo de esfuerzo se podría entender que hace el código, la brevedad de los nombres hace mas difícil el seguir la lógica de el código.

## Reglas de Indentación

En Go se trabajan mayormente con tabs a la hora de indentar por lo que se prefiere evitar el uso de espacios a no ser que sea necesario.

Se facilita la herramienta de gofmt para todo lo que tenga que ver con arreglar los estilos del código, sin embargo no por ello se deben de perder las buenas practicas.

## Errores comunes a la hora de escribir código

### Interfaces

El estilo de interfaces que existen en Go permite que todas las estructuras que implementen los métodos de una interfaz, automáticamente implemente la interfaz implícitamente, esto muchas veces puede llegar a generar ciertos errores si es que no se tienen todos los métodos correctamente implementados, en Go hay ciertas formas de solucionar estos errores aunque de las más usadas son:

```go
type Writer interface {
	Write(p []byte) (n int, err error)
}

type NullWriter struct {}

func (writer *NullWriter) Write(data []byte) (n int, err error) {
    // do nothing
    return len(data), nil
}

func NewNullWriter() io.Writer {
    return &NullWriter{}
}
```

En este caso gracias al uso del constructor se puede forzar a que cierta estructura implemente una interfaz para poder ser generada.

Otra solución podría ser:

```go
type Writer interface {
	Write(p []byte) (n int, err error)
}

type NullWriter struct {}
var writer io.Writer = &NullWriter{}
```

En este ejemplo se puede observar que el tipo de la variable no se define implícitamente, sino que es el usuario que define que sea un tipo `Writer` y por ende que lo que contenga implemente la interface.

### Concatenación de sentencias if

Muchas veces a la hora de escribir código se comete la necesidad de concatenar y generar diversos niveles de Indentación por el uso de los `if`, si bien muchas veces pueden llegar a ser necesarias y no se pueden evitar, muchas otras si que se pueden por lo que se procederá a plantear algunos ejemplos sobre como evitar este tipo de situaciones.

#### A través de errores

```go
func (g *Gopher) WriteTo(w io.Writer) (size int64, err error) {
    err = binary.Write(w, binary.LittleEndian, int32(len(g.Name)))
    if err == nil {
        size += 4
        var n int
        n, err = w.Write([]byte(g.Name))
        size += int64(n)
        if err == nil {
            err = binary.Write(w, binary.LittleEndian, int64(g.AgeYears))
            if err == nil {
                size += 4
            }
            return
        }
        return
    }
    return
}
```

Si analizamos el código podemos ver una secuencia de ifs que manejan diversos errores a la hora de ejecutar el programa, para este tipo de situaciones lo mas recomendable es manejar primero los errores y luego generar la lógica.

```go
func (g *Gopher) WriteTo(w io.Writer) (size int64, err error) {
    err = binary.Write(w, binary.LittleEndian, int32(len(g.Name)))
    if err != nil {
        return
    }
    size += 4
    n, err := w.Write([]byte(g.Name))
    size += int64(n)
    if err != nil {
        return
    }
    err = binary.Write(w, binary.LittleEndian, int64(g.AgeYears))
    if err == nil {
        size += 4
    }
    return
}
```

El este caso ya vemos como se ha evitado la situación anterior manejando primero la existencia de errores y finalizando el programa en caso de existir alguno.

#### Por no invertir la condición

Muchas veces aunque no nos demos cuenta, hay diversos casos en donde tan solo invertir la sentencia nos podría evitar ciertas concatenaciones de ifs, esto se genera por que muchas veces se cuestiona el "Que pasa si?" y no se cuestiona el "Que pasa si no?".

```go
func Fibonacci(position int) int {
    if position != 0 && position != 1 {
        return Fibonacci(position-1) + Fibonacci(position-2)
    } else {
        if position == 1 {
            return 1
        } else {
            return 0
        }
    }
}
```

En este caso evaluamos un "Que pasa si *position* es diferente de 0 y de 1?", sin embargo no se ha considerado un "Que pasa si *position* no es diferente de 0 o de 1?",  al pensarlo de esta manera, permite refactorizar el código de la siguiente forma:

```go
func Fibonacci(position int) int {
    if position == 0 {
        return 0
    }
    if position == 1 {
        return 1
    }
    return Fibonacci(position-1) + Fibonacci(position-1)
}
```

### Creando referencias a la variable iterable

Cuando se trabaja con una variable que itera, muchas veces podría parecer conveniente a simple vista el usar una referencia a la misma, esto puede llegar a ser bastante eficiente pero también nos puede generar errores como el siguiente:

```go
func main() {
	var out []*int
	for i := 0; i < 3; i++ {
		out = append(out, &i)
	}
	fmt.Println("Values:", *out[0], *out[1], *out[2])
	fmt.Println("Addresses:", out[0], out[1], out[2])
}
```

Lo cual nos daría como salida:

```
Values: 3 3 3
Addresses: 0x40e020 0x40e020 0x40e020
```

Si bien el código ejecuta no nos da el resultado esperado puesto que nos esta devolviendo los valores "3 3 3" en lugar de "0 1 2", esto se da ya que al referenciar a la variable i, esta va cambiando y cuando se accede a su dirección se accede al valor actual del mismo i almacenado en las 3 posiciones del arreglo, de esta forma al llamarse al valor se estaría buscando el ultimo valor tomado por i en las iteraciones el cual seria 3.

Una forma de solucionar el error podría ser el siguiente:

```go
func main() {
	var out []*int
	for i := 0; i < 3; i++ {
        i := i
		out = append(out, &i)
	}
	fmt.Println("Values:", *out[0], *out[1], *out[2])
	fmt.Println("Addresses:", out[0], out[1], out[2])
}
```

Lo cual nos resultaría en:

```
Values: 0 1 2
Addresses: 0x40e020 0x40e024 0x40e028
```

En este caso i se esta re-definiendo como una variable nueva por cada iteración y por lo tanto al agregar la referencia al arreglo, se agrega la nueva variable creada y no la variable que itera.

### A través de Gorrutinas

Cuando se desea procesar cierto numero grande de información muchas veces se puede considerar el uso de hilos, si bien esto es muy utilizado en Go por la facilidad que nos ofrecen la Gorrutinas, si no se manejan bien pueden llegar a generar errores no muy bien identificados.

```go
for _, val := range values {
	go func() {
		fmt.Println(val)
	}()
}
```

En este ejemplo existe una gran posibilidad que el código imprima un valor incorrecto puesto que al estar ejecutando la función con la misma variable, hay muchas posibilidades de que las funciones recién se empiecen a ejecutar cuando el loop haya terminado y por ende imprimir el mismo valor en cada una de sus llamadas.

Si se quisiera solucionar el error anterior una buena opción podría ser:

```go
for _, val := range values {
	go func(val interface{}) {
		fmt.Println(val)
	}(val)
}
```

En este caso val esta siendo evaluada con cada iteración y su valor se esta almacenando en una pila para el uso de la Gorrutina por lo que su valor por cada iteración se conservaría.