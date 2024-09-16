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
8. Di dalam tag `<head></head>` masukkan app.css dan app.js dengan Vite
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
3. Setup Controller
```php
<?php

namespace App\Http\Controllers;

use Illuminate\View\View;
use Illuminate\Http\Request;
use Illuminate\Http\RedirectResponse;
use Illuminate\Support\Facades\Storage;
use App\Models\ExampleModel;

class ExampleModelController extends Controller
{
    public function index() {
        // Tampil semua produk dari urutan terbaru
        $title = 'Students';
        $students = Student::latest()->paginate(10);
        return view('student.index', compact('exampleModels', 'title'));
    }

    public function store(Request $request): RedirectResponse{
        # Request : objek yang mengandung semua data
        // yang dikirimkan oleh pengguna melalui form atau URL
    
        // $request : untuk mengakses data yang dikirimkan oleh pengguna
    
        // RedirectResponse : digunakan untuk mengarahkan kembali ke 
        // halaman lain setelah data baru berhasil disimpan
        
        $request->validate([
            'name' => 'required',
            'desc' => 'required',
            'stock'=>  'required|numeric',
            'expired_date'=> 'required',
            'image' => 'required|image|mimes:jpeg,jpg,png,svg|max:2048'
        ]);
        
        $image = $request->file('image'); #turn file into variable
        $hash = $image->hashName();
        $image->storeAs('public/exampleModel', $hash); # Store image

        // Create student
        exampleModel::create([
            'name' => $request->name,
            'desc' => $request->desc,
            'stock' => $request->stock,
            'expired_date' => $request->expired_date,
            'image' => $hash,
        ]);
    
        return redirect()->route('exampleModel.index');
        }

        public function show(string $id): View {
            // Menampilkan data berdasarkan ID
            $student = Student::findOrFail($id);
    
            // Mengirim data produk ke halaman view show
            return view('exampleModel.show', compact('exampleModels'));
        }

        public function update(Request $request, $id): RedirectResponse
    {
        //validate form
        $request->validate([
            'name' => 'required',
            'desc' => 'required',
            'stock'=>  'required|numeric',
            'expired_date'=> 'required',
            'image' => 'required|image|mimes:jpeg,jpg,png,svg|max:2048'
        ]);

        //get student by ID
        $exampleModels = ExampleModel::findOrFail($id);

        //check if image is uploaded
        if ($request->hasFile('image')) {

        Storage::delete('public/exampleModel/'.$student->image);

        $image = $request->file('image'); #turn file into variable
        $hash = $image->hashName();
        $image->storeAs('public/exampleModel', $hash); # Store image

        // Update exampleModel element
        $student->update([
            'name' => $request->name,
            'desc' => ($request->desc),
            'stock' => $request->expired_date,
            'expired_date' => $request->expired_date,
            'image' => $hash,
        ]);

        } else {

            //update student without image
            $student->update([
            'name' => $request->name,
            'desc' => ($request->desc),
            'stock' => ($request->stock),
            'expired_date' => $request->expired_date
            ]);
        }
        return redirect()->route('exampleModel.index');
    }
        public function destroy($id): RedirectResponse{
            $exampleModel = ExampleModel::findOrFail($id);

            Storage::delete('public/exampleModel/' . $exampleModel->image);
            $exampleModel->delete();

            return redirect()->route('exampleModel.index');
        }
}

```
4. Membuat folder bebas (Contoh: .\view\exampleApp\)
5. Membuat view index.blade.php  
Dalam terminal
```bash
php artisan make:view exampleApp\index
```
Dalam `exampleApp\index.blade.php`
```html
<x-layout>
<x-slot></x-slot>
<x-table>
@forelse($exampleModels as  $row)
<tr  class="border-b dark:border-gray-700">
<td  class="px-4 py-3">{{  $row->name }}</td>
<td  class="px-4 py-3">{{  $row->stock }}</td>
<td  class="px-4 py-3">{{  $row->expired_date }}</td>
<td  class="px-4 py-3">
<img  src="  {{  asset('storage/exampleModel/'  .  $row->image)  }}"  alt=""  class="w-20 h-20 object-cover">
</td>
<td  class="px-4 py-3 flex mt-8 gap-3">
<a  href="{{  route('student.index',  $row->id)}}"  class="bg-slate-800 text-white py-2 px-4 rounded-lg hover:opacity-90">Show</a>
<button  data-modal-target="edit-modal{{  $row->id  }}"  data-modal-toggle="edit-modal{{  $row->id  }}"  class="bg-sky-600 text-white py-2 px-4 rounded-lg hover:opacity-90">Edit</button>
{{-- Edit Modal --}}
<div  id="edit-modal{{  $row->id  }}"  tabindex="-1"  aria-hidden="true"  class="hidden overflow-y-auto overflow-x-hidden fixed top-0 right-0 left-0 z-50 justify-center items-center w-full md:inset-0 h-[calc(100%-1rem)] max-h-full">
<div  class="relative p-4 w-full max-w-2xl max-h-full">
<!-- Modal content -->
<x-edit  :content=$row></x-edit>
</div>
</div>
<form  action="{{  route('exampleModel.destroy',  $row->id)  }}"  method="POST"  onsubmit="return  confirm('Are you sure you want to delete exampleModel {{  $row->name  }}?');">
@csrf
@method('DELETE')
<button  type="submit"  class="bg-red-600 text-white py-2 px-4 rounded-lg hover:opacity-90">Delete</button>
</form>
</td>
</tr>
@empty
<div  class="text-red-400 p-2 fw-semibold">exampleModel data failed!</div>
@endforelse
</x-table>
</x-layout>
```
6. Membuat component layout.blade.php  
Dalam terminal
```bash
php artisan make:component layout
```
Dalam `.\resources\views\component\layout.blade.php`  
```html
<!DOCTYPE html>
<html lang="en" class="h-full bg-gray-200">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <link rel="stylesheet" href="https://rsms.me/inter/inter.css">
    <link rel="icon" type="image/x-icon" href="https://tailwindui.com/img/logos/mark.svg?color=indigo&shade=500">
    <script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.x.x/dist/cdn.min.js"></script>
    <title>{{ $title }}</title>
    @vite(['resources/css/app.css','resources/js/app.js'])
</head>
<body>
    
    {{ $slot }}
    
</body>
</html>
```
7. Membuat component table.blade.php
Dalam terminal
```bash
php artisan make:component table
```
Dalam `.\resources\views\component\table.blade.php`
```html
<section  class="bg-gray-50 dark:bg-gray-900 p-3 sm:p-5">
	<h1  class="font-black text-4xl text-center mb-3">exampleModel Data</h1>
	<div  class="mx-auto max-w-screen-xl px-4 lg:px-12">
	<!-- Start coding here -->
		<div  class="bg-white dark:bg-gray-800 relative shadow-md sm:rounded-lg overflow-hidden">
			<div  class="flex flex-col md:flex-row items-center justify-between space-y-3 md:space-y-0 md:space-x-4 p-4">
				<div  class="w-full md:w-1/2">
					<form  class="flex items-center">
					<label  for="simple-search"  class="sr-only">Search</label>
					<div  class="relative w-full">
					<div  class="absolute inset-y-0 left-0 flex items-center pl-3 pointer-events-none">
					<svg  aria-hidden="true"  class="w-5 h-5 text-gray-500 dark:text-gray-400"  fill="currentColor"  viewbox="0 0 20 20"  xmlns="http://www.w3.org/2000/svg">
					<path  fill-rule="evenodd"  d="M8 4a4 4 0 100 8 4 4 0 000-8zM2 8a6 6 0 1110.89 3.476l4.817 4.817a1 1 0 01-1.414 1.414l-4.816-4.816A6 6 0 012 8z"  clip-rule="evenodd"  />
					</svg>
					</div>
					<input  type="text"  id="simple-search"  class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-primary-500 focus:border-primary-500 block w-full pl-10 p-2 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-primary-500 dark:focus:border-primary-500"  placeholder="Search"  required="">
					</div>
					</form>
				</div>
			<div  class="w-full md:w-auto flex flex-col md:flex-row space-y-2 md:space-y-0 items-stretch md:items-center justify-end md:space-x-3 flex-shrink-0">

			<!-- Modal toggle -->
			<button  data-modal-target="default-modal"  data-modal-toggle="default-modal"  class="flex items-center justify-center text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 text-center dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800"  type="button">
			<svg  class="h-3.5 w-3.5 mr-2"  fill="currentColor"  viewbox="0 0 20 20"  xmlns="http://www.w3.org/2000/svg"  aria-hidden="true">
			<path  clip-rule="evenodd"  fill-rule="evenodd"  d="M10 3a1 1 0 011 1v5h5a1 1 0 110 2h-5v5a1 1 0 11-2 0v-5H4a1 1 0 110-2h5V4a1 1 0 011-1z"  />
			</svg>
			Add exampleModel
			</button>
			</div>
			</div>
			<div  class="overflow-x-auto">
			<table  class="w-full text-sm text-left text-gray-500 dark:text-gray-400">
			<thead  class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400">
			<tr>
				<th  scope="col"  class="px-4 py-3">Name</th>
				<th  scope="col"  class="px-4 py-3">Desc</th>
				<th  scope="col"  class="px-4 py-3">Stock</th>
				<th  scope="col"  class="px-4 py-3">Expired Date</th>
				<th  scope="col"  class="px-4 py-3">Image</th>
				<th  scope="col"  class="px-4 py-3 w-1">Actions</th>
			</tr>
			</thead>
			<tbody>
			{{  $slot }}
			</tbody>
			</table>
			<!-- Modal Create -->
				<div  id="default-modal"  tabindex="-1"  aria-hidden="true"  class="hidden overflow-y-auto overflow-x-hidden fixed top-0 right-0 left-0 z-50 justify-center items-center w-full md:inset-0 h-[calc(100%-1rem)] max-h-full">
				<div  class="relative p-4 w-full max-w-2xl max-h-full">
			<!-- Modal content -->
			<x-create></x-create>
			</div>
			</div>
			</div>
		</div>
	</div>
</section>
```
