# Laravel 20-2 
## Lunes

### Creación de nuevo proyecto

> laravel new MiProyecto 
>
---
## Martes
### Autenticación

Implementar  Registro e Inicio de sesión con ayuda de composer

> composer require laravel/ui


> php artisan ui vue --auth

> npm install

> npm run dev

Corremos las migraciones
> php artisan migrate

Si hemos configurado nuestra base de datos correctamente ya podemos hacer el registro de un nuevo usuario. Podemos probar también cerrando sesión e iniciando nuevamente. 

## Miércoles

### Rutas y controladores
---

### Integrando vistas 

Copiar las carpetas css, img, js dentro de la carpeta 
>/Public

Hasta ahorita tenemos las vistas de autenticación, home, welcome, y layout/app que nos sirve de plantilla para todas las páginas ya que la navbar es la misma 

La primera vista que modificaremos será la de welcome, que copiaremos de nuestro archivo `index.html`

Para no tener problema usando los archivos de la carpeta public, definiremos las rutas con ayuda de assets de la siguiente manera
 ```html
 <link rel="stylesheet" href="{{asset('css/styles.css')}}">`
```

Eso indica que el archivo se buscará directamente dentro de la carpeta 
>/Public
>
Hacemos lo mismo con todos los archivos **.css**

Para las rutas podemos hacerlo mediante *url* escribiendo la ruta relativa si es que esta no tiene nombre, o si lo tiene, podemos hacer uso de *route*, de la siguiente manera:

```html
<a href="{{ route('register') }}">
<a href="{{ route('login') }}">
 ```

Ahora, haremos uso de las plantillas para poder reciclar nuestra barra de navegación en todas nuestras páginas,  estas se encuentran e la carpeta 
>/Resources/views/layouts/

Copiamos el contenido del archivo `layout_app.html` y lo  reemplazamos por el que está en `app.blade.php` e insertamos la sección de contenido 

```php
@yield('content)
```
Para el idioma podemos ocupar la variable
```php
lang="{{ str_replace('_', '-', app()->getLocale()) }}"
```
Y para el nombre de nuestra aplicación 
```php
 <title>{{ config('app.name') }} - {{ config('app.inicio') }}</title>
 ```
 en config/app.php
 ```php
 'inicio' => "Inicio",
 ```

Modificamos las rutas 
```html
<a href="{{route('login')}}">Iniciar sesión</a>
<a href="{{route('register')}}">Registro</a>
<a href="{{route('home')}}">Inicio</a>
``` 
 Y para cerrar sesión copiamos la siguiente ruta
 ```html
  <a class="dropdown-item" href="{{ route('logout') }}"                            onclick="event.preventDefault();                           document.getElementById('logout-form').submit();">
Cerrar sesión</a>
````
Creamos un usuario para probar

### Integrando vista principal (home)

Copiamos el código que está dentro de la etiqueta ````<body>```` del archivo `home.html` y lo pegamos en
>Resources/views/home.blade.php

Ocultamos las opciones con 
@guest
@else
@endguest

En donde dice 'mi nombre' ponemos la variable una vez que estamos loggeados
 ``{{ Auth::user()->name }}``
----
Ahora sigue reemplazar nuestras vistas de autentificación
Copiamos el contenido de `register.html` y lo pegamos dentro de `register.blade.php` sin borrar lo que se encuentra ahí

Dentro de los inputs vamos reemplazando los atributos y los mensajes de error.

le ponemos a form un metodo post  action="{{route('register')}}"

Los mismo hacemos con `login.html` y `login.blade.php`

---

### Modificar tabla users
Agregar una columna que especifique si el usuario es administrador o no. 
> /database/migrations/create_users_table

Agregamos la columna de tipo booleano que por default tendrá falso, es decir, que no es administrador. 

```php
$table->boolean('admin')->default(false); 
``` 

Dentro de 
>App/User.php

Agregamos **'admin'** en la siguiente sección
````php
protected $fillable = [
        'name', 'email', 'password', 'admin'
]; 
````

y hacemos la migración
>php artisan migrate:fresh

Ahora ya podemos personalizar la barra de navegación para que la opción 'Administrador' solo aparezca a los que lo son.
>resources/views/layouts/app.blade.php

Lo hacemos con una condicional if
```php
@if(Auth::user()->admin)
	<a href="#">Administrador</a>
@endif
 ````

Por default, nuestro primer usuario no es administrador por lo que tenemos que cambiarlo directamente en la base de datos.

---


---
### Perfil de administrador
#### Middleware
Creamos un nuevo middleware
>php artisan make:middleware Admin

Y ponemos dentro de la función:
```php
public function handle($request, Closure $next)
{
	if(!auth()->check()){
	return redirect('/login');
	}
	if(!auth()->user()->admin){
	return redirect('/login');
	}
	
	return $next($request);
}
````
Esto permitirá que unicamente los usuarios administradores tengan acceso.

Ahora tenemos que hacer su ruta en 
>Routes/web.php

Haremos uso de los grupos de rutas ya que hay varias páginas a las que les aplicaremos el mismo Middleware

````php
Route::middleware(['admin'])->group(function () {

});
````

Y ahora todo lo que esté dentro de esa función, no será accesible para usuarios que no sean administradores. 

Únicamente falta darle un alias a ese Middleware para que sea reconocido
>App/Http/Kernel.php
````php
'admin'=>\App\Http\Middleware\Admin::class
````

Ahora odemos hacer una ruta solo para probar que funcione
>Routes/web.php

````php
Route::get('prueba', function () {
	return 5;
});
````
Si buscamos esa ruta en el servidor siendo usuarios administradores no debería haber problema, pero si hacemos otro usuario sin perfil de administrador no podríamos accesar. 

---

### Vista de administrador
Haremos un nuevo layout para que los administradores tengan su propia barra de navegación.
Cómo es muy parecida a la que ya tenemos, duplicaremos ese archivo bajo el nombre de `admin.blade.php` en la misma carpeta
>Resources/views/layouts/admin.blade.php

Ponemos las variable del nombre del usuario 

````php
{{ Auth::user()->name }}
````
Y las rutas necesarias
````php
<a href="{{route('home')}}">Regresar</a>

<a href="{{ route('logout') }}"                            onclick="event.preventDefault();                            document.getElementById('logout-form').submit();">Cerrar sesión</a>

<form id="logout-form" action="{{ route('logout') }}" method="POST" style="display: none;">
@csrf
</form>
````
La de logout la podemos copiar del archivo `app.blade.php`

Para verificar, en la ruta de prueba que habíamos hecho cambiamos el valor de regreso por la vista de nuestra nueva plantilla

````php
Route::get('prueba', function () {
	    return view('layouts.admin');
});
````

---
### Tareas

Ya tenemos la estructura principal de nuestra página y los permisos, ahora pasamos a la sección de tareas, donde los usuarios administradores podrán subir una tarea a la plataforma, y esta será visible para los alumnos. 

Hacemos modelo, controlador y migración

>php artisan make:model Tarea
>php artisan make:controller TareaController -r
>php artisan make:migration create_tareas_table

*El nombre del modelo deberá empezar con mayúscula*

#### Tabla de Tareas en nuestra Base de Datos
Empezaremos definiendo los datos que contendrá nuestra tabla de tareas, para eso, vamos al archivo de migración que acabamos de crear
>Database/migrations/create_tareas_table.php

Y definimos qué columnas llevará nuestra tabla, tomando el cuenta el tipo de dato

```php
$table->string('title');
$table->text('description');
$table->date('date');
$table->time('time');
$table->string('filetype');
$table->string('file')->nullable();
````
Hacemos la migración
>php artisan migrate

#### Controlador de recursos
Este tipo de controlador contiene las operaciones fundamentales que se pueden hacer con una tabla
Lo primero es especificar la ruta en el archivo
>Routes/web.php

Esta ruta estará dentro de nuestro Middleware de administrador puesto que unicamente ellos podrán subir tareas a la plataforma
````php
Route::resource('admintareas', 'TareaController');
````

El primer parámetro corresponde al nombre que nosotros definimos para esa ruta y el segundo corresponde al controlador

#### Tareas index
Creamos una nueva carpeta
> Resources/views/admin/tareas

Y dentro creamos un nuevo archivo `index.blade.php`
Lo primero será heredar nuestra plantilla con la barra de navegación
```php
@extends('layouts.admin')
@section('content')
...
@endsection
````
Y dentro de la sección pegamos el código de nuestro archivo `tareas_index.html`

Dentro del controlador
>App/Http/Controllers/TareaController.php

en la funcion **index** regresamos nuestra vista 
```php
return view('admin.tareas.index');
````
Y en la plantilla `layouts/admin.blade.php`ponemos la ruta correspondiente
```php
<a href="{{ route('admintareas.index') }}">Tareas</a>
````

También la pondremos en `layouts/app.blade.php` para que al dar click en la opción de **'Administrador*** sea la página principal

````php
<a href="{{ route('admintareas.index') }}">Administrador</a>
````

## Jueves

#### Crear Tareas
Hacemos nuestra nueva vista `create.blade.php`
>Resources/views/admin/tareas/create.blade.php

Heredamos la plantilla
```php
@extends('layouts.admin')
@section('content')
...
@endsection
````

Y dentro de la sección pegamos el contenido del archivo `tareas_create`

Dentro del controlador
>App/Http/Controllers/TareaController.php

en la funcion **create** regresamos nuestra vista 
```php
return view('admin.tareas.create');
````

Y en nuestro archivo `tareas/index.blade.php`buscamos el botón de **Agregar Tarea** y le ponemos la ruta
````php
<a href="{{ route('admintareas.create') }}">Agregar Tarea</a>
````

La función crear nos regresa la vista con el formulario donde podremos llenar los datos, pero la función que guardará esos datos es **@store**

#### Guardar Tareas
En la etiqueta `<form>`definimos los siguientes atributos:

- `action="{{route('admintareas.store')}}"`
Para poder usar el formulario hay que definirle una acción, en este caso nos mandará a la funcion **@store** que es la que guardará la tarea. 
- `enctype="multipart/form-data"`
Para que nos permita enviar archivos
- `method="POST"`

Además tenemos que agregar un `@csrf`por cuestiones de seguridad

Ahora el formulario ya sabe que hacer una vez que obtiene los datos, solo falta definir en donde va a guardar cada cosa.

Dentro del controlador
>App/Http/Controllers/TareaController.php
>
en la función **store** :

Creamos una nueva tarea 

```php
$tarea=new Tarea();
````

*Cada que hagamos uso de un modelo, hay que importarlo con `use App\Tarea;` ya que no se encuentran dentro de la misma carpeta*

Vamos a ir por columnas específicando cual de todos los datos que obtuvimos del formulario es el que se va a guardar ahí

```php
$tarea->title=request($key='title');
$tarea->description=request($key = 'description');
$tarea->date=request($key = 'date');
$tarea->time=request($key = 'time');
$tarea->filetype=request($key = 'filetype');
````

*En `$key`pondremos el nombre de nuestro input para que pueda identificarlo, ese nombre está bajo el atributo `name` en nuestr archivo .blade*

Los archivos no se guardan directamente en la base de datos, sino como un archivo en nuestro proyecto, por lo que en la base unicamente se guardará la ruta a ese archivo.

Hacemos una variable donde se guardará el archivo obtenido del formulario
```php
$file=request($key='file');
```
Hacemos otra variable donde se guardará el nombre de ese archivo
```php
$name=$file->getClientOriginalName();
```
Hacemos una nueva carpeta donde se almacenarán nuestros archivos
>/Public/tareas

Y hacemos una variable en donde se guardará esa ruta
```php
$destination = public_path() . '/tareas/';
```
Ahora movemos el archivo tal cual que está guardado en la variable `$file`, la función `move` recibe como primer parámetro el destino y como segundo el nombre.
```php
$file->move($destination, $name);
```
Por último guardaremos en la base de datos el nombre de nuestro archivo
```php
$tarea->file=$name;
```
Guardamos todos los cambios
```php
$tarea->save();
```
Y regresamos al usuario a la vista principal
```php
return redirect('admintareas');
```
Si al momento de crear la tarea dejamos el campo del archivo vacío nos puede mandar un error, por lo que pondremos todo dentro de una condicional if

```php
if ($request->hasFile('file')) {
	$file = $request->file('file');
	$name=$file->getClientOriginalName();
	$destination = public_path() . '/tareas/';
	$file->move($destination, $name);
	$tarea->file=$name;
}
```
### Mostrar tareas en la vista Tareas index

Ya que tenemos guardadas nuestras tareas en la base de datos lo que sigue sería mostrarlas en la página index.
Esto lo haremos pasando los datos desde el controlador hasta la vista

Dentro del controlador
>App/Http/Controllers/TareaController.php
>
en la función **index** :

Guardaremos todos los datos en una variable
```php
$tareas = Tarea::all();
```
Ya que los guardamos, los tenemos que pasar a la vista

````php
return view('admin.tareas.index',['tareas'=>$tareas]);
````

*Como primer parametro es la ruta de la vista y como segundo todas las variables que vayamos a pasar, en este caso nadamas es una y esa contiene todos nuestros datos*

Ya los tenemos disponibles en la vista ahora solo falta mostrarlos. Queremos tener el mismo numero de tarjetas que de tareas sin tener que repetir el código, por lo que meteremos la tarjeta dentro de un ciclo for. 

```php
@foreach($tareas as $tarea)
	<div class="card__tareas azul">
	</div>
@endforeach
```
Los datos los ponemos en donde corresponden por medio de variables que se ponen entre llaves `{{ $tarea->title }}`

```php
<h4>{{$tarea->title}}</h4>
<b>{{$tarea->date}}</b>
<b>{{$tarea->time}}</b>
<p>{{$tarea->description}}</p>
<span>{{$tarea->filetype}}</span>
````

Y para el archivo:
```html
<a target="_blank" href="{{ asset("tareas/$tarea->file") }}">{{ $tarea->file }}</a>
```

#### Editar Tareas

La función **@edit** nos regresará la vista de un formulario donde podremos editar la tarea, este formulario será el mismo que en la función **@create** pero con los datos visibles.

Hacemos nuestra nueva vista `edit.blade.php`

>Resources/views/admin/tareas/edit.blade.php

Y pegamos el mismo código de `create.blade.php`

En  `index.blade.php` le ponemos ruta al botón **'Editar'** junto con el parámetro para que sepa exactamente que tarea editar
```html
<a href="{{ route('admintareas.edit', $tarea->id) }}">Editar</a>
```
En la función **'@edit'**
>App/Http/Controller/TareaController 

Primero buscamos la tarea haciendo uso del parámetro id
```php
$tarea=Tarea::findOrFail($id);
return view('admin.tareas.edit', ['tarea'=>$tarea]);
```
Ahora mostraremos los datos en el formulario para  los podamos editar, cómo son input, podemos hacer uso del atributo "value"

```php
<input value="{{$tarea->titulo}}">
<input value="{{$tarea->date}}"
<input value="{{$tarea->time}}">
```
Para textarea lo pondremos como contenido dentro de las etiquetas
```php
<textarea class="form-control" rows="3" name="description">{{$tarea->description}}
</textarea>
```

Y para el archivo, como es opcional, hay que poner una condicional 

Si el archivo existe, lo mostramos
```php
@if(!$tarea->file==NULL)
<div class="form-group">
<label>Archivo Actual:</label>
<a target="_blank" href="{{ asset("tareas/$tarea->file") }}" class="text-white">{{ $tarea->file }}</a>
</div>
<br>
```
Y además damos la opción de modificarlo:
```php
<div class="form-group">
<label>Modificar Archivo:</label>
<input  type="file" name="file" enctype  >
</div>
```
Si no hay archivo, únicamente damos la opción de subirlo
```php
@else
<div class="form-group">
<label>Subir Archivo:</label>
<input type="file" name="file" enctype  >
</div>
@endif
```

Ya tenemos los datos, solo hay que especificar lo que se hará al momento de dar en Submit 
```php
<form method="POST" action="{{ route('admintareas.update', $tarea->id) }}">
```
Para poder guardar archivos
```php
enctype="multipart/form-data"
```
Y para poder guardar
```php
@method('PATCH')
@csrf
```
Ahora definimos donde guardar los datos modificados. Es muy parecido a la funcion **@store**

En la función **'@update'**
>App/Http/Controller/TareaController 

Buscamos la tarea con la que estamos trabajando
```php      
$tarea=Tarea::findOrFail($id);
```
Y copiamos todo lo de la función **'@store'**
```php
$tarea->title=request($key='title');
$tarea->description=request($key = 'description');
$tarea->date=request($key = 'date');
$tarea->time=request($key = 'time');
$tarea->filetype=request($key = 'filetype');
if ($request->hasFile('file')) {
	$file = $request->file('file');
	$name=$file->getClientOriginalName();
	$destination = public_path() . '/tareas/';
	$file->move($destination, $name);
	$tarea->file=$name;
}
```
Guardamos los cambios hechos y redirigimos a la vista principal
```php
$tarea->update();
return redirect('admintareas');
```
#### Ver tareas por separado
Hacemos nueva vista
> /tareas/show.blade.php

Y pegamos mismo código que `tareas/index`, únicamente borramos todo el contenido de bienvenidos, unicamente borramos el botón 'Ver', 'Agregar Tareas' y el ciclo @foreach

En la función **'@show'**
>App/Http/Controller/TareaController 

```php
$tarea=Tarea::findOrFail($id);
return view('admin.tareas.show', ['tarea'=>$tarea]);
```
Y le damos ruta a nuestro boton ‘Ver’ en `tareas/index`, recordemos que recibe un id también
```php
<a href="{{ route('admintareas.show', $tarea->id) }}">
Ver</a>
```

#### Eliminar tareas

Utilizaremos el botón que está en `tareas/index.blade.php` que es tipo submit, entonces al darle click hará la action que tenga el formulario

```php
<form method="POST" action="{{ route('admintareas.destroy', $tarea->id) }}" >

@csrf
@method('DELETE')
```
En la función **'@destroy'**
>App/Http/Controller/TareaController 

```php
$tareas=Tarea::findOrFail($id);
$tareas->delete();
return redirect('admintareas');
```

### Mostrar tareas a los alumnos en la vista principal

En la función **'@index'**
>App/Http/Controller/HomeController 

Guardamos los datos en una variable y los pasamos a la vista
 ```php
$tareas = Tarea::all();
return view('home',['tareas'=>$tareas]);
```
*Importamos con use App\Tarea;*

Y en nuestra vista `home.blade.php` hacemos un ciclo foreach
```php
@foreach($tareas as $tarea)
    <div class="card__tareas azul">   
    </div>
@endforeach
```
Ponemos las variables donde corresponden 
```php
<h4>{{$tarea->title}}</h4>
<b>{{$tarea->date}}</b>
<b>{{$tarea->time}}</b>
<p>{{$tarea->description}}</p>
<span>{{$tarea->filetype}}</span>
```
Y para el archivo
```php
<a target="_blank" href="{{ asset("tareas/$tarea->file") }}">{{ $tarea->file }}</a>
```

### Entregas

Hacemos modelo, controlador y migración

>php artisan make:model Entrega
>php artisan make:controller EntregaController -r
>php artisan make:migration create_entregas_table

Ahora definimos las columnas que tendrá mi tabla
>Database/migrations/create_entregas_table

#### Tabla
A qué usuario pertenece y que tarea está subiendo
```php
$table->unsignedBigInteger('user_id');
$table->unsignedBigInteger('tarea_id');
```
El archivo que está entregando
```php
$table->string('file')->nullable();
```
La calificación
```php
$table->integer('cal')->default(0)->nullable();
```
Y definimos las que serán llaves foráneas
```php
$table->foreign('user_id')->references('id')->on('users');
$table->foreign('tarea_id')->references('id')->on('tareas');
```
Hacemos la migración
>php artisan migrate

Ahora hay que definir de que manera se van a relacionar los modelos User, Tarea y Entrega por medio de funciones

>App/User.php

```php
public function entregas(){
return $this->hasMany('App\Entrega');
}
```
>App/Entrega.php

```php
public function user(){
return $this->belongsTo('App\User');
}

public function tarea(){
return $this->belongsTo('App\Tarea');
}
```
>App/Tarea.php

```php
public function entregas(){
return $this->hasMany('App\Entrega');
}
```
Hacemos la ruta del controlador en 
>Routes/web.php

```php
Route::resource('entrega', 'EntregaController');
```
## Viernes

#### Vistas 
En la vista `home.blade.php` damos ruta a nuestro botón de subir archivo, el cuál nos llevará a otra vista

```php
<a href="{{ route('entrega.show', $tarea->id) }}">Subir archivo</a>
```
Hacemos vista `show.blade.php` y nueva carpeta
>Resources/views/entregas/show.blade.php

Heredamos nuestra plantilla
```php
@extends('layouts.app')
@section('content')
@endsection
```
Y pegamos código de `entregas.html` dentro de la sección

En nuestro formulario en `show.blade.php` tenemos un input que es invisble para el usuario, lo usaremos de auxiliar para pasar el id de la tarea 
```php
value="{{ $tarea->id }}"
```
Y le ponemos action a nuestro formulario
```php
action="{{ route('entrega.store') }}"
```
sin olvidarnos de estas tres lineas importantes
```php
method=”POST”
enctype="multipart/form-data"
@csrf
````

En la función **'@show'**
>App/Http/Controller/EntregaController 

Identificamos la tarea que está subiendo el usuario y pasamos esa información a la vista
```php
$tarea=Tarea::findOrFail($id);
return view('entregas.show',['tarea'=>$tarea]);
```
*Importamos con use App\Tarea;*

En la función **'@store'**
>App/Http/Controller/EntregaController 

Hacemos un nuevo objeto 'Entrega'
```php
$entrega=new Entrega();
```
Pasamos el valor del id de la tarea y del usuario
```php
$entrega->tarea_id=request($key = 'tarea_id');
$entrega->user_id=Auth::user()->id;
```
Para el archivo es lo mismo que habíamos hecho en la sección de tareas
```php
$file = $request->file('file');
$username=Auth::user()->name;
$name=$username.$file->getClientOriginalName();
```
Hacemos nueva carpeta 
>Public/entregas

```php
$destination = public_path() . '/entregas/';
$file->move($destination, $name);
$entrega->file=$name;
$entrega->save();
return redirect('home');
```
*Importamos use App\Entrega y 
use Illuminate\Support\Facades\Auth;*

### Mostrar tabla de entregas de cada tarea

Las entregas se mostrarán en la vista particular de cada tarea por lo que modificaremos

En la función **'@show'**
>App/Http/Controller/TareaController 

Pasamos los datos de las entregas

```php
$tarea=Tarea::findOrFail($id);
$entregas=Entrega::all();
return view('admin.tareas.show', ['tarea'=>Tarea::findOrFail($id), 'entregas'=>$entregas]);
 ```
*Importamos use App\Entrega;*

En la vista `tareas/show.blade.php` haremos un ciclo foreach para recorrer las entregas y al mismo tiempo las vamos a ir filtrando para unicamente ver las entregas relacionadas con esa tarea

```php
@foreach($entregas as $entrega)
@if($entrega->tarea_id == $tarea->id)
	<tr>
	</tr>
@endif
@endforeach
``` 
Y ponemos los valores correspondientes

```php
<th scope="row">{{ $entrega->user->name }}</th>
<a target="_blank" href="{{ asset("entregas/$entrega->file") }}">{{ $entrega->file }}</a>
````

#### Para subir las calificaciones
Dentro de  `tareas/show.blade` en el formulario ponemos la acción que será la funcion **@update**

```php
<form action="{{ route('entrega.update', $entrega->id) }}" method="POST">
@method('PATCH')
@csrf
````

En la función **'@update'**
>App/Http/Controller/EntregaController 

```php
$entrega=Entrega::findOrFail($id);
entrega->grade=$request->get('grade');
$entrega->update();
return back();
```` 

Para que se muestren las calificaciones ya guardadas
```php
<input type="number" min="0" max="10" name="cal" 
@if(!$entrega->cal==NULL)
value="{{ $entrega->cal }}" 
@else
value="0"
@endif
>
````

### Tabla de calificaciones para el usuario

Hacemos una nueva vista
>Resources/Views/cal.blade.php

Y pegamos el código de nuestro archivo `cal.html`

Hacemos una nueva función en el controlador de Home
>App/Http/Controller/HomeController 
```php
public function cal($id){
	return view('cal');
}
````
Y hacemos la ruta en `web.php` pasandole un parámetro
```php
Route::get('/cal/{id}', 'HomeController@cal');
```
Ese parámetro lo pasaremos desde la plantilla `app.blade.php`
```php
<a href="{{ url('cal/'.Auth::user()->id) }}">Calificaciones</a>
 ```

Y ahora pasamos a la vista los datos necesarios para que nos pueda mostrar las calificaciones

```php
$entregas=Entrega::all();
$user=User::findOrFail($id);
return 
view('cal', ['entregas'=>$entregas, 'user'=>$user]);
````

*Importamos use App\User; y use App\Entrega;*

Ahora solo falta mostrar los datos en las vistas con un foreach

```php
@foreach($entregas as $entrega)
@if($entrega->user_id==$user->id)
@if(!$entrega->cal==NULL)
<tr>
	<th scope="row">{{ $entrega->tarea->title}}</th>
	<td>{{ $entrega->cal}}</td>
</tr>
@endif
@endif
@endforeach
```




