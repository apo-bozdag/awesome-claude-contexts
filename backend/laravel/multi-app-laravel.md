# Laravel Multi-App Backend Guidelines

Bu dosya, tek bir Laravel backend'in birden fazla mobil/web uygulamasına hizmet vermesi için modüler mimari kurallarını içerir. Her app kendi modülünde izole, ortak servisler paylaşımlı.

---

## Genel Mimari Yaklaşım

```
┌─────────────────────────────────────────────────────────────────┐
│                        Laravel Backend                          │
├─────────────────────────────────────────────────────────────────┤
│  Modules/                                                       │
│  ├── Core/          (Auth, User, Base services - ORTAK)        │
│  ├── AppRota/       (Rota app spesifik)                        │
│  ├── AppParolla/    (Parolla app spesifik)                     │
│  └── AppNewApp/     (Yeni app eklendiğinde)                    │
├─────────────────────────────────────────────────────────────────┤
│  app/               (Shared infrastructure)                     │
│  ├── Services/      (Ortak servisler)                          │
│  ├── Traits/        (Ortak traitler)                           │
│  └── Support/       (Helpers, utilities)                       │
└─────────────────────────────────────────────────────────────────┘
         │              │              │
         ▼              ▼              ▼
    [Rota App]    [Parolla App]   [Future Apps]
```

---

## Proje Yapısı

