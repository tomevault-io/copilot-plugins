## laravel-base-project

> - Pest kullan (projede zaten var).


# TESTING

- Pest kullan (projede zaten var).
- Test isimleri açıklayıcı olmalı.
- Her feature için test yazılabilir formatta üret.
- Factory'leri kullan.

```php
// ✅ İyi
it('can create a product', function () {
    $user = User::factory()->create();

    $response = $this->actingAs($user)
        ->post(route('admin.products.store'), [
            'name' => 'Test Product',
            'sku' => 'TEST-001',
            'price' => 99.99,
        ]);

    $response->assertRedirect();
    $this->assertDatabaseHas('products', [
        'name' => 'Test Product',
        'sku' => 'TEST-001',
    ]);
});
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/enesekinci) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
