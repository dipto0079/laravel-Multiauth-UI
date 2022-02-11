# Laravel 8 Multi Authentication With Laravel Ui Package

## Step: 1 Create a New Laravel 8 Application
`composer create-project laravel/laravel laravel-ui-multiauth`

After successfully create a project run some commands to generate default auth.
```
composer require laravel/ui
php artisan ui bootstrap --auth
npm install && npm run dev 
npm run watch
```

## Step: 2 Create a Model and Migration File for Admin
```
php artisan make:model Admin -m
```
Add some code in the admin model

<b>Models / Admin.php</b>
```
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Foundation\Auth\User as Authenticatable;
class Admin extends Authenticatable
{
    use HasFactory;
    protected $guard = 'admin';

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'name', 'email', 'password',
    ];

    /**
     * The attributes that should be hidden for arrays.
     *
     * @var array
     */
    protected $hidden = [
        'password', 'remember_token',
    ];
}
```
Admin some code for the admin migration file. You can add more according to your project.
```
public function up()
    {
        Schema::create('admins', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->string('password');
            $table->timestamps();
        });
    }

```
Connect the database and run this command.
```
php artisan migrate
```
<b>config / auth.php</b>
```
'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'api' => [
            'driver' => 'token',
            'provider' => 'users',
        ],
        
        'admin' => [
            'driver' => 'session',
            'provider' => 'admins',
        ],
        'admin-api' => [
            'driver' => 'token',
            'provider' => 'admins',
        ],
        
    ],

    'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => App\User::class,
        ],
        
        'admins' => [
            'driver' => 'eloquent',
            'model' => App\Models\Admin::class,
        ],
        
    ],
```

## Step: 3 Create Admin Login Page and Admin Dashboard Page
<b>auth / admin_login.blade.php</b>
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset=UTF-8>
  <meta name=viewport content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossorigin="anonymous">
  <title>Admin</title>
</head>
<body>
    <div class="container">
      <div class="row justify-content-center mt-4">
          <div class="col-md-4">
            <div class="card">
              <div class="card-body">
                <form id="sign_in_adm" method="POST" action="{{ route('admin.login.submit') }}">
                  {{ csrf_field() }}
                <h1>Admin Login</h1>
                <div >
                  <input type="email" name=email class="form-control" placeholder="Email Address" value="{{ old('email') }}" required autofocus>
                </div>
                @if ($errors->has('email'))
                <span ><strong>{{ $errors->first('email') }}</strong></span>
                @endif
                <br>
                <div >
                  <input type="password" name="password" class="form-control" required>
                </div>
                <br>
                <div >
                  <button type="submit" class="btn btn-primary">Admin Login</button>
                </div>
                </form>
              </div>
            </div>
          </div>
      </div>
    </div>
</body>
</html>
```

<b>admin.blade.php</b>
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset=UTF-8>
  <meta name=viewport content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossorigin="anonymous">
  <title>Admin Dashboard</title>
</head>
<body>
    <nav class="navbar navbar-light bg-light">
        <div class="container-fluid">
          <a class="navbar-brand">Admin Dashboard</a>
          <a href="/admin/logout">Logout</a>
        </div>
      </nav>

      <div class="alert alert-success" role="alert">
        Welcome to admin dashboard
      </div>

</body>
</html>
```
## Step: 4 Create AdminController and AdminLoginController
```
php artisan make:controller Auth/AdminController
```
Add some code in <b>AdminController.php</b>
```
<?php
namespace App\Http\Controllers\Auth;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;
class AdminController extends Controller
{
    /**
     * Create a new controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('auth:admin');
    }
    /**
     * show dashboard.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        return view('admin');
    }
}
```
Run this command to create an admin login Controller.
```
php artisan make:controller Auth/AdminLoginController
```
<b>AdminLoginController.php</b>
```
<?php

namespace App\Http\Controllers\Auth;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;
use Auth;
use Route;
class AdminLoginController extends Controller
{
   
    public function __construct()
    {
      $this->middleware('guest:admin', ['except' => ['logout']]);
    }
    
    public function showLoginForm()
    {
      return view('auth.admin_login');
    }
    
    public function login(Request $request)
    {
      // Validate the form data
      $this->validate($request, [
        'email'   => 'required|email',
        'password' => 'required|min:6'
      ]);
      // Attempt to log the user in
      if (Auth::guard('admin')->attempt(['email' => $request->email, 'password' => $request->password])) {
        // if successful, then redirect to their intended location
        return redirect()->intended(route('admin.dashboard'));
      } 
      // if unsuccessful, then redirect back to the login with the form data
      return redirect()->back()->withInput($request->only('email', 'remember'));
    }
    
    public function logout()
    {
        Auth::guard('admin')->logout();
        return redirect('/admin');
    }
}
```

