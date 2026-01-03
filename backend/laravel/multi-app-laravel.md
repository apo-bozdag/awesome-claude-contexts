# Multi-App Laravel Backend Architecture

Bu backend birden fazla mobil uygulamayı tek API üzerinden destekler.
Her app kendi modülünde yaşar, ortak servisler (auth, subscription, user) paylaşılır.

**Base URL**: `api.radkod.com/{app}/api/v1/...`

---

## Mimari Genel Bakış

```
┌─────────────────────────────────────────────────────────────────────┐
│                    api.radkod.com                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  /rota/api/v1/...    /namaz/api/v1/...    /market/api/v1/...       │
│  ┌─────────────┐     ┌─────────────┐      ┌─────────────┐          │
│  │  Rota App   │     │  Namaz App  │      │  Market App │          │
│  │  Module     │     │  Module     │      │  Module     │          │
│  └──────┬──────┘     └──────┬──────┘      └──────┬──────┘          │
│         │                   │                    │                  │
│         └───────────────────┼────────────────────┘                  │
│                             │                                       │
│                             ▼                                       │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                      SHARED KERNEL                             │ │
│  │  ┌─────────┐  ┌──────────────┐  ┌────────────┐  ┌───────────┐ │ │
│  │  │  Auth   │  │ Subscription │  │   User     │  │   IAP     │ │ │
│  │  └─────────┘  └──────────────┘  └────────────┘  └───────────┘ │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                      INFRASTRUCTURE                            │ │
│  │  PostgreSQL │ Redis │ S3 │ Queue │ Sanctum │ Scramble         │ │
│  └───────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Tech Stack

- **Laravel**: 12.x, PHP 8.3+
- **Database**: PostgreSQL 16
- **Auth**: Laravel Sanctum (token-based, app-aware)
- **Localization**: Spatie Laravel Translatable
- **IAP**: imdhemy/laravel-purchases (Apple + Google)
- **Cache/Queue**: Redis
- **API Docs**: Scramble (OpenAPI 3.1.0)
- **Modules**: nwidart/laravel-modules

---

## Proje Yapısı

```
├── app/
│   ├── Console/
│   ├── Exceptions/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   └── Controller.php
│   │   ├── Middleware/
│   │   │   ├── IdentifyApp.php           # App detection from URL
│   │   │   └── EnsureAppAccess.php       # User has access to app
│   │   └── Traits/
│   │       └── ApiResponse.php
│   ├── Models/                            # Core/Shared models
│   │   ├── User.php
│   │   ├── App.php
│   │   └── UserApp.php
│   ├── Enums/
│   │   ├── AppCode.php
│   │   ├── SubscriptionStatus.php
│   │   ├── BillingCycle.php
│   │   └── PaymentPlatform.php
│   ├── Builders/
│   ├── Repositories/
│   │   └── Contracts/
│   └── Services/
│       └── AppContext.php                 # Current app context singleton
│
├── modules/                               # nwidart/laravel-modules
│   │
│   ├── Core/                              # Shared Kernel
│   │   ├── Auth/
│   │   │   ├── Http/
│   │   │   │   ├── Controllers/
│   │   │   │   │   └── AuthController.php
│   │   │   │   └── Requests/
│   │   │   ├── Services/
│   │   │   │   └── AuthService.php
│   │   │   ├── Actions/
│   │   │   │   ├── RegisterUser.php
│   │   │   │   └── LoginUser.php
│   │   │   ├── Routes/
│   │   │   │   └── api.php
│   │   │   └── Providers/
│   │   │       └── AuthServiceProvider.php
│   │   │
│   │   ├── Subscription/
│   │   │   ├── Models/
│   │   │   │   ├── Plan.php
│   │   │   │   ├── Subscription.php
│   │   │   │   └── Receipt.php
│   │   │   ├── Services/
│   │   │   │   ├── SubscriptionService.php
│   │   │   │   └── IAPService.php
│   │   │   ├── Database/
│   │   │   │   └── Migrations/
│   │   │   └── Providers/
│   │   │
│   │   ├── User/
│   │   │   └── ...
│   │   │
│   │   └── Notification/
│   │       └── ...
│   │
│   ├── Rota/                              # Rota App Module
│   │   ├── Http/
│   │   │   ├── Controllers/
│   │   │   │   ├── ListController.php
│   │   │   │   ├── POIController.php
│   │   │   │   └── SearchController.php
│   │   │   ├── Requests/
│   │   │   │   ├── StoreListRequest.php
│   │   │   │   └── UpdateListRequest.php
│   │   │   └── Resources/
│   │   │       ├── ListResource.php
│   │   │       └── POIResource.php
│   │   ├── Models/
│   │   │   ├── POIList.php
│   │   │   ├── POI.php
│   │   │   └── Category.php
│   │   ├── Builders/
│   │   │   └── POIListBuilder.php
│   │   ├── Repositories/
│   │   │   ├── Contracts/
│   │   │   │   └── POIListRepositoryInterface.php
│   │   │   └── POIListRepository.php
│   │   ├── Services/
│   │   │   └── ListService.php
│   │   ├── Policies/
│   │   │   └── POIListPolicy.php
│   │   ├── Config/
│   │   │   └── config.php
│   │   ├── Database/
│   │   │   ├── Migrations/
│   │   │   ├── Seeders/
│   │   │   └── Factories/
│   │   ├── Routes/
│   │   │   └── api.php
│   │   ├── Tests/
│   │   │   ├── Feature/
│   │   │   └── Unit/
│   │   ├── Providers/
│   │   │   └── RotaServiceProvider.php
│   │   └── module.json
│   │
│   ├── Namaz/                             # Prayer/Namaz App Module
│   │   ├── Http/
│   │   │   └── Controllers/
│   │   │       ├── PrayerTimeController.php
│   │   │       ├── MosqueController.php
│   │   │       └── DhikrController.php
│   │   ├── Models/
│   │   │   ├── PrayerTime.php
│   │   │   ├── Mosque.php
│   │   │   ├── DhikrLog.php
│   │   │   └── QiblaCalculation.php
│   │   ├── Services/
│   │   │   ├── PrayerTimeService.php
│   │   │   ├── DiyanetService.php
│   │   │   └── QiblaService.php
│   │   ├── Database/
│   │   │   └── Migrations/
│   │   ├── Routes/
│   │   │   └── api.php
│   │   ├── Providers/
│   │   │   └── NamazServiceProvider.php
│   │   └── module.json
│   │
│   └── Market/                            # Market/Fiyat App Module
│       ├── Http/
│       │   └── Controllers/
│       │       ├── ProductController.php
│       │       ├── StoreController.php
│       │       └── PriceController.php
│       ├── Models/
│       │   ├── Product.php
│       │   ├── Store.php
│       │   ├── Price.php
│       │   └── PriceHistory.php
│       ├── Services/
│       │   ├── PriceComparisonService.php
│       │   └── ScraperService.php
│       ├── Database/
│       │   └── Migrations/
│       ├── Routes/
│       │   └── api.php
│       └── module.json
│
├── config/
│   ├── apps.php                           # Registered apps config
│   └── modules.php
│
├── database/
│   └── migrations/                        # Core migrations only
│
└── routes/
    └── api.php                            # Core routes
