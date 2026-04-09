## the-scent-php

> After reviewing your business information and requirements, I've designed a comprehensive e-commerce solution for "The Scent" that will create the "Wow Effect" while providing robust functionality.

# Enhanced E-Commerce Solution for The Scent

After reviewing your business information and requirements, I've designed a comprehensive e-commerce solution for "The Scent" that will create the "Wow Effect" while providing robust functionality.

## Database Structure (MySQL)

```sql
-- Create the database
CREATE DATABASE IF NOT EXISTS thescent_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE thescent_db;

-- ENUM types
CREATE TABLE IF NOT EXISTS user_roles (
    id TINYINT UNSIGNED PRIMARY KEY,
    name VARCHAR(20) NOT NULL UNIQUE
);

INSERT INTO user_roles (id, name) VALUES (1, 'user'), (2, 'admin');

CREATE TABLE IF NOT EXISTS order_statuses (
    id TINYINT UNSIGNED PRIMARY KEY,
    name VARCHAR(20) NOT NULL UNIQUE
);

INSERT INTO order_statuses (id, name) VALUES 
    (1, 'pending'), (2, 'processing'), (3, 'shipped'), (4, 'delivered'), (5, 'cancelled');

CREATE TABLE IF NOT EXISTS payment_statuses (
    id TINYINT UNSIGNED PRIMARY KEY,
    name VARCHAR(20) NOT NULL UNIQUE
);

INSERT INTO payment_statuses (id, name) VALUES 
    (1, 'pending'), (2, 'processing'), (3, 'completed'), (4, 'failed'), (5, 'refunded');

-- Users table
CREATE TABLE IF NOT EXISTS users (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    phone VARCHAR(20),
    role_id TINYINT UNSIGNED DEFAULT 1,
    login_attempts TINYINT UNSIGNED DEFAULT 0,
    lock_until DATETIME,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (role_id) REFERENCES user_roles(id)
);

-- Sessions table
CREATE TABLE IF NOT EXISTS sessions (
    id CHAR(36) PRIMARY KEY,
    user_id INT UNSIGNED NOT NULL,
    captcha_text VARCHAR(10),
    expires_at DATETIME NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Categories table
CREATE TABLE IF NOT EXISTS categories (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(120) NOT NULL UNIQUE,
    description TEXT,
    image_url VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Products table
CREATE TABLE IF NOT EXISTS products (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(280) NOT NULL UNIQUE,
    price DECIMAL(10,2) NOT NULL,
    description TEXT NOT NULL,
    short_description VARCHAR(255),
    image_url VARCHAR(255) NOT NULL,
    featured BOOLEAN DEFAULT FALSE,
    review_count INT UNSIGNED DEFAULT 0,
    average_rating DECIMAL(3,2) DEFAULT 0,
    category_id INT UNSIGNED,
    stock INT UNSIGNED DEFAULT 100,
    sku VARCHAR(50) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE SET NULL
);

-- Product ingredients
CREATE TABLE IF NOT EXISTS product_ingredients (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    product_id INT UNSIGNED NOT NULL,
    ingredient VARCHAR(100) NOT NULL,
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE
);

-- Product benefits
CREATE TABLE IF NOT EXISTS product_benefits (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    product_id INT UNSIGNED NOT NULL,
    benefit VARCHAR(255) NOT NULL,
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE
);

-- Product images (additional images)
CREATE TABLE IF NOT EXISTS product_images (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    product_id INT UNSIGNED NOT NULL,
    image_url VARCHAR(255) NOT NULL,
    sort_order TINYINT UNSIGNED DEFAULT 0,
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE
);

-- Scent profiles
CREATE TABLE IF NOT EXISTS scent_profiles (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    description TEXT NOT NULL,
    icon_class VARCHAR(50)
);

-- Product scent profiles
CREATE TABLE IF NOT EXISTS product_scent_profiles (
    product_id INT UNSIGNED NOT NULL,
    scent_profile_id INT UNSIGNED NOT NULL,
    intensity TINYINT UNSIGNED DEFAULT 5,
    PRIMARY KEY (product_id, scent_profile_id),
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE,
    FOREIGN KEY (scent_profile_id) REFERENCES scent_profiles(id) ON DELETE CASCADE
);

-- Mood categories
CREATE TABLE IF NOT EXISTS moods (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    description TEXT,
    icon_class VARCHAR(50)
);

-- Product moods (which moods a product helps with)
CREATE TABLE IF NOT EXISTS product_moods (
    product_id INT UNSIGNED NOT NULL,
    mood_id INT UNSIGNED NOT NULL,
    effectiveness TINYINT UNSIGNED DEFAULT 5,
    PRIMARY KEY (product_id, mood_id),
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE,
    FOREIGN KEY (mood_id) REFERENCES moods(id) ON DELETE CASCADE
);

-- [Additional tables following the same pattern as your PostgreSQL schema...]
```

