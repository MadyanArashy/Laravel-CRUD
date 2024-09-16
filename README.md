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
5. Membuat view index
```php
```php
<x-layout>
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

```