## Step: 5 Create some routes for admin.
```
Route::prefix('admin')->group(function() {
    Route::get('/login',[AdminLoginController::class, 'showLoginForm'])->name('admin.login');
    Route::post('/login',[AdminLoginController::class, 'login'])->name('admin.login.submit');
    Route::get('logout/', [AdminLoginController::class, 'logout'])->name('admin.logout');
    Route::get('/', [AdminController::class, 'index'])->name('admin.dashboard');
   }) ;
```

## Step: 6 Handle admin redirection to login page or dashboard
Add some function in <b>app/Exceptions/Handler.php</b>
```
<?php

namespace App\Exceptions;
use Illuminate\Auth\AuthenticationException;
use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;
use Illuminate\Support\Arr;
use Throwable;

class Handler extends ExceptionHandler
{
    /**
     * A list of the exception types that are not reported.
     *
     * @var array
     */
    protected $dontReport = [
        //
    ];
    public function report(Throwable $exception)
    {
        parent::report($exception);
    }

    
    public function render($request, Throwable $exception)
    {
        return parent::render($request, $exception);
    }

    protected function unauthenticated($request, AuthenticationException $exception)
    {
        if ($request->expectsJson()) {
            return response()->json(['error' => 'Unauthenticated.'], 401);
        }

        $guard = Arr::get($exception->guards(), 0);

       switch ($guard) {
         case 'admin':
           $login='admin.login';
           break;

         default:
           $login='login';
           break;
       }

        return redirect()->guest(route($login));
    }
    /**
     * A list of the inputs that are never flashed for validation exceptions.
     *
     * @var array
     */
    protected $dontFlash = [
        'current_password',
        'password',
        'password_confirmation',
    ];

    /**
     * Register the exception handling callbacks for the application.
     *
     * @return void
     */
    public function register()
    {
        $this->reportable(function (Throwable $e) {
            //
        });
    }
}
```
Update the following code in redirect if authenticated.php

<b>app\Http\Middleware\RedirectIfAuthenticated.php</b>
```
<?php

namespace App\Http\Middleware;

use App\Providers\RouteServiceProvider;
use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class RedirectIfAuthenticated
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @param  string|null  ...$guards
     * @return mixed
     */
    public function handle($request, Closure $next, $guard = null)
    {
 
        switch ($guard) {
            case 'admin':
              if (Auth::guard($guard)->check()) {
                return redirect()->route('admin.dashboard');
              }
             
            default:
              if (Auth::guard($guard)->check()) {
                  return redirect('/');
              }
              break;
          }
          return $next($request);
          
    }
}
```
Add an AdminSeeder For Admin Login Creadentials
```
php artisan make:seeder AdminSeeder 
```
```
public function run()
    {
        Admin::create([
            'name' => 'Admin',
            'email' => 'admin@towhid.com',
            'password' => Hash::make("123456"),
        ]);
    }
```
Register This Seeder In <b>seeders/DatabaseSeeder.php</b>
```
public function run()
    {
        $this->call(AdminSeeder::class);
    }
```