```

---

## URL Routing Strategy

### Route Pattern

```
api.radkod.com/{app}/api/v1/{resource}

Examples:
- api.radkod.com/rota/api/v1/lists
- api.radkod.com/namaz/api/v1/prayer-times
- api.radkod.com/market/api/v1/products
```

### Core Routes (routes/api.php)

```php
<?php

use Illuminate\Support\Facades\Route;

// Health check (no auth, no app context)
Route::get('/health', fn() => response()->json(['status' => 'ok']));

// App-specific routes
Route::prefix('{app}/api/v1')
    ->middleware(['app.identify'])
    ->group(function () {
        
        // Shared Auth Routes (no auth required)
        Route::prefix('auth')->group(function () {
            Route::post('/register', [AuthController::class, 'register']);
            Route::post('/login', [AuthController::class, 'login']);
            Route::post('/forgot-password', [AuthController::class, 'forgotPassword']);
            Route::post('/social/{provider}', [AuthController::class, 'social']);
        });
        
        // Protected Shared Routes
        Route::middleware(['auth:sanctum'])->group(function () {
            // Auth
            Route::post('/auth/logout', [AuthController::class, 'logout']);
            Route::get('/auth/me', [AuthController::class, 'me']);
            Route::put('/auth/profile', [AuthController::class, 'updateProfile']);
            Route::post('/auth/delete-account', [AuthController::class, 'deleteAccount']);
            
            // Subscription
            Route::prefix('subscription')->group(function () {
                Route::get('/plans', [PlanController::class, 'index']);
                Route::get('/current', [SubscriptionController::class, 'current']);
                Route::post('/verify', [SubscriptionController::class, 'verify']);
                Route::post('/restore', [SubscriptionController::class, 'restore']);
            });
        });
    });
