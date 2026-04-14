## nutriplan

> This rule defines best practices and standards for Laravel models.


# Laravel Model Standards

This rule defines best practices and standards for Laravel models.

## Rule Details

- **Name**: Laravel Model Standards
- **Type**: Model Guidelines
- **Scope**: Model files
- **Priority**: High

## Requirements

1. Model Structure:

    - Use proper namespacing under `App\Models`
    - Follow Laravel's model naming conventions (singular, PascalCase)
    - Tables should be plural, snake_case versions of model names
    - Models should be unguarded by default (no `$fillable` property)
    - Use constructor property promotion where applicable
    - Include proper type declarations for properties
    - Use proper PHPDoc blocks for properties and methods

2. Model Properties:

    - Define `$casts` array for type casting attributes
    - Use `$dates` array for additional date fields
    - Define `$appends` array for computed attributes
    - Use `$with` array for default eager loading
    - Define `$touches` array for touch relationships
    - Use proper attribute accessors and mutators
    - Implement proper hidden/visible attributes

3. Model Relationships:

    - Use proper relationship method return types
    - Implement inverse relationships
    - Use proper relationship naming conventions
    - Define foreign key constraints in migrations
    - Use proper eager loading strategies
    - Implement proper polymorphic relationships
    - Include proper relationship documentation

4. Model Functionality:
    - Use proper scopes for query filtering
    - Implement proper events and observers
    - Use proper trait implementations
    - Define proper query scopes
    - Implement proper attribute casting
    - Use proper model factories
    - Include proper model testing

## Example

```php
<?php

declare(strict_types=1);

namespace App\Models;

use App\Enums\RecipeStatus;
use App\Traits\HasSlug;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Database\Eloquent\SoftDeletes;

class Recipe extends Model
{
    use HasFactory;
    use HasSlug;
    use SoftDeletes;

    /**
     * The attributes that should be cast.
     *
     * @var array<string, string>
     */
    protected $casts = [
        'status' => RecipeStatus::class,
        'preparation_time' => 'integer',
        'cooking_time' => 'integer',
        'published_at' => 'datetime',
        'is_featured' => 'boolean',
        'metadata' => 'array',
    ];

    /**
     * The relationships that should be touched on save.
     *
     * @var array<string>
     */
    protected $touches = ['user'];

    /**
     * The attributes that should be hidden for arrays.
     *
     * @var array<string>
     */
    protected $hidden = [
        'metadata',
    ];

    /**
     * The attributes that should be appended to arrays.
     *
     * @var array<string>
     */
    protected $appends = [
        'total_time',
        'difficulty_level',
    ];

    /**
     * The relationships that should always be loaded.
     *
     * @var array<string>
     */
    protected $with = [
        'category',
    ];

    /**
     * Get the user that owns the recipe.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    /**
     * Get the category that owns the recipe.
     */
    public function category(): BelongsTo
    {
        return $this->belongsTo(Category::class);
    }

    /**
     * Get the ingredients for the recipe.
     */
    public function ingredients(): HasMany
    {
        return $this->hasMany(Ingredient::class);
    }

    /**
     * Get the instructions for the recipe.
     */
    public function instructions(): HasMany
    {
        return $this->hasMany(Instruction::class)->orderBy('order');
    }

    /**
     * Scope a query to only include published recipes.
     */
    public function scopePublished($query)
    {
        return $query->where('status', RecipeStatus::Published)
            ->whereNotNull('published_at');
    }

    /**
     * Scope a query to only include featured recipes.
     */
    public function scopeFeatured($query)
    {
        return $query->where('is_featured', true);
    }

    /**
     * Get the total time attribute.
     */
    public function getTotalTimeAttribute(): int
    {
        return $this->preparation_time + $this->cooking_time;
    }

    /**
     * Get the difficulty level attribute.
     */
    public function getDifficultyLevelAttribute(): string
    {
        return match (true) {
            $this->total_time <= 30 => 'Easy',
            $this->total_time <= 60 => 'Medium',
            default => 'Hard',
        };
    }

    /**
     * The "booted" method of the model.
     */
    protected static function booted(): void
    {
        static::creating(function (Recipe $recipe) {
            if (empty($recipe->slug)) {
                $recipe->slug = Str::slug($recipe->title);
            }
        });
    }
}
```

## Validation

The IDE will enforce these rules by:

1. Validating model class structure and organization
2. Checking for proper relationship definitions
3. Ensuring proper type declarations and casting
4. Verifying proper use of Laravel model features
5. Validating proper documentation and PHPDoc blocks

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/foxen-digital)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/foxen-digital)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