```
├── app/
│   ├── Exceptions/
│   │   └── Handler.php
│   ├── Http/
│   │   ├── Controllers/
│   │   │   └── Controller.php          # Base controller
│   │   ├── Middleware/
│   │   │   ├── IdentifyApp.php         # App identifier middleware
│   │   │   ├── ApiVersion.php          # API versioning middleware
│   │   │   └── TenantScope.php         # Optional: tenant scoping
│   │   └── Resources/
│   │       └── BaseResource.php        # API Resource base class
│   ├── Models/
│   │   └── Concerns/                   # Model traits
│   │       ├── HasUuid.php
│   │       ├── BelongsToApp.php        # app_id scope trait
│   │       └── Searchable.php
│   ├── Services/                       # Shared services
│   │   ├── FileUploadService.php
│   │   ├── NotificationService.php
│   │   ├── CacheService.php
│   │   └── ExternalApi/
│   │       ├── GooglePlacesService.php
│   │       └── FirebaseService.php
│   ├── Support/
│   │   ├── ApiResponse.php             # Standardized API responses
│   │   ├── QueryFilters.php
│   │   └── helpers.php
│   └── Providers/
│       └── AppServiceProvider.php
│
├── Modules/                            # nwidart/laravel-modules
│   ├── Core/                           # ORTAK - Tüm app'ler kullanır
│   │   ├── Config/
│   │   │   └── config.php
│   │   ├── Database/
│   │   │   ├── Migrations/
│   │   │   │   ├── create_users_table.php
│   │   │   │   ├── create_apps_table.php
│   │   │   │   └── create_devices_table.php
│   │   │   └── Seeders/
│   │   ├── Entities/                   # Models
│   │   │   ├── User.php
│   │   │   ├── App.php
│   │   │   └── Device.php
│   │   ├── Http/
│   │   │   ├── Controllers/
│   │   │   │   └── Api/
│   │   │   │       └── V1/
│   │   │   │           ├── AuthController.php
│   │   │   │           └── UserController.php
│   │   │   ├── Requests/
│   │   │   │   └── Api/V1/
│   │   │   │       ├── LoginRequest.php
│   │   │   │       └── RegisterRequest.php
│   │   │   └── Resources/
│   │   │       ├── UserResource.php
│   │   │       └── UserCollection.php
│   │   ├── Repositories/
│   │   │   ├── Contracts/
│   │   │   │   └── UserRepositoryInterface.php
│   │   │   └── UserRepository.php
│   │   ├── Services/
│   │   │   ├── AuthService.php
│   │   │   └── UserService.php
│   │   ├── Actions/                    # Single responsibility actions
│   │   │   ├── CreateUserAction.php
│   │   │   └── UpdateUserAction.php
│   │   ├── DTOs/
│   │   │   ├── CreateUserDTO.php
│   │   │   └── UpdateUserDTO.php
│   │   ├── Events/
│   │   │   └── UserRegistered.php
│   │   ├── Listeners/
│   │   │   └── SendWelcomeEmail.php
│   │   ├── Routes/
│   │   │   └── api.php
│   │   ├── Tests/
│   │   │   ├── Feature/
│   │   │   └── Unit/
│   │   ├── Providers/
│   │   │   ├── CoreServiceProvider.php
│   │   │   └── RepositoryServiceProvider.php
│   │   └── module.json
│   │
│   ├── AppRota/                        # Rota App Modülü
│   │   ├── Config/
│   │   │   └── config.php
│   │   ├── Database/
│   │   │   ├── Migrations/
│   │   │   │   ├── create_venues_table.php
│   │   │   │   ├── create_lists_table.php
│   │   │   │   └── create_list_items_table.php
│   │   │   └── Seeders/
│   │   ├── Entities/
│   │   │   ├── Venue.php
│   │   │   ├── VenueList.php
│   │   │   └── ListItem.php
│   │   ├── Http/
│   │   │   ├── Controllers/
│   │   │   │   └── Api/
│   │   │   │       └── V1/
│   │   │   │           ├── VenueController.php
│   │   │   │           └── ListController.php
│   │   │   ├── Requests/
│   │   │   └── Resources/
│   │   ├── Repositories/
│   │   ├── Services/
│   │   │   ├── VenueService.php
│   │   │   └── VenueSearchService.php
│   │   ├── Actions/
│   │   ├── DTOs/
│   │   ├── Routes/
│   │   │   └── api.php
│   │   ├── Providers/
│   │   │   └── AppRotaServiceProvider.php
│   │   └── module.json
│   │
│   └── AppParolla/                     # Parolla App Modülü
│       ├── Config/
│       ├── Database/
│       │   └── Migrations/
│       │       ├── create_games_table.php
│       │       ├── create_questions_table.php
│       │       └── create_scores_table.php
│       ├── Entities/
│       │   ├── Game.php
│       │   ├── Question.php
│       │   └── Score.php
│       ├── Http/
│       │   ├── Controllers/Api/V1/
│       │   ├── Requests/
│       │   └── Resources/
│       ├── Repositories/
│       ├── Services/
│       ├── Routes/
│       │   └── api.php
│       └── module.json
│
├── config/
│   ├── modules.php                     # Module configuration
│   ├── apps.php                        # App-specific configs
│   └── api.php                         # API settings
│
├── routes/
│   └── api.php                         # Global API routes (health, etc.)
│
├── database/
│   ├── migrations/                     # Core Laravel migrations
│   └── seeders/
│
└── tests/
    ├── Feature/
    └── Unit/
```

---

## Kurallar

### 1. Module Yapısı (nwidart/laravel-modules)

**Module oluşturma:**

```bash
php artisan module:make AppNewApp
```

**Her module içinde katmanlı yapı:**

```
Module/
├── Entities/         # Eloquent Models
├── Repositories/     # Data access layer
│   └── Contracts/    # Repository interfaces
├── Services/         # Business logic
├── Actions/          # Single-purpose actions
├── DTOs/             # Data transfer objects
├── Http/
│   ├── Controllers/  # Thin controllers
│   ├── Requests/     # Form requests
│   └── Resources/    # API resources
├── Events/
├── Listeners/
├── Jobs/
├── Routes/
└── Tests/
```

### 2. App İzolasyonu

**apps table:**

```php
// Modules/Core/Database/Migrations/create_apps_table.php
Schema::create('apps', function (Blueprint $table) {
    $table->id();
    $table->string('slug')->unique();      // 'rota', 'parolla'
    $table->string('name');
    $table->string('api_key')->unique();
    $table->json('settings')->nullable();
    $table->boolean('is_active')->default(true);
    $table->timestamps();
});
```

**App identification middleware:**

```php
// app/Http/Middleware/IdentifyApp.php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Modules\Core\Entities\App;

class IdentifyApp
{
    public function handle(Request $request, Closure $next)
    {
        $apiKey = $request->header('X-App-Key');
        
        if (!$apiKey) {
            return response()->json([
                'success' => false,
                'message' => 'App key required',
            ], 401);
        }
        
        $app = App::where('api_key', $apiKey)
            ->where('is_active', true)
            ->first();
            
        if (!$app) {
            return response()->json([
                'success' => false,
                'message' => 'Invalid app key',
            ], 401);
        }
        
        // App'i request'e bind et
        $request->merge(['app' => $app]);
        app()->instance('current_app', $app);
        
        return $next($request);
    }
}
```

**Model scope trait:**

```php
// app/Models/Concerns/BelongsToApp.php
namespace App\Models\Concerns;

use Illuminate\Database\Eloquent\Builder;

trait BelongsToApp
{
    protected static function bootBelongsToApp(): void
    {
        // Otomatik app_id ekleme
        static::creating(function ($model) {
            if (!$model->app_id && app()->has('current_app')) {
                $model->app_id = app('current_app')->id;
            }
        });
        
        // Global scope - sadece current app verileri
        static::addGlobalScope('app', function (Builder $builder) {
            if (app()->has('current_app')) {
                $builder->where('app_id', app('current_app')->id);
            }
        });
    }
    
    public function app()
    {
        return $this->belongsTo(\Modules\Core\Entities\App::class);
    }
}
```

### 3. API Versioning

**Route yapısı:**

```php
// Modules/Core/Routes/api.php
use Illuminate\Support\Facades\Route;

Route::prefix('v1')->group(function () {
    Route::post('auth/login', [AuthController::class, 'login']);
    Route::post('auth/register', [AuthController::class, 'register']);
    
    Route::middleware('auth:sanctum')->group(function () {
        Route::get('user', [UserController::class, 'me']);
        Route::put('user', [UserController::class, 'update']);
    });
});

// V2 eklendiğinde
Route::prefix('v2')->group(function () {
    // V2 routes...
});
```

**Module route registration:**

```php
// Modules/AppRota/Routes/api.php
use Illuminate\Support\Facades\Route;

Route::prefix('v1/rota')->middleware(['identify.app'])->group(function () {
    Route::get('venues', [VenueController::class, 'index']);
    Route::get('venues/{id}', [VenueController::class, 'show']);
    
    Route::middleware('auth:sanctum')->group(function () {
        Route::apiResource('lists', ListController::class);
    });
});
```

### 4. Controller Yapısı (Thin Controllers)

```php
// Modules/Core/Http/Controllers/Api/V1/AuthController.php
namespace Modules\Core\Http\Controllers\Api\V1;

use App\Http\Controllers\Controller;
use App\Support\ApiResponse;
use Modules\Core\Http\Requests\Api\V1\LoginRequest;
use Modules\Core\Http\Requests\Api\V1\RegisterRequest;
use Modules\Core\Services\AuthService;
use Modules\Core\Http\Resources\UserResource;

class AuthController extends Controller
{
    public function __construct(
        private AuthService $authService
    ) {}

    public function login(LoginRequest $request)
    {
        $result = $this->authService->login($request->validated());
        
        if (!$result['success']) {
            return ApiResponse::error($result['message'], 401);
        }
        
        return ApiResponse::success([
            'user' => new UserResource($result['user']),
            'token' => $result['token'],
        ], 'Login successful');
    }

    public function register(RegisterRequest $request)
    {
        $result = $this->authService->register(
            $request->validated(),
            app('current_app')
        );
        
        return ApiResponse::success([
            'user' => new UserResource($result['user']),
            'token' => $result['token'],
        ], 'Registration successful', 201);
    }
}
```

### 5. Service Layer

```php
// Modules/Core/Services/AuthService.php
namespace Modules\Core\Services;

use Illuminate\Support\Facades\Hash;
use Modules\Core\Repositories\Contracts\UserRepositoryInterface;
use Modules\Core\DTOs\CreateUserDTO;
use Modules\Core\Actions\CreateUserAction;
use Modules\Core\Entities\App;

class AuthService
{
    public function __construct(
        private UserRepositoryInterface $userRepository,
        private CreateUserAction $createUserAction
    ) {}

    public function login(array $credentials): array
    {
        $user = $this->userRepository->findByEmail($credentials['email']);
        
        if (!$user || !Hash::check($credentials['password'], $user->password)) {
            return [
                'success' => false,
                'message' => 'Invalid credentials',
            ];
        }
        
        $token = $user->createToken('auth-token')->plainTextToken;
        
        return [
            'success' => true,
            'user' => $user,
            'token' => $token,
        ];
    }

    public function register(array $data, App $app): array
    {
        $dto = new CreateUserDTO(
            name: $data['name'],
            email: $data['email'],
            password: $data['password'],
            appId: $app->id
        );
        
        $user = $this->createUserAction->execute($dto);
        $token = $user->createToken('auth-token')->plainTextToken;
        
        return [
            'user' => $user,
            'token' => $token,
        ];
    }
}
```

### 6. Repository Pattern

```php
// Modules/Core/Repositories/Contracts/UserRepositoryInterface.php
namespace Modules\Core\Repositories\Contracts;

use Modules\Core\Entities\User;

interface UserRepositoryInterface
{
    public function find(int $id): ?User;
    public function findByEmail(string $email): ?User;
    public function create(array $data): User;
    public function update(int $id, array $data): User;
    public function delete(int $id): bool;
}
```

```php
// Modules/Core/Repositories/UserRepository.php
namespace Modules\Core\Repositories;

use Modules\Core\Repositories\Contracts\UserRepositoryInterface;
use Modules\Core\Entities\User;

class UserRepository implements UserRepositoryInterface
{
    public function __construct(
        private User $model
    ) {}

    public function find(int $id): ?User
    {
        return $this->model->find($id);
    }

    public function findByEmail(string $email): ?User
    {
        return $this->model->where('email', $email)->first();
    }

    public function create(array $data): User
    {
        return $this->model->create($data);
    }

    public function update(int $id, array $data): User
    {
        $user = $this->find($id);
        $user->update($data);
        return $user->fresh();
    }

    public function delete(int $id): bool
    {
        return $this->model->destroy($id) > 0;
    }
}
```

**Repository binding:**

```php
// Modules/Core/Providers/RepositoryServiceProvider.php
namespace Modules\Core\Providers;

use Illuminate\Support\ServiceProvider;

class RepositoryServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->bind(
            \Modules\Core\Repositories\Contracts\UserRepositoryInterface::class,
            \Modules\Core\Repositories\UserRepository::class
        );
    }
}
```

### 7. Action Classes

```php
// Modules/Core/Actions/CreateUserAction.php
namespace Modules\Core\Actions;

use Illuminate\Support\Facades\Hash;
use Modules\Core\DTOs\CreateUserDTO;
use Modules\Core\Entities\User;
use Modules\Core\Events\UserRegistered;

class CreateUserAction
{
    public function execute(CreateUserDTO $dto): User
    {
        $user = User::create([
            'name' => $dto->name,
            'email' => $dto->email,
            'password' => Hash::make($dto->password),
            'app_id' => $dto->appId,
        ]);
        
        event(new UserRegistered($user));
        
        return $user;
    }
}
```

### 8. DTOs (Data Transfer Objects)

```php
// Modules/Core/DTOs/CreateUserDTO.php
namespace Modules\Core\DTOs;

readonly class CreateUserDTO
{
    public function __construct(
        public string $name,
        public string $email,
        public string $password,
        public int $appId,
    ) {}

    public static function fromRequest(array $data, int $appId): self
    {
        return new self(
            name: $data['name'],
            email: $data['email'],
            password: $data['password'],
            appId: $appId,
        );
    }
}
```

### 9. API Response Standardization

```php
// app/Support/ApiResponse.php
namespace App\Support;

use Illuminate\Http\JsonResponse;

class ApiResponse
{
    public static function success(
        mixed $data = null, 
        string $message = 'Success', 
        int $code = 200
    ): JsonResponse {
        return response()->json([
            'success' => true,
            'message' => $message,
            'data' => $data,
        ], $code);
    }

    public static function error(
        string $message = 'Error', 
        int $code = 400, 
        mixed $errors = null
    ): JsonResponse {
        $response = [
            'success' => false,
            'message' => $message,
        ];
        
        if ($errors !== null) {
            $response['errors'] = $errors;
        }
        
        return response()->json($response, $code);
    }

    public static function paginated(
        $paginator, 
        string $resourceClass, 
        string $message = 'Success'
    ): JsonResponse {
        return response()->json([
            'success' => true,
            'message' => $message,
            'data' => $resourceClass::collection($paginator->items()),
            'meta' => [
                'current_page' => $paginator->currentPage(),
                'last_page' => $paginator->lastPage(),
                'per_page' => $paginator->perPage(),
                'total' => $paginator->total(),
            ],
        ]);
    }
}
```

### 10. Form Requests

```php
// Modules/Core/Http/Requests/Api/V1/RegisterRequest.php
namespace Modules\Core\Http\Requests\Api\V1;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Contracts\Validation\Validator;
use Illuminate\Http\Exceptions\HttpResponseException;
use App\Support\ApiResponse;

class RegisterRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'name' => ['required', 'string', 'max:255'],
            'email' => ['required', 'email', 'unique:users,email'],
            'password' => ['required', 'string', 'min:8', 'confirmed'],
        ];
    }

    protected function failedValidation(Validator $validator): void
    {
        throw new HttpResponseException(
            ApiResponse::error(
                'Validation failed',
                422,
                $validator->errors()
            )
        );
    }
}
```

### 11. API Resources

```php
// Modules/Core/Http/Resources/UserResource.php
namespace Modules\Core\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'avatar_url' => $this->avatar_url,
            'created_at' => $this->created_at->toISOString(),
            'updated_at' => $this->updated_at->toISOString(),
        ];
    }
}
```

---

## Cross-Module Communication

**Modüller arası iletişim Events üzerinden:**

```php
// Modules/Core/Events/UserRegistered.php
namespace Modules\Core\Events;

use Modules\Core\Entities\User;

class UserRegistered
{
    public function __construct(
        public User $user
    ) {}
}

// Modules/AppRota/Listeners/CreateDefaultList.php
namespace Modules\AppRota\Listeners;

use Modules\Core\Events\UserRegistered;
use Modules\AppRota\Services\ListService;

class CreateDefaultList
{
    public function __construct(
        private ListService $listService
    ) {}

    public function handle(UserRegistered $event): void
    {
        if ($event->user->app->slug !== 'rota') {
            return;
        }
        
        $this->listService->createDefault($event->user);
    }
}
```

---

## Database Migration Strategy

**Her modül kendi migrations'ını yönetir:**

```bash
# Tüm modül migration'larını çalıştır
php artisan module:migrate

# Spesifik modül
php artisan module:migrate Core
php artisan module:migrate AppRota
```

**Shared tables (Core modülünde):**
- users
- apps
- devices
- personal_access_tokens

**App-specific tables (kendi modülünde):**
- AppRota: venues, venue_lists, list_items
- AppParolla: games, questions, scores

---

## Config Yapısı

```php
// config/apps.php
return [
    'rota' => [
        'name' => 'Rota',
        'features' => ['venues', 'lists', 'recommendations'],
        'settings' => [
            'max_lists_per_user' => 50,
            'max_items_per_list' => 100,
        ],
    ],
    'parolla' => [
        'name' => 'Parolla',
        'features' => ['daily_game', 'leaderboard'],
        'settings' => [
            'daily_game_time' => '00:00',
            'timezone' => 'Europe/Istanbul',
        ],
    ],
];
```

---

## Yeni App Ekleme Checklist