```

### Module Routes (modules/Rota/Routes/api.php)

```php
<?php

use Illuminate\Support\Facades\Route;
use Modules\Rota\Http\Controllers\ListController;
use Modules\Rota\Http\Controllers\POIController;
use Modules\Rota\Http\Controllers\FeedController;

// All routes automatically prefixed with: /rota/api/v1
Route::prefix('rota/api/v1')
    ->middleware(['app.identify', 'app.restrict:rota', 'auth:sanctum'])
    ->group(function () {
        
        // Lists
        Route::apiResource('lists', ListController::class);
        Route::post('lists/{list}/like', [ListController::class, 'like']);
        Route::delete('lists/{list}/like', [ListController::class, 'unlike']);
        Route::post('lists/{list}/save', [ListController::class, 'save']);
        Route::post('lists/{list}/pois', [ListController::class, 'addPOI']);
        Route::delete('lists/{list}/pois/{poi}', [ListController::class, 'removePOI']);
        
        // POIs
        Route::get('pois/search', [POIController::class, 'search']);
        Route::get('pois/nearby', [POIController::class, 'nearby']);
        Route::get('pois/{poi}', [POIController::class, 'show']);
        
        // Feed
        Route::get('feed/trending', [FeedController::class, 'trending']);
        Route::get('feed/nearby', [FeedController::class, 'nearby']);
        Route::get('feed/following', [FeedController::class, 'following']);
        
        // Categories & Cities (public data)
        Route::get('categories', [CategoryController::class, 'index']);
        Route::get('cities', [CityController::class, 'index']);
    });
```

---

## App Identification System

### AppCode Enum

```php
// app/Enums/AppCode.php
<?php

namespace App\Enums;

enum AppCode: string
{
    case ROTA = 'rota';
    case NAMAZ = 'namaz';
    case MARKET = 'market';
    
    public function label(): string
    {
        return match($this) {
            self::ROTA => 'Rota',
            self::NAMAZ => 'Namaz Vakti',
            self::MARKET => 'Market Asistanı',
        };
    }
    
    public function bundleIdIos(): string
    {
        return match($this) {
            self::ROTA => 'com.radkod.rota',
            self::NAMAZ => 'com.radkod.namaz',
            self::MARKET => 'com.radkod.market',
        };
    }
    
    public function packageNameAndroid(): string
    {
        return match($this) {
            self::ROTA => 'com.radkod.rota',
            self::NAMAZ => 'com.radkod.namaz',
            self::MARKET => 'com.radkod.market',
        };
    }
    
    public function moduleNamespace(): string
    {
        return match($this) {
            self::ROTA => 'Modules\\Rota',
            self::NAMAZ => 'Modules\\Namaz',
            self::MARKET => 'Modules\\Market',
        };
    }
    
    public static function values(): array
    {
        return array_column(self::cases(), 'value');
    }
    
    public static function fromRoute(string $routeParam): ?self
    {
        return self::tryFrom($routeParam);
    }
}
```

### IdentifyApp Middleware

```php
// app/Http/Middleware/IdentifyApp.php
<?php

namespace App\Http\Middleware;

use App\Enums\AppCode;
use App\Services\AppContext;
use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

