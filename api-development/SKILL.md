---
name: api-development
description: "Use when creating API endpoints, REST resources, or CRUD APIs. Guides the generation of complete, production-ready API endpoints following Laravel best practices with JSON:API support for Laravel 13+."
metadata:
  author: corgab
---

# API Development

Best practices for building production-ready API endpoints. This skill must be used together with `laravel-best-practices`. For exact API syntax, verify with `search-docs`.

## Before You Start

Ask the user before generating any code:

1. **Which entity?** (e.g., "Product", "Order")
2. **Which CRUD operations?** (index, show, store, update, destroy)
3. **Does the model already exist?**

## Architecture

**Controller → Action → Model.** The controller must never contain business logic. It only coordinates: receives the validated request, calls an Action, returns the Resource.

- One Action class per operation (e.g., `CreateProductAction`, `UpdateProductAction`)
- Place Actions in `app/Actions/{Entity}/`
- Use `DB::transaction()` in Actions for all write operations
- Never pass raw arrays between layers — use DTOs

## Data Transfer Objects

Use PHP 8.2+ readonly classes for typed data between controller and action. Create a static `fromRequest()` factory method on the DTO. Place DTOs in `app/DataTransferObjects/`.

## Version Detection

Read `composer.json` to determine the Laravel version:

- **Laravel 13+** → Use `JsonApiResource`, `JsonApiCollection`, with `toAttributes()`, `toRelationships()`, `toResourceIdentifier()`, `toLinks()`, `toResourceMeta()`
- **Laravel < 13** → Use `JsonResource`, `ResourceCollection`, with `toArray()`

Use `search-docs` for the exact syntax of each resource type.

## File Generation

| File | When |
|------|------|
| Model + Migration | Only if model doesn't exist |
| Action classes | Always — one per operation |
| DTO | If store/update operations exist |
| Controller in `Api\V1\` | Always |
| FormRequest (Store + Update) | If store/update operations exist |
| Resource + Collection | Always |
| Policy | If operations require authorization |
| Route in `routes/api.php` | Always |
| Feature Test | Always |

## Controller

- Namespace: `App\Http\Controllers\Api\V1`
- One controller per resource
- Methods must be thin: validate via FormRequest, authorize via Policy, delegate to Action, return Resource
- Return type must be the Resource class, not `JsonResponse` — this is critical for Scramble documentation

## FormRequest

- Separate request for Store and Update — never share one
- Validation rules must be strict and specific (avoid `sometimes` without `required`)
- Use `sometimes|required` on Update requests for partial updates
- `authorize()` returns `true` when a Policy handles authorization

## Resource

- Relationships always wrapped in `whenLoaded()` (classic) or closures (JSON:API)
- Timestamps in ISO 8601 format
- ID cast to string in JSON:API mode
- Use `search-docs` with queries like `['eloquent resources', 'json api resources']` for version-specific patterns

## Policy

Generate only when the endpoint has operations that need authorization (store, update, destroy). Check `composer.json` for `spatie/laravel-permission`:

- **If installed**: use `$user->can('permission-name')` pattern
- **If not installed**: use standard ownership checks

Register authorization in the controller with `$this->authorize()`.

## Routes

Use `Route::apiResource()` in `routes/api.php` with versioned prefix:

- Prefix: `/v1/`
- Named: `api.v1.`
- Middleware: `auth:sanctum`, `throttle:api`
- Use `->only()` when not all CRUD operations are needed

## Filtering

Use `when()` for query filters in the `index` method. No external packages — keep it simple:

- `->when($request->search, fn ($q, $v) => ...)` for each filterable field
- Sorting with `-` prefix for descending (e.g., `?sort=-created_at`)

## Error Handling

Do not override Laravel's default error handling. FormRequest returns structured 422 errors automatically. Route model binding returns 404. Policy denials return 403. Keep it consistent.

## Rate Limiting

Apply `throttle:api` middleware in the route group. Customize in `AppServiceProvider` if needed. Use `search-docs` with query `rate limiting` for current syntax.

## Testing

Generate a Feature test for each CRUD operation. Follow the `pest-testing` skill for conventions on naming, factories, and assertions. Each test must:

- Authenticate with `actingAs()`
- Assert correct HTTP status
- Assert response structure
- Assert database state for write operations
- Test validation failures for store/update

## API Documentation — Scramble

Check if `dedoc/scramble` is in `composer.json`. If not, suggest installing it.

To ensure Scramble generates accurate docs:

- Type hints on all method parameters and return types
- Return the Resource class directly, not `response()->json()`
- PHPDoc on every public controller method with a description
- FormRequest rules fully typed — Scramble reads them to document parameters