1. [ ] Module oluştur: `php artisan module:make AppNewApp`
2. [ ] `apps` tablosuna yeni app ekle (seeder veya migration)
3. [ ] API key generate et
4. [ ] Module config dosyasını düzenle
5. [ ] Routes tanımla: `Modules/AppNewApp/Routes/api.php`
6. [ ] Entities (Models) oluştur
7. [ ] Migrations oluştur ve çalıştır
8. [ ] Repositories + Interfaces tanımla
9. [ ] Services oluştur
10. [ ] Controllers + Requests + Resources oluştur
11. [ ] Cross-module event listeners (gerekirse)
12. [ ] Tests yaz

---

## Paket Listesi

```json
{
    "require": {
        "php": "^8.2",
        "laravel/framework": "^12.0",
        "laravel/sanctum": "^4.0",
        "nwidart/laravel-modules": "^12.0",
        "spatie/laravel-query-builder": "^6.0",
        "spatie/laravel-data": "^4.0",
        "spatie/laravel-permission": "^6.0"
    },
    "require-dev": {
        "barryvdh/laravel-ide-helper": "^3.0",
        "laravel/telescope": "^5.0",
        "pestphp/pest": "^3.0"
    }
}
```

---

## Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Module | PascalCase, App prefix | `AppRota`, `AppParolla` |
| Controller | PascalCase, Controller suffix | `VenueController` |
| Service | PascalCase, Service suffix | `VenueService` |
| Repository | PascalCase, Repository suffix | `VenueRepository` |
| Interface | PascalCase, Interface suffix | `VenueRepositoryInterface` |
| Action | PascalCase, Action suffix | `CreateVenueAction` |
| DTO | PascalCase, DTO suffix | `CreateVenueDTO` |
| Event | PascalCase, past tense | `VenueCreated` |
| Listener | PascalCase, verb phrase | `NotifyUserOfVenue` |
| Request | PascalCase, Request suffix | `CreateVenueRequest` |
| Resource | PascalCase, Resource suffix | `VenueResource` |

---

## API URL Structure

```
Base URL: https://api.domain.com

# Core endpoints (tüm app'ler)
POST   /api/v1/auth/login
POST   /api/v1/auth/register
POST   /api/v1/auth/logout
GET    /api/v1/user
PUT    /api/v1/user

# Rota app endpoints
GET    /api/v1/rota/venues
GET    /api/v1/rota/venues/{id}
GET    /api/v1/rota/venues/search
POST   /api/v1/rota/lists
GET    /api/v1/rota/lists
GET    /api/v1/rota/lists/{id}
PUT    /api/v1/rota/lists/{id}
DELETE /api/v1/rota/lists/{id}
POST   /api/v1/rota/lists/{id}/items

# Parolla app endpoints
GET    /api/v1/parolla/daily
POST   /api/v1/parolla/daily/submit
GET    /api/v1/parolla/leaderboard
```

---

## Anti-Patterns (YAPMA)

```php
// ❌ Controller'da business logic
public function store(Request $request)
{
    $user = User::create([
        'name' => $request->name,
        'password' => Hash::make($request->password),
    ]);
    
    Mail::to($user)->send(new WelcomeEmail());
    Cache::forget('users_count');
    
    return response()->json($user);
}

// ✅ Service/Action kullan
public function store(CreateUserRequest $request)
{
    $user = $this->createUserAction->execute(
        CreateUserDTO::fromRequest($request->validated())
    );
    
    return ApiResponse::success(
        new UserResource($user),
        'User created',
        201
    );
}
```

```php
// ❌ Modüller arası doğrudan model erişimi
// Modules/AppRota/Services/SomeService.php
use Modules\AppParolla\Entities\Score; // YANLIŞ!

// ✅ Events/Contracts üzerinden iletişim
event(new UserScoredHigh($user, $score));
```

```php
// ❌ App scope olmadan query
User::where('email', $email)->first();

// ✅ Global scope otomatik uygulanır veya explicit
User::withoutGlobalScope('app')
    ->where('email', $email)
    ->where('app_id', $appId)
    ->first();
```

---

## Notes

- Core modülü TÜM app'ler tarafından paylaşılır
- Her app kendi modülünde izole
- Cross-module communication sadece Events ile
- API versioning baştan planla (/v1/, /v2/)
- Her module kendi testlerini içermeli
