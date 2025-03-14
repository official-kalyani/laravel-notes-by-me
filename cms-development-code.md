 ### Step 3: Create a Model, Migration, and Controller
```
php artisan make:model MenuItem -mcr
This will generate:

app/Models/MenuItem.php
database/migrations/YYYY_MM_DD_create_menu_items_table.php
app/Http/Controllers/MenuItemController.php

```
### Step 4: Define Database Schema
Edit the migration file in database/migrations/:
```
public function up()
{
    Schema::create('menu_items', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->string('type'); // 'page' or 'pdf'
        $table->string('url'); // Page link or PDF file path
        $table->timestamps();
    });
}
```
Run the migration:
```
php artisan migrate
```
### Step 5: Configure the Model
Edit app/Models/MenuItem.php:
```
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class MenuItem extends Model
{
    use HasFactory;
    
    protected $fillable = ['title', 'type', 'url'];
}
```
### Step 6: Implement Controller Logic
Edit app/Http/Controllers/MenuItemController.php:
```
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\MenuItem;

class MenuItemController extends Controller
{
    public function index()
    {
        $menuItems = MenuItem::all();
        return view('menu.index', compact('menuItems'));
    }

    public function create()
    {
        return view('menu.create');
    }

    public function store(Request $request)
    {
        $request->validate([
            'title' => 'required',
            'type' => 'required|in:page,pdf',
            'url' => 'required',
        ]);

        MenuItem::create($request->all());

        return redirect()->route('menu.index')->with('success', 'Menu item added successfully.');
    }

    public function destroy(MenuItem $menuItem)
    {
        $menuItem->delete();
        return redirect()->route('menu.index')->with('success', 'Menu item deleted successfully.');
    }
}
```
# Step 7: Create Views
menu/index.blade.php
```
@extends('layouts.app')

@section('content')
<div class="container">
    <h2>Manage Sidebar Menu</h2>
    <a href="{{ route('menu.create') }}" class="btn btn-primary">Add Menu Item</a>
    
    <table class="table mt-3">
        <tr>
            <th>Title</th>
            <th>Type</th>
            <th>URL</th>
            <th>Actions</th>
        </tr>
        @foreach ($menuItems as $item)
        <tr>
            <td>{{ $item->title }}</td>
            <td>{{ ucfirst($item->type) }}</td>
            <td><a href="{{ $item->url }}" target="_blank">View</a></td>
            <td>
                <form action="{{ route('menu.destroy', $item->id) }}" method="POST">
                    @csrf
                    @method('DELETE')
                    <button type="submit" class="btn btn-danger">Delete</button>
                </form>
            </td>
        </tr>
        @endforeach
    </table>
</div>
@endsection

```
menu/create.blade.php
```
@extends('layouts.app')

@section('content')
<div class="container">
    <h2>Add Menu Item</h2>
    
    <form action="{{ route('menu.store') }}" method="POST">
        @csrf
        <div class="mb-3">
            <label>Title</label>
            <input type="text" name="title" class="form-control" required>
        </div>
        
        <div class="mb-3">
            <label>Type</label>
            <select name="type" class="form-control" required>
                <option value="page">Page</option>
                <option value="pdf">PDF</option>
            </select>
        </div>

        <div class="mb-3">
            <label>URL (Page link or PDF path)</label>
            <input type="text" name="url" class="form-control" required>
        </div>

        <button type="submit" class="btn btn-success">Save</button>
    </form>
</div>
@endsection

```
### Step 8: Define Routes
Edit routes/web.php:

```
use App\Http\Controllers\MenuItemController;

Route::get('/menu', [MenuItemController::class, 'index'])->name('menu.index');
Route::get('/menu/create', [MenuItemController::class, 'create'])->name('menu.create');
Route::post('/menu/store', [MenuItemController::class, 'store'])->name('menu.store');
Route::delete('/menu/{menuItem}', [MenuItemController::class, 'destroy'])->name('menu.destroy');

```

### Step 9: Display Menu in Sidebar
Modify resources/views/layouts/app.blade.php:

```
<div class="sidebar">
    <h3>Sidebar Menu</h3>
    <ul>
        @foreach(App\Models\MenuItem::all() as $item)
        <li>
            <a href="{{ $item->url }}" target="_blank">{{ $item->title }}</a>
        </li>
        @endforeach
    </ul>
</div>

```
### Step 10: Run and Test
Start the server:

```
php artisan serve
```
Visit: http://127.0.0.1:8000/menu to manage menu items.


Additional Features
✅ Add File Upload for PDFs
✅ Implement User Authentication
✅ Create a Dynamic Page Editor for Content Management

Let me know if you need enhancements! 🚀

Enhancements: File Upload for PDFs & Dynamic Page Editor
We'll extend the CMS with:

File Upload for PDFs – Allow admins to upload PDFs instead of manually adding URLs.
Dynamic Page Editor – Provide a rich text editor for creating and managing page content.

## 1. File Upload for PDFs
Step 1: Update Migration
Modify the menu_items table to support file uploads.
```
php artisan make:migration add_pdf_path_to_menu_items --table=menu_items

public function up()
{
    Schema::table('menu_items', function (Blueprint $table) {
        $table->string('pdf_path')->nullable()->after('url');
    });
}

public function down()
{
    Schema::table('menu_items', function (Blueprint $table) {
        $table->dropColumn('pdf_path');
    });
}

php artisan migrate
```

### Step 2: Update Model
Modify app/Models/MenuItem.php:
```
protected $fillable = ['title', 'type', 'url', 'pdf_path'];

```
### Step 3: Modify Form for PDF Upload
Modify menu/create.blade.php:

