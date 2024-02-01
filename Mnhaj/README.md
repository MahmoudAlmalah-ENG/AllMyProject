# Mnhaj Academy - Laravel Project

<p align="center">
        <img src="image/logo.png" alt="logo" style="background-color: #ffffff; color: #ffffff; padding: 5px 10px; border-radius: 5px; text-decoration: none;" width="200">
</p>

<p align="center">
    <a href="https://mnhajakadmy.online/">
        <img src="https://img.shields.io/badge/website-mnhajakadmy.online-blue" alt="website">
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
- [Database Schema](#database-schema)

## Project Overview

The Educational Platform is a web application built with Laravel, providing a platform for the distribution of courses
across different academic disciplines. The project aims to facilitate the sharing of educational content and enhance the
learning experience for students.

## Tools Used

- **PHP**: ^8.1
- **Blade UI Kit/Blade Icons**: ^1.5
- **Blade UI Kit/Blade UI Kit**: ^0.4.0
- **Blade UI Kit/Blade Zondicons**: ^1.4
- **Guzzle HTTP**: ^7.2
- **Laravel Framework**: ^10.10
- **Laravel Sanctum**: ^3.2
- **Laravel Telescope**: ^4.16
- **Laravel Tinker**: ^2.8
- **Laravel UI**: ^4.2
- **Livewire**: ^3.0
- **mcamara/laravel-localization**: ^1.8
- **pbmedia/laravel-ffmpeg**: ^8.3

## Code Editor

- **PHPStorm**: [https://www.jetbrains.com/phpstorm/](https://www.jetbrains.com/phpstorm/)

## Project Idea

The project aims to create an educational platform using Laravel, where users can access and publish courses for
different academic subjects. This platform will provide an interactive and user-friendly interface for both educators
and learners.

## Application Description

The application is a web platform that allows users to access and publish courses for different academic subjects. The
platform provides an interactive and user-friendly interface for both educators and learners.

## Developer Information

- **Developer**: Mahmoud Almalah
- **Contact**: +201026475912

## Build Timeline

- **WebSite**: 7 Days

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

4. **Generate Application Key
   **[AllCourseController.php](..%2F..%2F..%2F..%2F..%2Fmedia%2Fmahmoud%2FNew%20Volume%2FProject%2FLaravel%2FVISOFT%2Flearn%2Fapp%2FHttp%2FControllers%2FCourses%2FAllCourseController.php)

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

### AllCourseController.php

```php
<?php
    public function index()
    {
        return view('courses.all', [
            'courses' => Course::whereStatus(true)
                ->has('sections.lessons')
                ->latest()
                ->paginate($this->paginate)
        ]);
    }

    public function show(Course $course)
    {
        return view('courses.show', [
            'course' => $course,
            'similarCourses' => Course::whereStatus(true)
                ->where('id', '<>', $course->id)
                ->whereRelation('category', 'id', $course->category->id)
                ->has('sections.lessons')
                ->latest()
                ->take(8)
                ->get()
        ]);
    }
?>
```

### Course.php

```php
final class Course extends Model
{
    use HasFactory;

    const TABLE = 'courses';

    protected $table = self::TABLE;

    protected $with = ['category', 'sections'];

    protected $fillable = [
        'title',
        'lecturer',
        'price',
        'description',
        'cat_id',
        'user_id',
        'image',
        'slug',
        'status'
    ];

    protected $casts = [
        'cat_id' => 'integer',
        'user_id' => 'integer',
        'status' => 'boolean'
    ];
}
```

## Database Schema

![Database Schema](image/database.png)
