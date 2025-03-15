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

```
<form action="{{ route('menu.store') }}" method="POST" enctype="multipart/form-data">
    @csrf
    <div class="mb-3">
        <label>Title</label>
        <input type="text" name="title" class="form-control" required>
    </div>
    
    <div class="mb-3">
        <label>Type</label>
        <select name="type" id="menuType" class="form-control" required>
            <option value="page">Page</option>
            <option value="pdf">PDF</option>
        </select>
    </div>

    <div class="mb-3" id="urlField">
        <label>URL (For Page)</label>
        <input type="text" name="url" class="form-control">
    </div>

    <div class="mb-3" id="pdfField" style="display: none;">
        <label>Upload PDF</label>
        <input type="file" name="pdf_file" class="form-control">
    </div>

    <button type="submit" class="btn btn-success">Save</button>
</form>

<script>
    document.getElementById("menuType").addEventListener("change", function () {
        if (this.value === "pdf") {
            document.getElementById("pdfField").style.display = "block";
            document.getElementById("urlField").style.display = "none";
        } else {
            document.getElementById("pdfField").style.display = "none";
            document.getElementById("urlField").style.display = "block";
        }
    });
</script>

```

### Step 4: Modify Controller

Modify MenuItemController.php:

```
use Illuminate\Support\Facades\Storage;

public function store(Request $request)
{
    $request->validate([
        'title' => 'required',
        'type' => 'required|in:page,pdf',
        'url' => 'nullable|required_if:type,page',
        'pdf_file' => 'nullable|required_if:type,pdf|mimes:pdf|max:2048',
    ]);

    $pdfPath = null;
    if ($request->hasFile('pdf_file')) {
        $pdfPath = $request->file('pdf_file')->store('pdfs', 'public');
    }

    MenuItem::create([
        'title' => $request->title,
        'type' => $request->type,
        'url' => $request->type == 'page' ? $request->url : null,
        'pdf_path' => $pdfPath,
    ]);

    return redirect()->route('menu.index')->with('success', 'Menu item added successfully.');
}


```

### Step 5: Update Sidebar to Show PDFs

Modify layouts/app.blade.php:

```
@foreach(App\Models\MenuItem::all() as $item)
<li>
    @if($item->type == 'page')
        <a href="{{ $item->url }}" target="_blank">{{ $item->title }}</a>
    @else
        <a href="{{ asset('storage/' . $item->pdf_path) }}" target="_blank">{{ $item->title }} (PDF)</a>
    @endif
</li>
@endforeach

```
### 2. Dynamic Page Editor for Content Management
Now, let’s allow admins to create dynamic pages.

Step 1: Create a Page Model, Migration, and Controller
```
php artisan make:model Page -mcr
```
```
Modify database/migrations/YYYY_MM_DD_create_pages_table.php:
public function up()
{
    Schema::create('pages', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->text('content');
        $table->timestamps();
    });
}

php artisan migrate
```

### Step 2: Update the Model
Modify app/Models/Page.php:

```
protected $fillable = ['title', 'content'];

```
### Step 3: Implement Controller Logic
Modify app/Http/Controllers/PageController.php:

``` 
use App\Models\Page;
use Illuminate\Http\Request;

public function index()
{
    $pages = Page::all();
    return view('pages.index', compact('pages'));
}

public function create()
{
    return view('pages.create');
}

public function store(Request $request)
{
    $request->validate([
        'title' => 'required',
        'content' => 'required',
    ]);

    Page::create($request->all());

    return redirect()->route('pages.index')->with('success', 'Page created successfully.');
}

public function show(Page $page)
{
    return view('pages.show', compact('page'));
}

```
### Step 4: Install & Configure Rich Text Editor

```
composer require unisharp/laravel-ckeditor

Modify menu/create.blade.php and pages/create.blade.php:

<script src="https://cdn.ckeditor.com/4.16.2/standard/ckeditor.js"></script>
<script>
    CKEDITOR.replace('content');
</script>

Modify pages/create.blade.php:

<form action="{{ route('pages.store') }}" method="POST">
    @csrf
    <div class="mb-3">
        <label>Title</label>
        <input type="text" name="title" class="form-control" required>
    </div>

    <div class="mb-3">
        <label>Content</label>
        <textarea name="content" id="content" class="form-control" required></textarea>
    </div>

    <button type="submit" class="btn btn-success">Save</button>
</form>
```
### Step 5: Define Routes
Modify routes/web.php:
```
use App\Http\Controllers\PageController;

Route::get('/pages', [PageController::class, 'index'])->name('pages.index');
Route::get('/pages/create', [PageController::class, 'create'])->name('pages.create');
Route::post('/pages/store', [PageController::class, 'store'])->name('pages.store');
Route::get('/pages/{page}', [PageController::class, 'show'])->name('pages.show');

Step 6: Link Pages to Sidebar
Modify layouts/app.blade.php:

@foreach(App\Models\Page::all() as $page)
<li>
    <a href="{{ route('pages.show', $page->id) }}">{{ $page->title }}</a>
</li>
@endforeach
```
Final Steps
Run php artisan serve
Visit /menu to manage menu items
Visit /pages to create and view pages

This section contains:

Title: "NAAC"
Two Links: "IIQA" and "SSR" with icons
Video Embed: A YouTube video
To manage this section dynamically in your Laravel CMS, follow these steps:
### 1. Create a Table for Storing Section Content

```
php artisan make:migration create_naac_section_table --create=naac_section
Modify the migration file:
public function up()
{
    Schema::create('naac_section', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->string('iiqa_link')->nullable();
        $table->string('ssr_link')->nullable();
        $table->text('video_embed')->nullable();
        $table->timestamps();
    });
}

php artisan migrate
```
### 2. Create a Model
Run:

```
php artisan make:model NaacSection

Modify app/Models/NaacSection.php:

```
protected $fillable = ['title', 'iiqa_link', 'ssr_link', 'video_embed'];
```

3. Create a Controller

php artisan make:controller NaacSectionController
app/Http/Controllers/NaacSectionController.php:

use App\Models\NaacSection;
use Illuminate\Http\Request;

public function edit()
{
    $naac = NaacSection::first();
    return view('admin.naac.edit', compact('naac'));
}

public function update(Request $request)
{
    $request->validate([
        'title' => 'required',
        'iiqa_link' => 'nullable|url',
        'ssr_link' => 'nullable|url',
        'video_embed' => 'nullable',
    ]);

    $naac = NaacSection::first();
    if (!$naac) {
        $naac = new NaacSection();
    }
    $naac->update($request->all());

    return redirect()->back()->with('success', 'NAAC section updated successfully.');
}

```

### 4. Create an Admin Form
Create resources/views/admin/naac/edit.blade.php:


