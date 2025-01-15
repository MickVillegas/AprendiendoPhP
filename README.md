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
























## DUDAS A RESPONDER, CODIGO SIGUIENTE

DATABASESEEDER  

```
<?php

namespace Database\Seeders;

use App\Models\User;
// use Illuminate\Database\Console\Seeds\WithoutModelEvents;
use Illuminate\Database\Seeder;
use App\Models\Category;
use App\Models\Comment;
use App\Models\Question;
use App\Models\Tag;

class DatabaseSeeder extends Seeder
{
    /**
     * Seed the application's database.
     */
    public function run(): void
    {
        //User::factory(10)->create();
        //Category::factory(5)->create();
        //Question::factory(40)->create();
        //Comment::factory(80)->create();
        //Tag::factory(18)->create();
        // User::factory(10)->create();

        Tag::factory(6)->create();
        Category::factory(5)->has(Question::factory(10)->hasComments(8))->create();

        $etiquetas = Tag::all();
        Question::all()->each(function ($pregunta) use ($etiquetas){
            $etiquetaGenerada = $etiquetas->random(rand(1,6))->pluck('id')->toArray();
            $pregunta->tags()->attach($etiquetaGenerada);
        });
    }
}
```
```
Tag::factory(6)->create();
```
Este código utiliza el método factory para generar 6 registros en la tabla tags usando el modelo Tag. Esto supone que has definido una fábrica de datos para el modelo Tag, que especifica cómo se deben generar los datos ficticios
```
Category::factory(5)->has(Question::factory(10)->hasComments(8))->create();
```
Category::factory(5)->create(): Genera 5 registros en la tabla categories.
.has(Question::factory(10)): Por cada categoría generada, se crearán 10 preguntas asociadas en la tabla questions. Esto establece una relación "uno a muchos" entre categorías y preguntas.
.hasComments(8): Por cada pregunta generada, se crearán 8 comentarios asociados en la tabla comments. Esto establece una relación "uno a muchos" entre preguntas y comentarios.
En resumen, esta línea crea:

5 categorías.
Cada categoría tendrá 10 preguntas.
Cada pregunta tendrá 8 comentarios.
```
$etiquetas = Tag::all();
```
Recupera todas las etiquetas generadas anteriormente (6 etiquetas).
```
Question::all()->each(function ($pregunta) use ($etiquetas) {
```
Recupera todas las preguntas generadas y para cada pregunta realiza una operación. Al hacer un question:all() generas un array con las preguntas hechas., each es un for each, en for each se le puede pasar una funcion donde function(itemActual) será una posicioion del array. En este caso pregunta puede ser ese indice.
```
$etiquetaGenerada = $etiquetas->random(rand(1, 6))->pluck('id')->toArray();
```
$etiquetas->random(rand(1,6)): Selecciona aleatoriamente entre 1 y 6 etiquetas de las disponibles.
pluck('id'): Extrae únicamente los IDs de las etiquetas seleccionadas.
toArray(): Convierte el conjunto de IDs en un arreglo para que pueda ser procesado fácilmente.
```
$pregunta->tags()->attach($etiquetaGenerada);
```
Asocia las etiquetas seleccionadas con la pregunta actual. Esto utiliza la relación "muchos a muchos" (tags() en el modelo Question) para insertar datos en la tabla intermedia que relaciona preguntas y etiquetas.


MODELOS 
CATEGORY.PHP

```
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Category extends Model
{
    /** @use HasFactory<\Database\Factories\CategoryFactory> */
    use HasFactory;
    public function questions(){
        return $this->hasMany(Question::class);
    }
}
```


QUESTION.PHP

```
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Question extends Model
{
    /** @use HasFactory<\Database\Factories\QuestionFactory> */
    use HasFactory;
    public function comments(){
        return $this->hasMany(Comment::class);
    }

    public function tags(){
        return $this->belongsToMany(Tag::class);
    }

    public function category(){
        return $this->belongsTo(category::class);
    }

    public function users(){
        return $this->belongsTo(User::class);
    }
}

```








