# Square - Laravel Project

<p align="center">
   <img src="image/logo.png" alt="logo" width="200" height="200">
</p>

<p align="center"> 
   <a href="http://square.visooft.com/public/docs">
       <img src="https://img.shields.io/badge/Documentation-View%20API%20Documentation-blue" alt="API Documentation">
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

Square is a real estate application that allows users to buy, sell, and rent real estate.

## Tools Used

- **PHP**: ^8.1
- **Laravel Framework**: ^10.10
- **Blade UI Kit/Blade Icons**: ^1.5
- **Blade UI Kit/Blade UI Kit**: ^0.4.0
- **Blade UI Kit/Blade Zondicons**: ^1.4
- **Guzzle HTTP**: ^7.2
- **Kreait/Laravel-Firebase**: ^5.4
- **livewire/livewire**: ^3.0,
- **opcodesio/log-viewer**: ^3.1,
- **power-components/livewire-powergrid**: ^5.1,
- **spatie/laravel-permission**: ^6.0,
- **laravel/pulse**: ^1.0*

## Code Editor

- **PHPStorm**: [https://www.jetbrains.com/phpstorm/](https://www.jetbrains.com/phpstorm/)

## Project Idea

## Application Description

## Developer Information

- **Developer**: Mahmoud Almalah
- **Contact**: +201026475912

## Build Timeline

- **Admin**: 8 Days
- **API**: 6 Days

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

### PropertyController.php

```php
<?php
    public function index()
    {
        $property = $this->property::query()
            ->latest()
            ->active()
            ->notShare()
            ->paginate(perPage: $this->perPage);

        return $this->success(message: __(key: 'http-statuses.200'), data: PropertyResource::collection(resource: $property));
    }
    
    public function store(StoreADS $storeADS)
    {
        try {
            $property = $this->property::create(
                array_merge($storeADS->validated(),
                    ['user_id' => auth()->id(), 'slug' => Str::slug(title: auth()->user()->code . '-' . rand())]
                )
            );

            $this->handleImages(storeADS: $storeADS, property: $property);
            $this->handlePropertyData(storeADS: $storeADS, property: $property, key: PropertyInfos::QUESTIONS);
            $this->handlePropertyData(storeADS: $storeADS, property: $property, key: PropertyInfos::TOOLS);

            return $this->success(message: __(key: 'api.your_ads_in_review'));
        } catch (\Exception $e) {
            $this->logError(file: __FILE__, message: $e->getMessage(), function: __FUNCTION__);

            return $this->error(message: __(key: 'http-statuses.500'));
        }
    }
    
    private function handleImages(StoreADS $storeADS, Property $property)
    {
        if ($storeADS->hasFile(key: 'first_image')) {
            $file = $this->uploadFile(file: $storeADS->file(key: 'first_image'), path: Asset::PROPERTY_IMAGE, id: $property->id);
            $property->first_image = $file;
            $property->save();
        }

        if ($storeADS->hasFile(key: 'images')) {
            foreach ($storeADS->file(key: 'images') as $item) {
                $file = $this->uploadFile(file: $item, path: Asset::PROPERTY_IMAGE, id: $property->id);
                $this->ImageCreate(path: $file, id: $property->id);
            }
        }
    }
    
    private function handlePropertyData(StoreADS $storeADS, Property $property, string $key)
    {
        if ($storeADS->has(key: $key)) {
            $items = is_array(value: $storeADS->get(key: $key)) ? $storeADS->get(key: $key) : explode(separator: ',', string: $storeADS->get(key: $key));
            foreach ($items as $item) {
                $property->propertyData()->create(attributes: [
                    'property_id' => $property->id,
                    'property_info_id' => is_array(value: $item) ? $item['id'] : $item,
                    'value' => is_array(value: $item) ? $item['value'] : null,
                    'key' => $key,
                ]);
            }
        }
    }
?>
```

### PropertyResource.php

```php
<?php

class PropertyResource extends JsonResource
{
    public function toArray($request): array
    {
        $info = $this->propertyData;
        $user = $this->user;

        return [
            'id' => $this->id,
            'city' => $this->city?->name,
            'area' => $this->area?->name,
            'category' => $this->category?->name,
            'title' => $this->title,
            'type' => PropertiesState::getLabelType($this->type),
            'type_status' => $this->type_status,
            'price' => number_format($this->price),
            'age' => $this->age,
            'lat' => $this->lat,
            'lng' => $this->lng,
            'description' => $this->description,
            'space' => $this->space,
            'payment' => $this->payment,
            'first_image' => $this->ImageUrl(),
            'user' => $this->whenLoaded('user', function () use ($user) {
                return [
                    'id' => $user->id,
                    'name' => $user?->full_name,
                    'phone' => $user?->phone,
                    'email' => $user?->email,
                    'image' => $user?->imageUrl(),
                ];
            }),
            'property' => [
                'tools' => $info->where('key', PropertyInfos::TOOLS)->map(function ($item) {
                    return [
                        'id' => $item->property_info?->id,
                        'title' => $item->property_info?->title,
                    ];
                })->toArray(),
                'questions' => $info->where('key', PropertyInfos::QUESTIONS)->map(function ($item) {
                    return [
                        'id' => $item->property_info?->id,
                        'title' => str_replace('عدد', '', $item->property_info?->title),
                        'value' => $item->value,
                    ];
                })->toArray(),
            ],
            'share' => $this->share,
            'images' => new ImagesCollection($this->images),
            'status' => PropertiesState::getLabelStatus($this->status),
            'code' => $this->slug,
            'views' => formatNumber((string)$this->views),
            'created_at' => $this->created_at->diffForHumans(),
        ];
    }
}
?>
```

### ADS.php

```php
final class Property extends Model
{
    use HasFactory;

    const TABLE = 'properties';

    protected $table = self::TABLE;

    protected $primaryKey = 'id';

    protected $with = [
        'user',
        'city',
        'area',
        'images',
        'propertyData',
        'category',
        'propertyData.property_info'
    ];

    protected $fillable = [
        'user_id',
        'city_id',
        'area_id',
        'cat_id',
        'title',
        'type',
        'price',
        'age',
        'lat',
        'lng',
        'description',
        'first_image',
        'status',
        'views',
        'slug',
        'space',
        'payment',
        'type_status',
        'share',
    ];

    protected $attributes = [
        'status' => PropertiesState::APPROVED,
        'views' => 0,
        'share' => 0,
    ];

    public function ImageUrl(): string
    {
        return asset(path: Storage::url(Asset::PROPERTY_IMAGE . $this->first_image));
    }
}
?>
```


