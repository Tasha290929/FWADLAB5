# Лабораторная работа №5. Компоненты безопасности в Laravel

## Цель работы

Познакомиться с основами компонентов безопасности в Laravel, таких как аутентификация, авторизация, защита от CSRF, а также использование встроенных механизмов для управления доступом.

Освоить подходы к безопасной разработке, включая создание защищенных маршрутов и управление ролями пользователей.

## Аутентификация пользователей

За основу работы было взято продолжение 2 лабораторной работы.

1. Создаём Контроллер для аутентификации

```
php artisan make:controller AuthController
```

2. Реализуем методы и прописываем маршруты.

```php
Route::controller(AuthController::class)->group(function () {
    Route::get('/register', 'register')->name('auth.register');
    Route::post('/register', 'storeRegister')->name('store.register');
    Route::get('/login', 'login')->name('login');
    Route::post('/login', 'storeLogin')->name('store.login');
    Route::post('/logout', 'logout')->name('logout');
});
```

3. Создаём формы регистрации и авторизации ```login.blade.php```

```php
@extends('layouts.app')
@section('title', 'Логин')
@section('content')
<h1>Форма Логин</h1>
<form action="{{route('store.login')}}" method="POST">
    @csrf
    <div class="form-group">
        <label for="email">E-mail</label>
        <input type="text" name="email" id="email" class="form-control" required>
        @error('email')
        <p>{{ $message }}</p>
        @enderror
    </div>
    <div class="form-group">
        <label for="password">Password</label>
        <input type="password" name="password" id="password" class="form-control" />
        @error('password')
        <p>{{ $message }}</p>
        @enderror
    </div>
    <button type="submit" class="btn btn-primary">LogIn</button>
</form>
@endsection
```

для решистрации ```register.blade.php```

```php
@extends('layouts.app')
@section('title', 'Регистрация')
@section('content')
<h1>Форма регистрации</h1>
<form action="{{route('store.register')}}" method="POST">
    @csrf
    <div class="form-group">
        <label for="email">E-mail</label>
        <input type="text" name="email" id="email" class="form-control" required>
        @error('email')
        <p>{{ $message }}</p>
        @enderror
    </div>
    <div class="form-group">
        <label for="name">Name</label>
        <input type="text" name="name" id="name" class="form-control" required>
        @error('name')
        <p>{{ $message }}</p>
        @enderror
    </div>
    <div class="form-group">
        <label for="password">Password</label>
        <input type="password" name="password" id="password" class="form-control" />
        @error('password')
        <p>{{ $message }}</p>
        @enderror
    </div>
    <div class="form-group">
        <label for="password_confirmation">Confirm Password</label>
        <input type="password" name="password_confirmation" id="password_confirmation" class="form-control" />
        @error('password_confirmation')
        <p>{{ $message }}</p>
        @enderror
    </div>

    <button type="submit" class="btn btn-primary">Register</button>
</form>
@endsection
```

4. Создаём личный кабинет для пользователя ```profie.blade.php``` 

```php
@extends('layouts.app')

@section('content')
    <h1>Личный кабинет</h1>
    <p>Добро пожаловать, {{ $user->name }}!</p>
    <p>Email: {{ $user->email }}</p>
    <p>Дата регистрации: {{ $user->created_at->format('d.m.Y') }}</p>

    <form action="{{ route('logout') }}" method="POST">
        @csrf
        <button type="submit" class="btn btn-danger">Выйти</button>
    </form>
@endsection
```

* и ```adminprofile.blade.php``` для амина, что бы посмотреть данные всех зарегистрированных пользотелей 

```php
@extends('layouts.app')

@section('title', 'Админ-панель')

@section('content')
    <h1>Админ-панель</h1>
    <p>Добро пожаловать, {{ Auth::user()->name }}!</p>

    <h2>Список пользователей</h2>
    <table>
        <thead>
            <tr>
                <th>ID</th>
                <th>Имя</th>
                <th>Email</th>
                <th>Роль</th>
                <th>Дата регистрации</th>
            </tr>
        </thead>
        <tbody>
            @foreach ($users as $user)
                <tr>
                    <td>{{ $user->id }}</td>
                    <td>{{ $user->name }}</td>
                    <td>{{ $user->email }}</td>
                    <td>{{ $user->role }}</td>
                    <td>{{ $user->created_at->format('d.m.Y') }}</td>
                </tr>
            @endforeach
        </tbody>
    </table>
    <form action="{{ route('logout') }}" method="POST">
        @csrf
        <button type="submit" class="btn btn-danger">Выйти</button>
    </form>
@endsection
```

5. Создан маршрут с middleware auth:

```php
Route::middleware('auth')->group(function () {

    // Админ может видеть все кабинеты
    Route::get('/admin/users', [AuthController::class, 'adminprofile'])
        ->middleware('role:admin')
        ->name('admin.users');

    // Пользователь может видеть только свой кабинет
    Route::get('/profile', [AuthController::class, 'showProfile'])->name('profile');
});
```

6. Создаём миграции с добавлением колонки роли пользователя в базу данных

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('role')->default('user');
});

```

7. В контроллер добавляем 

```php 
    public function showDashboard()
    {
        $user = Auth::user();
    
        return view('profile', compact('user'));
    }


    public function adminprofile()
    {
        $users = User::all();
        return view('adminprofile', compact('users'));
    }
    public function showProfile()
{
    $user = Auth::user();
    return view('profile', compact('user'));
}

```

## Контрольные вопросы

* Какие готовые решения для аутентификации предоставляет Laravel?

Laravel предлагает пакет laravel/ui или встроенный механизм Breeze и Fortify для аутентификации.

* Какие методы аутентификации пользователей вы знаете?

По логину и паролю.
По уникальному имени и паролю.
Через подключение к гугл почте или любой другой. 
С использованием API-токенов.
Через OAuth (например, Laravel Passport).

* Чем отличается аутентификация от авторизации?

Аутентификация подтверждает личность пользователя.
Авторизация проверяет права доступа.

* Как обеспечить защиту от CSRF-атак в Laravel?

Laravel автоматически защищает формы с помощью токенов CSRF, добавленных через директиву @csrf.