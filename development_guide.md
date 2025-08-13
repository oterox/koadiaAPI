## 📁 DEVELOPMENT_GUIDE.md

# Development Guide
# Laravel Secure API Project

## 🚀 Quick Start Guide

### Prerequisites
- Docker & Docker Compose installed
- Git for version control
- Postman or similar API testing tool
- Code editor (VS Code, PhpStorm, etc.)

### Initial Setup
```bash
# 1. Clone the repository
git clone <repository-url> laravel-api
cd laravel-api

# 2. Copy environment file
cp .env.example .env

# 3. Build and start containers
docker-compose up -d --build

# 4. Install dependencies
docker-compose exec app composer install

# 5. Generate application key
docker-compose exec app php artisan key:generate

# 6. Run migrations and seeders
docker-compose exec app php artisan migrate:fresh --seed

# 7. Install Sanctum
docker-compose exec app php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"

# 8. Clear caches
docker-compose exec app php artisan config:clear
docker-compose exec app php artisan cache:clear
```

### Verify Installation
```bash
# Check if API is responding
curl http://localhost:7001/api/health

# Expected response:
{
  "status": "healthy",
  "database": "connected", 
  "cache": "connected",
  "timestamp": "2024-01-15T10:30:00.000000Z"
}
```

## 🔧 Development Workflow

### Daily Development
```bash
# Start development environment
docker-compose up -d

# Watch logs (optional)
docker-compose logs -f app

# Access application container
docker-compose exec app bash

# Run tests
docker-compose exec app php artisan test

# Stop environment
docker-compose down
```

### Code Generation Commands
```bash
# Generate Controller
php artisan make:controller Api/ExampleController --api --requests

# Generate Model with migration
php artisan make:model Example -m -f -s

# Generate Form Request
php artisan make:request StoreExampleRequest

# Generate Resource
php artisan make:resource ExampleResource

# Generate Policy
php artisan make:policy ExamplePolicy --model=Example

# Generate Service class
php artisan make:service ExampleService

# Generate Test
php artisan make:test ExampleControllerTest
```

### Database Operations
```bash
# Create migration
php artisan make:migration create_examples_table

# Run migrations
php artisan migrate

# Rollback migrations
php artisan migrate:rollback

# Fresh migration with seeders
php artisan migrate:fresh --seed

# Generate seeder
php artisan make:seeder ExampleSeeder

# Run specific seeder
php artisan db:seed --class=ExampleSeeder
```

## 🏗️ Project Structure Guide

### Controllers (`app/Http/Controllers/Api/`)
```php
<?php
// Example Controller Structure
namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Requests\StoreTestRecordRequest;
use App\Http\Resources\TestRecordResource;
use App\Services\TestRecordService;

class TestRecordController extends Controller
{
    public function __construct(
        private TestRecordService $testRecordService
    ) {
        $this->middleware(['auth:sanctum']);
        $this->middleware(['permission:test-records.read'])->only(['index', 'show']);
        $this->middleware(['permission:test-records.create'])->only(['store']);
        $this->middleware(['permission:test-records.update'])->only(['update']);
        $this->middleware(['permission:test-records.delete'])->only(['destroy']);
    }

    public function index(Request $request)
    {
        $records = $this->testRecordService->getPaginated($request->all());
        
        return TestRecordResource::collection($records)
            ->additional([
                'meta' => [
                    'filters_applied' => $request->only(['status', 'priority', 'user_id'])
                ]
            ]);
    }
}
```

### Services (`app/Services/`)
```php
<?php
// Example Service Structure
namespace App\Services;

use App\Models\TestRecord;
use Illuminate\Pagination\LengthAwarePaginator;

class TestRecordService
{
    public function getPaginated(array $filters = []): LengthAwarePaginator
    {
        $query = TestRecord::with('user');
        
        // Apply filters
        if (isset($filters['status'])) {
            $query->where('status', $filters['status']);
        }
        
        if (isset($filters['priority'])) {
            $query->where('priority', $filters['priority']);
        }
        
        if (isset($filters['search'])) {
            $query->where(function ($q) use ($filters) {
                $q->where('title', 'like', "%{$filters['search']}%")
                  ->orWhere('description', 'like', "%{$filters['search']}%");
            });
        }
        
        // Apply sorting
        $sortField = $filters['sort'] ?? 'created_at';
        $sortDirection = $filters['direction'] ?? 'desc';
        
        $query->orderBy($sortField, $sortDirection);
        
        return $query->paginate(10);
    }
}
```