readonly class IdentifyApp
{
    public function __construct(
        private AppContext $appContext
    ) {}

    public function handle(Request $request, Closure $next): Response
    {
        $appParam = $request->route('app');
        
        if (!$appParam) {
            return response()->json([
                'status' => 'error',
                'message' => 'App identifier is required',
            ], 400);
        }
        
        $app = AppCode::fromRoute($appParam);
        
        if (!$app) {
            return response()->json([
                'status' => 'error',
                'message' => 'Invalid app identifier',
                'valid_apps' => AppCode::values(),
            ], 400);
        }
        
        $this->appContext->setApp($app);
        
        // Set app in request for easy access
        $request->attributes->set('app', $app);
        $request->attributes->set('app_code', $app->value);
        
        return $next($request);
    }
}
```

### EnsureAppAccess Middleware

```php
// app/Http/Middleware/EnsureAppAccess.php
<?php

namespace App\Http\Middleware;

use App\Enums\AppCode;
use App\Services\AppContext;
use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

readonly class EnsureAppAccess
{
    public function __construct(
        private AppContext $appContext
    ) {}

    public function handle(Request $request, Closure $next, string $requiredApp): Response
    {
        $required = AppCode::from($requiredApp);
        
        if (!$this->appContext->is($required)) {
            return response()->json([
                'status' => 'error',
                'message' => "This endpoint is only available for {$required->label()} app",
            ], 403);
        }
        
        return $next($request);
    }
}
```

### AppContext Service

```php
// app/Services/AppContext.php
<?php

namespace App\Services;

use App\Enums\AppCode;
use App\Models\App;

class AppContext
{
    private ?AppCode $currentApp = null;
    private ?App $appModel = null;

    public function setApp(AppCode $app): void
    {
        $this->currentApp = $app;
        $this->appModel = null;
    }

    public function getApp(): ?AppCode
    {
        return $this->currentApp;
    }
    
    public function getAppCode(): ?string
    {
        return $this->currentApp?->value;
    }
    
    public function getAppModel(): ?App
    {
        if (!$this->currentApp) {
            return null;
        }
        
        if (!$this->appModel) {
            $this->appModel = App::query()
                ->where('code', $this->currentApp->value)
                ->first();
        }
        
        return $this->appModel;
    }
    
    public function getAppId(): ?int
    {
        return $this->getAppModel()?->id;
    }

    public function is(AppCode $app): bool
    {
        return $this->currentApp === $app;
    }
    
    public function isRota(): bool
    {
        return $this->is(AppCode::ROTA);
    }
    
    public function isNamaz(): bool
    {
        return $this->is(AppCode::NAMAZ);
    }
    
    public function isMarket(): bool
    {
        return $this->is(AppCode::MARKET);
    }
    
    public function require(): AppCode
    {
        if (!$this->currentApp) {
            throw new \RuntimeException('App context is not set');
        }
        
        return $this->currentApp;
    }
}
```

### Register Middleware (bootstrap/app.php)

```php
// bootstrap/app.php
<?php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        api: __DIR__.'/../routes/api.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->alias([
            'app.identify' => \App\Http\Middleware\IdentifyApp::class,
            'app.restrict' => \App\Http\Middleware\EnsureAppAccess::class,
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })
    ->create();
```

---

## Database Schema

### Core Tables

```php
// database/migrations/xxxx_create_apps_table.php
Schema::create('apps', function (Blueprint $table) {
    $table->id();
    $table->string('code')->unique();              // 'rota', 'namaz', 'market'
    $table->string('name');
    $table->string('bundle_id_ios')->nullable();
    $table->string('package_name_android')->nullable();
    $table->boolean('is_active')->default(true);
    $table->json('settings')->nullable();
    $table->timestamps();
});

// database/migrations/xxxx_create_user_apps_table.php
Schema::create('user_apps', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->cascadeOnDelete();
    $table->foreignId('app_id')->constrained()->cascadeOnDelete();
    $table->timestamp('registered_at');
    $table->timestamp('last_active_at')->nullable();
    $table->json('preferences')->nullable();
    $table->timestamps();
    
    $table->unique(['user_id', 'app_id']);
});

