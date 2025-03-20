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
```
@extends('layouts.admin')

@section('content')
<h2>Edit NAAC Section</h2>
<form action="{{ route('naac.update') }}" method="POST">
    @csrf
    @method('PUT')

    <div class="mb-3">
        <label>Title</label>
        <input type="text" name="title" value="{{ $naac->title ?? '' }}" class="form-control" required>
    </div>

    <div class="mb-3">
        <label>IIQA Link</label>
        <input type="url" name="iiqa_link" value="{{ $naac->iiqa_link ?? '' }}" class="form-control">
    </div>

    <div class="mb-3">
        <label>SSR Link</label>
        <input type="url" name="ssr_link" value="{{ $naac->ssr_link ?? '' }}" class="form-control">
    </div>

    <div class="mb-3">
        <label>Video Embed Code</label>
        <textarea name="video_embed" class="form-control">{{ $naac->video_embed ?? '' }}</textarea>
    </div>

    <button type="submit" class="btn btn-success">Update</button>
</form>
@endsection

```
### 5. Define Routes
Modify routes/web.php:

```
use App\Http\Controllers\NaacSectionController;

Route::get('/admin/naac/edit', [NaacSectionController::class, 'edit'])->name('naac.edit');
Route::put('/admin/naac/update', [NaacSectionController::class, 'update'])->name('naac.update');
```
### 6. Display the Section on the Frontend
Modify the frontend view resources/views/naac.blade.php:

```
@php
$naac = App\Models\NaacSection::first();
@endphp

@if($naac)
    <h2 class="text-center">{{ $naac->title }}</h2>
    <div class="text-center">
        <a href="{{ $naac->iiqa_link }}" class="btn btn-primary">IIQA</a>
        <a href="{{ $naac->ssr_link }}" class="btn btn-primary">SSR</a>
    </div>

    @if($naac->video_embed)
        <div class="text-center mt-3">
            {!! $naac->video_embed !!}
        </div>
    @endif
@endif
```

Final Steps
Run php artisan serve
Visit /admin/naac/edit to update the section
The frontend will dynamically update the content

If you want to manage multiple dynamic pages like NAAC (e.g., IQAC, Research, Admissions, etc.), you need a CMS-like system in your Laravel backend. Here’s how you can handle it:

1. Create a Table for Dynamic Pages
```
php artisan make:migration create_pages_table --create=pages
public function up()
{
    Schema::create('pages', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->string('slug')->unique(); // For URL handling
        $table->text('content')->nullable(); // Main content
        $table->string('image')->nullable(); // Image URL (e.g., Director's image)
        $table->string('email')->nullable();
        $table->string('phone')->nullable();
        $table->timestamps();
    });
}
php artisan migrate

```

### 2. Create a Model

```
php artisan make:model Page
protected $fillable = ['title', 'slug', 'content', 'image', 'email', 'phone'];

```
### 3. Create a Controller

```
php artisan make:controller PageController
use App\Models\Page;
use Illuminate\Http\Request;

class PageController extends Controller
{
    public function index()
    {
        $pages = Page::all();
        return view('admin.pages.index', compact('pages'));
    }

    public function create()
    {
        return view('admin.pages.create');
    }

    public function store(Request $request)
    {
        $request->validate([
            'title' => 'required',
            'slug' => 'required|unique:pages',
            'content' => 'required',
            'image' => 'nullable|image|mimes:jpg,jpeg,png',
            'email' => 'nullable|email',
            'phone' => 'nullable'
        ]);

        $imagePath = null;
        if ($request->hasFile('image')) {
            $imagePath = $request->file('image')->store('uploads', 'public');
        }

        Page::create([
            'title' => $request->title,
            'slug' => $request->slug,
            'content' => $request->content,
            'image' => $imagePath,
            'email' => $request->email,
            'phone' => $request->phone
        ]);

        return redirect()->route('pages.index')->with('success', 'Page created successfully.');
    }

    public function edit($id)
    {
        $page = Page::findOrFail($id);
        return view('admin.pages.edit', compact('page'));
    }

    public function update(Request $request, $id)
    {
        $page = Page::findOrFail($id);

        $request->validate([
            'title' => 'required',
            'content' => 'required',
            'image' => 'nullable|image|mimes:jpg,jpeg,png',
            'email' => 'nullable|email',
            'phone' => 'nullable'
        ]);

        if ($request->hasFile('image')) {
            $imagePath = $request->file('image')->store('uploads', 'public');
            $page->image = $imagePath;
        }

        $page->update($request->except('image'));

        return redirect()->route('pages.index')->with('success', 'Page updated successfully.');
    }

    public function show($slug)
    {
        $page = Page::where('slug', $slug)->firstOrFail();
        return view('pages.show', compact('page'));
    }
}
```
### 4. Define Routes
Modify routes/web.php:

