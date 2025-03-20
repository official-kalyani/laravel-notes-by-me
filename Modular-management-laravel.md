### Step 1: Create FormSubmission Module
php artisan module:make FormSubmission

### Step 2: Create a Migration for Form Data
php artisan module:make-migration create_form_submissions_table --module=FormSubmission
Then, open the migration file located at:
Modules/FormSubmission/Database/Migrations/xxxx_xx_xx_xxxxxx_create_form_submissions_table.php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up() {
        Schema::create('form_submissions', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email');
            $table->text('message');
            $table->timestamps();
        });
    }

    public function down() {
        Schema::dropIfExists('form_submissions');
    }
};
Run the migration:
php artisan module:migrate FormSubmission

###  Step 3: Create a Model for Form Submissions

php artisan module:make-model FormSubmission --module=FormSubmission

Open Modules/FormSubmission/Entities/FormSubmission.php and define the model:

namespace Modules\FormSubmission\Entities;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class FormSubmission extends Model {
    use HasFactory;
    
    protected $fillable = ['name', 'email', 'message'];
}

### Step 4: Create a Controller

php artisan module:make-controller FormSubmissionController --module=FormSubmission

Modify Modules/FormSubmission/Http/Controllers/FormSubmissionController.php:
```
namespace Modules\FormSubmission\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Routing\Controller;
use Modules\FormSubmission\Entities\FormSubmission;

class FormSubmissionController extends Controller {

    // Show the form
    public function create() {
        return view('formsubmission::create');
    }

    // Store form data
    public function store(Request $request) {
        $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email',
            'message' => 'required'
        ]);

        FormSubmission::create($request->all());

        return redirect()->route('formsubmission.index')->with('success', 'Form submitted successfully!');
    }

    // Display submitted forms
    public function index() {
        $submissions = FormSubmission::latest()->get();
        return view('formsubmission::index', compact('submissions'));
    }
}
```
### Step 5: Define Routes
Open Modules/FormSubmission/Routes/web.php and add:

```
use Modules\FormSubmission\Http\Controllers\FormSubmissionController;

Route::prefix('formsubmission')->group(function () {
    Route::get('/', [FormSubmissionController::class, 'index'])->name('formsubmission.index');
    Route::get('/form', [FormSubmissionController::class, 'create'])->name('formsubmission.create');
    Route::post('/form', [FormSubmissionController::class, 'store'])->name('formsubmission.store');
});

```

### Step 6: Create Views

Create Modules/FormSubmission/Resources/views/create.blade.php for the form:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Submit Form</title>
</head>
<body>
    <h1>Submit Form</h1>
    <form action="{{ route('formsubmission.store') }}" method="POST">
        @csrf
        <label>Name:</label>
        <input type="text" name="name" required><br><br>
        <label>Email:</label>
        <input type="email" name="email" required><br><br>
        <label>Message:</label>
        <textarea name="message" required></textarea><br><br>
        <button type="submit">Submit</button>
    </form>
</body>
</html>

```

Create Modules/FormSubmission/Resources/views/index.blade.php to display submissions:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Submitted Forms</title>
</head>
<body>
    <h1>Submitted Forms</h1>
    <table border="1">
        <tr>
            <th>Name</th>
            <th>Email</th>
            <th>Message</th>
            <th>Submitted At</th>
        </tr>
        @foreach($submissions as $submission)
            <tr>
                <td>{{ $submission->name }}</td>
                <td>{{ $submission->email }}</td>
                <td>{{ $submission->message }}</td>
                <td>{{ $submission->created_at }}</td>
            </tr>
        @endforeach
    </table>
</body>
</html>

```

### Step 7: Test the Module

After uploading and activating the module, visit:

1️⃣ Form Page:
http://127.0.0.1:8000/formsubmission/form

2️⃣ Submissions Page:
http://127.0.0.1:8000/formsubmission