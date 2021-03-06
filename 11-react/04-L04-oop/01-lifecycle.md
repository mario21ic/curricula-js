# Estado local y ciclo de vida

Hay ocasiones en las que el uso de un estado global y compartido, no es la forma ideal de expresar el estado de nuestro componente, ya sea porque es demasiado complicado, porque no queremos contaminar el estado global con estados internos de muy bajo nivel, u otros motivos.

Tomemos el siguiente ejemplo:

```javascript
function Reloj({ fecha }) {
  return (
    <div>
      <h1>Hola Mundo!</h1>
      <h2>Son las {fecha.toLocaleTimeString()}.</h2>
    </div>
  );
}

function tick() {
  ReactDOM.render(
    <Reloj fecha={new Date()} />,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

[Try it on CodePen.](http://codepen.io/gaearon/pen/gwoJZk?editors=0010)

> Fíjate que para mantener la continuidad de la actualización tenemos que usar `ReactDOM.render()` para cambiar el `output` cada vez.

En esta sección veremos cómo hacer de `Reloj` un componente reusable y encapsulado, pero en lugar de usar el enfoque funcional, vamos a utilizar el enfoque ***orientado a objetos*** que provee `React`: `Reloj` se encargará de configurar su propio timer y actualizarse por si sólo cada segundo.

Nuestro objetivo es escribir una única vez el siguiente código

```javascript
ReactDOM.render(
  <Reloj />,
  document.getElementById('root')
);
```
y que `Reloj` se encargue el mismo de todo lo necesario para seguir funcionando.

Para implementar esto, necesitamos agregar un **estado local** (`state`) al componente `Reloj`. El `state` local es similar a las `props`, con la diferencia de que es privado y completamente controlado por el componente.

Los componentes definidos como `class` tienen algunos `features` adicionales, no presentes en los componentes `stateless` . El `state` local es uno de ellos.


## Conviertiendo una función en clase

Para convertir un componente `stateless` como `Reloj` en una clase, sigue estos 4 pasos:

1. Crea una [clase de `ES6`]([ES6 class](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes) (`class`) y has que extienda de `React.Component`.

2. Crean un método en tu clase, llamado `render()`.

3. Mueve el cuerpo de tu antigua funcion al cuerpo de tu método `render()`.

4. Tus `props` que antes recibias como parámetro, ahora están accesibles a través `this.props` dentro de `render()`.

```js
class Reloj extends React.Component {
  render() {
    const { fecha } = this.props
    return (
      <div>
        <h1>Hola Mundo!</h1>
        <h2>Son las {fecha.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

[Try it on CodePen.](http://codepen.io/gaearon/pen/zKRGpo?editors=0010)

`Reloj` ahora esta definido como class en lugar de como función. Esto nos permitirá usar funcionalidades adicionales como el estado local y los *hooks* para las diferentes fases del ciclo de vida del componente.

## Agregando un estado local

Ahora, moveremos la `fecha` de `props` al `state` en 3 pasos:

1. Reemplazamos la asignación desde `this.props` a `this.state`

  ```js
  class Reloj extends React.Component {
    render() {
      const { fecha } = this.state
      return (
        <div>
          <h1>Hola Mundo!</h1>
          <h2>Son las {fecha.toLocaleTimeString()}.</h2>
        </div>
      );
    }
  }
  ```

2. Agregamos un [constructor](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes#Constructor) que asigne el valor inicial de `this.state`:

  ```js
  class Reloj extends React.Component {
    constructor(props) {
      super(props);
      this.state = { fecha: new Date() };
    }

    render() {
      const { fecha } = this.state
      return (
        <div>
          <h1>Hola Mundo!</h1>
          <h2>Son las {fecha.toLocaleTimeString()}.</h2>
        </div>
      );
    }
  }
  ```

  Nota como pasamos las `props` al constructor de `React.Component`:

  ```js
    constructor(props) {
      super(props);
      this.state = { fecha: new Date() };
    }
  ```

  > Los componentes `statefull` **siempre** deverían invocar al constructor de la superclase con `props` como parámetro.

3. Quita la propiedad `fecha` del elemento `<Reloj />`:

  ```js
  ReactDOM.render(
    <Reloj />,
    document.getElementById('root')
  );
  ```

The result looks like this:

```js
class Reloj extends React.Component {
  constructor(props) {
    super(props);
    this.state = { fecha: new Date() };
  }

  render() {
    return (
      <div>
        <h1>Hola Mundo!</h1>
        <h2>Son las {fecha.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Reloj />,
  document.getElementById('root')
);
```

[Try it on CodePen.](http://codepen.io/gaearon/pen/KgQpJd?editors=0010)

Ahora, vamos a hacer que `Reloj` configure su propio timer y se actualice por si mismo cada segundo.

## Agregando métodos del ciclo de vida (lifecycle)

En las aplicaciones con muchos componentes, es muy importante liberar los recursos bloqueados, una vez que el componente que los necesita es destruído.

Entonces, nosotros queremos [configurar un timer](https://developer.mozilla.org/es-ES/docs/Web/API/WindowTimers/setInterval) al momento que `Reloj` es inyectado en el `DOM` por primera vez. Esto se llama *montar* (`mount`) un componente en la jerga `React`.

Y de manera análoga, queremos [limpiar el  timer](https://developer.mozilla.org/es-ES/docs/Web/API/WindowTimers/clearInterval) cuando el DOM producido por `Reloj` es removido. Esto se llama *desmontar* (`unmount`) un componente.

Definimos métodos espciales dentro de nuestra clase para ejecutar instruciones cada vez que el componente se *monta* o *desmonta*:

```js
class Reloj extends React.Component {
  constructor(props) {
    super(props);
    this.state = { fecha: new Date() };
  }

  componentDidMount() {

  }

  componentWillUnmount() {

  }

  render() {
    return (
      <div>
        <h1>Hola Mundo!</h1>
        <h2>Son las {fecha.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

Estos métodos se llaman "lifecycle hooks".

El hook `componentDidMount()` se ejecuta después de que el output del componente haya sido renderizado en el DOM. Este es un buen lugar en donde inicializar nuestro timer:

```js
  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }
```

Fíjate que guardamos el ID del timer creado, directamente como una propiedad de `this`.

Mientras que `this.props` es setteado por React y `this.state` tiene un significado especial, como en cualquier clase, eres libre de agregar manualmente campos adicionales a tu clase si necesitas guardar alguna información, siempre y cuando su valor no esté relacionado con el output de tu `render()`.

> Si no lo estás usando en el `render()`, no debería estar en el `state`.

Y ahora limpiamos el timer en `componentWillUnmount()`:

```js
  componentWillUnmount() {
    clearInterval(this.timerID);
  }
```

Por último, vamos a implementar el método `tick()` que se ejecuta a cada segundo.

Este método hará uso de `this.setState()` para disparar las modificaciones del estado local:

```js
class Reloj extends React.Component {
  constructor(props) {
    super(props);
    this.state = { fecha: new Date() };
  }

  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      fecha: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hola Mundo!</h1>
        <h2>Son las {fecha.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Reloj />,
  document.getElementById('root')
);
```

[Try it on CodePen.](http://codepen.io/gaearon/pen/amqdNA?editors=0010)

## Recomendaciones para usar `state` correctamente

Hay 3 cosas que debes saber de `setState()`.

### Nunca modifiques el `state` directamente

```js
// Mal
this.state.saludo = 'Hello'
```

```js
// Bien
this.setState({ saludo: 'Hello' })
```

> El único lugar en donde podrías asignar `this.state` directamente es en el constructor.

### Las modificaciones al `state` pueden ser asíncronas

Por performance, `React` puede juntar multiples llamadas `setState()` en un solo update.

Como `this.props` y `this.state` podrían modificarse asíncronamente, no deberías depender de sus valores para calcular el nuevo estado.

Por ejemplo, el siguiente código prodría fallar:

```js
// Mal
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

En estas situaciones, utiliza una segunda forma de `setState()` que acepta una función en lugar de un objeto. Esta función recibirá un como primer argumento, el estado anterior y las `props` al ese momento como segundo argumento:

```js
// Bien
this.setState((prevState, props) => ({
  counter: prevState.counter + props.increment
}));
```

### Las actualizaciones del `state` se "funden"

Cada vez que tu llamas a `setState()`, `React` toma como base el estado actual y pisa las propiedades definidas en tu objeto.

Por ejemplo, tenemos un estado con dos propiedades independientes

```js
  constructor(props) {
    super(props);
    this.state = {
      posts: [],
      comments: []
    };
  }
```

Que podemos modificar granularmente

```js
  componentDidMount() {
    fetchPosts({ err, response }) => {
      // handle err
      this.setState({
        posts: response.posts
      });
    });

    fetchComments({ err, response }) => {
      // handle err
      this.setState({
        comments: response.comments
      });
    });
  }
```

Cuando se ejecuta `this.setState({ comments })`, `this.state.posts` queda intacto, mientras que `this.state.comments` es completamente reemplazado.

## Encapsulamiento

Por último, para demostrar cuán verdaderamente encampsulado es nuestro `Reloj`, te proponemos el siguiente exprimento: crea un componente `App` que renderice 3 `<Reloj>`es:

```js
function App() {
  return (
    <div>
      <Reloj />
      <Reloj />
      <Reloj />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

[Try it on CodePen.](http://codepen.io/gaearon/pen/vXdGmd?editors=0010)

Como puedes ver cada `Reloj` inicia su propio timer y se actualiza de manera independiente.

> En `React` si un componente es `statefull` o `stateless` se considera un detalle de implementación del componente y esta condición puede cambiar con el tiempo. Esto quiere decir que puedes usar componentes `stateless` dentro de componentes `stateful` y vice-versa.

> Sólo debes utilizar el estado local de un componente cuando **puedas asegurar** que no necesitarás compartir esa información con el "exterior"

<br />

> Chequea todos los *hooks* disponibles en la [documentación de `React`](https://facebook.github.io/react/docs/react-component.html#the-component-lifecycle)

> ![Component lifecycle](https://github.com/Laboratoria/curricula-js/raw/react-L05/11-react/04-L04-oop/static/lifecycle.jpg)