```
use App\Http\Controllers\PageController;

Route::prefix('admin')->group(function () {
    Route::get('/pages', [PageController::class, 'index'])->name('pages.index');
    Route::get('/pages/create', [PageController::class, 'create'])->name('pages.create');
    Route::post('/pages/store', [PageController::class, 'store'])->name('pages.store');
    Route::get('/pages/edit/{id}', [PageController::class, 'edit'])->name('pages.edit');
    Route::put('/pages/update/{id}', [PageController::class, 'update'])->name('pages.update');
});

Route::get('/pages/{slug}', [PageController::class, 'show'])->name('pages.show');
```

### 5. Create an Admin Panel for Pages
Admin Page List (resources/views/admin/pages/index.blade.php)

```
@extends('layouts.admin')

@section('content')
<h2>Manage Pages</h2>
<a href="{{ route('pages.create') }}" class="btn btn-success">Add New Page</a>

<table class="table">
    <thead>
        <tr>
            <th>Title</th>
            <th>Slug</th>
            <th>Actions</th>
        </tr>
    </thead>
    <tbody>
        @foreach($pages as $page)
        <tr>
            <td>{{ $page->title }}</td>
            <td>{{ $page->slug }}</td>
            <td>
                <a href="{{ route('pages.edit', $page->id) }}" class="btn btn-primary">Edit</a>
                <a href="{{ route('pages.show', $page->slug) }}" class="btn btn-info">View</a>
            </td>
        </tr>
        @endforeach
    </tbody>
</table>
@endsection


Create New Page (resources/views/admin/pages/create.blade.php)

@extends('layouts.admin')

@section('content')
<h2>Create Page</h2>
<form action="{{ route('pages.store') }}" method="POST" enctype="multipart/form-data">
    @csrf

    <div class="mb-3">
        <label>Title</label>
        <input type="text" name="title" class="form-control" required>
    </div>

    <div class="mb-3">
        <label>Slug (Unique URL Identifier)</label>
        <input type="text" name="slug" class="form-control" required>
    </div>

    <div class="mb-3">
        <label>Content</label>
        <textarea name="content" class="form-control"></textarea>
    </div>

    <div class="mb-3">
        <label>Image (Optional)</label>
        <input type="file" name="image" class="form-control">
    </div>

    <div class="mb-3">
        <label>Email (Optional)</label>
        <input type="email" name="email" class="form-control">
    </div>

    <div class="mb-3">
        <label>Phone (Optional)</label>
        <input type="text" name="phone" class="form-control">
    </div>

    <button type="submit" class="btn btn-success">Save</button>
</form>
@endsection


6. Display the Dynamic Page on Frontend
Create resources/views/pages/show.blade.php:

@extends('layouts.app')

@section('content')
<div class="container">
    <h2>{{ $page->title }}</h2>
    <div>{!! $page->content !!}</div>

    @if($page->image)
        <img src="{{ asset('storage/' . $page->image) }}" alt="Page Image" class="img-fluid">
    @endif

    @if($page->email)
        <p>Email: <a href="mailto:{{ $page->email }}">{{ $page->email }}</a></p>
    @endif

    @if($page->phone)
        <p>Phone: {{ $page->phone }}</p>
    @endif
</div>
@endsection

```

### Final Steps
Run Laravel Server:
php artisan serve
Go to Admin Panel (/admin/pages) and create a new page (e.g., IQAC).
View Dynamic Pages via /pages/iqac or /pages/naac.
Now, any new page can be added and managed dynamically without coding! 🚀 Let me know if you need modifications. 🎯

Yes! If you want to call the sidebar section inside the content area, you can modify the textarea to use a rich text editor like CKEditor or TinyMCE, which allows adding dynamic components like menus.

Two Ways to Include Sidebar in Content:
Hardcode Sidebar Inside the Page Template (Recommended)
Use Shortcodes in Content for Sidebar (Dynamic)