## PHP Framework Structure (Laravel)

```php
<?php
// app/Models/Product.php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Factories\HasFactory;

class Product extends Model
{
    use HasFactory;

    protected $fillable = [
        'name', 'slug', 'price', 'description', 'short_description',
        'image_url', 'featured', 'category_id', 'stock', 'sku'
    ];

    public function category()
    {
        return $this->belongsTo(Category::class);
    }

    public function ingredients()
    {
        return $this->hasMany(ProductIngredient::class);
    }

    public function benefits()
    {
        return $this->hasMany(ProductBenefit::class);
    }

    public function additionalImages()
    {
        return $this->hasMany(ProductImage::class)->orderBy('sort_order');
    }

    public function scentProfiles()
    {
        return $this->belongsToMany(ScentProfile::class, 'product_scent_profiles')
                    ->withPivot('intensity');
    }

    public function moods()
    {
        return $this->belongsToMany(Mood::class, 'product_moods')
                    ->withPivot('effectiveness');
    }

    public function reviews()
    {
        return $this->hasMany(Review::class);
    }

    public function getAverageRatingAttribute()
    {
        return $this->reviews()->avg('rating') ?? 0;
    }
}
```

## Frontend Design and "Wow Effect" Features

```php
<!-- resources/views/home.blade.php -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Scent - Natural Aromatherapy Products</title>
    <link rel="stylesheet" href="{{ asset('css/app.css') }}">
    <!-- Custom fonts and icons -->
    <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Cormorant+Garamond:wght@300;400;500;600&family=Montserrat:wght@300;400;500&display=swap">
</head>
<body class="bg-soft-cream font-montserrat text-deep-brown">
    <!-- Animated hero section with parallax effect -->
    <section class="relative h-screen overflow-hidden">
        <div class="absolute inset-0 bg-gradient-to-b from-transparent to-soft-cream/80 z-10"></div>
        <div class="absolute inset-0 bg-cover bg-center" style="background-image: url('{{ asset('images/hero-bg.jpg') }}');">
            <div class="absolute inset-0 bg-soft-green/20"></div>
        </div>
        
        <div class="relative h-full flex items-center z-20">
            <div class="container mx-auto px-6">
                <h1 class="font-cormorant text-5xl md:text-7xl font-light text-deep-brown mb-4 opacity-0 animate-fade-in">
                    Experience <span class="italic">The Scent</span>
                </h1>
                <p class="text-lg md:text-xl text-sage-green max-w-xl mb-8 opacity-0 animate-fade-in animation-delay-300">
                    Discover premium aromatherapy products that nourish your body and calm your mind.
                </p>
                <div class="opacity-0 animate-fade-in animation-delay-600">
                    <a href="{{ route('products.index') }}" class="btn-primary">
                        Explore Our Collection
                    </a>
                </div>
            </div>
        </div>
        
        <div class="absolute bottom-8 left-1/2 transform -translate-x-1/2 animate-bounce z-20">
            <svg class="w-8 h-8 text-deep-brown" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 14l-7 7m0 0l-7-7m7 7V3"></path>
            </svg>
        </div>
    </section>

    <!-- Featured products with hover effect -->
    <section class="py-20 bg-soft-cream">
        <div class="container mx-auto px-6">
            <h2 class="font-cormorant text-4xl text-center mb-16">Our Featured Collection</h2>
            
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-8">
                @foreach($featuredProducts as $product)
                <div class="product-card">
                    <div class="relative overflow-hidden rounded-lg mb-4 group">
                        <img src="{{ $product->image_url }}" alt="{{ $product->name }}" class="w-full h-80 object-cover transform transition-transform duration-500 group-hover:scale-105">
                        <div class="absolute inset-0 bg-gradient-to-t from-deep-brown/60 to-transparent opacity-0 group-hover:opacity-100 transition-opacity duration-300 flex items-end">
                            <div class="p-6 w-full">
                                <div class="flex justify-between items-center">
                                    <a href="{{ route('products.show', $product->slug) }}" class="btn-light-sm">View Details</a>
                                    <button class="btn-icon add-to-cart" data-product-id="{{ $product->id }}">
                                        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 11V7a4 4 0 00-8 0v4M5 9h14l1 12H4L5 9z"></path>
                                        </svg>
                                    </button>
                                </div>
                            </div>
                        </div>
                    </div>
                    <h3 class="font-cormorant text-xl mb-1">{{ $product->name }}</h3>
                    <div class="flex justify-between items-center">
                        <span class="text-deep-brown">${{ number_format($product->price, 2) }}</span>
                        <div class="flex items-center">
                            <div class="flex text-amber-400">
                                @for($i = 1; $i <= 5; $i++)
                                    @if($i <= $product->average_rating)
                                        <svg class="w-4 h-4 fill-current" viewBox="0 0 24 24"><path d="M12 17.27L18.18 21l-1.64-7.03L22 9.24l-7.19-.61L12 2 9.19 8.63 2 9.24l5.46 4.73L5.82 21z"></path></svg>
                                    @else
                                        <svg class="w-4 h-4 fill-current text-gray-300" viewBox="0 0 24 24"><path d="M12 17.27L18.18 21l-1.64-7.03L22 9.24l-7.19-.61L12 2 9.19 8.63 2 9.24l5.46 4.73L5.82 21z"></path></svg>
                                    @endif
                                @endfor
                            </div>
                            <span class="text-sm text-gray-500 ml-1">({{ $product->review_count }})</span>
                        </div>
                    </div>
                </div>
                @endforeach
            </div>
        </div>
    </section>

    <!-- Interactive Scent Profile Finder -->
    <section class="py-20 bg-sage-green/10">
        <div class="container mx-auto px-6">
            <div class="max-w-4xl mx-auto text-center">
                <h2 class="font-cormorant text-4xl mb-6">Find Your Perfect Scent</h2>
                <p class="text-deep-brown/80 mb-12">Answer a few questions about your preferences and we'll recommend products tailored just for you.</p>
                
                <div id="scent-finder" class="bg-white rounded-xl shadow-soft p-8">
                    <!-- Step 1: Initial question -->
                    <div class="scent-step active" data-step="1">
                        <h3 class="font-cormorant text-2xl mb-6">What are you looking for today?</h3>
                        <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                            <button class="scent-option p-6 border border-sage-green/30 rounded-lg hover:bg-sage-green/5 transition" data-value="relax">
                                <div class="w-16 h-16 mx-auto mb-4 rounded-full bg-sage-green/20 flex items-center justify-center">
                                    <svg class="w-8 h-8 text-sage-green" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 3v4M3 5h4M6 17v4m-2-2h4m5-16l2.286 6.857L21 12l-5.714 2.143L13 21l-2.286-6.857L5 12l5.714-2.143L13 3z"></path></svg>
                                </div>
                                <span class="block text-lg font-medium">Relaxation</span>
                            </button>
                            
                            <button class="scent-option p-6 border border-sage-green/30 rounded-lg hover:bg-sage-green/5 transition" data-value="energy">
                                <div class="w-16 h-16 mx-auto mb-4 rounded-full bg-amber-400/20 flex items-center justify-center">
                                    <svg class="w-8 h-8 text-amber-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z"></path></svg>
                                </div>
                                <span class="block text-lg font-medium">Energy</span>
                            </button>
                            
                            <button class="scent-option p-6 border border-sage-green/30 rounded-lg hover:bg-sage-green/5 transition" data-value="focus">
                                <div class="w-16 h-16 mx-auto mb-4 rounded-full bg-blue-400/20 flex items-center justify-center">
                                    <svg class="w-8 h-8 text-blue-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z"></path></svg>
                                </div>
                                <span class="block text-lg font-medium">Focus</span>
                            </button>
                        </div>
                    </div>
                    
                    <!-- Additional steps would follow here -->
                    
                    <!-- Navigation buttons -->
                    <div class="mt-10 flex justify-between">
                        <button id="prev-step" class="btn-outline hidden">Previous</button>
                        <button id="next-step" class="btn-primary ml-auto">Continue</button>
                    </div>
                </div>
            </div>
        </div>
    </section>

    <!-- Aromatherapy Mood Mapping -->
    <section class="py-20 bg-white">
        <div class="container mx-auto px-6">
            <h2 class="font-cormorant text-4xl text-center mb-16">Aromatherapy Mood Mapping</h2>
            
            <div class="relative">
                <!-- Interactive mood map will be rendered here with JavaScript -->
                <div id="mood-map" class="aspect-video bg-soft-cream rounded-xl shadow-soft overflow-hidden relative">
                    <!-- This will be populated with JavaScript -->
                </div>
            </div>
        </div>
    </section>

    <!-- Ingredient Transparency -->
    <section class="py-20 bg-soft-cream">
        <div class="container mx-auto px-6">
            <h2 class="font-cormorant text-4xl text-center mb-6">Nature's Finest Ingredients</h2>
            <p class="text-center text-deep-brown/80 max-w-2xl mx-auto mb-16">We believe in complete transparency about what goes into our products. Explore our premium, ethically sourced ingredients.</p>
            
            <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
                <div class="bg-white rounded-xl shadow-soft p-8 transform transition-transform hover:-translate-y-2">
                    <div class="w-20 h-20 rounded-full bg-sage-green/20 flex items-center justify-center mb-6 mx-auto">
                        <svg class="w-10 h-10 text-sage-green" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 7v10a2 2 0 002 2h14a2 2 0 002-2V9a2 2 0 00-2-2h-6l-2-2H5a2 2 0 00-2 2z"></path></svg>
                    </div>
                    <h3 class="font-cormorant text-2xl text-center mb-4">Ethically Sourced</h3>
                    <p class="text-center text-deep-brown/80">We partner with small farms and cooperatives around the world to ensure fair compensation and sustainable practices.</p>
                </div>
                
                <div class="bg-white rounded-xl shadow-soft p-8 transform transition-transform hover:-translate-y-2">
                    <div class="w-20 h-20 rounded-full bg-amber-400/20 flex items-center justify-center mb-6 mx-auto">
                        <svg class="w-10 h-10 text-amber-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 21a4 4 0 01-4-4V5a2 2 0 012-2h4a2 2 0 012 2v12a4 4 0 01-4 4zm0 0h12a2 2 0 002-2v-4a2 2 0 00-2-2h-2.343M11 7.343l1.657-1.657a2 2 0 012.828 0l2.829 2.829a2 2 0 010 2.828l-8.486 8.485M7 17h.01"></path></svg>
                    </div>
                    <h3 class="font-cormorant text-2xl text-center mb-4">Pure & Natural</h3>
                    <p class="text-center text-deep-brown/80">We use only 100% pure, natural ingredients with no synthetic fragrances, preservatives, or additives.</p>
                </div>
                
                <div class="bg-white rounded-xl shadow-soft p-8 transform transition-transform hover:-translate-y-2">
                    <div class="w-20 h-20 rounded-full bg-blue-400/20 flex items-center justify-center mb-6 mx-auto">
                        <svg class="w-10 h-10 text-blue-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z"></path></svg>
                    </div>
                    <h3 class="font-cormorant text-2xl text-center mb-4">Scientifically Tested</h3>
                    <p class="text-center text-deep-brown/80">All our formulations are tested for efficacy and safety, ensuring they deliver the therapeutic benefits we promise.</p>
                </div>
            </div>
        </div>
    </section>

    <!-- Footer with newsletter -->
    <footer class="bg-deep-brown text-white py-16">
        <div class="container mx-auto px-6">
            <div class="grid grid-cols-1 md:grid-cols-4 gap-8">
                <div class="md:col-span-2">
                    <h3 class="font-cormorant text-2xl mb-6">The Scent</h3>
                    <p class="mb-6 text-white/80">Our mission is to promote mental and physical health through the power of natural aromatherapy.</p>
                    <form class="flex max-w-md">
                        <input type="email" placeholder="Your email address" class="flex-grow px-4 py-2 rounded-l-lg focus:outline-none text-deep-brown">
                        <button type="submit" class="bg-sage-green text-white px-4 py-2 rounded-r-lg hover:bg-sage-green/90 transition">Subscribe</button>
                    </form>
                </div>
                
                <div>
                    <h4 class="font-medium text-lg mb-4">Quick Links</h4>
                    <ul class="space-y-2 text-white/80">
                        <li><a href="#" class="hover:text-white transition">About Us</a></li>
                        <li><a href="#" class="hover:text-white transition">Products</a></li>
                        <li><a href="#" class="hover:text-white transition">Aromatherapy Guide</a></li>
                        <li><a href="#" class="hover:text-white transition">Contact</a></li>
                    </ul>
                </div>
                
                <div>
                    <h4 class="font-medium text-lg mb-4">Contact Us</h4>
                    <ul class="space-y-2 text-white/80">
                        <li class="flex items-center">
                            <svg class="w-4 h-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 8l7.89 5.26a2 2 0 002.22 0L21 8M5 19h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z"></path></svg>
                            <a href="mailto:info@thescent.com">info@thescent.com</a>
                        </li>
                        <li class="flex items-center">
                            <svg class="w-4 h-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 5a2 2 0 012-2h3.28a1 1 0 01.948.684l1.498 4.493a1 1 0 01-.502 1.21l-2.257 1.13a11.042 11.042 0 005.516 5.516l1.13-2.257a1 1 0 011.21-.502l4.493 1.498a1 1 0 01.684.949V19a2 2 0 01-2 2h-1C9.716 21 3 14.284 3 6V5z"></path></svg>
                            <a href="tel:+18001234567">+1 (800) 123-4567</a>
                        </li>
                    </ul>
                    
                    <h4 class="font-medium text-lg mt-6 mb-4">Follow Us</h4>
                    <div class="flex space-x-4">
                        <a href="#" class="text-white/80 hover:text-white transition">
                            <svg class="w-6 h-6" fill="currentColor" viewBox="0 0 24 24"><path d="M9 8h-3v4h3v12h5v-12h3.642l.358-4h-4v-1.667c0-.955.192-1.333 1.115-1.333h2.885v-5h-3.808c-3.596 0-5.192 1.583-5.192 4.615v3.385z"></path></svg>
                        </a>
                        <a href="#" class="text-white/80 hover:text-white transition">
                            <svg class="w-6 h-6" fill="currentColor" viewBox="0 0 24 24"><path d="M12 2.163c3.204 0 3.584.012 4.85.07 3.252.148 4.771 1.691 4.919 4.919.058 1.265.069 1.645.069 4.849 0 3.205-.012 3.584-.069 4.849-.149 3.225-1.664 4.771-4.919 4.919-1.266.058-1.644.07-4.85.07-3.204 0-3.584-.012-4.849-.07-3.26-.149-4.771-1.699-4.919-4.92-.058-1.265-.07-1.644-.07-4.849 0-3.204.013-3.583.07-4.849.149-3.227 1.664-4.771 4.919-4.919 1.266-.057 1.645-.069 4.849-.069zm0-2.163c-3.259 0-3.667.014-4.947.072-4.358.2-6.78 2.618-6.98 6.98-.059 1.281-.073 1.689-.073 4.948 0 3.259.014 3.668.072 4.948.2 4.358 2.618 6.78 6.98 6.98 1.281.058 1.689.072 4.948.072 3.259 0 3.668-.014 4.948-.072 4.354-.2 6.782-2.618 6.979-6.98.059-1.28.073-1.689.073-4.948 0-3.259-.014-3.667-.072-4.947-.196-4.354-2.617-6.78-6.979-6.98-1.281-.059-1.69-.073-4.949-.073zm0 5.838c-3.403 0-6.162 2.759-6.162 6.162s2.759 6.163 6.162 6.163 6.162-2.759 6.162-6.163c0-3.403-2.759-6.162-6.162-6.162zm0 10.162c-2.209 0-4-1.79-4-4 0-2.209 1.791-4 4-4s4 1.791 4 4c0 2.21-1.791 4-4 4zm6.406-11.845c-.796 0-1.441.645-1.441 1.44s.645 1.44 1.441 1.44c.795 0 1.439-.645 1.439-1.44s-.644-1.44-1.439-1.44z"></path></svg>
                        </a>
                        <a href="#" class="text-white/80 hover:text-white transition">
                            <svg class="w-6 h-6" fill="currentColor" viewBox="0 0 24 24"><path d="M8.29 20.251c7.547 0 11.675-6.253 11.675-11.675 0-.178 0-.355-.012-.53A8.348 8.348 0 0022 5.92a8.19 8.19 0 01-2.357.646 4.118 4.118 0 001.804-2.27 8.224 8.224 0 01-2.605.996 4.107 4.107 0 00-6.993 3.743 11.65 11.65 0 01-8.457-4.287 4.106 4.106 0 001.27 5.477A4.072 4.072 0 012.8 9.713v.052a4.105 4.105 0 003.292 4.022 4.095 4.095 0 01-1.853.07 4.108 4.108 0 003.834 2.85A8.233 8.233 0 012 18.407a11.616 11.616 0 006.29 1.84"></path></svg>
                        </a>
                    </div>
                </div>
            </div>
            
            <div class="border-t border-white/20 mt-12 pt-8 text-center text-white/60">
                <p>&copy; 2025 The Scent. All rights reserved.</p>
            </div>
        </div>
    </footer>

    <!-- JavaScript -->
    <script src="{{ asset('js/app.js') }}"></script>
</body>
</html>
```

