## crowbar-mobile

> This rule explains Laravel conventions and best practices for backend development.


# Laravel rules

- Use `php artisan make:{option}` to create models, migrations, controllers, etc.
- `app\Console\Kernel.php` does not exist in Laravel 11+. If the file is not present, use the the [app.php](mdc:bootstrap/app.php) file for related configurations.
- In Laravel 11+ commands created in `app\Console\Commands\` are automatically registered and available to use.
- Add environment variables to config files and avoid using env variables directly in the code. For example `config('app.name')` instead of `env('APP_NAME')`.
- Avoid N+1 queries by using eager loading or batch loading. Examples:

```php
$users = User::with('posts')->get();
$posts = Post::whereIn('user_id', $users->pluck('id'))->get();
```

- Prefer soft deletes for models using the `SoftDeletes` trait:

```php
class User extends Model
{
    use SoftDeletes;
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aguileraz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