// Extend users table
Schema::table('users', function (Blueprint $table) {
    $table->string('primary_app_code')->nullable()->after('email');
});
```

### Subscription Tables (App-Aware)

```php
// modules/Core/Subscription/Database/Migrations/xxxx_create_plans_table.php
Schema::create('plans', function (Blueprint $table) {
    $table->id();
    $table->foreignId('app_id')->constrained();
    $table->string('code')->unique();
    $table->json('name');                          // Translatable
    $table->json('description')->nullable();
    $table->string('billing_cycle');               // monthly, yearly, lifetime
    $table->integer('price_cents');
    $table->string('currency')->default('TRY');
    $table->string('apple_product_id')->nullable();
    $table->string('google_product_id')->nullable();
    $table->json('features')->nullable();
    $table->integer('trial_days')->default(0);
    $table->integer('sort_order')->default(0);
    $table->boolean('is_active')->default(true);
    $table->timestamps();
    
    $table->index(['app_id', 'is_active']);
});

// modules/Core/Subscription/Database/Migrations/xxxx_create_subscriptions_table.php
Schema::create('subscriptions', function (Blueprint $table) {
    $table->id();
    $table->uuid('uuid')->unique();
    $table->foreignId('user_id')->constrained()->cascadeOnDelete();
    $table->foreignId('app_id')->constrained();
    $table->foreignId('plan_id')->constrained();
    $table->string('status');                      // SubscriptionStatus enum
    $table->string('platform');                    // PaymentPlatform enum
    $table->string('original_transaction_id')->nullable();
    $table->timestamp('starts_at');
    $table->timestamp('ends_at')->nullable();
    $table->timestamp('trial_ends_at')->nullable();
    $table->timestamp('canceled_at')->nullable();
    $table->json('metadata')->nullable();
    $table->timestamps();
    $table->softDeletes();
    
    $table->index(['user_id', 'app_id', 'status']);
    $table->index('original_transaction_id');
});
```

---

## Authentication Flow

### Register Action

```php
// modules/Core/Auth/Actions/RegisterUser.php
<?php

namespace Modules\Core\Auth\Actions;

use App\Models\User;
use App\Models\UserApp;
use App\Models\App;
use App\Services\AppContext;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;

readonly class RegisterUser
{
    public function __construct(
        private AppContext $appContext
    ) {}

    public function execute(array $data): array
    {
        return DB::transaction(function () use ($data) {
            $appCode = $this->appContext->getAppCode();
            $app = App::where('code', $appCode)->firstOrFail();
            
            // Check if user exists (could be from different app)
            $user = User::where('email', $data['email'])->first();
            
            if ($user) {
                // Check if already registered for this app
                $exists = UserApp::where('user_id', $user->id)
                    ->where('app_id', $app->id)
                    ->exists();
                    
                if ($exists) {
                    throw new \Exception('Bu e-posta adresi zaten kayıtlı');
                }
            } else {
                // Create new user
                $user = User::create([
                    'name' => $data['name'],
                    'email' => $data['email'],
                    'password' => Hash::make($data['password']),
                    'primary_app_code' => $appCode,
                ]);
            }
            
            // Register for this app
            UserApp::create([
                'user_id' => $user->id,
                'app_id' => $app->id,
                'registered_at' => now(),
            ]);
            
            // Create token with app scope
            $token = $user->createToken(
                name: $data['device_name'] ?? 'mobile',
                abilities: ["app:{$appCode}"]
            )->plainTextToken;
            
            return [
                'user' => $user->fresh(),
                'token' => $token,
                'app' => $appCode,
            ];
        });
    }
}
```

### Login Action

```php
// modules/Core/Auth/Actions/LoginUser.php
<?php

namespace Modules\Core\Auth\Actions;

use App\Models\User;
use App\Models\UserApp;
use App\Models\App;
use App\Services\AppContext;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