## Main JavaScript Functionality

```javascript
// public/js/app.js

// Initialize on page load
document.addEventListener('DOMContentLoaded', function() {
    // Initialize scent finder
    initScentFinder();
    
    // Initialize mood map
    initMoodMap();
    
    // Initialize product interactions
    initProductInteractions();
    
    // Initialize animations
    initAnimations();
});

// Scent Finder Functionality
function initScentFinder() {
    const scentFinder = document.getElementById('scent-finder');
    if (!scentFinder) return;
    
    const steps = scentFinder.querySelectorAll('.scent-step');
    const nextBtn = document.getElementById('next-step');
    const prevBtn = document.getElementById('prev-step');
    let currentStep = 1;
    let userSelections = {};
    
    // Option selection handling
    scentFinder.querySelectorAll('.scent-option').forEach(option => {
        option.addEventListener('click', function() {
            const step = this.closest('.scent-step');
            const stepNumber = parseInt(step.dataset.step);
            const value = this.dataset.value;
            
            // Store selection
            userSelections[`step${stepNumber}`] = value;
            
            // Mark as selected
            step.querySelectorAll('.scent-option').forEach(opt => {
                opt.classList.remove('border-sage-green', 'bg-sage-green/10');
            });
            this.classList.add('border-sage-green', 'bg-sage-green/10');
            
            // Enable next button
            nextBtn.disabled = false;
        });
    });
    
    // Next button handling
    nextBtn.addEventListener('click', function() {
        if (currentStep < steps.length) {
            steps[currentStep - 1].classList.remove('active');
            steps[currentStep].classList.add('active');
            currentStep++;
            
            // Show prev button after first step
            if (currentStep > 1) {
                prevBtn.classList.remove('hidden');
            }
            
            // Change next to 'See results' on last step
            if (currentStep === steps.length) {
                nextBtn.textContent = 'See Your Results';
            }
        } else {
            // Show results
            showScentResults(userSelections);
        }
    });
    
    // Previous button handling
    prevBtn.addEventListener('click', function() {
        if (currentStep > 1) {
            steps[currentStep - 1].classList.remove('active');
            currentStep--;
            steps[currentStep - 1].classList.add('active');
            
            // Hide prev on first step
            if (currentStep === 1) {
                prevBtn.classList.add('hidden');
            }
            
            // Change back to 'Continue' if not on last step
            if (currentStep < steps.length) {
                nextBtn.textContent = 'Continue';
            }
        }
    });
}

// Mood Map Visualization
function initMoodMap() {
    const moodMap = document.getElementById('mood-map');
    if (!moodMap) return;
    
    // This would be an interactive visualization - simplified version here
    const moods = [
        { id: 1, name: 'Relaxation', x: 30, y: 70, color: '#8BAD99' },
        { id: 2, name: 'Energy', x: 70, y: 30, color: '#F8B84E' },
        { id: 3, name: 'Focus', x: 80, y: 80, color: '#5E9ED6' },
        { id: 4, name: 'Sleep', x: 20, y: 40, color: '#786FA6' },
        { id: 5, name: 'Stress Relief', x: 50, y: 60, color: '#A0C3D2' }
    ];
    
    // Create the mood nodes
    moods.forEach(mood => {
        const node = document.createElement('div');
        node.className = 'absolute transform -translate-x-1/2 -translate-y-1/2 transition-all duration-500 ease-in-out';
        node.style.left = `${mood.x}%`;
        node.style.top = `${mood.y}%`;
        node.innerHTML = `
            <div class="mood-node cursor-pointer" data-mood-id="${mood.id}">
                <div class="w-20 h-20 rounded-full flex items-center justify-center text-white transition-transform hover:scale-110"
                     style="background-color: ${mood.color}">
                    <span class="font-medium">${mood.name}</span>
                </div>
            </div>
        `;
        moodMap.appendChild(node);
        
        // Add click event to show related products
        node.querySelector('.mood-node').addEventListener('click', function() {
            const moodId = this.dataset.moodId;
            fetchMoodProducts(moodId);
        });
    });
    
    // Add connecting lines (this would be more sophisticated in a real implementation)
    const svg = document.createElementNS('http://www.w3.org/2000/svg', 'svg');
    svg.setAttribute('class', 'absolute inset-0 w-full h-full');
    svg.innerHTML = `
        <path d="M ${moods[0].x}% ${moods[0].y}% L ${moods[2].x}% ${moods[2].y}% L ${moods[1].x}% ${moods[1].y}% L ${moods[3].x}% ${moods[3].y}% Z" 
              stroke="#E5E7EB" stroke-width="1" fill="none" />
        <path d="M ${moods[4].x}% ${moods[4].y}% L ${moods[0].x}% ${moods[0].y}%" 
              stroke="#E5E7EB" stroke-width="1" fill="none" />
        <path d="M ${moods[4].x}% ${moods[4].y}% L ${moods[3].x}% ${moods[3].y}%" 
              stroke="#E5E7EB" stroke-width="1" fill="none" />
    `;
    moodMap.insertBefore(svg, moodMap.firstChild);
}

// Additional functions for product interactions, animations, and API calls would be here
```