### 1. Hardcode Sidebar Inside the Page Template (Recommended)
Modify resources/views/pages/show.blade.php to include the sidebar directly:
```
@extends('layouts.app')

@section('content')
<div class="container">
    <div class="row">
        <!-- Sidebar Section -->
        <div class="col-md-3">
            @include('partials.sidebar')  {{-- Include sidebar here --}}
        </div>

        <!-- Main Content -->
        <div class="col-md-9">
            <h2>{{ $page->title }}</h2>
            <div>{!! $page->content !!}</div> {{-- Display content from DB --}}
        </div>
    </div>
</div>
@endsection

 Now, every dynamic page will automatically include the sidebar.
 ```
 ### 2. Use Shortcodes in Content for Sidebar (Dynamic)

 If you want to allow the admin to dynamically add the sidebar from the content editor, you can use shortcodes like [sidebar].

Step 1: Modify PageController
Edit show() in app/Http/Controllers/PageController.php:
```
public function show($slug)
{
    $page = Page::where('slug', $slug)->firstOrFail();

    // Replace [sidebar] shortcode with actual sidebar view
    $page->content = str_replace('[sidebar]', view('partials.sidebar')->render(), $page->content);

    return view('pages.show', compact('page'));
}
```
Now, if the admin adds [sidebar] inside the content editor, the sidebar will appear there dynamically.

### Final Step: Use CKEditor for Rich Editing
Modify resources/views/admin/pages/create.blade.php:

```
<div class="mb-3">
    <label>Content</label>
    <textarea id="editor" name="content" class="form-control"></textarea>
</div>
Then add CKEditor in the footer:

<script src="https://cdn.ckeditor.com/4.16.0/standard/ckeditor.js"></script>
<script>
    CKEDITOR.replace('editor');
</script>
```
 Now, the admin can easily insert [sidebar] dynamically inside the content! 🚀

 Final Summary:
✔️ If you always want a sidebar → Hardcode it in the page template.
✔️ If you want the admin to insert the sidebar dynamically → Use [sidebar] shortcode.

Would you like to customize the sidebar further? 😊

### To allow the admin to dynamically insert the sidebar using a [sidebar] shortcode, follow these steps:

Step 1: Modify PageController to Replace [sidebar]
Edit app/Http/Controllers/PageController.php:

```
public function show($slug)
{
    $page = Page::where('slug', $slug)->firstOrFail();

    // Replace [sidebar] shortcode with actual sidebar view
    $sidebarHtml = view('partials.sidebar')->render();
    $page->content = str_replace('[sidebar]', $sidebarHtml, $page->content);

    return view('pages.show', compact('page'));
}

```
Now, whenever [sidebar] appears in the database content, it will be replaced with the sidebar view dynamically.

Step 2: Allow Admin to Insert [sidebar] in Content
In your admin panel where the admin adds/edit pages:

Modify resources/views/admin/pages/create.blade.php and edit.blade.php:

```
<div class="mb-3">
    <label>Content</label>
    <textarea id="editor" name="content" class="form-control">{{ old('content', $page->content ?? '') }}</textarea>
</div>

```
Then, add CKEditor for a rich text editor:
```
<script src="https://cdn.ckeditor.com/4.16.0/standard/ckeditor.js"></script>
<script>
    CKEDITOR.replace('editor');
</script>

```
Now, the admin can insert [sidebar] anywhere in the content.

Step 3: Create the Sidebar Partial
Create a sidebar file: resources/views/partials/sidebar.blade.php
```
<div class="sidebar">
    <ul class="list-group">
        <li class="list-group-item"><a href="#">Link 1</a></li>
        <li class="list-group-item"><a href="#">Link 2</a></li>
        <li class="list-group-item"><a href="#">Link 3</a></li>
    </ul>
</div>
This is the sidebar that will replace [sidebar] in content.

```
Step 4: Display the Page with Sidebar
Modify resources/views/pages/show.blade.php:
```
@extends('layouts.app')

@section('content')
<div class="container">
    <h2>{{ $page->title }}</h2>
    <div>{!! $page->content !!}</div> {{-- Content with sidebar replaced --}}
</div>
@endsection
```
Now, whenever the admin adds [sidebar] in the content, the sidebar will be displayed dynamically! 🎉

Final Summary
✔ Admin can insert [sidebar] in the content using CKEditor.
✔ Laravel will replace [sidebar] with the actual sidebar HTML dynamically.
✔ Pages will show the sidebar inside the content at the exact position where [sidebar] was inserted.

Now, you can dynamically manage sidebars inside any page without hardcoding! 🚀 Let me know if you need modifications. 😊




