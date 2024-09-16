# Step-by-step CRUD Laravel 
## Catatan untuk PTS praktik produktif RPL
### BAB I: Precode
 1. Dalam directory `./laragon/www/`
```bash
composer create-project laravel/laravel example-project
```
2. Buat database dengan nama bebas (contoh: db_crud_madyan)
3. Dalam terminal proyekmu jalankan 
```bash
php artisan migrate
php artisan serve
```
4. Dalam terminal download semua plugin yang perlu (tailwindcss + flowbite) kemudian buat tailwind config file
```bash
npm install -D tailwindcss postcss autoprefixer flowbite
npx tailwindcss init -p
```
5. Di dalam file `tailwind.config.js` masukkan
```javascript
module.exports = {
    content: [
      "./resources/**/*.blade.php",
      "./resources/**/*.js",
      "./resources/**/*.vue",
      "./node_modules/flowbite/**/*.js"
    ],
    theme: {
      extend: {},
    },
    plugins: [
        require('flowbite/plugin')
    ],
  }
```
6. Di dalam file `./resources/css/app.css`
```css
@tailwind base;  
@tailwind components;  
@tailwind utilities;  
```
7. Insert Flowbite (atau plugin lain) dalam `./resources/js/app.js`
```javascript
import('flowbite');
```
8. Di dalam tag <head></head> masukkan app.css dan app.js dengan Vite
```js
@vite(['resources/css/app.css','resources/js/app.js'])
```
9. Dalam terminal proyekmu jalankan 
 ```bash
 npm run dev
 ```

### Bab II: code
1. Membuat **Model, Migration & Controller** bebas
```bash
php artisan make:model exampleModel -mcr
```
2. Setup **Model & Migration**  
Migration:
```php
<?php
use  Illuminate\Database\Migrations\Migration;
use  Illuminate\Database\Schema\Blueprint;
use  Illuminate\Support\Facades\Schema;

return  new  class  extends  Migration
{
	/**
	* Run the migrations.
	*/
	public  function  up():  void
	{
	Schema::create('exampleModel',  function  (Blueprint  $table)  {
		$table->id();
		$table->timestamps();
		$table->string('name');
		$table->text('desc');
		$table->integer('stock');
		$table->date('expired_date');
		$table->string('image')->nullable();
	});
}
/**
* Reverse the migrations.
*/
public  function  down():  void
{
Schema::dropIfExists('exampleModel');
}
};
```
Model:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use App\Factories\StudentFactory;

class student extends Model
{
    use HasFactory;

    protected $fillable = [
        'name',
        'text',
        'stock',
	'expired_date',
        'image'
    ];
}

```
