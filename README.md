# AskMe - Laravel Project

<p align="center">
        <img src="image/logo.png" alt="logo" width="200" height="200">
</p>

<p align="center">
    <a href="https://play.google.com/store/apps/details?id=com.asos.asalny">
        <img src="https://play.google.com/intl/en_us/badges/static/images/badges/en_badge_web_generic.png" alt="Google Play" width="200" height="80">
    </a>
    <a href="#">
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
- [Database Schema](#database-schema)
- [File Structure](#file-structure)

## Project Overview

Welcome to AskMe, a Laravel-based application that allows users to ask questions and receive responses based on
real-world experiences. The application introduces a unique concept where users can get quick responses by subscribing
to a premium service. The project leverages various tools and technologies to create a seamless and interactive
platform.

## Tools Used

- **PHP**: ^8.1
- **Blade UI Kit/Blade Icons**: ^1.5
- **Blade UI Kit/Blade UI Kit**: ^0.4.0
- **Blade UI Kit/Blade Zondicons**: ^1.4
- **Guzzle HTTP**: ^7.2
- **Kreait/Laravel-Firebase**: ^5.4
- **Laravel Framework**: ^10.10
- **Laravel Sanctum**: ^3.3
- **Laravel Telescope**: ^4.16
- **Laravel Tinker**: ^2.8
- **Livewire**: ^3.0
- **Power Components/Livewire Powergrid**: ^5.1
- **Spatie/Laravel-Permission**: ^6.0

## Code Editor

- **PHPStorm**: [Download PHPStorm](https://www.jetbrains.com/phpstorm/)

## Project Idea

AskMe is a platform that allows users to post questions and receive answers from others in the same city. For users in a
hurry, a subscription model is available, allowing them to pay a fee for quicker responses. The application includes
features for selecting a city, categorizing questions, adding images, and two options for posting questions: instant
post or subscription-based post.

## Application Description

If you have a question or inquiry, you can enter the AskMe application, write your question, and people from the same
city will respond based on their experiences. If you need a quick response, you can pay a certain amount, for example,
$10, for a 5-hour priority subscription for your question. Users can post questions by choosing the city and category,
typing the question, and uploading images if needed. There are two options: choosing to subscribe for instant question
prioritization or posting a regular question.

## Developer Information

- **Developer**: Mahmoud Almalah
- **Contact**: +201026475912

## Build Timeline

- **API**: 2 days
- **Admin Panel**: 4 days

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

   Access the application at [http://127.0.0.1:8000](http://127.0.0.1:8000).

## Contact

For inquiries or support, please contact Mahmoud Almalah at +201026475912.

Thank you for choosing AskMe! Happy coding!

## Example Code

### API Routes

```php
<?php
Route::group([
    'controller' => 'EndpointController',
    'prefix' => 'endpoints',
    'as' => 'endpoints.',
    'namespace' => 'Endpoints',
], function () {
    Route::get('splash', 'splash')->name('splash');
    Route::get('country', 'country')->name('country');
    Route::get('city', 'city')->name('city');
    Route::get('city_show/{country}', 'cityShow')->name('city_show');
    Route::get('category', 'category')->name('category');
    Route::get('guidelines', 'guidelines')->name('guidelines');
    Route::get('nationality', 'nationality')->name('nationality');
    Route::get('year', 'year')->name('year');
    Route::get('privacy', 'privacy')->name('privacy');
    Route::get('setting', 'settings')->name('settings');
});
```

### WEB Routes

```php
<?php
Route::group([
    'prefix' => 'guidelines',
    'as' => 'guidelines.',
    'middleware' => ['can:guidelines'],
], function () {
    Route::view('/', 'admin.guidelines.index')->name('index');
});
```

### API Controller

```php
<?php
 public function register(RegisterRequest $registerRequest)
    {
        try {
            // Create a new user with the validated data from the request
            $user = User::create($registerRequest->validated());
            // Load the 'nationality', 'city', and 'country' relationships for the user
            $user->load(['nationality', 'city', 'country']);

            // Assign the 'user' role to the user
            $user->assignRole(roles: 'user');

            // Return a success response with a message, the user data, and a 201 status code
            return $this->success(__('auth.register'), new UserResource($user), 201);
        } catch (\Exception $e) {
            // Log the error and return an error response with a 500 status code
            $this->logError(file: __FILE__, message: $e->getMessage(), function: __FUNCTION__, line: $e->getLine(), trace: $e->getTraceAsString());

            return $this->error(message: __('http-statuses.500'));
        }
    }
?>
```

### Query Builder Example

```php
<?php
    public static function getPaymentQuestions(int $city_id = null, int $cat_id = null, int $perPage = null): LengthAwarePaginator
    {
        // Start a new query
        return self::query()
            // Apply category filter if a category ID is provided
            ->when($cat_id, function ($query, $cat_id) {
                return $query->where('category_id', $cat_id);
            })
            // Filter for questions that are of type 'PAID' and status 'ACTIVE'
            ->where(['questions.type' => QuestionsState::PAID, 'questions.status' => QuestionsState::ACTIVE])
            // Eager load related 'plan' and 'questionLocation' data from 'questionPlans'
            ->with(['questionPlans.plan', 'questionPlans.questionLocation'])
            // Join with 'question_plans' table to fetch related data
            ->leftJoin('question_plans', function ($join) {
                $join->on('question_plans.question_id', '=', 'questions.id')
                    ->where('question_plans.status', PaymentState::SUCCESS)
                    ->where('question_plans.expired_at', '>=', now());
            })
            // Join with 'plans' table to fetch related data
            ->leftJoin('plans', 'plans.id', '=', 'question_plans.plan_id')
            // Join with 'question_locations' table to fetch related data
            ->leftJoin('question_locations', 'question_locations.question_id', '=', 'questions.id')
            // Apply city filter if a city ID is provided
            ->where(function ($query) use ($city_id) {
                $query->where(function ($query) use ($city_id) {
                    $query->where('plans.type', PlansStats::ThisCity)
                        ->whereHas('questionLocations', function ($query) use ($city_id) {
                            $query->where('city_id', $city_id);
                        });
                })
                    ->orWhere(function ($query) use ($city_id) {
                        $query->where('plans.type', PlansStats::AllCityInCountry)
                            ->whereRelation('questionLocations', 'country_id', CountryCity::where('city_id', $city_id)->first()->country_id);
                    })
                    ->orWhere('plans.type', PlansStats::AllCountry);
            })
            // Order the results by the latest Modified At date
            ->latest()
            // Paginate the results
            ->paginate($perPage ?? self::perPage);
    }
?>
```

### Model Example

```php
<?php
final class User extends Authenticatable
{
    use HasApiTokens, HasFactory, HasRoles, Notifiable, SoftDeletes, UserHelper;

    const TABLE = 'users';

    protected $table = self::TABLE;

    protected $primaryKey = 'id';

    protected $fillable = [
        'name',
        'phone',
        'email',
        'birthday',
        'gender',
        'status',
        'nationality_id',
        'city_id',
        'country_id',
        'last_login',
        'password',
        'fcm_token',
    ];

    protected $attributes = [
        'status' => UserState::ACTIVE,
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected $casts = [
        'status' => 'integer',
        'email_verified_at' => 'datetime',
        'password' => 'hashed',
        'created_at' => 'datetime',
        'birthday' => 'string',
    ];
}
?>
```

### Livewire Component Example

```php
<?php
final class AddCategoryForm extends Livewire
{
 #[Rule('required|string|max:255|unique:categories,name')]
    public $name;

    const HIDE_ADD_MODAL = 'hideAddCategoryModal';

    const REFRESH_TABLE = 'refreshTable';

    public function hydrate(): void
    {
        $this->resetErrorBag();
        $this->resetValidation();
    }

    public function store(): void
    {
        $this->validate();

        try {
            Category::create([
                'name' => $this->name,
            ]);

            $this->dispatch(self::REFRESH_TABLE);
            $this->dispatch(self::HIDE_ADD_MODAL);

            $this->reset();

            $this->alertByEvent('success', __('http-statuses.200'), __('admin.:name created_successfully', ['name' => __('admin.category')]));
        } catch (\Exception $e) {
            $this->alertByEvent('error', __('http-statuses.500'), $e->getMessage());
        }
    }
}
 ?>
```

## Database Schema

![Database Schema](image/asalny.png)

## File Structure

```bash
.
├── app
│   ├── Enum
│   │   ├── AnswerStatus.php
│   │   ├── QuestionStatus.php
│   │   └── Assets.php
│   │   └── NotificationState.php
│   │   └── PaymentState.php
│   │   └── UserState.php
│   ├── Helpers
│   │   ├── Helper.php
│   ├── Http
│   │   ├── Controllers
│   │   │   ├── API
│   │   │   │   ├──V1
│   │   │   │   │   ├── Auth
│   │   │   │   │   │   ├── AuthenticationController.php
│   │   │   │   │   ├── Question
│   │   │   │   │   │   ├── QuestionController.php
│   │   │   │   │   ├── Answers
│   │   │   │   │   │   ├── AnswerController.php
│   │   │   │   │   ├── Notification
│   │   │   │   │   │   ├── NotificationController.php
│   │   │   │   │   ├── Plans
│   │   │   │   │   │   ├── PlanController.php
│   │   │   │   │   ├── Actions
│   │   │   │   │   │   ├── ActionController.php
│   │   │   │   │   ├── QuestionPlan
│   │   │   │   │   │   ├── QuestionPlanController.php
│   │   │   │   │   ├── Endpoints
│   │   │   │   │   │   ├── EndpointController.php
│   │   │   │   │   ├── User
│   │   │   │   │   │   ├── UserController.php
│   │   │   ├── WEB
│   │   │   │   ├── Admin 
│   │   │   │   │   ├── Auth
│   │   │   │   │   │   ├── AuthenticationController.php
│   │   │   │   │   ├── Blocks
│   │   │   │   │   │   ├── BlockController.php
│   │   │   │   │   ├── Country
│   │   │   │   │   │   ├── CountryController.php
│   │   │   │   │   ├── Home
│   │   │   │   │   │   ├── HomeController.php
│   │   │   │   │   ├── Questions
│   │   │   │   │   │   ├── QuestionController.php
│   │   │   │   │   ├── Roles
│   │   │   │   │   │   ├── RoleController.php
│   │   │   │   │   ├── Users
│   │   │   │   │   │   ├── UserController.php
│   │   │   │   │   ├── Years
│   │   │   │   │   │   ├── YearController.php
│   │   │   │   |── User
│   │   │   │   │   ├── Share
│   │   │   │   │   │   ├── ShareController.php
│   │   ├── Controllers
│   │   │   ├── Controller.php
│   │   ├── Middleware
│   │   │   ├── AdminMiddleware.php
│   │   │   ├── AuthGardMiddleware.php
│   │   │   ├── TrustCodeMiddleware.php
│   │   ├── Requests
│   │   │   ├── API
│   │   │   │   ├── V1
│   │   │   │   │   ├── Auth
│   │   │   │   │   │   ├── ForgotPasswordRequest.php
│   │   │   │   │   │   ├── LoginRequest.php
│   │   │   │   │   │   ├── RegisterRequest.php
│   │   │   │   │   │   ├── ResetPasswordRequest.php
│   │   │   │   │   ├── Question
│   │   │   │   │   │   ├── QuestionRequest.php
│   │   │   │   │   ├── Answers
│   │   │   │   │   │   ├── AnswerRequest.php
│   │   │   │   │   ├── Profile
│   │   │   │   │   │   ├── ProfileRequest.php
│   │   │   │   │   ├── QuestionPlan
│   │   │   │   │   │   ├── QuestionPlanRequest.php
│   │   │   ├── API 
│   │   │   │   ├── Request.php
│   │   ├── Resources
│   │   │   ├── API
│   │   │   │   ├── V1
│   │   │   │   │   ├── Answer
│   │   │   │   │   │   ├── AnswerResource.php
│   │   │   │   │   ├── Auth
│   │   │   │   │   │   ├── AuthResource.php
│   │   │   │   │   ├── Block
│   │   │   │   │   │   ├── BlockResource.php
│   │   │   │   │   ├── Endpoints
│   │   │   │   │   │   ├── EndpointResource.php
│   │   │   │   │   ├── Home
│   │   │   │   │   │   ├── HomeResource.php
│   │   │   │   │   ├── Notification
│   │   │   │   │   │   ├── NotificationResource.php
│   │   │   │   │   ├── Plans
│   │   │   │   │   │   ├── PlanResource.php
│   │   │   │   │   ├── Privacy
│   │   │   │   │   │   ├── PrivacyResource.php
│   │   │   │   │   ├── QuestionPlan
│   │   │   │   │   │   ├── QuestionPlanResource.php
│   │   │   │   │   ├── Questions
│   │   │   │   │   │   ├── QuestionResource.php
│   │   │   │   │   ├── Ticket
│   │   │   │   │   │   ├── TicketResource.php
│   ├── Models
│   │   ├── Answer.php
│   │   ├── Block.php
│   │   ├── Category.php
│   │   ├── City.php
│   │   ├── Country.php
│   │   ├── CountryCity.php
│   │   ├── Favorite.php
│   │   ├── Guidelines.php
│   │   ├── Image.php
│   │   ├── Like.php
│   │   ├── Nationality.php
│   │   ├── NickName.php
│   │   ├── Notifications.php
│   │   ├── Otp.php
│   │   ├── Plans.php
│   │   ├── Privacy.php
│   │   ├── Question.php
│   │   ├── QuestionLocation.php
│   │   ├── QuestionPlan.php
│   │   ├── Report.php
│   │   ├── Slider.php
│   │   ├── Splash.php
│   │   ├── Ticket.php
│   │   ├── User.php
│   │   ├── Year.php
│   ├── Observers
│   │   ├── AnswerObserver.php
│   │   ├── CategoryObserver.php
│   │   ├── CityObserver.php
│   │   ├── CountryCityObserver.php
│   │   ├── CountryObserver.php
│   │   ├── GuidelinesObserver.php
│   │   ├── ImageObserver.php
│   │   ├── NationalitiesObserver.php
│   │   ├── QuestionObserver.php
│   │   ├── ReportObserver.php
│   │   ├── SliderObserver.php
│   │   ├── SplashObserver.php
│   │   ├── YearObserver.php
│   ├── Traits
│   │   ├── API
│   │   │   ├── V1
│   │   │   │   ├── FileManage.php
│   │   │   │   ├── ModelHelper.php
│   │   │   │   ├── Response.php
│   │   │   │   ├── UserHelper.php
│   │   ├── WEB
│   │   │   ├── V1
│   │   │   │   ├── Alert.php
│   │   │   │   ├── FirebaseMessaging.php
│   ├── Livewire
│   │   ├──All Models CRUD Livewire Components
.
