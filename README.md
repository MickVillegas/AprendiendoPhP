Esto lo separaremos por:

- la vista (la web)
- las rutas (que nos llevará a una url)
- el controlador (que contienen los metodos)

## Crear un cotrolador
Comando
```
php artisan make:controller PageController 
```
Creará automaticamente una clase llamada PageController

### Metodos de controladores
Metodo del controlador de categorias:
```
public function handleCategory(string $nombreCategoria){    
        return view("category", compact("nombreCategoria")); 
    }
```
El string por parametro será concatenado en la ruta, esta variable lo tiene que devolver con un return    
El return devuelve category el cual es una referencia a la vista php (un archivo que
llamaremos category.blade.php)


## Ruta categoria
Primero debemos que importar la clase donde se encuentra el controlador
```
use App\Http\Controllers\PageController;
```
Ejemplo de ruta al controlador con el metodo handleCategory (creado anteriormente):  
```
Route::get('/categoria/{nombreCategoria}', [PageController::class, "handleCategory"]);
```
Llamamos al controlador, le ponemos un nombre/{la variable del controlador que tambien es su parametro}[nombre de la clase del controlador, metodo que devuelve la vista]


## Vista
Se divide en dos partes

- por un lado la app donde se mostrarán los datos

```
<html lang="en">
<head>
    <title>@yield('title')</title>              
</head>                                       
<body>
    <div>
        <h2>@yield('title')</h2>
        <p>@yield('description')</p>
    </div>
    <div>
    @yield('content')
    </div>
</body>
</html>
```

@yield() Es una variable que pintará la pagina si le metemos un dato adentro, la variable ira entre comillas dentro de los para metros de yield:

```
<title>@yield('title')</title>  
```

- ahora aprovecharemos el mismo archivo para variar su contenido segun la ruta que visitemos, por ejemplo en el caso de categoria seria:

```
@extends('app')                 //para que funcione tiene que eredar de app que es donde esta los yield que alteraremos
@section('title', 'Categoria: ' .$nombreCategoria)          
@section('description')                                 
Esta es la categoria                                  
@endsection
@section('content')
Home
@endsection                                                 

```
Aqui hay un ejemplo de instanciacion del yield:  
```
@section('title', 'Categoria: ' .$nombreCategoria)  
```
section es la seccion que elegimos donde guardaremos el dato, en este caso hemos elegido el yield de la seccion titulo, se usa asi:  
- se llama a la seccion mediante el nombre del yield entre comillas
- una coma
- entre comillas lo que va a pintar si es una dato escrito a mano
- en este caso concatenamos $nombreCategoria que es el parametro que usamos en el metodo del controlador categoria  

El archivo se llama category.blade.php

De esta forma si busco /categoria/gatitos En titulo deveria aparecerme "categoria gatitos" puesto que en la ruta le estoy pasando la variable entre parametros que uso en el metodo del controlador y que despues pasamos al yield de la app

## Modelos

En php podemos crear los modelos con: 
```
php artisan make:model Category -fm 
```
Al crear modelo es importante que sea en ingles, singular y mayuscula  

Esto crea migraciones con la base de datos, factorys y el modelo
las migraciones y factorias está en la carpeta migrations y factories  

En migrations hacemos las migraciones con la base de datos donde creamos las tablas a la base de datos por ejemplo, en categorias:  

- La mayoria del trabajo ya está auto completado, lo importante y donde trabajamos es en donde pone schema:  

```
    public function up(): void
    {
        Schema::create('categories', function (Blueprint $table) {
            $table->id();                                                   
            $table->timestamps();
            $table->string('name');                                         
            $table->string('slug')->unique();
        });
    }
```

Las dos primeras columnas estan auto generadas
Creamos en la tablas de la siguiente manera  

```
$table->string('name');    
```

Estamos creando una tabla llamada name que guarda cadenas de caracteres  

Una vez hecho eso haremos migraciones a la base de datos, usamos el comando:  
```
php artisan migrate 
```
En factories creamos datos falsos, esto está hecho para testear si funciona bien la base de datos, crea datos en las tablas hechas en las migraciones, nos centramos en definition, segimos con categorias:

    public function definition(): array
    {
        return [
            'name' => $nombre = fake()->unquite()->word(),
            'slug' => Str::slug($nombre)
        ];
    }

Aquí estoy creando un nombre y un slug falso

En seeders/databaseSedders necesitamos hacer llamadas a nuestros factorys de los modelos que hemos creado donde el modelo que importamos es una clase   

```
use App\Models\Category;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        Category::factory(5)->create();
    }
}
```
Aqui en Category::factory(5)->create(); le estamos diciendo que nos genere 5 categorias en nuestra base de datos en la tabla categoria

Para ejecutar esto usamos el comando:  
```
php artisan migrate:fresh --seed
```








































