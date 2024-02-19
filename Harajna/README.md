# Harajna - Laravel Project

<p align="center">
        <img src="image/logo.png" alt="logo" width="200" height="200">
</p>

<p align="center">
    <a href="https://play.google.com/store/apps/details?id=com.visooft.harajna">
        <img src="https://play.google.com/intl/en_us/badges/static/images/badges/en_badge_web_generic.png" alt="Google Play" width="200" height="80">
    </a>
    <a href="https://apps.apple.com/us/app/harajna/id6472904170">
        <img src="https://developer.apple.com/assets/elements/badges/download-on-the-app-store.svg" alt="App Store" width="160" height="80">
    </a>
</p>

## Table of Contents

- [Project Overview](#project-overview)
- [Tools Used](#tools-used)
- [Code Editor](#code-editor)
- [Project Idea](#project-idea)
- [Application Description](#application-description)
- [Developer Information](#developer-information)
- [Build Timeline](#build-timeline)
- [Getting Started](#getting-started)
- [Contact](#contact)
- [Example Code](#example-code)

## Project Overview

Harajna is a mobile application that allows users to buy and sell products in a safe and easy way.
The application provides a platform for users to create and manage their own stores,
and to buy and sell products from other users.

## Tools Used

- **PHP**: ^8.1
- **Laravel Framework**: ^9.19
- **Laravel Sanctum**: ^3.2
- **Laravel Telescope**: ^4.16
- **Livewire**: ^3.0
- **mcamara/laravel-localization**: ^1.8

## Code Editor

- **PHPStorm**: [https://www.jetbrains.com/phpstorm/](https://www.jetbrains.com/phpstorm/)

## Project Idea

The project aims to create a mobile application using Laravel,
where users can buy and sell products in a safe and easy way.
The application will provide a platform for users to create and manage their own stores, and to buy and sell products
from other users.

## Application Description

The application is a mobile platform that allows users to buy and sell products in a safe and easy way.
The application provides a platform for users to create and manage their own stores,
and to buy and sell products from other users.

## Developer Information

- **Developer**: Mahmoud Almalah
- **Contact**: +201026475912

## Build Timeline

- **Admin**: 7 Days
- **API**: 5 Days

## Getting Started

1. **Clone the Repository**

    ```bash
    git clone https://github.com/*/*.git
    ```

2. **Install Dependencies**

    ```bash
    composer install
    ```

3. **Copy Environment File**

    ```bash
    cp .env.example .env
    ```

4. **Generate Application Key**

    ```bash
    php artisan key:generate
    ```

5. **Configure Database**

   Update the database configuration in the `.env` file with your database credentials.

6. **Run Migrations and Seed Database**

    ```bash
    php artisan migrate --seed
    ```

7. **Start the Development Server**

    ```bash
    php artisan serve
    ```

   The application will be accessible at [http://127.0.0.1:8000](http://127.0.0.1:8000).

## Contact

For inquiries or support, please contact Mahmoud Almalah at +201026475912.

Thank you for choosing AskMe! Happy coding!

## Example Code

### ADSController.php

```php
<?php
    protected function Ads()
    {
        return $this->ADS->query()
            ->with(['category', 'user', 'city', 'license', 'sub_Category'])
            ->withExists(['favorites as favorite' => function ($query) {
                $query->where('user_id', auth()->id());
            }])
            ->active()
            ->withoutUserBlock();
    }

    public function index()
    {
        try {
            $ads = $this->Ads()
                ->latest()
                ->paginate(15);

            return $this->success(message: 'تم جلب البيانات', data: ADSResources::collection($ads));
        } catch (Exception $e) {
            return $this->error(message: 'حدث خطأ ما', data: $e->getMessage());
        }
    }
    
    public function filterLocation(Request $request)
    {
        try {
            $ads = $this->Ads()
                ->when(value: $request->get('cat_id'), callback: function ($query) use ($request) {
                    return $query->where(['category_id' => $request->get('cat_id')]);
                })
                ->whereBetween(column: 'lat', values: [$request->get('lat') - 0.5, $request->get('lat') + 0.5])
                ->whereBetween(column: 'long', values: [$request->get('long') - 0.5, $request->get('long') + 0.5])
                ->latest()
                ->paginate(15);

            return $this->success(message: 'تم جلب البيانات', data: ADSResources::collection($ads));
        } catch (Exception $e) {
            return $this->error(message: 'حدث خطأ ما', data: $e->getMessage());
        }
    }
?>
```

### ADSResource.php

```php
<?php
class ADSResources extends JsonResource
{
    public function toArray($request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'slug' => $this->slug,
            'description' => $this->description,
            'price' => $this->price,
            'city' => $this->city->name,
            'lat' => $this->lat,
            'long' => $this->long,
            'phone' => $this->phone ?? $this->user->phone,
            'status' => $this->state ? __('api.state.active') : __('api.state.inactive'),
            'image' => array_map(function ($image) {
                return asset($image);
            }, $this->image),
            'user' => $this->whenLoaded('user', function () {
                return [
                    'id' => $this->user->id,
                    'name' => $this->user->name,
                    'image' => asset($this->user->avatar),
                    'fcm_token' => $this->user?->fcm_token,
                ];
            }),
            'category' => $this->whenLoaded('category', function () {
                return [
                    'id' => $this->category->id,
                    'name' => $this->category->name,
                ];
            }),
            'sub_category' => $this->whenLoaded('sub_Category', function () {
                return [
                    'id' => $this->sub_Category->id,
                    'name' => $this->sub_Category->name,
                ];
            }),
            'created_at' => $this->created_at->format('d-m-Y'),
            'time' => $this->created_at->format('H:i'),
            'from_now' => $this->created_at->diffForHumans(),
            'favorite' => $this->favorite, // 1 => favorite, 0 => not favorite
            'license' => $this->whenLoaded('license', fn() => $this->license),
        ];
    }
}
?>
```

### ADS.php

```php
<?php
final class ADS extends Model
{
    use HasFactory;

    protected $table = 'a_d_s';

    protected $fillable = [
        'id',
        'category_id',
        'user_id',
        'city_id',
        'title',
        'slug',
        'description',
        'price',
        'image',
        'state',
        'lat',
        'long',
        'phone',
        'sub_category_id',
    ];

    protected $hidden = [
        'updated_at',
    ];

    protected $attributes = [
        'state' => 1,
    ];

    protected $casts = [
        'id' => 'integer',
        'category_id' => 'integer',
        'user_id' => 'integer',
        'city_id' => 'integer',
        'price' => 'float',
        'image' => 'array',
        'state' => 'boolean',
        'lat' => 'float',
        'long' => 'float',
        'phone' => 'string',
        'sub_category_id' => 'integer',
        'created_at' => 'datetime:Y-m-d H:i:s',
    ];
}
?>
```


