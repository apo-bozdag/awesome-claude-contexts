# Laravel 12 Development Guide

> Best practices and conventions for Laravel 12 projects with PHP 8.3+

## Project Structure

### Architecture Patterns
- **Repository Pattern**: Data access abstraction with interfaces in `app/Repositories/Contracts/`
- **Service Layer**: Business logic in `app/Services/` using readonly classes
- **Policy-Based Authorization**: Use Laravel policies for fine-grained access control
- **Form Request Validation**: Separate validation logic in `app/Http/Requests/`
- **API Resources**: Transform responses with `app/Http/Resources/`
- **Event-Driven**: Use Events + Listeners + Jobs for async operations

### Directory Organization
```
app/
├── Enums/              # PHP 8.3 backed enums
├── Events/             # Domain events
├── Jobs/               # Queueable jobs
├── Listeners/          # Event listeners (implement ShouldQueue for async)
├── Http/
│   ├── Controllers/    # Thin controllers (delegate to services)
│   ├── Requests/       # Form request validation classes
│   └── Resources/      # API response transformers
├── Models/             # Eloquent models
├── Policies/           # Authorization policies
├── Repositories/
│   ├── Contracts/      # Repository interfaces
│   └── *Repository.php # Repository implementations
├── Services/           # Business logic (readonly classes)
└── Traits/             # Reusable traits (e.g., ApiResponse)
```

## Coding Standards

### PHP 8.3+ Features
- Use **readonly classes** for services and DTOs
- Use **backed enums** for fixed value sets (status, roles, etc.)
- Use **typed properties** and **return types** everywhere
- Use **constructor property promotion** when possible
- Use **named arguments** for clarity in complex method calls

### Laravel 12 Conventions
- Use **Route Model Binding** instead of manual queries
- Use **API Resources** for all JSON responses
- Use **Form Requests** for validation (never validate in controllers)
- Use **Policies** for authorization (never check permissions in controllers)
- Use **Service Layer** for business logic (keep controllers thin)
- Use **Repository Pattern** for data access (abstract database queries)

### Database
- Always use **migrations** for schema changes
- Use **factories** for test data generation
- Use **seeders** for initial/demo data
- Add **indexes** on foreign keys and frequently queried columns
- Use **soft deletes** when data should be retained

### Testing
- Write **feature tests** for API endpoints
- Write **unit tests** for services and repositories
- Use `RefreshDatabase` trait for test isolation
- Use **factories** instead of manual data creation
- Mock external services and APIs
- Test both success and error cases

### Queue & Events
- Use **ShouldQueue** interface for async listeners
- Use **ShouldBeEncrypted** for sensitive job data
- Always set **tries** and **timeout** for jobs
- Use **queued listeners** instead of direct job dispatching when possible
- Log important queue failures

## Common Commands

### Development
```bash
./vendor/bin/sail up -d                    # Start Docker containers
./vendor/bin/sail artisan migrate          # Run migrations
./vendor/bin/sail artisan db:seed          # Seed database
./vendor/bin/sail artisan queue:work       # Start queue worker
./vendor/bin/sail test                     # Run tests
./vendor/bin/sail artisan tinker           # REPL for debugging
```

### Cache Management
```bash
./vendor/bin/sail artisan cache:clear      # Clear application cache
./vendor/bin/sail artisan config:cache     # Cache config files
./vendor/bin/sail artisan route:cache      # Cache routes
./vendor/bin/sail artisan view:cache       # Cache blade views
```

### Code Quality
```bash
./vendor/bin/sail artisan pint             # Format code with Laravel Pint
./vendor/bin/sail artisan test --coverage  # Run tests with coverage
./vendor/bin/sail artisan event:list       # Show event-listener mappings
```

## API Development

### Response Structure
```php
// Success response
{
    "success": true,
    "message": "Operation successful",
    "data": { ... }
}

// Error response
{
    "success": false,
    "message": "Error message",
    "errors": { ... }  // Validation errors
}
```

### Authentication
- Use **Laravel Sanctum** for API token authentication
- Store tokens securely on client side
- Include `Authorization: Bearer {token}` header in requests
- Implement rate limiting on auth endpoints

### Error Handling
- Return proper HTTP status codes (200, 201, 401, 403, 404, 422, 500)
- Use Form Requests for validation errors (422)
- Use Policies for authorization errors (403)
- Log unexpected errors for debugging

## Performance

### Caching
- Use **Redis** for cache storage
- Use **cache tags** for grouped invalidation
- Cache expensive queries and API responses
- Set appropriate TTL values (not too long, not too short)
- Invalidate cache on data mutations

### Database Optimization
- Use **eager loading** to prevent N+1 queries (`with()`, `load()`)
- Add indexes on frequently queried columns
- Use `select()` to fetch only needed columns
- Use `chunk()` for large dataset processing
- Monitor slow queries with Laravel Telescope

### Queue Optimization
- Use **async jobs** for time-consuming operations
- Use **job batching** for bulk operations
- Use **unique jobs** to prevent duplicate processing
- Monitor queue metrics (pending, failed, processed)

## Security

### Input Validation
- Always validate user input with Form Requests
- Use `validated()` method to get sanitized data
- Define validation rules clearly and specifically
- Use custom validation rules for complex logic

### Authorization
- Use Laravel Policies for all authorization logic
- Check permissions with `$this->authorize()` in controllers
- Never implement authorization logic in controllers or services
- Test authorization with feature tests

### Common Vulnerabilities
- Prevent **SQL Injection**: Use Eloquent ORM or parameterized queries
- Prevent **XSS**: Escape output in Blade templates (automatic with `{{ }}`)
- Prevent **CSRF**: Use `@csrf` in forms (automatic with API tokens)
- Prevent **Mass Assignment**: Define `$fillable` or `$guarded` in models
- Rate limit authentication and sensitive endpoints

## Troubleshooting

### Common Issues
- **Cache issues**: Run `php artisan cache:clear` and `php artisan config:clear`
- **Route issues**: Run `php artisan route:clear` and check `routes/api.php`
- **Queue issues**: Ensure queue worker is running and check failed jobs
- **Migration issues**: Check database connection and run `migrate:fresh`
- **Test issues**: Ensure test database is configured in `phpunit.xml`

### Debugging
- Use `dd()` and `dump()` for quick debugging
- Use `ray()` for advanced debugging (requires Ray app)
- Use Laravel Telescope for request/query monitoring
- Check logs in `storage/logs/laravel.log`
- Use `php artisan tinker` for interactive debugging

## Resources

- [Laravel 12 Documentation](https://laravel.com/docs/12.x)
- [Laravel News](https://laravel-news.com/)
- [Laracasts](https://laracasts.com/)
- [PHP 8.3 Documentation](https://www.php.net/releases/8.3/en.php)