## CSS Styling (app.css)

```css
/* Base styles */
:root {
    --color-deep-brown: #4A3B31;
    --color-soft-cream: #F9F5F0;
    --color-sage-green: #8BAD99;
    --color-amber: #F8B84E;
    --color-blue: #5E9ED6;
}

.font-cormorant {
    font-family: 'Cormorant Garamond', serif;
}

.font-montserrat {
    font-family: 'Montserrat', sans-serif;
}

.bg-soft-cream {
    background-color: var(--color-soft-cream);
}

.bg-sage-green {
    background-color: var(--color-sage-green);
}

.text-deep-brown {
    color: var(--color-deep-brown);
}

.text-sage-green {
    color: var(--color-sage-green);
}

/* Buttons */
.btn-primary {
    @apply inline-block px-6 py-3 bg-sage-green text-white rounded-lg font-medium transition-colors hover:bg-sage-green/90;
}

.btn-outline {
    @apply inline-block px-6 py-3 border border-sage-green text-sage-green rounded-lg font-medium transition-colors hover:bg-sage-green/5;
}

.btn-light-sm {
    @apply inline-block px-4 py-2 bg-white text-deep-brown rounded-lg text-sm font-medium transition-colors hover:bg-white/90;
}

.btn-icon {
    @apply p-2 rounded-full bg-white text-deep-brown hover:bg-white/90 transition-colors;
}

/* Animations */
.animate-fade-in {
    animation: fadeIn 0.8s ease forwards;
}

.animation-delay-300 {
    animation-delay: 300ms;
}

.animation-delay-600 {
    animation-delay: 600ms;
}

@keyframes fadeIn {
    from {
        opacity: 0;
        transform: translateY(20px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

/* Card effects */
.product-card {
    @apply transition-transform duration-300 hover:-translate-y-2;
}

/* Shadow effects */
.shadow-soft {
    box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.05), 0 8px 10px -6px rgba(0, 0, 0, 0.025);
}

/* Scent finder */
.scent-step {
    display: none;
}

.scent-step.active {
    display: block;
}
```

## Key "Wow Effect" Features

1. **Interactive Scent Profile Finder**: Personalized quiz that helps customers discover products matching their preferences, moods, and needs

2. **Aromatherapy Mood Mapping**: Visual interactive map showing connections between moods and scents, allowing users to explore products based on emotional benefits

3. **Product Visualization**: High-quality product imagery with smooth hover animations and interactive elements

4. **Ingredient Transparency System**: Visual breakdown of natural ingredients with details on their benefits and origins

5. **Personalized "Aromatherapy Journey"**: Custom recommendations based on previous purchases and preferences

6. **Dynamic Product Detail Pages**: Each product features scent intensity visualizations, mood benefits, and detailed information about therapeutic properties

7. **Seasonal Collections**: Curated product collections that change with seasons and target specific seasonal wellness needs

8. **Custom Blend Builder**: Allow customers to create their own custom essential oil blends based on aromatherapy principles

Would you like me to explain or elaborate on any particular aspect of this solution?

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nordeim)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nordeim)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