readonly class LoginUser
{
    public function __construct(
        private AppContext $appContext
    ) {}

    public function execute(array $credentials): array
    {
        $user = User::where('email', $credentials['email'])->first();
        
        if (!$user || !Hash::check($credentials['password'], $user->password)) {
            throw ValidationException::withMessages([
                'email' => ['Geçersiz e-posta veya şifre'],
            ]);
        }
        
        $appCode = $this->appContext->getAppCode();
        $app = App::where('code', $appCode)->firstOrFail();
        
        // Check or create app registration
        $userApp = UserApp::firstOrCreate(
            ['user_id' => $user->id, 'app_id' => $app->id],
            ['registered_at' => now()]
        );
        
        $userApp->update(['last_active_at' => now()]);
        
        // Revoke old tokens for this device/app
        if (isset($credentials['device_name'])) {
            $user->tokens()
                ->where('name', $credentials['device_name'])
                ->delete();
        }
        
        $token = $user->createToken(
            name: $credentials['device_name'] ?? 'mobile',
            abilities: ["app:{$appCode}"]
        )->plainTextToken;
        
        return [
            'user' => $user,
            'token' => $token,
            'app' => $appCode,
        ];
    }
}
```

### Auth Controller

```php
// modules/Core/Auth/Http/Controllers/AuthController.php
<?php

namespace Modules\Core\Auth\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Http\Traits\ApiResponse;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use Modules\Core\Auth\Actions\RegisterUser;
use Modules\Core\Auth\Actions\LoginUser;
use Modules\Core\Auth\Http\Requests\RegisterRequest;
use Modules\Core\Auth\Http\Requests\LoginRequest;
use Modules\Core\User\Http\Resources\UserResource;

class AuthController extends Controller
{
    use ApiResponse;

    public function register(
        RegisterRequest $request,
        RegisterUser $action
    ): JsonResponse {
        $result = $action->execute($request->validated());
        
        return $this->created([
            'user' => new UserResource($result['user']),
            'token' => $result['token'],
        ], 'Kayıt başarılı');
    }

    public function login(
        LoginRequest $request,
        LoginUser $action
    ): JsonResponse {
        $result = $action->execute($request->validated());
        
        return $this->success([
            'user' => new UserResource($result['user']),
            'token' => $result['token'],
        ], 'Giriş başarılı');
    }

    public function logout(Request $request): JsonResponse
    {
        $request->user()->currentAccessToken()->delete();
        
        return $this->success(null, 'Çıkış yapıldı');
    }

    public function me(Request $request): JsonResponse
    {
        return $this->success(new UserResource($request->user()));
    }
}
```

---

## User Model (Multi-App Aware)

```php
// app/Models/User.php
<?php

namespace App\Models;

use App\Enums\AppCode;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens;
use Modules\Core\Subscription\Models\Subscription;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;

    protected $fillable = [
        'name',
        'email',
        'password',
        'avatar',
        'primary_app_code',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'password' => 'hashed',
            'primary_app_code' => AppCode::class,
        ];
    }

    // Relationships
    public function apps(): BelongsToMany
    {
        return $this->belongsToMany(App::class, 'user_apps')
            ->withPivot(['registered_at', 'last_active_at', 'preferences'])
            ->withTimestamps();
    }

    public function userApps(): HasMany
    {
        return $this->hasMany(UserApp::class);
    }

    public function subscriptions(): HasMany
    {
        return $this->hasMany(Subscription::class);
    }

    // Helpers
    public function hasApp(AppCode $app): bool
    {
        return $this->apps()->where('code', $app->value)->exists();
    }

    public function getSubscription(AppCode $app): ?Subscription
    {
        return $this->subscriptions()
            ->whereHas('app', fn($q) => $q->where('code', $app->value))
            ->active()
            ->first();
    }

    public function isPremium(AppCode $app): bool
    {
        return $this->getSubscription($app) !== null;
    }
    
    public function registeredApps(): array
    {
        return $this->apps()->pluck('code')->toArray();
    }
    
    public function appPreferences(AppCode $app): ?array
    {
        $userApp = $this->userApps()
            ->whereHas('app', fn($q) => $q->where('code', $app->value))
            ->first();
            
        return $userApp?->preferences;
    }
}
```

---

## Module Service Provider

```php
// modules/Rota/Providers/RotaServiceProvider.php
<?php

namespace Modules\Rota\Providers;

use Illuminate\Support\ServiceProvider;
use Modules\Rota\Repositories\Contracts\POIListRepositoryInterface;
use Modules\Rota\Repositories\POIListRepository;

class RotaServiceProvider extends ServiceProvider
{
    protected string $moduleName = 'Rota';
    protected string $moduleNameLower = 'rota';

    public function boot(): void
    {
        $this->registerConfig();
        $this->registerMigrations();
        $this->loadRoutesFrom(module_path($this->moduleName, 'Routes/api.php'));
    }

    public function register(): void
    {
        // Repository bindings
        $this->app->bind(
            POIListRepositoryInterface::class,
            POIListRepository::class
        );
    }

    protected function registerConfig(): void
    {
        $this->mergeConfigFrom(
            module_path($this->moduleName, 'Config/config.php'),
            $this->moduleNameLower
        );
    }

    protected function registerMigrations(): void
    {
        $this->loadMigrationsFrom(
            module_path($this->moduleName, 'Database/Migrations')
        );
    }
}
```

---

## Subscription Service (App-Aware)

```php
// modules/Core/Subscription/Services/SubscriptionService.php
<?php

namespace Modules\Core\Subscription\Services;

use App\Models\User;
use App\Services\AppContext;
use Illuminate\Support\Collection;
use Modules\Core\Subscription\Models\Plan;
use Modules\Core\Subscription\Models\Subscription;

readonly class SubscriptionService
{
    public function __construct(
        private AppContext $appContext,
        private IAPService $iapService
    ) {}

    public function getPlans(): Collection
    {
        return Plan::query()
            ->where('app_id', $this->appContext->getAppId())
            ->where('is_active', true)
            ->orderBy('sort_order')
            ->get();
    }

    public function getCurrentSubscription(User $user): ?Subscription
    {
        return Subscription::query()
            ->where('user_id', $user->id)
            ->where('app_id', $this->appContext->getAppId())
            ->active()
            ->with('plan')
            ->first();
    }

    public function verifyPurchase(User $user, array $receiptData): Subscription
    {
        $verified = $this->iapService->verify($receiptData);
        
        if (!$verified->isValid()) {
            throw new \Exception('Invalid receipt');
        }
        
        $plan = Plan::query()
            ->where('app_id', $this->appContext->getAppId())
            ->where(function ($q) use ($verified) {
                $q->where('apple_product_id', $verified->productId)
                  ->orWhere('google_product_id', $verified->productId);
            })
            ->firstOrFail();
        
        return Subscription::updateOrCreate(
            [
                'user_id' => $user->id,
                'app_id' => $this->appContext->getAppId(),
                'original_transaction_id' => $verified->originalTransactionId,
            ],
            [
                'plan_id' => $plan->id,
                'status' => 'active',
                'platform' => $verified->platform,
                'starts_at' => $verified->purchaseDate,
                'ends_at' => $verified->expiresDate,
            ]
        );
    }
}
```

---

## Yeni App Ekleme

### 1. Enum'a Ekle

```php
// app/Enums/AppCode.php
case YENI_APP = 'yeniapp';

// Match ifadelerine ekle
public function label(): string
{
    return match($this) {
        // ...existing
        self::YENI_APP => 'Yeni Uygulama',
    };
}
```

### 2. Module Oluştur

```bash
php artisan module:make YeniApp
```

### 3. Database Seed

```php
// database/seeders/AppsSeeder.php
App::create([
    'code' => 'yeniapp',
    'name' => 'Yeni Uygulama',
    'bundle_id_ios' => 'com.radkod.yeniapp',
    'package_name_android' => 'com.radkod.yeniapp',
    'is_active' => true,
]);
```

### 4. Plans Oluştur

```php
Plan::create([
    'app_id' => $app->id,
    'code' => 'yeniapp_monthly',
    'name' => ['tr' => 'Aylık', 'en' => 'Monthly'],
    'billing_cycle' => 'monthly',
    'price_cents' => 4999,
    'apple_product_id' => 'com.radkod.yeniapp.monthly',
    'google_product_id' => 'yeniapp_monthly',
]);
```

### 5. Service Provider Register

```php
// config/modules.php veya bootstrap/providers.php
Modules\YeniApp\Providers\YeniAppServiceProvider::class,
```

---

## API Request Headers

```http
Accept: application/json
Content-Type: application/json
Authorization: Bearer <token>      # Protected routes
Accept-Language: tr                # tr, en
X-Device-ID: <uuid>               # Analytics
X-App-Version: 1.0.0              # Version gating
X-Platform: ios                   # ios, android
```

---

## Testing

```php
// tests/Feature/MultiAppAuthTest.php
<?php

namespace Tests\Feature;

use App\Models\User;
use App\Models\App;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class MultiAppAuthTest extends TestCase
{
    use RefreshDatabase;

    protected function setUp(): void
    {
        parent::setUp();
        $this->seed(\Database\Seeders\AppsSeeder::class);
    }

    public function test_user_can_register_for_multiple_apps(): void
    {
        // Register for Rota
        $response = $this->postJson('/rota/api/v1/auth/register', [
            'name' => 'Test User',
            'email' => 'test@example.com',
            'password' => 'password123',
            'password_confirmation' => 'password123',
        ]);
        
        $response->assertCreated();
        
        // Same user registers for Namaz
        $response = $this->postJson('/namaz/api/v1/auth/register', [
            'name' => 'Test User',
            'email' => 'test@example.com',
            'password' => 'password123',
            'password_confirmation' => 'password123',
        ]);
        
        $response->assertCreated();
        
        // User should have both apps
        $user = User::where('email', 'test@example.com')->first();
        $this->assertCount(2, $user->apps);
    }

    public function test_invalid_app_returns_error(): void
    {
        $response = $this->postJson('/invalid/api/v1/auth/register', [
            'name' => 'Test',
            'email' => 'test@example.com',
            'password' => 'password123',
        ]);
        
        $response->assertStatus(400)
            ->assertJsonPath('message', 'Invalid app identifier');
    }
}
```

---

## Caching Strategy

```php
// App-specific cache keys
$cacheKey = "app:{$appCode}:plans";
$plans = Cache::remember($cacheKey, 3600, fn() => 
    Plan::where('app_id', $appId)->active()->get()
);

// User+App specific
$cacheKey = "user:{$userId}:app:{$appCode}:subscription";

// Invalidation
Cache::forget("app:{$appCode}:plans");
Cache::tags(["app:{$appCode}"])->flush();
```

---

## Quick Reference

### Get Current App

```php
// In Controller/Service
$appContext = app(AppContext::class);
$appCode = $appContext->getAppCode();      // 'rota'
$appId = $appContext->getAppId();          // 1
$app = $appContext->getApp();              // AppCode::ROTA

// Check specific app
if ($appContext->isRota()) {
    // Rota-specific logic
}

// From Request
$app = $request->attributes->get('app');   // AppCode enum
```

### User App Helpers

```php
$user->hasApp(AppCode::ROTA);              // bool
$user->isPremium(AppCode::ROTA);           // bool
$user->getSubscription(AppCode::ROTA);     // Subscription|null
$user->registeredApps();                   // ['rota', 'namaz']
```

### API Response

```php
// Success
return $this->success($data, 'İşlem başarılı');
return $this->created($data, 'Oluşturuldu');
return $this->paginated($collection);

// Error
return $this->error('Hata', 400);
return $this->notFound('Bulunamadı');
return $this->unauthorized();
return $this->forbidden();
return $this->validationError($errors);
```

---

## Commands

```bash
# Module commands
php artisan module:make ModuleName
php artisan module:make-controller ControllerName ModuleName
php artisan module:make-model ModelName ModuleName -m
php artisan module:make-migration create_table ModuleName
php artisan module:migrate ModuleName

# Standard commands
php artisan migrate
php artisan db:seed
php artisan test
php artisan route:list --path=rota
```

---

**Last Updated**: 2026-01-03
