# Laravel Tips

Daily Laravel/PHP tips I share on my [X](https://x.com/OussamaMater) and [LinkedIn](https://www.linkedin.com/in/oussamamater/). For more in-depth articles about Laravel, feel free to check out my [blog](https://blog.oussama-mater.tech/).

## Tip #1 💡: Use rebinding events to refresh dependencies

When a bound instance in the Laravel container is rebound, a rebinding event is triggered. You can listen to this event to ensure that the classes using the instance stay up to date. You can achieve this by using the rebinding method or simply using the refresh shortcut 🚀

```php
<?php

namespace App\Providers;

use App\Services\PaymentService;
use Illuminate\Support\ServiceProvider;
use Illuminate\Contracts\Foundation\Application;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->singleton('payment.service', function (Application $app) {
            $paymentService = new PaymentService();

            $paymentService->setTaxService($app['tax.service']);

            // Will set a new TaxService instance for the Payment Service
            $app->refresh('tax.service', $paymentService, 'setTaxService');

            return $paymentService;
        });
    }
}
```

## Tip #2 💡: Get Original Attributes

Laravel accessors allow you to transform your model attributes when retrieving them. But sometimes, you may wish to get the original value. Well, Laravel provides a method just for that: `getRawOriginal` 🚀

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Casts\Attribute;

class User extends Model
{
    protected function username(): Attribute
    {
        return Attribute::make(
            function (string $value, array $attributes) {
                return sprintf('%s#%s', $value, $attributes['app_id']);
            }
        );
    }
}

$user = User::create([
    'username' => 'oussama',
]);

$user->getOriginal('username'); // oussama#1234
$user->getRawOriginal('username'); // oussama
```

## Tip #3 💡: Model Binding in Form Requests

Route Model Binding allows you to inject the model instances directly into your routes. It is usually used within a controller, but did you know you could also access the model instance in your Form Request?

```php
<?php

// Route definition
Route::put('/post/{post}', [PostsController::class, 'create']);

  
// Form Request defintion
class PostController extends FormRequest
{
    public function authorize(): bool
    {
        // The post instance, if found
        return $this->user()->can('create', $this->route('post'));
    }

    public function rules(): array
    {
        return [
            // ...
        ];
    }
}
```

## Tip #4 💡: Mapping Exceptions via Handler

Did you know that you can map exceptions to other exceptions? This is helpful when you have custom exceptions that you'd like to be handled like internal ones, or when you want to wrap third-party exceptions and map them to your own 🚀

```php
<?php

namespace App\Exceptions;

use App\Exceptions\MyCustomException;
use Illuminate\Database\Eloquent\ModelNotFoundException;
use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;

class Handler extends ExceptionHandler
{
    // MyCustomException is now handled as if it were ModelNotFoundException
    protected $exceptionMap = [
        MyCustomException::class => ModelNotFoundException::class,
    ];
}
```

## Tip #5 💡: Mapping Exception via Custom Exception Classes

When defining a custom exception, there is a secret way to map the exception to another one, and this is done by defining the `getInnerException` method 🚀

Curious about how the mapping is done? Take a look at `Illuminate\Foundation\Exceptions\Handler::mapException()`

```php
<?php

namespace App\Exceptions;

use Exception;
use Throwable;
use Illuminate\Database\Eloquent\ModelNotFoundException;

class MyCustomException extends Exception
{
    // MyCustomException is now handled as if it were ModelNotFoundException
    public function getInnerException(): Throwable
    {
        return new ModelNotFoundException;
    }
}
```

## Tip #6 💡: Route Absolute and Relative Path

In Laravel, the route() helper is used to generate paths for named routes; by default, it generates the absolute path. Did you know that if you pass False as your third parameter, it will generate a relative path? Now you do 🚀

```php
<?php

route('tips', $tip->id); // https://oussama-mater.tech/tips/1

route('tips', $tip->id, false); // /tips/1
```

## Tip #7 💡: Dynamic Wheres

Did you know that Laravel allows you to define dynamic "where" conditions? For example, you could do, `whereNameAndAge(name_value, age_value)` 🤯

Make sure to add the method name to your model's PHPDoc so your IDE does not complain, that's a bit too much magic for it to understand.

Curious about how it's done? Take a look at `Illuminate\Database\Query\Builder::dynamicWhere()`

```php
<?php

// select * from `users` where `name` = 'oussama' and `last_name` = 'mater'"
User::whereNameAndLastName('oussama', 'mater')->first();
```

## Tip #8 💡: Specify Custom Factories Path

Laravel automatically resolves model factories if they exist in the Database\\Factories namespace. Sometimes you wish to move them. For example, if you're using Domain-Driven Design (DDD), you may want each domain to have its own factories. In such cases, you can instruct Laravel on how to resolve the factory by defining a newFactory method in your model. By doing so, you can perform additional logic too! For example return different factories for different environments.

```php
<?php

namespace App\Models;

use Database\Factories\UserFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    // ...

    protected static function newFactory()
    {
        // ... custom logic
        return new UserFactory();
    }
}
```

## Tip #9 💡: Quietly update your models

When updating your Laravel models, you always trigger "Model Events," which are hooks enabling you to perform extra actions. You can disable this behavior by updating them quietly 🤫

```php
<?php

$user = Auth::user();

$user->name = 'Oussama Mater';

// Won't trigger Model Events
$user->saveQuietly();
$user->deleteQuietly();
$user->forceDeleteQuietly();
$user->restoreQuietly();
```

## Tip #10 💡: Eager Loading with Specific Columns

Did you know that when using eager loading with relationships, you may specify the exact columns you need? This will decrease memory usage 🚀

```php
<?php

$users = Post::with('author:id,name')->get();
```

## Tip #11 💡: `is()` **method in Laravel**

Did you know you can check whether or not two models are the same by using the `is()` helper? 🚀

```php
<?php

$user = User::find(1);  
$sameUser = User::find(1);  
$differentUser = User::find(2);

$user->is($sameUser); // true  
$user->is($differentUser); // false
```

## Tip #12 💡: A shorter "whereHas"

While Laravel's `whereHas` is excellent for retrieving records based on a specified relationship along with additional query constraints, there's a shortcut called "whereRelation" that accomplishes the same task 🚀

```php
<?php

// Before
User::whereHas('comments', function ($query) {
    $query->where('created_at', '>', now()->subDay());
})->get();

// After
User::whereRelation('comments', 'created_at', '>', now()->subDay())->get();
```

## Tip #13 💡: Dispatch After Response

At times, certain tasks, like sending emails, don't necessarily need to be queued and processed by a worker. In such cases, you can make use of "dispatchAfterResponse()." True to its name, this method dispatches the job right after the server responds to the user. A quick response for the client without burdening the worker with minor tasks 🚀

```php
<?php

use App\Jobs\SendNotification;

SendNotification::dispatchAfterResponse();
```

## Tip #14 💡: Find Related IDs on a BelongsToMany Relationship

Did you know that Laravel ships with the 'allRelatedId()' method to help you fetch all IDs for a belongsToMany relationship? Now you do 🚀

```php
<?php

class User extends Model
{
    public function roles()
    {
        return $this->belongsToMany(Role::class);
    }
}

$user = User::find(1);

$roleIds = $user->roles()->pluck('id')->toArray();
$roleIds = $user->roles()->allRelatedIds()->toArray();
```

## Tip #15 💡: Get All Executed Queries

Did you know you can listen to all executed queries in Laravel? This is not only useful for quick debugging; for instance, you can send a Slack notification if the query is slower than expected🚀

```php
<?php

DB::listen(function (QueryExecuted $query) {
    dump($query->sql); // select * from `users` where `users`.`id` = ? limit 1
    dump($query->bindings); // [0 => 1]
    dump($query->time); // 6.05
});
```

## Tip #16 💡: Customizing Missing Model Behavior

Did you know that Laravel provides the "missing()" method to customize the default route model binding behavior when a model is not found? 🚀

```php
<?php

use App\Http\Controllers\LocationsController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;

Route::get('/locations/{location:slug}', [LocationsController::class, 'show'])
    ->name('locations.view')
    ->missing(function (Request $request) {
        return Redirect::route('locations.index');
    });
```

## Tip #17 💡: isset()

Did you know you could pass multiple arguments to the isset() function?

```php
<?php

if(isset($var1) && isset($var2)) { ... }
if(isset($var1, $var2)) { ... }
```

## Tip #18 💡: The "withToken()" method

Did you know that the Laravel HTTP Client comes with a fluent method called "withToken()" that you can use to set bearer tokens? 🚀

```php
<?php

// The following code
Http::withHeaders([
    'Authorization' => 'Bearer eyJhbGciOiJIUz...',
]);

// Is equivalent to this
Http::withToken('eyJhbGciOiJIUz...');
```

## Tip #19 💡: Much Cooler Command Output

It is true that Laravel provides methods like "info()" and "error()" to send output to the console, but since Laravel 9, the core itself started using "termwind", which you can see when seeding your database or in case of an error, and you can leverage that as well by using the "components" attributes.

```php
<?php

/**
 * Execute the console command.
 */
public function handle()
{
    // A standard error message
    $this->error('The old fashioned way.');

    // A much cooler error message
    $this->components->error('What Laravel uses internally.');

    // And more
    $this->components->info('Hello, World!');
    $this->components->warn('Hello, World!');
    $this->components->alert('Hello, World!');
    // choice, confirm, ask, warn, task, line, bulletList ..

    return Command::SUCCESS;
}
```

## Tip #20 💡: The "Prunable" trait

Did you know that Laravel comes with a Prunable trait to permanently remove records, including the soft-deleted ones, based on a condition you define? 🚀

```php
<?php

// Define the pruning condition
class User extends Authenticatable
{
    use Prunable;

    public function prunable()
    {
        return static::query()
            ->whereNull('email_verified_at')
            ->where('created_at', '<', now()->subMonths(6));
    }
}

// Schedule the pruning command to run daily for example
$schedule->command(PruneCommand::class)->daily();
```

## Tip #21 💡: Prevent Stray Requests

Did you know that Laravel ships with the "preventStrayRequests()" method to avoid making actual requests during testing? This is handy not only for third-party APIs but also for local APIs. While your tests may pass locally because the entire environment is up and running, they could fail in the CI pipeline. This happens because, in the CI pipeline, typically only the service you are testing is active, so making actual requests to unavailable APIs will cause your suite to fail.

```php
<?php

use Illuminate\Support\Facades\Http;

// Define in your setUp()
Http::preventStrayRequests();

// Now in your tests
Http::fake([
    'example.com/*' => Http::response('ok'),
]);

// An "ok" response is returned ...
Http::get('https://example.com/users');

// An exception is thrown ...
Http::get('https://oussama-mater.tech');
```

## Tip #22 💡: The "whereBelongsTo" method

Did you know that Laravel ships with a "whereBelongsTo" to get the parent model? This will make the code much more readable 🚀

```php
<?php

// This
$posts = Post::where('user_id', $user->id)->get();
// Or this is okay
$posts = Post::whereUserId($user->id)->get();

// But this is more readable
$posts = Post::whereBelongsTo($user)->get();
```

## Tip #23 💡: The "squish" method

I'm sure you've trimmed a string at least once. While the built-in "trim" function that PHP provides is good, it may not be sufficient if you want to eliminate extra spaces between text. In such cases, give the "squish" method a go 🚀

```php
<?php

use Illuminate\Support\Str;

$joke = Str::squish(' PHP is dead 🤣 ');

// PHP is dead 🤣 
```

## Tip #24 💡: The "whenQueryingForLongerThan" method

Did you know that you can use "whenQueryingForLongerThan" to monitor slow queries? You can set the threshold in milliseconds. If a query exceeds the threshold, you can send notifications or grab a coffee with the mastermind behind the query 😂

```php
<?php

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        DB::whenQueryingForLongerThan(500, function (Connection $connection, QueryExecuted $event) {
            // Mastermind behind this query, let's grab a coffee ..
        });
    }
}
```

## Tip #25 💡: The Scheduler's "skip" Method

Sometimes, you might want to skip executing a command based on a specific condition. Laravel includes the "skip" method to accomplish exactly that 🚀

```php
<?php

$schedule->command('emails:send')->daily()->skip(function () {
    return Carbon::today()->isHoliday();
});
```

## Tip #26 💡: Faker Formatters

Since Laravel uses FakerPHP for generating fake data, you can employ both, "numerify" and "bothify", to generate data in a specific pattern 🚀

```php
<?php

$faker = Faker\Factory::create();

// 2067-7317-8584-7068
$creditCardNumber = $faker->numerify('####-####-####-####');

// TRK-cwq-546-shx
$trackingNumber = $faker->bothify('TRK-???-###-???');
```

## Tip #27 💡: The "foreignIdFor" Method

When defining foreign IDs, Laravel offers multiple methods, one of which is "foreignIdFor()". This method snake cases the model name and appends "id" to it. Not only does it make your code more readable, but you can quickly navigate to the model from the migration 🚀

```php
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('posts', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->foreignIdFor(User::class); // user_id
    $table->timestamps();
});
```

## Tip #28 💡: Careful with "whereYear()"

Be careful when using `whereYear`; even if your column is indexed, it won't be used, and the database will perform a full table scan. Instead, opt for using ranges 🚀

```php
<?php

DB::table('user_posts')
    ->select('user_id', DB::raw('COUNT(*) as count'))
    ->whereYear('created_at', 2023)
    ->groupBy('user_id')
    ->get();
// select `user_id`, COUNT(*) as count from `user_posts` where year(`created_at`) = ? group by `user_id`

DB::table('user_posts')
    ->select('user_id', DB::raw('COUNT(*) as count'))
    ->whereBetween('created_at', ['2023-01-01 00:00:00', '2024-01-01 00:00:00'])
    ->groupBy('user_id')
    ->get();
// select `user_id`, COUNT(*) as count from `user_posts` where `created_at` between ? and ? group by `user_id`
```

## Tip #29 💡: The "withCount" Method

Did you know that you can use "withCount" to get the count of a related relationship without loading it? This is really useful, for example, when displaying statistics 🚀

```php
<?php

$users = User::withCount(['posts'])->get();

// $users->posts_count
```

## Tip #30 💡: Inline Validation

While Laravel offers a plethora of validation rules, there are times when you need custom ones. These are usually written in a custom class. But, did you know you can perform inline validation also? 🚀

```php
<?php

request()->validate([
    'email' => [
        'required',
        'email',
        function ($attribute, $value, $fail) {
            if (substr($value, -12) !== '@example.com') {
                $fail($attribute, 'The email must belong to example.com domain');
            }
        },
    ],
]);
```

## Tip #31 💡: The "json" method

If you are using Laravel 10 and upwards, there is an elegant way to read JSON files by using "File::json()". You can also pass the flags that you would normally pass to "json_decode()", in case you want to throw an exception 🚀

```php
<?php

// Laravel < 10
$data = json_decode(File::get('data.json'), flags: JSON_THROW_ON_ERROR);

// Laravel >= 10
$data = File::json('data.json', JSON_THROW_ON_ERROR);
```

## Tip #32 💡: Custom Route Model Binding Resolution

Laravel's route model binding is a powerful feature that automatically injects your model into your controller. By default, it uses the ID to fetch the instance. If you wish, you can implement custom logic for resolution by defining "resolveRouteBinding" 🚀

```php
<?php

// In your model
class Post extends Model
{
    public function resolveRouteBinding($value, $field = null)
    {
        return $this
            ->where('slug', $value)
            ->orWhere('uuid', $value)
            ->firstOrFail();
    }
}

/*
Now both these will work
/post/resolve-route-binding-is-cool
/post/638fd97b-4a96-469e-b497-9e553f13c8ef
*/
Route::get('/post/{post}', fn (Post $post) => $post);
```

## Tip #33 💡: The "rescue" Helper

Sometimes we are forced to use a try-catch block just to ignore an expected exception that an external service throws upon failure. Laravel provides a more elegant solution for such scenarios through the "rescue" helper  🚀

```php
<?php

// If an exception is thrown, we default to "en"
try {
    $language = LanguageDetector::detect($request->ip());
} catch (Exception $e) {
    $language = 'en';
}

// A cleaner way to achieve the same, and the exception will still be reported
$language = rescue(fn () => LanguageDetector::detect($request->ip()), 'en');
```

## Tip #34 💡: The "toQuery" method

Did you know that Laravel ships with a method called "toQuery"? This method allows you to update a collection using a single query statement by running a "whereIn" 🚀

```php
<?php

$users = User::where('status', 'VIP')->get();

// Instead of this
foreach ($users as $user) {
    $user->update(['status' => 'Administrator']);
}

// Do this instead
$users->toQuery()->update(['status' => 'Administrator']);
```

## Tip #35 💡: Preview Mailables

When working with mailables, we often send them to MailHog or Mailtrap to quickly preview the rendered email. Did you know that Laravel allows you to preview emails in the browser as if they were regular Blade files? 🚀

```php
<?php

Route::get('/mailable', function () {
    $invoice = App\Models\Invoice::find(1);

    return new App\Mail\InvoicePaid($invoice);
});
```

## Tip #36 💡: The "whenTableDoesntHaveColumn" and "whenTableHasColumn" methods

Laravel 9 and onward ship with 2 schema methods, 'whenTableDoesntHaveColumn' and 'whenTableHasColumn', which allow you to drop or create a column if it exists or not. This is really helpful when you have multiple environments where schemas can get out of sync quickly.

```php
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

// Will create the email column only if it does not exist
Schema::whenTableDoesntHaveColumn('users', 'email', function(Blueprint $table){
    $table->string('email')->unique();
});

// Will drop the email column only if it exists
Schema::whenTableHasColumn('users', 'email', function(Blueprint $table){
    $table->dropColumn('email');
});
```

## Tip #37 💡: Report Internal Exceptions

Sometimes, you just wish to report internal exceptions. For instance, if you want to know exactly where that 404 error is being thrown, you can do so by overriding the `internalDontReport` array 🚀

```php
<?php

namespace App\Exceptions;

use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;

// The ModelNotFoundException exception will now be reported
class Handler extends ExceptionHandler
{
    protected $internalDontReport = [
        AuthenticationException::class,
        AuthorizationException::class,
        BackedEnumCaseNotFoundException::class,
        HttpException::class,
        HttpResponseException::class,
        //ModelNotFoundException::class,
        MultipleRecordsFoundException::class,
        RecordsNotFoundException::class,
        SuspiciousOperationException::class,
        TokenMismatchException::class,
        ValidationException::class,
    ];
}
```

## Tip #38 💡: Validate Nested Arrays Easily

Sometimes, when validating nested arrays, you may have custom rules that require access to the value of the item being validated. Laravel 9 and onwards comes with 'forEach,' enabling you to do just that 🚀

```php
<?php

use App\Rules\HasPermission;
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

$validator = Validator::make($request->all(), [
    'companies.*.id' => Rule::forEach(function (string|null $value, string $attribute) {
        return [
            Rule::exists(Company::class, 'id'),
            new HasPermission('manage-company', $value),
        ];
    }),
]);
```

## Tip #39 💡: The "withoutTimestamps" method

Did you know that if you want to update a model without modifying its `updated_at` timestamp, you can use the 'withoutTimestamps' method? 🚀

```php
<?php

// The updated_at column will not be updated
Post::withoutTimestamps(fn () => $post->increment(['reads']));
```

## Tip #40 💡: Random Ordering

Did you know that Laravel comes with the "inRandomOrder" method, which sorts query results randomly? 🚀

```php
$randomUser = DB::table('users')
->inRandomOrder()
->first();
```

## Tip #41 💡: On Command Result

Did you know that Laravel allows you to define callbacks to be executed based on the result of a scheduled task? This helps log failures or execute related actions on success 🚀

```php
<?php

use Illuminate\Console\Scheduling\Schedule;

$schedule->command('emails:send')
            ->daily()
            ->onSuccess(function () {
                // The task succeeded...
               })
            ->onFailure(function () {
                 // The task failed...
            });
```

## Tip #42 💡: Delete a Job When Models Are Missing

Eloquent models injected into jobs are auto-serialized for the queue and may trigger a `ModelNotFoundException` if deleted while waiting for a worker; you can quietly discard such jobs by setting `deleteWhenMissingModels` to true 🚀

```php
<?php

use Illuminate\Contracts\Queue\ShouldQueue;

class UpdateSearchIndex implements ShouldQueue
{
    /**
     * Delete the job if its models no longer exist.
     *
     * @var bool
     */
    public $deleteWhenMissingModels = true;
}
```

## Tip #43 💡: Touching Relationships

Laravel automatically updates "updated_at" in many-to-many relationships, and it also ships with "setTouchedRelations" method to manually update related models in one-to-one and one-to-many relationships 🚀

```php
<?php

$user = User::firstOrFail();
$user->setTouchedRelations(['posts']);

// The 'updated_at' of all related posts will be updated
$user->save();
```

## Tip #44 💡: Recursively Saving Models and Relationships

Did you know that Laravel ships with the "push" method, allowing you to save models and all related relationships recursively, without having to go over them one by one? 🚀

```php
<?php

$post = Post::find(1);

$post->comments[0]->message = 'Message';
$post->comments[0]->author->name = 'Author Name';

// Instead of these
$post->comments[0]->author->save();
$post->comments[0]->save();

// You can do this
$post->push();
```

## Tip #45 💡: The "saveMany" method

Did you know that Laravel allows you to save multiple related models at once by using the 'saveMany' method? 🚀

```php
<?php

$post = Post::find(1);

$post->comments()->saveMany([
    new Comment(['message' => 'A new comment.']),
    new Comment(['message' => 'Another new comment.']),
]);
```

## Tip #46 💡: Query JSON Fields

Did you know that Laravel allows you to query JSON fields in databases that support JSON column types? 🚀

```php
<?php

// Will return all users where the preferences.dining.meal field is equal to 'salad'
DB::table('users')
    ->where('preferences→dining→meal', 'salad')
    ->get();

// Will return all users where the languages array contains 'en' and 'de'
DB::table('users')
    ->whereJsonContains('options→languages', ['en', 'de'])
    ->get();

// Will return all users where the languages array has more than one element
DB::table('users')
    ->whereJsonLength('options→languages', '>', 1)
    ->get();
```

## Tip #47 💡: The "toBase()" Method

Sometimes, you may need to load a large amount of data, but you don't need the hydrated models. In these scenarios, you can use the "toBase()" method provided by Laravel to save on memory usage 🚀

```php
<?php

// This collection will consist of PHP objects
// and will not include hydrated models
$users = User::toBase()->get();
```

## Tip #48 💡: The "value()" Method
  
Sometimes, you only need a single value instead of the entire row. Laravel comes with the "value()" method, which return the value of the column directly 🚀

```php
<?php

$email = DB::table('users')->where('name', 'John')->value('email');

dd($email); // john@example.com
```

## Tip #49 💡: Group Resource Controllers

Did you know that you can group resource controllers using the "resources()" method? It makes routing even cleaner! 🚀

```php
<?php

use App\Http\Controllers\PostsController;
use App\Http\Controllers\PhotosController;

// Instead of this
Route::resource('posts', PostsController::class);
Route::resource('photos', PhotosController::class);

// Do this
Route::resources([
    'posts' => PostsController::class,
    'photos' => PhotosController::class,
]);
```

## Tip #50 💡: The "whereAll" and "whereAny" Methods

Laravel v10.47.0 has just been released, featuring four new methods: "whereAll," "whereAny," "orWhereAll," and "orWhereAny." These methods allow you to compare a value against multiple columns 🚀

```php
<?php

$search = 'ous%';

// Instead of this
User::query()
    ->where(function($query) use ($search) {
        $query
            ->where('first_name', 'LIKE', $search)
            ->where('last_name', 'LIKE', $search);
    })
    ->get();

// You can now do this
User::query()
    ->whereAll(['first_name', 'last_name'], 'LIKE', $search)
    ->get();

User::query()
    ->whereAny(['first_name', 'last_name'], 'LIKE', $search)
    ->get();

// Which results in the following queries

// select * from `users` where (`first_name` LIKE 'ous%' and `last_name` LIKE 'ous%')
// select * from `users` where (`first_name` LIKE 'ous%' or `last_name` LIKE 'ous%')

// You can also use "orWhereAll" and "orWhereAny".
```

## Tip #51 💡: Command Input Auto-Completion

When building console commands, you can improve the user experience by implementing auto-completion for the user. This can be done using the "anticipate" method provided by Laravel 🚀

```php
<?php

// You can use arrays 
$animal = $this->anticipate("What's your favourite animal?", ['dogs', 'cats']);

// Or run a closure, triggered every time a user types a character
$animal = $this->anticipate("What's your favourite animal?", function (string $input) {
    return Animal::query()
                ->where('name', 'LIKE', "$input%")
                ->pluck('name')
                ->toArray();
});
```

## Tip #52 💡: Mail Command Output

Did you know that the Laravel scheduler allows you to email the output of a command to an email address of your choosing? 🚀

```php
<?php

$schedule->command(SendEmailsCommand::class)
    ->daily()
    ->emailOutputTo('oussama@example.com');

// You can also use `emailOutputOnFailure`.
```

## Tip #53 💡: Hook into Authentication Events

Did you know that the Laravel authentication component comes with multiple events you can listen to? Whether a user attempts to log in, or fails to do so, you can handle these events as you wish 🚀

```php
<?php

protected $listen = [
    Illuminate\Auth\Events\Registered::class => [],
    Illuminate\Auth\Events\Attempting::class => [],
    Illuminate\Auth\Events\Authenticated::class => [],
    Illuminate\Auth\Events\Login::class => [],
    Illuminate\Auth\Events\Failed::class => [],
    Illuminate\Auth\Events\Validated::class => [],
    Illuminate\Auth\Events\Verified::class => [],
    Illuminate\Auth\Events\Logout::class => [],
    Illuminate\Auth\Events\CurrentDeviceLogout::class => [],
    Illuminate\Auth\Events\OtherDeviceLogout::class => [],
    Illuminate\Auth\Events\Lockout::class => [],
    Illuminate\Auth\Events\PasswordReset::class => [],
];
```

## Tip #54 💡: The "scan" Helper

Did you know that you can use the "scan" helper to parse a string input into a collection according to a format supported by the built-in sscanf PHP function? 🚀

```php
<?php

$rgb = '#402A2A';

[$r, $g, $b] = str($rgb)->scan('#%2x%2x%2x');

// $r = 64, $g = 42, $b = 42
```

## Tip #55 💡: Hide Console Commands

Did you know that you can conditionally hide console commands? This is useful when you are building packages and you want a command to be executed only once 🚀

```php
<?php

namespace Package\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Artisan;
use Package\PackageServiceProvider;

class Install extends Command
{
    protected $signature = 'package:install';

    public function __construct()
    {
        parent::__construct();

        if (file_exists(config_path('package-config.php'))) {
            $this->setHidden();
        }
    }

    public function handle()
    {
        Artisan::call('vendor:publish', ['--provider' => PackageServiceProvider::class]);
        
        $this->info('Package was installed successfully 🎉');
    }
}
```

## Tip #56 💡: Optional Faker Values

If you have optional columns in your table that you want to seed randomly, FakerPHP (what Laravel uses under the hood) allows for optional values out of the box 🚀

```php
<?php

fake()->optional()->randomDigit(); // a random digit, but also null sometimes

fake()->optional($weight = 0.1)->randomDigit(); // 90% chance of NULL
fake()->optional($weight = 0.9)->randomDigit(); // 10% chance of NULL

fake()->optional($weight = 10)->randomDigit; // 90% chance of NULL
fake()->optional($weight = 100)->randomDigit; // 0% chance of NULL

fake()->optional($weight = 0.5, $default = false)->randomDigit(); // 50% chance of FALSE
fake()->optional($weight = 0.9, $default = 'abc')->word(); // 10% chance of 'abc'
```

## Tip #57 💡: Chain Scheduled Commands

You can chain commands in your scheduler using the `then()` method, an undocumented feature that I often find myself using 🚀

```php
<?php

$schedule->command('db:backup')
            ->daily()
            ->then(function(){
                $this->command('notify:slack');
            });
```

## Tip #58 💡: The New "once" Helper

Laravel 11 has introduced a new helper called "once," which computes a result once and caches it. Subsequent calls will return the previously cached result. You can use this instead of static properties for better code readability! 🚀

```php
<?php

function random(): int
{
    return once(function () {
        return random_int(1, 1000);
    });
}

random(); // 123
random(); // 123 (cached result)
random(); // 123 (cached result)
```

## Tip #59 💡: Test Queue/Job Interactions

Laravel 11 has introduced a new way of testing failed, released or deleted jobs, which was a challenging in previous versions! You can now simply chain "withFakeQueueInteractions" on your job and assert one of the mentioned actions 🚀

```php
<?php

use App\Jobs\ProcessPodcast;

$job = (new ProcessPodcast)->withFakeQueueInteractions();

$job->handle();

$job->assertReleased(delay: 30);
$job->assertDeleted();
$job->assertFailed();
```

## Tip #60 💡: Factory Sequences

Did you know that Laravel allows you to define sequences when using factories? This makes setting up complex tests a breeze 🚀

```php
<?php

$users = User::factory()
    ->count(2)
    ->sequence(
        ['name' => 'First User'],
        ['name' => 'Second User'],
    )
    ->create();
```

## Tip #61 💡: Limit Eager Loaded Relationships

In Laravel versions 10 and below, we couldn't limit eager loaded relationships natively. Well, guess what? In Laravel 11, we can! 🚀

```php
<?php

User::with([
    'posts' => fn ($query) => $query->limit(5)
])->paginate();
```

## Tip #62 💡: Retry Concurrent Requests

In Laravel versions 10 and below, retrying failed concurrent requests wasn't possible. Well, guess what? In Laravel 11, we can! 🚀

```php
<?php

use Illuminate\Http\Client\Pool;
use Illuminate\Support\Facades\Http;

$responses = Http::pool(fn (Pool $pool) => [
    $pool->as('first')->retry(2)->get('http://localhost/first'),
    $pool->as('second')->retry(2)->get('http://localhost/second'),
    $pool->as('third')->retry(2)->get('http://localhost/third'),
]);
```

## Tip #63 💡: Define Casts as a Method

In Laravel versions 10 and below, we had to define casts as properties which made it a bit messy to pass arguments. In Laravel 11, we can define casts as a method! 🚀

```php
<?php

namespace App\Models;

use App\Casts\Json;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
   // Laravel ≤ 10 😔
   protected $casts = [
       'statuses' => AsEnumCollection::class.':'.ServerStatus::class,
   ];

   // Laravel 11 😎
   protected function casts(): array
   {
       return [
           'statuses' => AsEnumCollection::of(ServerStatus::class),
       ];
   }
}
```

## Tip #64 💡: The "withExists" Method

Did you know that Laravel ships with a method called `withExists` which allows you to check if a model has a relationship or not? This is really helpful for conditional logics 🚀

```php
<?php

$user = User::query()
    ->withExists('posts as is_author')
    ->get();

/*
    select
        `users`.*,
        exists (select * from `posts` where `users`.`id` = `posts`.`user_id`) as `is_author`
    from `users`
*/

$user->is_author; // Will be either true or false.
```

## Tip #65 💡: The "whereKey" Method

Did you know that Laravel ships with the "whereKey" method? It makes your "where in" statements more readable, and well, you don't have to remember the name of the primary key 🚀

```php
<?php

// 😕 Instead of doing this
Post::whereIn('id', [1,2,3])->get();
Post::whereNotIn('id', [1,2,3])->get();

// 😎 You can do this
Post::whereKey([1,2,3])->get();
Post::whereKeyNot([1,2,3])->get();
```

## Tip #66 💡: Avoid Columns Ambiguity

I'm sure we've all encountered column name ambiguity when building queries at least once. To avoid that, you can use the "qualifyColumn" method, which prefixes the column with the table name 🚀

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Team extends Model
{
    use HasFactory;

    protected $fillable = [
        'name',
    ];

    public function servers(): HasMany
    {
        // Use qualifyColumn() to avoid collision
        // with the 'name' column on the Team model
        return $this->hasMany(Server::class)->orderBy(
            (new Server)->qualifyColumn('name')
        );
    }
}
```

## Tip #67 💡: The "throw_if" and "throw_unless" Helpers

Did you know that Laravel ships with two helpers, "throw_if" and "throw_unless," which not only make your code shorter but also much more readable? 🚀

```php
<?php

// This is okay
if (!Auth::user()->isAdmin()) {
    throw new AuthorizationException;
}

// This reads better
// Throw the exception if they're not the admin.
throw_if(!Auth::user()->isAdmin(), AuthorizationException::class);

// Throw the exception unless they're an admin.
throw_unless(Auth::user()->isAdmin(), AuthorizationException::class);
```

## Tip #68 💡: The "blank" and "filled" Helpers

Did you know that Laravel ships with two cool helpers, "blank" and "filled"? You can now have a standardized way to test if a variable is empty or not, regardless of its type. Even collections are supported! 🚀

```php
<?php

// Will return true
blank('');
blank('    ');
blank(null);
blank(collect());
blank([]);

// Will return false
blank(0);
blank('string');
blank(true);
blank(false);
blank([1]);

// For the inverse, you can make use of filled()
```

## Tip #69 💡: The "Lottery" Class

Did you know that Laravel comes with a Lottery class that allows you to execute callbacks based on odds? I found this to be extremely helpful, especially when implementing A/B tests.

```php
<?php
use Carbon\CarbonInterval;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Lottery;

// 1 out of every 100 slow queries will be reported.
DB::whenQueryingForLongerThan(
    CarbonInterval::seconds(2),
    Lottery::odds(1, 100)->winner(fn () => report('Querying > 2 seconds.')),
);
```

## Tip #70 💡: Extend the "PersonalAccessToken" model

Sanctum is a powerful package for managing API tokens. Sometimes you may wish the token model had more methods. Well, guess what? You can extend the model and register your custom one instead! 🚀

```php
<?php
use App\Models\Sanctum\PersonalAccessToken;
use Laravel\Sanctum\Sanctum;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    // Custom token model that adds new methods such as "lastUsedAt()"
    Sanctum::usePersonalAccessTokenModel(PersonalAccessToken::class);
}

// Now this will return an instance of the custom model
$token = $request->user()->currentAccessToken();
```

## Tip #71 💡: The "doesntHave" Method

Sometimes, you may want to retrieve all the models that do not have a relationship. While you can achieve this using a query builder, Laravel already ships with the "doesntHave" method, which reads so well 🚀

```php
<?php

use App\Models\Post;

// Posts that do not have the "comments" relationship
$posts = Post::doesntHave('comments')->get();

/*
select *
from `posts`
where
    not exists (
        select *
        from comments
        where
            `posts`.`id` = `comments`.`post_id`
    )
*/
```

## Tip #72 💡: The "Sleep" Helper

Did you know that Laravel ships with a fluent wrapper for the PHP "sleep" method? This not only makes the code readable but also testable, as you can fake the Sleep class 🚀

```php
<?php

// Pause execution for 90 seconds
Sleep::for(1.5)->minutes();

// Pause execution for 2 seconds
Sleep::for(2)->seconds();

// Pause execution for 500 milliseconds
Sleep::for(500)->milliseconds();

// Pause execution for 5,000 microseconds
Sleep::for(5000)->microseconds();

// Pause execution until a given time
Sleep::until(now()->addMinute());

// Alias of PHP's native "sleep" function
Sleep::sleep(2);

// Alias of PHP's native "usleep" function
Sleep::usleep(5000);

// You can also chain methods
Sleep::for(1)->second()->and(10)->milliseconds();
```

## Tip #73 💡: Mute All Model Events

Sometimes you may want to mute all events fired by a model. Laravel ships with the "withoutEvents" method that accomplishes exactly that 🚀

```php
<?php

use App\Models\User;

// This will mute all model events 🤔
$user = User::withoutEvents(function () {
    User::findOrFail(1)->delete();
    
    return User::find(2);
});
```

## Tip #74 💡: Make use of "unguard"

Did you know that Laravel allows you to unguard a model? While you can still use the forceFill method, this approach enables you to unguard multiple models at once, which is really useful for seeding the database 🚀

```php
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    protected $fillable = ['name'];
}

Model::unguard();

// You can have *multiple* models unguarded
Flight::create([
    'name' => 'flight',
    'not_in_fillable' => true,
]);

Model::reguard();
```

## Tip #75 💡: The "containsOneItem" Method

Sometimes we want to ensure a collection has a single item. Instead of calling the count method on the collection, did you know there is an elegant method called "containsOneItem" that does the same? 🚀

```php
<?php
// Instead of this
collect([1])->count() === 1;

// You can do this
collect([1])->containsOneItem(); // true
collect([])->containsOneItem(); // false
collect([1, 2])->containsOneItem(); // false
```

## Tip #76 💡: The "back" Helper

I know we've all redirected users back at some point, and we typically use the "request()->back()" method. However, you can simply call the "back()" helper function 🚀

```php
<?php
// Instead of this
return redirect()->back($status = 302, $headers = [], $fallback = '/');

// You can do this
return back($status = 302, $headers = [], $fallback = '/');
```

## Tip #77 💡: Customize the pivot Attribute Name

We all use many-to-many relationships. In those cases, the intermediate table is accessed via the pivot attribute. Renaming it to something more expressive can make the code more readable 🚀

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class User extends Model
{
    public function podcasts(): BelongsToMany
    {
        return $this->belongsToMany(Podcast::class)
            ->as('subscription')
            ->withTimestamps();
    }
}

$users = User::with('podcasts')->get();

foreach ($users->flatMap->podcasts as $podcast) {
    // Instead of $podcast->pivot->created_at
    echo $podcast->subscription->created_at;
}
```

## Tip #78 💡: The "words" Helper

Sometimes, you may want to limit the number of words in a string. Well, Laravel comes with a handy helper "words" to do exactly that! 🚀

```php
<?php

use Illuminate\Support\Str;

return Str::words("Don't refactor without tests.", 2);
// Don't refactor ...

return Str::words("Don't refactor without tests.", 2, ' 👀');
// Don't refactor 👀

// You can also use fluent string 😎
return str("Don't refactor without tests.")->words(2);
// Don't refactor ...
```

## Tip #79 💡: Increment and Decrement Methods

Sometimes we need to update a value by incrementing or decrementing it. Usually, we would write a query to achieve this, but Laravel comes with elegant methods to do so 🚀

```php
<?php

// Increment votes by 1
DB::table('users')->increment('votes');

// Increment votes by 5
DB::table('users')->increment('votes', 5);

// Decrement votes by 1
DB::table('users')->decrement('votes');

// Decrement votes by 5
DB::table('users')->decrement('votes', 5);

// Increment votes by 1 and set the name to John
DB::table('users')->increment('votes', 1, ['name' => 'John']);

// You can increment multiple columns at once
DB::table('users')->incrementEach([
    'votes' => 5,    // Will increment votes by 5
    'balance' => 100, // Will increment balance by 100
]);

// You can also use them with Eloquent
User::query()->incrementEach([
    'votes' => 5,    // Will increment votes by 5
    'balance' => 100  // Will increment balance by 100
]);
```

## Tip #80 💡: HTTP Client Handler Stats

The Laravel HTTP Client uses Guzzle under the hood, providing access to statistics for each request you make, including total time, download speed and much more 🚀

```php
<?php

$stats = Http::get('https://oussama-mater.tech')->handlerStats();

/*
All the statistics related to the request:
[
    "http_code" => 200
    "total_time" => 8.430478
    "speed_download" => 135.0
    "speed_upload" => 0.0
    ...
]
*/
```

## Tip #81 💡: Make Use Of Factory States

Did you know that Laravel factories allow you to define states? You can use multiple states to describe the object and apply discrete modifications to it. This also makes the code more readable! 🚀

```php
<?php

use Illuminate\Database\Eloquent\Factories\Factory;

// Define the state in the user factory
public function suspended(): Factory
{
    return $this->state(function (array $attributes) {
        return [
            'account_status' => Status::SUSPENDED,
        ];
    });
}

// Now you can use it like so
$user = User::factory()->suspended()->create();
```

## Tip #82 💡: The "Benchmark" Helper

Did you know that Laravel 9.32 and above come with a handy helper called "Benchmark", allowing you to measure the number of milliseconds it takes for the given callbacks to complete? 🚀

```php
<?php

use App\Models\User;
use Illuminate\Support\Benchmark;

// This will measure and return the value
Benchmark::measure(fn () => User::find(1)); // 0.1 ms

// This will measure and dump the value
Benchmark::dd(fn () => User::find(1)); // 0.1 ms

// For both measure() and dd(), you can pass an array
Benchmark::dd([
    'Scenario 1' => fn () => User::count(), // 0.5 ms
    'Scenario 2' => fn () => User::all()->count(), // 20.0 ms
]);

// You can also save the values
[$count, $duration] = Benchmark::value(fn () => User::count());
```

## Tip #83 💡: Retrieve Soft Deleted Models

When using Laravel Model Binding, you may wish to include the soft deleted models as well. Luckily, Laravel comes with a handy routing method called "withTrashed" to do exactly that 🚀

```php
<?php

use App\Models\User;

// Now soft-deleted users will be included.
Route::get('/users/{user}', function (User $user) {
    return $user->email;
})->withTrashed();
```

## Tip #84 💡: Check if the Value of a Given Model Key Has Changed

Sometimes, we wish to check if the value of a given model key has been affected by a change or not. Laravel ships with the "originalIsEquivalent()" method to do exactly that 🚀

```php
<?php

$user = User::firstOrFail(); // ['name' => 'old']

$user->name = 'old'; // Keep the old value

$user->originalIsEquivalent('name'); // true

$user->name = 'new'; // Change the value

$user->originalIsEquivalent('name'); // false
```

## Tip #85 💡: The "getOrPut" Method

When using collections, sometimes we want to retrieve the value of an existing key but insert it if it does not exist. While this can be achieved using the "get" and "put" methods, Laravel offers a handy method called "getOrPut" that does the same 🚀

```php
$collection = collect(['price' => 100]);

// Check and set value if not exists
if (!$collection->has('name')) {
    $collection->put('name', 'Desk');
}
$value = $collection->get('name');

// Or use getOrPut() for the same operation
$value = $collection->getOrPut('name', 'Desk');
```

## Tip #86 💡: The "dot" and "undot" Methods

When working with Laravel collections, you may want to flatten a multi-dimensional collection into a single-level collection, or vice versa. Luckily, there are 2 methods just for that, "dot" and "undot" 🚀

```php
$collection = collect(['products' => ['desk' => ['price' => 100]]]);

$dotted = $collection->dot();    // ['products.desk.price' => 100]
$undotted = $collection->undot(); // ['products' => ['desk' => ['price' => 100]]]
```

## Tip #87 💡: The "times" Methods

Did you know that Laravel ships with a cool collection method "times," which allows you to create a collection by invoking a closure N times? This could be helpful when working with days or generating random strings 🚀

```php
$collection = Collection::times(10, function (int $number) {
    return $number * 9;
});

$collection->all(); 
// [9, 18, 27, 36, 45, 54, 63, 72, 81, 90]
```

## Tip #88 💡: Faker Random Element

Sometimes, when defining factories, you may want to pick a random element from an array. Since Laravel uses FakerPHP under the hood, you can do this by calling the "randomElement" method 🚀

```php
// Get a random subscription type
$random = fake()->randomElement(['basic', 'premium']);

// Get a random letter
$random = fake()->randomElement(['a', 'b', 'c']);
```

## Tip #89 💡: Redirect with URL Fragments

Did you know that Laravel comes with the "withFragment" method, which allows you to add a URI fragment when redirecting? 🚀

```php
<?php

return redirect()
    ->back()
    ->withFragment('testimonials');
    // exmaple.com/#testimonials

return redirect()
    ->route('product.show')
    ->withFragment('reviews');
    // exmaple.com/product/1#reviews
```

## Tip #90 💡: Deny As Not Found

When defining gates or policies, for security reasons, we often opt to return a 404 instead of a 403. Laravel provides the "denyAsNotFound()" method for this purpose 🚀

```php
<?php

use App\Models\User;
use Illuminate\Auth\Access\Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
        ? Response::allow()
        : Response::denyAsNotFound();
});
```

## Tip #91 💡: Mask Strings

Ever found yourself in need of masking an email or a verification code for security purposes? Laravel ships with the "mask" method to do exactly that! You can always combine it with other methods to avoid leaking the length of the string 🚀

```php
<?php

use Illuminate\Support\Str;

Str::mask('john@example.com', '*', 3); // joh*************
Str::mask('98552157', '*', -5, 4); // 985****7
```

## Tip #92 💡: More Readable Numbers

If you work with numbers, the longer they are, the harder they are to read. Did you know you can use underscores for better readability? 🚀

```php
<?php

$amount = 1000;
$amount = 1_000;

$amount = 100000;
$amount = 100_000;

$amount = 100000000;
$amount = 100_000_000;
```

## Tip #93 💡: Human Readable Dates

Have you ever wanted to display human-readable dates instead of exact ones? Like "1 day ago" or "a month ago"? Laravel allows you to do exactly that by using the "diffForHumans" method 🚀

```php
<?php

// 5 days ago
$post->created_at->diffForHumans();

// 5 days 23 minutes ago
$post->created_at->diffForHumans(['parts' => 2]);

// 5 days 24 minutes 36 seconds ag
$post->created_at->diffForHumans(['parts' => 3]);
```

## Tip #94 💡: Higher Order Expectations

Did you know that [@pestphp](https://twitter.com/pestphp) ships with "Higher Order Expectations"? It allows you to perform expectations on the properties and/or methods of the given object. This results in a much cleaner code

```php
<?php

// Instead of this 😫
expect($user->name)->toBe('John');
expect($user->country)->toBe('us');
expect($user->isAdmin())->toBeFalse();

// Do this 😎
expect($user)
    ->name->toBe('John')
    ->country->toBe('us')
    ->isAdmin()->toBeFalse();
```

## Tip #95 💡: Intercept Exceptions

One of the cool features in Pest is "intercepting" exceptions. You can override built-in expectations with your own implementation and still fallback to the default behavior for regular cases 🚀

```php
<?php

use Illuminate\Database\Eloquent\Model;
use App\Models\User;

// In your tests/Expectations.php file
expect()->intercept('toBe', Model::class, function (Model $expected) {
    expect($this->value->is($expected))
    // You can also set a custom message
    ->toBeTrue(message: 'Failed asserting that both models are the same.');
});

// Now you can use it in your tests 😎
test('models', function () {
    $user = User::find(1);
    $sameUser = User::find(1);
    
    expect($user)->toBe($sameUser);
});

// It will still work as it used to 👍
test('old expectation', function () {
    expect('hello-world')->toBe('hello-world');
});
```

## Tip #96 💡: Access the Parent Loop Variable

Sometimes, when dealing with nested loops, you may want to keep track of the parent's iteration. Blade makes it incredibly easy, as you have access to the parent loop variable 🚀

```php
@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            // This is the first iteration of the parent loop.
        @endif
    @endforeach
@endforeach
```

## Tip #97 💡: Redirect Away

Sometimes, you may want to redirect your users away from your Laravel application. Luckily, there is a helper method called "away" that allows you to do exactly that 🚀

```php
<?php

redirect()->away('https://blog.oussama-mater.tech');
```

## Tip #98 💡: The "whenFilled" Method

We often test whether or not a request parameter is filled to decide to perform logic. Laravel ships with a cool helper called "whenFilled" to do just that! 🚀

```php
<?php

$request->whenFilled('name', function (string $input) {
    // The "name" value is filled...
}, function () {
    // The "name" value is not filled...
});
```

## Tip #99 💡: The "startOfHour" Method

When working with time, you may want to have a sharp hour, like 18:00:00. Since Laravel uses Carbon under the hood, you have access to "startOfHour," which does exactly that 🚀

```php
<?php
// 2024-04-26 18:46:39
echo now();

// 2024-04-26 18:46:00
echo now()->setSeconds(0);

// 2024-04-26 18:00:00
echo now()->setSeconds(0)->setMinutes(0);

// 2024-04-26 18:00:00
echo now()->startOfHour();
```

## Tip #100 💡: Use the Higher Order "orWhere" Method

Laravel supports "Higher Order Messages" with collections, which are cool shortcuts that we use. But did you know that you can make use of them when writing eloquent queries? 🚀

```php
// tip-100.php
<?php

// Instead of this 😫
User::popular()->orWhere(function (Builder $query) {
    $query->active();
})->get()

// You can do this 😎
User::popular()->orWhere->active()->get();
```

## Tip #101 💡: Short Attribute Syntax

Did you know that Blade allows for short attribute syntax when passing to components? 🚀

```php
// Instead of this 😫
<x-profile :user-id="$userId"></x-profile>

// You can do this 😎
<x-profile :$userId></x-profile>
```

## Tip #102 💡: Faster Queries with "whereIntegerInRaw"

When using a whereIn query with non-user input, opt for whereIntegerInRaw. This speeds up your query by skipping PDO bindings and Laravel's security measures against SQL injection 🚀

```php
<?php

// Instead of using whereIn()
Product::whereIn('id', range(1, 10000))->get();

// Use WhereIntegerInRaw()
Product::whereIntegerInRaw('id', range(1, 10000))->get();
```

## Tip #103 💡: Recycle Existing Models

When defining factories, you may want to use a single model for all the relationships instead of creating a new one for each of them. Laravel ships with a cool method "recycle" to do exactly that 🚀

```php
<?php

$airlines = Airline::factory()->count(3)->create();

// Only 3 airlines will be used for the 100 tickets
Ticket::factory()
    ->count(100)
    ->recycle($airlines) // You can pass a single one
    ->create();
```

## Tip #104 💡: The "upsert" Method

Sometimes you may wish to update a bunch of records or create them if they do not exist. Laravel ships with a cool method "upsert" to do exactly that 🚀

```php
<?php

/*
This will update the price of all the records that match
the given departure and destination or create them
*/
Flight::upsert([
    ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
    ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150]
], uniqueBy: ['departure', 'destination'], update: ['price']);
```

## Tip #105 💡: Catch Flaky Tests

Sometimes when you write tests, you might end up with some flaky ones that are just unstable; those tests fail once in a thousand. PestPHP comes with a really cool helper, "repeat," to catch those 🚀

```php
<?php

it('can repeat a test', function() {
    $result = /** Some code that may be unstable */;

    expect($result)->toBeTrue();
})->repeat(100); // Repeat the test 100 times
```

## Tip #106 💡: View Routes

Did you know that Laravel ships with the "view" method, which allows you to render a view directly without having to define a closure? It's much more elegant! 🚀

```php
<?php

use Illuminate\Support\Facades\Route;

// Instead of this 🥱
Route::get('/welcome', function () {
    return view('welcome', ['foo' => 'bar']);
});

// You can do this 😎
Route::view('/welcome', 'welcome', ['foo' => 'bar']);
```

## Tip #107 💡: No timestamp columns

Sometimes, your table might not have the "created_at" and "updated_at" columns. You can instruct Laravel not to update them by setting "timestamps" to false 🚀

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * Indicates if the model should be timestamped.
     *
     * @var bool
     */
    public $timestamps = false;
}
```

## Tip #108 💡: The "simplePaginate" Method

Sometimes, when paginating your models, you may only need a "next" and "previous" buttons. For this, you can use the "simplePaginate" method instead of the regular paginate 🚀

```php
<?php

$users = User::paginate(15);

/*
Will result in the following keys
[
    "current_page",
    "data",
    "first_page_url",
    "from",
    "last_page",
    "last_page_url",
    "links",
    "next_page_url",
    "path",
    "per_page",
    "prev_page_url",
    "to",
    "total",
];
*/
$users = User::simplePaginate(15);

/*
Will result in the following keys
[
    "current_page",
    "data",
    "first_page_url",
    "from",
    "next_page_url",
    "path",
    "per_page",
    "prev_page_url",
    "to",
];
*/
```

## Tip #109 💡: Bootable Traits

Did you know that Laravel automatically boots your traits if they follow the `boot[TraitName]` convention? This allows you to easily define shared logic for model events. And here's a secret: that's where multi-tenancy starts 🚀

```php
<?php

trait Sluggable
{
    public static function bootSluggable()
    {
        static::saving(function ($model) {
            $model->slug = str($model->title)->slug()->toString();
        });
    }
}

class Post extends Model
{
    use Sluggable; // The trait will be booted automatically
}
```

## Tip #110 💡: Blade To HTML

Did you know you can use Blade to render views as strings wherever you want? This is helpful as you can use Blade to build dynamic strings or even shell scripts, similar to how Envoy does 🚀

```php
<?php

// This renders the blade file welcome.blade.php into an HTML string
$rendered = view('welcome', ['foo' => 'bar'])->render();
```

## Tip #111 💡: Clean Up After Failed Jobs

When jobs fail, you may want to send notifications or perform some cleanups. Luckily, Laravel allows you to define a "failed" method to do exactly that 🚀

```php
<?php

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Throwable;

class ProcessPodcast implements ShouldQueue
{
    use InteractsWithQueue, Queueable, SerializesModels;

    // ...

    public function failed(?Throwable $exception): void
    {
        // Handle job failure, such as sending notifications
    }
}
```

## Tip #112 💡: Schedule Jobs Based On Time Zones

Did you know that you can schedule jobs based on specific time zones? You can do so by chaining the "timezone" method 🚀

```php
<?php

use Illuminate\Support\Facades\Schedule;

Schedule::command('report:generate')
    ->timezone('Africa/Tunis')
    ->at('9:00')
```

## Tip #113 💡: Singleton Routes

Sometimes you may need singleton resources that can't be created but only shown or edited, like a user profile, for example. Laravel ships with a singleton helper to allow you to define these routes 🚀

```php
// Image 4: tip-114.php
<?php

use App\Http\Controllers\ProfileController;
use Illuminate\Support\Facades\Route;

Route::singleton('profile', ProfileController::class);
/*
This will result in the following routes:
GET         /profile
GET         /profile/edit
PUT/PATCH   /profile
*/
```

## Tip #114 💡: Authenticate the User Once

Did you know that Laravel ships with the "once" method for authenticating the user only for the current request? This can be handy in various scenarios, such as building single-use links or RESTful APIs 🚀

```php
<?php

if (Auth::once($credentials)) {
    // ...
}

```

## Tip #115 💡: The "doesntExist" Method

Sometimes you may want to check if certain records do not exist in the database. While checking the count or using the exists() method can do the trick, Laravel ships with the "doesntExist" method to do it elegantly 🚀

```php
// Image 5: tip-116.php
<?php

// This is okay 😊
if (User::count() === 0) {
}

// This is good 😊
if (! User::exists()) {
}

// This is better 😎
if (User::doesntExist()) {
}
```

## Tip #116 💡: Clone Your Queries

Sometimes you may need to reuse the same base query for multiple filtering. Laravel ships with a "clone" method to do exactly that 🚀

```php
<?php

// Base query, common conditions
$query = User::query()->where('created_at', '<', now()->subMonths(3));

$verified_users = $query->clone()->whereNotNull('email_verified_at')->get();

// This can be customized further if needed
$unverified_users = $query->clone()->whereNull('email_verified_at')->get();
```

## Tip #117 💡: Assert Model Missing

When writing tests, we often use `assertDatabaseMissing` to check whether a model has been deleted. Did you know that Laravel ships with a cool helper called `assertModelMissing` to do exactly that? 🚀

```php
<?php

use App\Models\User;

$user = User::factory()->create();

$user->delete();

// Instead of this 🥱
$this->assertDatabaseMissing('users', [
    'email' => $user->email,
]);

// You can do this 😎
$this->assertModelMissing($user);
```

## Tip #118 💡: Check Collection Item Types

Sometimes you may want to ensure that the collection items are all of a specific type. While `map` paired with an `instanceof` check might do the trick, Laravel already ships with the `ensure` method to do that 🚀

```php
<?php

// Instead of this 🥱
return $collection->each(function ($item) {
    if (!$item instanceof User) {
        throw new UnexpectedValueException('😕');
    }
});

// You can do this 😎
return $collection->ensure(User::class);

// Or allow multiple types, which is equivalent to OR
return $collection->ensure([User::class, Customer::class]);

// You can also ensure primitive types
return $collection->ensure('int');
```

## Tip #119 💡: Stop On First Failure

Sometimes, when validating a request, you may want to stop at the first failure. Laravel allows you to do this by setting the "stopOnFirstFailure" property to true on the form request class 🚀

```php
<?php

/**
 * Indicates if the validator should stop on the first rule failure.
 *
 * @var bool
 */
protected $stopOnFirstFailure = true;
```

## Tip #120 💡: The "bail" Validation Rule

Sometimes, when validating a field, you may want to stop at the first validation failure. Laravel ships with a rule called "bail" to do exactly that 🚀

```php
<?php

// When validating the title, if the required rule fails, 
// Laravel won't test against the other rules.
$request->validate([
    'title' => 'bail|required|unique:posts|max:255',
    'body' => 'required',
]);
```

## Tip #121 💡: Faker "valid()" Modifier

Since Laravel uses FakerPHP under the hood, you can use the "valid()" modifier to ensure that the generated fake data follows certain rules 🚀

```php
<?php

// This will only generate even numbers
$evenNumber = fake()->valid(fn (int $digit) => $digit % 2 === 0)->randomDigit();
```

## Tip #122 💡: Hide Columns On The Fly

Sometimes, you may want to hide model attributes that were not defined in the "hidden" array. Laravel allows you to do this on the fly using the "makeHidden" method 🚀

```php
<?php

$users = User::all()->makeHidden(['address', 'phone_number']);
```

## Tip #123 💡: Without Data Wrapping

Eloquent API resources are automatically wrapped in a "data" object. Sometimes, you may want to remove this wrapping. Laravel includes the "withoutWrapping" method to do exactly that 🚀

```php
<?php

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        JsonResource::withoutWrapping();
    }
}

/*
[
    {
        "id": 1,
        "name": "Eladio Schroeder Sr.",
        "email": "therese28@example.com"
    },
    {
        "id": 2,
        "name": "Liliana Mayert",
        "email": "evandervort@example.com"
    }
]
*/
```

## Tip #124 💡: Collect API Responses

Usually, when working with the HTTP Client, we manually collect the JSON response from the API. However, did you know that Laravel ships with the "collect" method directly on the HTTP Client? 🚀

```php
<?php

use Illuminate\Support\Facades\Http;

// Instead of this 😒
$response = Http::get('http://example.com')->json();
collect($response);

// Do this 😎
$collection = Http::get('http://example.com')->collect();
```

## Tip #125 💡: The "data_get" Helper

When working with nested arrays, Laravel provides a cool helper called "data_get". This helper allows you to use "dot" syntax and wildcards to retrieve values 🚀

```php
<?php

$data = [
    'product-one' => ['name' => 'Desk 1', 'price' => 100],
    'product-two' => ['name' => 'Desk 2', 'price' => 150],
];
 
data_get($data, 'product-one.name'); // ['Desk 1'];
data_get($data, '*.name'); // ['Desk 1', 'Desk 2'];
```

## Tip #126 💡: Better Checks For Input Presence

We often need to check if a request contains certain values. Did you know that Laravel ships with two cool methods, "has" and "hasAny", to perform these checks elegantly? 🚀

```php
<?php

// True if ALL of the values are present
if ($request->has(['name', 'email'])) {
    //
}

// True if ANY of the values are present
if ($request->hasAny(['name', 'email'])) {
    //
}
```

## Tip #127 💡: Bind Typed Variadics

Did you know that you can bind typed variadics to the container? Laravel ships with 3 methods to allow you to do so: "when()", "needs()", and "give()". You can keep using DI without worries! 🚀

```php
<?php

use App\Models\Filter;
use App\Services\Logger;

class Firewall
{
    protected array $filters;

    public function __construct(
        protected Logger $logger,
        Filter ...$filters,
    ) {
        $this->filters = $filters;
    }
}

// In the service provider
$this->app->when(Firewall::class)
    ->needs(Filter::class)
    ->give(function (Application $app) {
        return [
            $app->make(NullFilter::class),
            $app->make(ProfanityFilter::class),
            $app->make(TooLongFilter::class),
        ];
    });
```

## Tip #128 💡: The "every" Collection Method

Sometimes you may want to check if every element in the collection passes a condition. Luckily, Laravel ships with the "every" method to do exactly that 🚀

```php
<?php

$result = collect([1, 2, 3])->every(fn (int $value, int $key) => $value > 2);
// $result will be false

$result = collect([])->every(fn (int $value, int $key) => $value > 2);
// Since the collection is empty, $result will be true
```

## Tip #129 💡: The "forget" Collection Method

Sometimes, when working with collections, you may want to remove an element by its key. Luckily, collections come with the "forget" method to do exactly that 🚀

```php
<?php

$collection = collect(['name' => 'John Doe', 'framework' => 'laravel']);

$collection->forget('name');

$collection->all(); // ['framework' => 'laravel']
```

## Tip #130 💡: Spell Numbers

Did you know that you can spell numbers in different locales using the "Number" helper that Laravel ships with? 🚀

```php
<?php

use Illuminate\Support\Number;

$number = Number::spell(102); // one hundred and two

$number = Number::spell(88, locale: 'fr'); // quatre-vingt-huit
```

## Tip #131 💡: Human-readable Numbers

Sometimes you may want to format numbers for your users in a human-readable format. The "Number" helper allows you to do just that 🚀

```php
<?php

use Illuminate\Support\Number;
 
$number = Number::forHumans(1000); // 1 thousand
 
$number = Number::forHumans(489939); // 490 thousand
 
$number = Number::forHumans(1230000, precision: 2); // 1.23 million
```

## Tip #132 💡: Skip Collection Items Until a Condition is Met

Sometimes, when working with collections, you may want to skip all the elements until a condition is met. Laravel comes with the "skipUntil" method to do exactly that 🚀

```php
<?php

$collection = collect([1, 2, 3, 4]);

$subset = $collection->skipUntil(function (int $item) {
    return $item >= 3;
});

$subset->all(); // [3, 4]
```

## Tip #133 💡: The "zip" Collection Method

When working with collections, you might want to merge two collections by their index, combining the values of the first index, then the second, and so on. Luckily, Laravel includes the "zip" method to do exactly that 🚀

```php
<?php

$collection = collect(['Chair', 'Desk']);

// This will merge values by index, so "Chair" with 100, and "Desk" with 200
$zipped = $collection->zip([100, 200]);

$zipped->all(); // [['Chair', 100], ['Desk', 200]]
```

## Tip #134 💡: The "WhenNotEmpty" Collection Method

While working with collections, you might want to execute some logic when the collection is not empty. Instead of manually checking, Laravel ships with a cool method, "whenNotEmpty()", to do exactly that 🚀

```php
<?php

$collection = collect(['michael', 'tom']);

$collection->whenNotEmpty(function (Collection $collection) {
    return $collection->push('adam');
});

$collection->all(); // ['michael', 'tom', 'adam']

$collection = collect(); // empty collection

$collection->whenNotEmpty(function (Collection $collection) {
    return $collection->push('adam');
});

$collection->all(); // []
```

## Tip #135 💡: Send Concurrent Requests

Laravel's HTTP client wraps Guzzle, which allows you to make concurrent requests to speed things up. This is very helpful for various cases, such as health checks! 🚀

```php
<?php

use Illuminate\Http\Client\Pool;
use Illuminate\Support\Facades\Http;

$responses = Http::pool(fn (Pool $pool) => [
    $pool->get('http://localhost/first'),
    $pool->get('http://localhost/second'),
    $pool->get('http://localhost/third'),
]);

return $responses[0]->ok() &&
       $responses[1]->ok() &&
       $responses[2]->ok();
```

## Tip #136 💡: Search Collection Items

Did you know that Laravel allows you to search collection items? You can even pass a condition to search for the first element that meets it 🚀

```php
<?php
$collection = collect([2, 4, 6, 8]);

$collection->search('4'); // 1 (the index)

$collection->search('4', strict: true); // false (not found)

$collection->search(fn (int $item, int $key) => $item > 5); // 2 (the index)
```

## Tip #137 💡: Disable Global Scopes

Laravel allows you to apply global scopes to your models, but sometimes you may wish to disable them for a specific query. You can do this by chaining the "withoutGlobalScope" method 🚀

```php
<?php

// Remove all of the global scopes
User::withoutGlobalScopes()->get();

// Remove a single global scope
User::withoutGlobalScope(FirstScope::class)->get();

// Remove some of the global scopes
User::withoutGlobalScopes([
    FirstScope::class, SecondScope::class
])->get();
```

## Tip #138 💡: Run Commands In Maintenance Mode

When Laravel's maintenance mode is on, all scheduled commands won't run. If you wish to change this behavior, you can chain the "evenInMaintenanceMode()" method 🚀

```php
<?php

// Command will run even when maintenance mode is on
Schedule::command('emails:send')->evenInMaintenanceMode();
```

## Tip #139 💡: Run Commands In the Background

Scheduled commands run sequentially. If you have a long-running task, it could take longer than anticipated and cause a delay for other tasks. Luckily, in such cases, you can use the "runInBackground" method 🚀

```php
<?php

use Illuminate\Support\Facades\Schedule;

Schedule::command('analytics:report')
        ->daily()
        ->runInBackground();
```

## Tip #140 💡: The "literal" Helper

Did you know that Laravel ships with a cool helper called "literal" that allows you to create a PHP object using named arguments? 🚀

```php
<?php

$obj = literal(
    name: 'Joe',
    languages: ['PHP', 'Ruby'],
);

$obj->name;      // 'Joe'
$obj->languages; // ['PHP', 'Ruby']
```

## Tip #141 💡: The "abort_if" Helper

When writing middlewares, we often abort the request if a condition is met. For such cases, "abort_if" allows you to do exactly that! 🚀

```php
<?php

// Instead of this 😞
if (!Auth::user()->isAdmin()) {
    abort(403);
}

// Do this 😎
abort_if(!Auth::user()->isAdmin(), 403);
```

## Tip #142 💡: The "sole" Method

When working with collections, whether regular or Eloquent, if you want to get the first item that matches the condition and ensure it is the only one, use the "sole" method 🚀

```php
<?php

// Returns 2
collect([1, 2, 3, 4])->sole(fn (int $value, int $key) => $value === 2);

// Throws: Illuminate\Support\MultipleItemsFoundException 2 items were found.
collect([1, 2, 2, 4])->sole(fn (int $value, int $key) => $value === 2);
```

## Tip #143 💡: Hash Passwords Automatically

When creating users, we often use the Hash facade, but did you know that Laravel comes with a "hashed" cast that will automatically hash your user's password? 🚀

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected function casts(): array
    {
        return [
            'password' => 'hashed',
        ];
    }
}

$user = User::create([
    // ...
    'password' => 'password', // Instead of Hash::make('password')
]);
```

## Tip #144 💡: The "createOr" Laravel

Sometimes, you may want to execute some actions when no record is found, beyond just creating a new instance. The "createOr" method allows you to do exactly that 🚀

```php
<?php

$user = User::where('email', $request->input('email'))->firstOr(function () {
    // Execute some logic

    // Create and return a new user
    return User::create([
        // ...
    ]);
});
```

## Tip #145 💡: Append and Prepend to files

When working with files, you may need to prepend or append content. Luckily, Laravel ships with two helpers to do exactly that 🚀

```php
<?php

Storage::prepend('file.log', 'Prepended Text');

Storage::append('file.log', 'Appended Text');
```

## Tip #146 💡: Formatting to Percentages

Did you know Laravel ships with a "percentage" helper to get the percentage of any representative value? 🚀

```php
<?php

use Illuminate\Support\Number;
 
$percentage = Number::percentage(10); // 10%
 
$percentage = Number::percentage(10, precision: 2); // 10.00%
 
$percentage = Number::percentage(10.123, maxPrecision: 2); // 10.12%
 
$percentage = Number::percentage(10, precision: 2, locale: 'de'); // 10,00%
```

## Tip #147 💡: Formatting to a Human-Readable File Size

Did you know Laravel ships with a "fileSize" helper to get the file size representation of a given byte value as a string? 🚀

```php
<?php

use Illuminate\Support\Number;
 
$size = Number::fileSize(1024); // 1 KB
 
$size = Number::fileSize(1024 * 1024); // 1 MB
 
$size = Number::fileSize(1024, precision: 2); // 1.00 KB
```

## Tip #148 💡: Freeze Time

When writing tests, we sometimes need to "freeze" time to make assertions. Laravel provides an elegant method "freezeTime" to do exactly that 🚀

```php
<?php

// Instead of this 🥱
Carbon::setTestNow(now());

// Do this 😎
$this->freezeTime();
```

## Tip #149 💡: Log Out Other Devices

When users log out, you might want to ask them if they want to log out from other devices while keeping the current one. Luckily, Laravel ships with the "logoutOtherDevices" method that does exactly that 🚀

```php
<?php

use Illuminate\Support\Facades\Auth;
 
Auth::logoutOtherDevices($currentPassword);
```

## Tip #150 💡: Working with Base64 Strings

We often work with Base64 strings, especially when building API integrations. Laravel comes with built-in helpers to work with Base64 right out of the box 🚀

```php
<?php

use Illuminate\Support\Str;
 
$base64 = Str::toBase64('Laravel'); // TGFyYXZlbA==
$base64 = str('Laravel')->toBase64(); // TGFyYXZlbA==

$base64 = Str::fromBase64('TGFyYXZlbA=='); // Laravel
$base64 = str('TGFyYXZlbA==')->fromBase64(); // Laravel
```

## Tip #151 💡: The "abort_unless" Helper

When writing middlewares, we often need to abort the request if a condition is met. For such cases, "abort_unless" allows you to do exactly that! 🚀

```php
<?php

// Instead of this 🥱
if (!Auth::user()->isAdmin()) {
    abort(403);
}

// Do this 😎
abort_unless(Auth::user()->isAdmin(), 403);
```

## Tip #152 💡: The "latest" and "oldest" Methods

We often order models in ascending or descending order using the "orderBy" method. But did you know that Laravel comes with two methods, "latest" and "oldest," that do exactly that? 🚀

```php
<?php

// Instead of this 🥱
User::orderBy('created_at', 'desc')->get();
User::orderBy('created_at', 'asc')->get();

// Do this 😎
User::latest()->get();
User::oldest()->get();

// You can specify keys other than created_at
User::latest('id')->get();
User::oldest('id')->get();
```

## Tip #153 💡: Cache Headers On The Fly

Did you know that Laravel ships with a middleware "SetCacheHeaders", which you can use to set cache headers, such as "max_age" and "etag"? 🚀

```php
<?php

Route::middleware('cache.headers:public;max_age=2628000;etag')->group(function () {
    Route::get('/privacy', function () {
        // ...
    });

    Route::get('/terms', function () {
        // ...
    });
});
```

## Tip #154 💡: Insert Or Ignore

Sometimes, you might want to ignore errors when inserting data. Laravel comes with the "insertOrIgnore" method that does exactly that 🚀

```php
<?php

DB::table('users')->insertOrIgnore([
    ['id' => 1, 'email' => 'sisko@example.com'],
    ['id' => 2, 'email' => 'archer@example.com'],
]);

// insert ignore into `users` (`email`, `id`) values (?, ?), (?, ?)
```

## Tip #155 💡: Schedule Shell Commands

Did you know that the Laravel Scheduler allows you to execute commands in the operating system? 🚀

```php
<?php

use Illuminate\Support\Facades\Schedule;
 
Schedule::exec('node /home/forge/script.js')->daily();
```

## Tip #156 💡: Typed Configs

If you are using static analysis tools like PHPStan (which you should), you can make use of typed configs. Not only does this assist the tools, but it also ensures that the config matches the required type 🚀

```php
<?php

Config::string('config-key'); // config()->string('config-key')
Config::integer('config-key'); // config()->integer('config-key')
Config::float('config-key'); // config()->float('config-key')
Config::boolean('config-key'); // config()->boolean('config-key')
Config::array('config-key'); // config()->array('config-key')
```

## Tip #157 💡: Pluralize Words

Did you know that Laravel ships with the "plural" method to help you pluralize words? This is really helpful when you want to show a quantity-dependent message to your users. 🚀

```php
<?php

use Illuminate\Support\Str;

// The count is optional, if 1 is passed, the same word will be returned
$plural = Str::plural('car', count: 2); // cars

// Also available as a fluent method
$plural = str('child')->plural(); // children
```

## Tip #158 💡: Generate Fake User Agents

Since Laravel uses FakerPHP under the hood, you can generate fake user agents for your tests. 🚀

```php
<?php

echo fake()->userAgent();
// Mozilla/5.0 (iPad; CPU OS 8_0_2 like Mac OS X; sl-SI) AppleWebKit/532.22.4 (KHTML, like Gecko) Version/3.0.5 Mobile/8B114 Safari/6532.22.4

echo fake()->chrome();
// Mozilla/5.0 (X11; Linux i686) AppleWebKit/5352 (KHTML, like Gecko) Chrome/38.0.899.0 Mobile Safari/5352

echo fake()->firefox();
// Mozilla/5.0 (Windows 98; Win 9x 4.90; sl-SI; rv:1.9.1.20) Gecko/20220314 Firefox/35.0

echo fake()->safari();
// Mozilla/5.0 (Windows; U; Windows NT 5.1) AppleWebKit/531.4.3 (KHTML, like Gecko) Version/5.1 Safari/531.4.3

echo fake()->opera();
// Opera/8.71 (Windows NT 4.0; nl-NL) Presto/2.11.217 Version/12.00

echo fake()->internetExplorer();
// Mozilla/5.0 (compatible; MSIE 5.0; Windows NT 5.1; Trident/4.0)

echo fake()->msedge();
// Mozilla/5.0 (iPhone; CPU iPhone OS 13_1 like Mac OS X) AppleWebKit/535.2 (KHTML, like Gecko) Version/15.0 EdgiOS/98.01106.3 Mobile/15E148 Safari/535.2
```

## Tip #159 💡: Ping URLs When Running Commands

Did you know you can ping URLs before and after your command has run? This is really useful if you want to notify an external service or a webhook. 🚀

```php
<?php

Schedule::command('emails:send')
    ->daily()
    ->pingBefore($webhookUrl)
    ->thenPing($webhookUrl);
```

## Tip #160 💡: Find Many

Did you know that you can pass multiple IDs to the `find()` method? Laravel also ships with a slightly more readable method, `findMany()`, which does the same thing! 🚀

```php
<?php

// Instead of this
$users = User::query()->whereIn('id', [1, 2, 3])->get();

// Do this
$users = User::find([1, 2, 3]);

// Even better, find() calls findMany() internally, so skip it and be expli
$users = User::findMany([1, 2, 3]);
```

## Tip #161 💡: Check If Your Model Has Changed Since Last Retrieval

Did you know Laravel ships with the `isDirty()` method, which allows you to check if one or more attributes have changed since the last time you retrieved the model? 🚀

```php
<?php

use App\Models\User;

$user = User::create([
    'first_name' => 'John',
    'last_name' => 'Doe',
    'age' => 20,
]);

$user->age = 21;

$user->isDirty(); // true
$user->isDirty('age'); // true
$user->isDirty('first_name'); // false
$user->isDirty(['first_name', 'age']); // true
$user->isDirty(['first_name', 'last_name']); // false

$user->save();

$user->isDirty(); // false
```

## Tip #162 💡: Log Out The Current Device Only

Did you know that Laravel ships with the "logoutCurrentDevice" method, which allows you to log out only the currently authenticated device? 🚀

```php
<?php

use Illuminate\Support\Facades\Auth;
 
Auth::logoutCurrentDevice();
```

## Tip #163 💡: Customize the Default Timestamp Columns

Sometimes, you might need to customize the default timestamp columns, or you might already have an old table for which you are creating a model. Luckily, it is super simple to do so 🚀

```php
<?php

class Flight extends Model
{
    const CREATED_AT = 'creation_date';
    const UPDATED_AT = 'updated_date';
}
```

## Tip #164 💡: Prevent Filling Unfillable Attributes

Did you know you can configure Laravel to throw an exception when attempting to fill an unfillable attribute? This is helpful during development to catch missed or forgotten attributes before getting to prod 🚀

```php
<?php

use Illuminate\Database\Eloquent\Model;

Model::preventSilentlyDiscardingAttributes(! $this->app->isProduction());
```

## Tip #165 💡: Create New Records or Update Existing Ones

We've all been in a situation where we want to check if a record exists so we can update it, or create it if it does not. Laravel ships with the `updateOrCreate` method to do exactly that 🚀

```php
<?php

$flight = Flight::updateOrCreate(
    // If we can't find a flight with the following criteria,
    ['departure' => 'Oakland', 'destination' => 'San Diego'],
    // we will create it with the following data
    ['price' => 99, 'discounted' => 1]
);
```

## Tip #166 💡: Delete (Destroy) Records

Did you know that Laravel ships with the `destroy` method, which allows you to delete records by their primary key? 🚀

```php
<?php

// Instead of this 😢
Flight::find(1)->delete()

// Do this 😎
Flight::destroy(1);

Flight::destroy(1, 2, 3); // works with variadic arguments

Flight::destroy([1, 2, 3]); // arrays

Flight::destroy(collect([1, 2, 3])); // and also collections!
```

## Tip #167 💡: Time Travel in Your Tests

Did you know that Laravel comes with a cool time helper that allows you to travel into the future or the past in your tests? 🚀

```php
<?php

// Travel into the future...
$this->travel(5)->milliseconds(); // Travel 5 milliseconds into the future
$this->travel(5)->seconds();
$this->travel(5)->minutes();
$this->travel(5)->hours();
$this->travel(5)->days();
$this->travel(5)->weeks();
$this->travel(5)->years();

// Travel into the past...
$this->travel(-5)->hours(); // Travel 5 years into the past

// Travel to an explicit time...
$this->travelTo(now()->subHours(6));

// Return back to the present time...
$this->travelBack();
```

## Tip #168 💡: Cast Values On The Fly

Sometimes you may need to apply casts while executing a query. Luckily, you can do that on the fly with the "withCasts" method that Laravel ships with 🚀

```php
<?php

$users = User::select([
    'users.*',
    'last_posted_at' => Post::selectRaw('MAX(created_at)')
        ->whereColumn('user_id', 'users.id')
])->withCasts([
    // Casting the raw string 'last_posted_at' to a datetime
    'last_posted_at' => 'datetime'
])->get();
```

## Tip #169 💡: Extract Validated Input

When working with validated input, we often need to extract only a few items from the request. Instead of manually unsetting or filtering, use Laravel's "safe()" method to do this elegantly 🚀

```php
<?php

$validated = $request->safe()->only(['name', 'email']);
 
$validated = $request->safe()->except(['name', 'email']);
 
$validated = $request->safe()->all();
```

## Tip #170 💡: The "valueOrFail" Method

We often use the "firstOrFail" method to get a single value from the resulting model. Did you know that Laravel ships with the "valueOrFail" method, which allows you to do exactly that? 🚀

```php
<?php

// Instead of this
$flight = Flight::where('legs', '>', 3)->firstOrFail();
$flightName = $flight->name;

// Do this
$flightName = Flight::where('legs', '>', 3)->valueOrFail('name');
```

## Tip #171 💡: The "to_route" Method

We often redirect our users to specific routes using the "redirect()" method. Did you know there is a shorter and more expressive method called "to_route"? 🚀

```php
<?php

// Instead of this 🥱
return redirect()->route('profile');

// You can do this 😎
return to_route('profile');
```

## Tip #172 💡: Generate Random Passwords

Did you know that Laravel ships with a "password" method that generates random, strong passwords? This is helpful when you want to suggest passwords for your users 🚀

```php
<?php

use Illuminate\Support\Str;

// The password will include letters, numbers, symbols, and spaces.
// The default length is 32 characters.
$password = Str::password(); // '?;D7zlsMmZ87R0aBmIH.>GU77nagX26U'

$password = Str::password(12); // 'q_2j00<#gr{'
```

## Tip #173 💡: Abbreviate Numbers

Did you know that Laravel ships with the "abbreviate" method, which allows you to format numbers in a human-readable way, with an abbreviation for the units? 🚀

```php
<?php

use Illuminate\Support\Number;

$number = Number::abbreviate(1000); // 1K

$number = Number::abbreviate(489939); // 490K

$number = Number::abbreviate(1230000, precision: 2); // 1.23M
```

## Tip #174 💡: The New "chopStart" and "chopEnd" Methods

Laravel 11.14 introduces 2 new string helpers that allow you to remove characters from the beginning or end of a string 🚀

```php
<?php

use Illuminate\Support\Str;
 
$url = Str::chopStart('http://laravel.com', ['https://', 'http://']); // 'laravel.com'
$url = Str::chopEnd('app/Models/Photograph.php', '.php'); // 'app/Models/Photograph'
```

## Tip #175 💡: Conditionally Adding Rules

When working with dynamic forms, you might want to validate certain inputs only if another input is checked. Laravel ships with the "exclude_if" validation rule, which does exactly that 🚀

```php
<?php

use Illuminate\Support\Facades\Validator;

// 'appointment_date' and 'doctor_name' will not be validated if
// 'has_appointment' is false
$validator = Validator::make($data, [
    'has_appointment' => 'required|boolean',
    'appointment_date' => 'exclude_if:has_appointment,false|required|date',
    'doctor_name' => 'exclude_if:has_appointment,false|required|string',
]);
```

## Tip #176 💡: Swap Multiple Strings

When working with strings, we often need to find and replace occurrences of multiple strings. Laravel ships with an elegant method "swap" to do exactly that 🚀

```php
<?php

use Illuminate\Support\Str;

// The swap method is also available as a fluent method.
$string = Str::swap([
    'Tacos' => 'Burritos',
    'great' => 'fantastic',
], 'Tacos are great!');
 
// $string: Burritos are fantastic!
```

## Tip #177 💡: Shape Your Strings

Since Laravel uses FakerPHP for generating fake data, you can use "lexify" to generate strings in a specific pattern 🚀

```php
<?php

fake()->lexify(); // 'sakh', 'qwei', 'adsj'

fake()->lexify('id-????'); // 'id-xoqe', 'id-pqpq', 'id-zpeu'
```

## Tip #178 💡: Requests Fingerprints

Have you ever needed to code a unique identifier for a request, such as for caching purposes? Laravel ships with the "fingerprint" method that allows you to generate a unique identifier for your requests 🚀

```php
<?php

// Generates a unique fingerprint using the domain, URI, method and IP address
request()->fingerprint() // fbb969117edfa916b86dfb67fd11decf1e336df0
```

## Tip #179 💡: Retrieve and Delete Items From the Session

We often need to retrieve an item from the session and then delete it. While you can use the usual combo of get and forget, Laravel ships with the "pull" method that does exactly that 🚀

```php
<?php

// Instead of this 😫
$value = session()->get('key', 'default-value');

session()->forget('key');

// Do this 😎
$value = session()->pull('key', 'default-value');
```

## Tip #180 💡: Count the Words

Have you ever needed to count the words in a string? Laravel ships with the "countWords" method to do exactly that 🚀

```php
<?php

use Illuminate\Support\Str;
 
Str::wordCount('Okay, this helper is so cool!'); // 6
```

## Tip #181 💡: The "last" and "head" helpers

Did you know that Laravel ships with two helpers, "last" and "head"? They allow you to retrieve the first and last elements of an array 🚀

```php
<?php

$array = [100, 200, 300];

$first = head($array); // 100

$last = last($array); // 300
```

## Tip #182 💡: Check Your Application Enviroment

We often need to check the application environment. While you can use the environment method to do so, Laravel ships with elegant methods "isProduction" and "isLocal" to do exactly that 🚀

```php
<?php

// Could be better 😫
app()->environment() === 'production'

// Okay 😐
app()->environment('production')

// Better 😎
app()->isProduction()

// Also applies for 'local' environment with "isLocal()"
```

## Tip #183 💡: Normalize Validated Data

Have you ever needed to normalize the validated data before using it? Laravel Form Requests come with a "passedValidation" hook which allows you to tweak the validated data 🚀

```php
<?php

/**
 * Handle a passed validation attempt.
 */
protected function passedValidation(): void
{
    $this->replace([
        'name' => ucwords(strtolower($this->name)),
    ]);
}
```

## Tip #184 💡: Filter Null Values

When working with arrays, we sometimes need to filter out null values. Laravel ships with an elegant helper "whereNotNull" to do exactly that 🚀

```php
<?php

use Illuminate\Support\Arr;

$array = [0, null, 'hello, world'];

$filtered = Arr::whereNotNull($array); // [0 => 0, 2 => "hello, world"]
```

## Tip #185 💡: Truncate Long Strings

Sometimes you may want to truncate long descriptions for display. Laravel ships with the "limit" method to do just that, and in the upcoming version, you can preserve whole words for a better UX 🚀

```php
<?php

use Illuminate\Support\Str;

Str::limit('This will be a long description', 10); // This will ...

Str::limit('This will be a long description', 10, ' ( ... )'); // This will ( ... )

// In the upcoming Laravel version we can preserve words as well 😎
$before = Str::limit('We can preserve words', 8) // We can p...

$after = Str::limit('We can preserve words', 8, preserveWords: true) // We can...
```

## Tip #186 💡: The "firstWhere" Method

We often need to get the first record matching a where statement. While "where()" combined with "first()" does the job, Laravel ships with a shortcut "firstWhere()" to do exactly that 🚀

```php
<?php

// Instead of this 😞
$user = User::query()->where('name', 'john')->first();

// Do this 😎
$user = User::query()->firstWhere('name', 'john');
```

## Tip #187 💡: Exclude Middleware

Sometimes you may need to exclude a middleware from a specific route. You can do this using the "withoutMiddleware()" method 🚀

```php
<?php

use App\Http\Middleware\EnsureTokenIsValid;

Route::middleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/', function () {
        // ...
    });

    Route::get('/profile', function () {
        // ...
    })->withoutMiddleware([EnsureTokenIsValid::class]); // Excludes the middleware
});
```

## Tip #188 💡: Work with IPs

Sometimes you may need to work with IP addresses. Laravel uses the HttpFoundation component from Symfony under the hood, which comes with useful IP helpers 🚀

```php
use Symfony\Component\HttpFoundation\IpUtils;

$ipv4 = '192.168.1.1';

IpUtils::checkIp4($ipv4); // true, "checkIp6" is also available
IpUtils::isPrivateIp($ipv4); // true, works with IPv6 as well
IpUtils::anonymize($ipv4); // '192.168.1.0', works with IPv6 as well
```

## Tip #189 💡: Assert JSON Fragments

When testing APIs, we often need to check if the response contains a specific key with the expected data. Laravel ships with the "assertJsonFragment" to do exactly that 🚀

```php
<?php

Route::get('/users', function () {
    return [
        'users' => [
            [
                'name' => 'John Doe',
            ],
        ],
    ];
});

$response->assertJsonFragment(['name' => 'John Doe']);
```

## Tip #190 💡: A Better Content Negotiation

Sometimes you might have multiple response formats that you return. You can use the "getAcceptableContentTypes" method to map your response to what's best for the user 🚀

```php
<?php

request()->getAcceptableContentTypes();

// [
//     0 => "text/html",
//     1 => "application/xhtml+xml",
//     2 => "image/avif",
//     3 => "image/webp",
//     4 => "image/apng",
//     5 => "application/xml",
//     6 => "x/e",
//     7 => "application/signed-exchange",
// ];
```

## Tip #191 💡: Magic Factories Methods

We use factories a lot. Did you know about the for[Relation] and has[Relation] magic methods? You just need to make sure you have the relationship set up in your model and you are set 🚀

```php
<?php

// You need to have User and Post factories.
$user = User::factory()
    // The User model has a hasMany "posts" relationship.
    ->hasPosts(3)
    ->create();

$posts = Post::factory()
    ->count(3)
    // The Post model has a belongsTo "user" relationship.
    // The array is optional. Use it to override an attribute if needed.
    ->forUser([
        'name' => 'John Doe',
    ])
    ->create();
```

## Tip #192 💡: Generate Fake Credit Card Numbers

Since Laravel uses FakerPHP under the hood, you can generate fake credit card numbers for your tests 🚀

```php
<?php

fake()->creditCardNumber(); // '5151791946409422'

fake()->creditCardNumber('Visa'); // 4147628831960830

fake()->creditCardNumber('Visa', formatted: true); // 4147-6288-3196-0830

// You can even generate card types
fake()->creditCardType(); // MasterCard
```

## Tip #193 💡: Generate Currency Code

Since Laravel uses FakerPHP under the hood, you can generate random currency codes. This is really useful for fintech apps 🚀

```php
<?php

fake()->currencyCode();
// 'TND', 'AED', 'SAR', 'KZT'
```

## Tip #194 💡: Type Hinting for Blade

We use Blade a lot, and if I have one thing to complain about, it's type hints. However, we can solve this issue by defining a @php block for all the variables used 🚀

```php
@php
    /* @var App\Models\Flight $flight */
@endphp

<div>
    // Your IDE will provide type hints for the property
    {{ $flight->name }}
</div>
```

## Tip #195 💡: Validate Dates Elegantly

Did you know that when validating dates with Laravel, you can pass strings like "today" or "tomorrow" instead of actual dates? This makes the validation rules much more readable 🚀

```php
<?php

// You can use any string supported by strtotime() date validation
$rules = [
    'start_date' => 'required|date|after:tomorrow',
    'end_date' => 'required|date|after_or_equal:start_date',
    'past_date' => 'required|date|before:yesterday',
    'deadline' => 'required|date|before_or_equal:today',
];
```

## Tip #196 💡: Prevent N+1 Issues

Eager loading can significantly improve performance. Use the "preventLazyLoading" method to ensure all relationships are eager-loaded during development and customize its behavior for violations 🚀

```php
<?php

use Illuminate\Database\Eloquent\Model;

// In your AppServiceProvider
public function boot(): void
{
    // This ensures lazy loading is prevented in your dev environment
    Model::preventLazyLoading(! $this->app->isProduction());

    // You can customize the behavior for lazy loading violations
    Model::handleLazyLoadingViolationUsing(function (Model $model, string $relation) {
        $class = $model::class;

        info("Attempted to lazy load [{$relation}] on model [{$class}].");
    });
}
```

## Tip #197 💡: Check if valid JSON

We often need to check if a given string is valid JSON. Laravel provides an elegant method, "isJson", to help you with this. It uses the new "json_validate" function in PHP 8.3, and "json_decode" for earlier versions 🚀

```php
<?php

use Illuminate\Support\Str;

// The cool thing is, on the day you upgrade to PHP 8.3, if you haven't already,
// you will automatically be using the new "json_validate()" function
// instead of "json_decode()"

Str::isJson('[1,2,3]'); // true

Str::isJson('{"first": "John", "last": "Doe"}'); // true

Str::isJson('{first: "John", last: "Doe"}'); // false
```

## Tip #198 💡: Count words occurances

Ever needed to count the occurrences of a word in a sentence? Laravel ships with the "substrCount" method to do exactly that 🚀

```php
<?php

use Illuminate\Support\Str;

Str::substrCount('My name is Bond, James Bond', 'Bond'); // 2
```

## Tip #199 💡: Shortcuts for Dropping Columns

Need to drop some framework-specific columns? You don’t have to specify them manually, Laravel provides shortcuts to do exactly that 🚀

```php
<?php

Schema::table('users', function (Blueprint $table) {
    $table->dropColumn(['created_at', 'updated_at']);
    $table->dropTimestamps();

    $table->dropColumn(['deleted_at']);
    $table->dropSoftDeletes();

    $table->dropColumn(['remember_token']);
    $table->dropRememberToken();

    $table->dropColumn(['morphable_id', 'morphable_type']);
    $table->dropMorphs('morphable');
});
```

## Tip #200 💡: Deduplicate Characters

Laravel v11.20 introduces a new "deduplicate" method which allows you to remove duplicates from spaces or any character you choose 🚀

```php
<?php

Str::deduplicate('Laravel Framework') // Laravel Framework
Str::deduplicate('Laravel Frameworkkkkk', 'k') // Laravel Framework
```

## Tip #201 💡: Ensure Env Keys Exist

If you want to be absolutely sure that a key exists in your .env file, use the "getOrFail()" method. It will throw a runtime exception if the key is missing. This is really useful for API keys 🚀

```php
<?php

use Illuminate\Support\Env;

return [
    'postmark' => [
        'token' => Env::getOrFail('POSTMARK_TOKEN'),
    ],
];

// If the POSTMARK_TOKEN key is missing from your .env file,
// an exception will be thrown with the message:
// "Environment variable [POSTMARK_TOKEN] has no value."
```

## Tip #202 💡: Customize the Redirect Location

We often use form requests for validation. Did you know you can customize the redirect location upon failure? You are never limited to redirecting users back to the previous page 🚀

```php
<?php

// You can use a path
protected $redirect = '/dashboard';

// Or named routes
protected $redirectRoute = 'dashboard';

// Or even use actions
protected $redirectAction = [DashboardController::class, 'index'];
```

## Tip #203 💡: Invisible Columns

If you are using MySQL/MariaDB as your database, you can leverage invisible columns. These columns remain hidden in 'SELECT * ' statements, which is perfect for handling sensitive information and precomputed data 🚀

```php
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

// Note that this works for MariaDB/MySQL
Schema::table('users', function (Blueprint $table) {
    // Useful for storing secrets
    $table->string('secret')->nullable()->invisible();

    // Or for precomputed columns
    $table->string('posts_count')->default(0)->invisible();
});

// The attribute is not even present on the Eloquent object
$user = User::first();
$user->secret; // null

// You have to explicitly select the column
$user = User::select('secret')->first();
$user->secret; // SUEGQs8klF30CLSi
```

## Tip #204 💡: Schedule Commands on Specific Environments

Did you know that you can schedule your commands for specific environments? If you frequently use "schedule:work", you'll find this helpful for excluding commands from the dev environment 🚀

```php
<?php

use Illuminate\Support\Facades\Schedule;

Schedule::command('newsletter:send')
    ->environments('production') // You can also pass an array of environments
    ->weekends();
```

## Tip #205 💡: Get Bearer Tokens Elegantly

Building an API with Laravel? You can retrieve the bearer token using the "bearerToken" method on the request object without having to manually parse it 🚀

```php
<?php

// Instead of this 😞
$token = substr(request()->header('Authorization'), 7);

// Do this 😊
$token = request()->bearerToken();
```

## Tip #206 💡: Skip Relationships in Queues

When passing a model to a job, consider using the "WithoutRelations" attribute to skip serializing the relationships if you don't need them. This will keep the payload minimal and memory efficient 🚀

```php
<?php

use Illuminate\Queue\Attributes\WithoutRelations;

public function __construct(
    #[WithoutRelations]
    public Podcast $podcast
) {}
```

## Tip #207 💡: Generated Columns

Did you know that Laravel can handle generated columns in migrations out of the box? No need to write raw SQL in your migration to create these columns 🚀

```php
<?php

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('products', function (Blueprint $table) {
            // ...

            $table->decimal('unit_price');
            $table->integer('quantity');

            // Works on MariaDB / MySQL / PostgreSQL / SQLite
            $table->decimal('full_price')->storedAs('unit_price * quantity');

            // ...
        });
    }
};
```

## Tip #208 💡: Disable Model Events When Seeding

In most cases, when seeding the database, you don't need to fire model events. You can use the "WithoutModelEvents" trait to mute those events, making your seeders slightly faster 🚀

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Database\Console\Seeds\WithoutModelEvents;

class DatabaseSeeder extends Seeder
{
    use WithoutModelEvents; // Will mute all model events

    public function run(): void
    {
        $this->call([
            UserSeeder::class,
            PostSeeder::class,
            CommentSeeder::class,
        ]);
    }
}
```

## Tip #209 💡: Lazily Refresh Your Database

We often need to refresh the database when testing the code. In such cases, you can lazily refresh your database, so migrations are run only when you are hitting the db. This will help speed up your tests 🚀

```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\LazilyRefreshDatabase;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    use RefreshDatabase;
    // use LazilyRefreshDatabase;

    public function test_basic_example(): void
    {
        $response = $this->get('/');

        // ...
    }
}
```

## Tip #210 💡: Use Default Models

When working with "hasOne" or "belongsTo" relationships, we often check whether they are nullable before accessing their properties. In such cases, you can use default models to ensure you never get null values 🚀

```php
<?php

public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault([
        'name' => 'Guest Author',
    ]);
}

// Now, instead of checking this every time
Post::first()->user->name ?? 'Guest Author'; // 'Guest Author'

// You can be confident that when a user is null, the default will be used
Post::first()->user->name; // 'Guest Author'
```

## Tip #211 💡: Permanently Delete Soft-Deleted Models

Sometimes you may want to permanently remove soft-deleted models. You can use "forceDelete()" for that, or the new "forceDestroy()" method introduced in Laravel v11.21 🚀

```php
<?php

// Laravel < v11.20
$flight = Flight::find(1);
$flight->forceDelete();

// Laravel v11.21
Flight::forceDestroy(1); // You can also pass an array of IDs
```

## Tip #212 💡: Exclude Validated Input

Sometimes, you may want to exclude an input from the validated array. Instead of manually unsetting it, you can use the "exclude" rule, which does exactly that 🚀

```php
<?php

public function store(Request $request): RedirectResponse
{
    $validated = $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'captcha' => 'required|exclude',
    ]);

    dd($validated); // only 'title' and 'body' are set on $validated
}
```

## Tip #213 💡: Filter Only Real Emails

Tired of high bounce rates from invalid emails? Laravel comes with the "dns" validation rule to ensure you're getting real emails. It won't magically fix the issue, but it definitely improves deliverability  🚀

```php
<?php

public function store(Request $request): RedirectResponse
{
    $validated = $request->validate([
        'email' => 'required|email:dns'
    ]);
}

// In your tests, make sure to replace fake()->email() with fake()->freeEmail()
// to avoid flaky tests. freeEmail() will always generate an email with
// valid DNS records.
```

## Tip #214 💡: Better Error Messages for Arrays

When validating arrays, it's a better UX to tell the user which item has failed rather than throwing a generic message. For this, you can use the ":index" and ":position" placeholders 🚀

```php
<?php

use Illuminate\Support\Facades\Validator;

$validator = Validator::make($request->all(), [
    'photos.*.description' => 'required',
], [
    'photos.*.description.required' => 'Please describe photo #:position.',
]);

// This will result in "Please describe photo #1".
// You can use :index to start from 0, and if you have deeply nested arrays,
// you can use :second-index, :second-position, :third-index, :third-position
```

## Tip #215 💡: The "checked" Blade Directive

Often, we need to conditionally mark an input as checked. While this can be done manually, Laravel provides a cool blade directive "checked" to do exactly that 🚀

```php
<input type="checkbox" name="active" value="active"
    {{ old('active', $user->active) ? 'checked' : '' }}
    @checked(old('active', $user->active)) />
```

## Tip #216 💡: Filter Falsy Values

We've all used the "filter" method on collections, but did you know that if no callback is passed, it will filter out all the falsy values? 🚀

```php
<?php

$collection = collect([1, 2, 3, null, false, '', 0, []]);
$collection->filter()->all(); // [1, 2, 3]
```

## Tip #217 💡: Get Only Trashed Records

When working with soft-deleted models, you may need to get only the trashed records. While you can manually filter the query using the "deleted_at" column, there's an "onlyTrashed()" method to do exactly that 🚀

```php
<?php

// Instead of manually specifying 'deleted_at'
$trashedUsers = User::query()->whereNotNull('deleted_at')->get();

// You can use onlyTrashed(), which is both easier and more readable
$trashedUsers = User::query()->onlyTrashed()->get();
```

## Tip #218 💡: A Better Implode

We've all used PHP's "implode" function, but did you know about the "join" helper? It does the same thing but also allows you to customize the last separator 🚀

```php
<?php

// We've all done this, the regular implode()
collect(['a', 'b', 'c'])->join(', ', ''); // 'a, b, c'

// But did you know you can specify the last separator?
collect(['a', 'b', 'c'])->join(', ', ', and '); // 'a, b, and c'

// And it's smart enough to handle edge cases
collect(['a'])->join(', ', ', and '); // 'a'
collect([])->join(', ', ', and '); // ''
```

## Tip #219 💡: Bulk Dispatch

While Laravel offers batches to dispatch jobs, sometimes you just want to fire and forget. In that case, you can bulk dispatch instead of doing it one by one 🚀

```php
<?php

// If you are using the database driver, this will result in N queries
$users->each(fn(User $user) => GenerateInvoice::dispatch($user));

// This will result in a single query
$jobs = $users->map(fn(User $user) => new GenerateInvoice($user))->toArray();
Queue::bulk($jobs);
```

## Tip #220 💡: Safer Passwords for Users

Users tend to use the same password for all websites, which puts them in danger if their password has been leaked. You can make sure that the user inputs an uncompromised password using the "uncompromised" rule 🚀

```php
<?php

// Under the hood, uncompromised uses the Have I Been Pwned public API
// with k-anonymity, where only a prefix of 5 characters of the hash will be sent over
Password::min(8)
    ->mixedCase()
    ->numbers()
    ->letters()
    ->uncompromised()
```

## Tip #221 💡: The  Conditionable Trait

Do you find yourself writing multiple if statements to call different methods on a class? Consider making your class "conditionable" by using Laravel's "Conditionable" trait 🚀

```php
<?php

use Illuminate\Support\Traits\Conditionable;

class NotificationService
{
    use Conditionable;

    public function sendEmailNotification()
    {
        echo "Email notification sent.";
    }

    public function sendSmsNotification()
    {
        echo "SMS notification sent.";
    }
}

$notificationService = new NotificationService;

$shouldSendEmail = true;
$shouldSendSms = false;

// Instead of this
if ($shouldSendEmail) {
    $notificationService->sendEmailNotification();
}

if ($shouldSendSms) {
    $notificationService->sendSmsNotification();
}

// You can do this
$notificationService
    ->when($shouldSendEmail, fn(NotificationService $service) => $service->sendEmailNotification())
    ->when($shouldSendSms, fn(NotificationService $service) => $service->sendSmsNotification());
```

## Tip #222 💡: Run Seeders During Tests

Did you know you can run database seeders within your tests? This allows you to keep your tests clean by moving setup logic to seeders 🚀

```php
<?php

use function Pest\Laravel\seed;

uses(LazilyRefreshDatabase::class);

// For feature tests with PHPUnit, you can simply call $this->seed()
test('orders can be created', function () {
    // Run the DatabaseSeeder ...
    seed();

    // Run a specific seeder ...
    seed(OrderStatusSeeder::class);

    // Run an array of specific seeders ...
    seed([
        OrderStatusSeeder::class,
        TransactionStatusSeeder::class,
    ]);
});
```

## Tip #223 💡: Faker's "randomDigitNot()"

When defining factories, you may sometimes need to generate a random digit while excluding a specific one. Since Laravel uses FakerPHP under the hood, you can use "randomDigitNot" to do exactly that 🚀

```php
<?php

// This will return a digit from 0 to 9
fake()->randomDigit();

// You can also exclude a digit
fake()->randomDigitNot(3); // This will return any digit from 0 to 9, except 3
```

## Tip #224 💡: Add Multiple Columns After Another

Did you know that you can add multiple columns after a specific column using the "after" method?  🚀

```php
<?php

Schema::table('users', function (Blueprint $table) {
    $table->after('password', function (Blueprint $table) {
        $table->string('address_line1');
        $table->string('address_line2');
        $table->string('city');
    });
});
```

## Tip #225 💡: Auto Capitalize Your Translations

Did you know that not only can you parameterize your translation strings, but you can also auto-capitalize them? 🚀

```php
<?php

// in your en/messages.json
return [
    'welcome' => 'Welcome, :NAME',
    'goodbye' => 'Goodbye, :NAME',
];

// This will auto-capitalize it for you
echo __('messages.welcome', ['name' => 'dayle']); // Welcome, DAYLE
echo __('messages.goodbye', ['name' => 'dayle']); // Goodbye, Dayle
```

## Tip #226 💡: Connect to Your DB via Artisan

Ever needed to quickly connect to your database via the CLI? There’s an Artisan command to do exactly that! 🚀

```php
// This will connect you directly to the database
php artisan db

// Have multiple connections? No worries you can specify which one to use
php artisan db mysql
```

## Tip #227 💡: Cache Accessor Result

When working with accessors, objects are cached by default. However, if you have a computationally intensive accessor for a primitive value, like a string, you can enable caching for it using the "shouldCache()" method 🚀

```php
<?php

protected function hash(): Attribute
{
    return Attribute::make(
        get: fn (string $value) => bcrypt(gzuncompress($value)),
        shouldCache: true
    );
}
```

## Tip #228 💡: Ensure Proper Table Name in Migrations

When creating migrations, we sometimes don't format the name in a way that Laravel can understand to fill in the table name. However, you can always specify it manually 🚀

```php
<?php

/**
 * If you want Laravel to fill in the Schema::table() for you,
 * use the --table argument when generating the migration.
 */
php artisan make:migration "tweak_users_table" --table=users

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('users', function (Blueprint $table) {
            //
        });
    }

    public function down(): void
    {
        Schema::table('users', function (Blueprint $table) {
            //
        });
    }
};
```

## Tip #229 💡: On-Demand Notifications

Sometimes, you may want to send notifications to "anonymous" users who are not yet stored in the database. Laravel's "route" method allows you to do exactly that 🚀

```php
<?php

use Illuminate\Broadcasting\Channel;
use Illuminate\Support\Facades\Notification;

Notification::route('mail', 'john@example.com')
    ->route('twilio', '9419287384')
    ->route('slack', '#slack-channel')
    ->notify(new InvoicePaid($invoice));
```

## Tip #230 💡: Cool Artisan DB Commands

Have you ever needed to check if your db connection is working as expected? Or wondered how many open connections you have? Maybe you want to know the total size of a db? Well, Artisan comes with some cool commands to do exactly that 🚀

```php
# This command allows you to inspect a table
php artisan db:table

# This command checks if your db connections are working
php artisan db:monitor

# This command displays the db version, total size, open connections, and more
php artisan db:show
```

## Tip #231 💡: The Pipeline Helper

Often, we need to process input through multiple steps, such as applying request filters to a query or cleaning data in a multi-stage chain. When you find yourself in such cases, use Laravel's Pipeline helper 🚀

```php
<?php
$user = Pipeline::send($user)
    ->through([
        // Those are invokable classes
        GenerateProfilePhoto::class,
        ActivateSubscription::class,
        SendWelcomeEmail::class,
    ])
    ->then(fn(User $user) => $user);

// Example of the GenerateProfilePhoto class
class GenerateProfilePhoto
{
    public function __invoke(User $user, Closure $next)
    {
        // Execute the logic to generate the profile photo

        return $next($user);
    }
}
```

## Tip #232 💡: Skip Jobs When Batch is Cancelled

When working with batched jobs, it's best to check if the batch is canceled before running the job, and you don't have to do it manually because the "SkipIfBatchCancelled" middleware does exactly this 🚀

```php
<?php
use App\Jobs\ImportContacts;
use Illuminate\Support\Collection;

// A batched job
public function handle(): void
{
    if ($this->batch()->cancelled()) {
        return;
    }
}

public function middleware(): array
{
    return [new SkipIfBatchCancelled];
}
```

## Tip #233 💡: The "whereLike" method

We often use "where like" statements in our apps. Did you know Laravel ships with a "whereLike" method, which takes it a step further by allowing the like statement to be case insensitive? 🚀

```php
<?php
// Instead of this
User::query()->where('name', 'like', 'Jo%')->get();

// You can do this
User::query()->whereLike('name', 'Jo%')->get();

// You can even specify whether it should be case sensitive
User::query()->whereLike('name', 'Jo%', caseSensitive: true)->get();
// Query: select * from `users` where `name` like binary 'Jo%'
```

## Tip #234 💡: The "tap" Helper

How many times have you needed to return an object right after a basic action? Yes many times I know. The "tap" helper allows you to do just that 🚀

```php
<?php
public function markAsPaid(): self
{
    // Not bad
    $this->update(['status' => 'paid']);
    return $this;

    // Better
    return tap($this)->update(['status' => 'paid']);
}
```

## Tip #235 💡: Using Multiple IDs and Specific Columns with "find"

We often use "find()", but did you know you can pass an array of IDs and also select specific columns? 🚀

```php
<?php
// You can pass an array of ids, an select specific columns
User::find(id: [1, 2], columns: ['email']);
// select `email` from `users` where `users`.`id` in (1, 2)
```

## Tip #236 💡: Execute Tasks Concurrently

Starting from Laravel v11.23, you can execute tasks concurrently. This can speed things up when you have independent tasks that can be run simultaneously 🚀

```php
<?php
use App\Services\Metrics;
use Illuminate\Support\Facades\Concurrency;

Concurrency::defer([
    fn () => Metrics::report('users'),
    fn () => Metrics::report('orders'),
]);
```

## Tip #237 💡: Load Relationship Count on the Fly

When working with models, you might need the count of a relationship. If you forget to eager load it, you can always use "loadCount" to load the count on the fly 🚀

```php
<?php
$user = User::first();

// Load the count of related posts on the fly
$user->loadCount('posts');

// You can even constraint it
$user->loadCount('posts', fn(Builder $query) => $query->whereNull('published'));

// Access the count via <relationship>_count
$user->posts_count
```

## Tip #238 💡: The Required If Accepted Rule

When validating forms, sometimes you want to conditionally require a field if another one was filled. Laravel ships with "required_if_accepted" to do exactly that 🚀

```php
<?php
// subscribe_to_newsletter must be true for the email field to be required
$validator = Validator::make($data, [
    'subscribe_to_newsletter' => 'boolean',
    'email' => 'required_if_accepted:subscribe_to_newsletter|email',
]);
```

## Tip #239 💡: Ask for Confirmation in Commands

Did you know that you can request confirmation for risky commands before executing them? You can do this using the "confirm" method 🚀

```php
<?php

// Displays the old Laravel prompt view
$this->confirm('Do you want to continue?');

// With "components", it displays the modern Laravel prompt view
$this->components->confirm('Do you want to continue?');

// In your terminal, you should see
/*
 * Do you want to continue? (yes/no) [no]
 *
 */
```

## Tip #240 💡: Skip Jobs

Ever needed to skip job execution? While you can do it manually, Laravel ships with a "Skip" middleware to do exactly that 🚀

```php
<?php

use Illuminate\Queue\Middleware\Skip;

public function middleware(): array
{
    return [
        Skip::when($someCondition),
        // You can also use `unless` and pass a closure
        Skip::unless(fn () => $this->shouldNotSkip()),
    ];
}
```

## Tip #241 💡: A Better dd()

When debugging an Eloquent query, we often use "dd()" to check the result. Did you know you can just chain it directly? 🚀

```php
<?php

// This is okay 😊
$user = User::all();
dd($users);

// This is better 🔥
User::all()->dd();
```

## Tip #242 💡: The "today()" Helper

Sometimes you may want to get today's date. While you can do this in multiple ways, Laravel ships with a readable "today()" helper to do exactly that. You can pass timezones and chain other helpful methods as well 🚀

```php
<?php

// Returns today's date
$today = today();

// But you can do more 👀
today()->isWeekend(); // true
today()->isWeekday(); // false

today()->isSaturday(); // true, and you can check for all days
today()->isMidday(); // false

// You can also pass timezones 🔥
today($timezeone);
```

## Tip #243 💡: Show Model Infos

When a model grows, it can be hard to check all the relationships, including those from third-party packages, the registered events, and the observers. When that's the case, you can use the "model:show" command instead 🚀

```shell
php artisan model:show User

# This will list: The table, attributes, relationships, events and observers.
```

## Tip #244 💡: Send All Mail to a Specific Email

Ever needed to send all mails to a single address? Whether you're working in a specific environment or managing a small project with just one contact email, you can use the "alwaysTo" method to do exactly that 🚀

```php
<?php

use Illuminate\Support\Facades\Mail;

public function boot(): void
{
    if (app()->environment('staging')) {
        Mail::alwaysTo('company@domain.com');
    }
}
```

## Tip #245 💡: Move Column to First Position

Did you know you can move a column to the first position in your table, even if it was added later on? Use the "first()" method to do that 🚀

```php
<?php

Schema::table('posts', function (Blueprint $table) {
    $table->string('uuid')->first();
});
```

## Tip #246 💡: Use the "set" Method in Factories

Sometimes you may want to override or pass a single attribute to your factory. While you can pass a whole array for that, you can use the "set" method, which does exactly that 🚀

```php
<?php

// Rather than passing an array for just a single attribute
User::factory()->create(['email' => 'john@example.com']);

// You can use set() 🔥 Plus, It's more readable 😊
User::factory()
    ->set('email', 'john@example.com')
    ->create();
```

## Tip #247 💡: The New Flexible Cache Method

Starting with Laravel v11.23.0, you can use the new "flexible" method for caching. If you've struggled with revalidating your cache before it expires, make use of it 🚀

```php
<?php

// If a request is made between the 5 and 10s interval,
// the cache will be recalculated 🔥 The cache will only expire after 10 seconds
// if no requests are made within the 5 to 10s window.
$value = Cache::flexible('users', [5, 10], fn () => User::all());
```

## Tip #248 💡: Higher Order Messages

When working with Laravel collections, remember that they come with higher order messages, which are shortcuts for the most common actions 🚀

```php
<?php

use App\Models\User;
 
$users = User::where('votes', '>', 500)->get();
 
// While you can do this
$users->each(fn(User $user) => $user->markAsVip());

// You can simplify it to this using higher order messages 🔥
$users->each->markAsVip();
```

## Tip #249 💡: A Better Pluck

We often need to get the IDs of some models. While you can use the "pluck()" method to do this, you can also use "modelKeys()", which reads better and won't break if you change the primary key at any point 🚀

```php
<?php

use App\Models\User;

// Instead of this
$ids = User::all()->pluck('id');

// You can do this 🔥
$ids = User::all()->modelKeys();
```

## Tip #250 💡: Laravel's Signed Routes

A reminder that Laravel ships with signed routes. They are perfect for magic login links, temporary access, and one-time actions like unsubscribing users, all in a safe way by making sure the URL isn't tampered with 🚀

```php
<?php

use Illuminate\Support\Facades\URL;

// This will generate a signed route with an absolute path
URL::signedRoute('unsubscribe', ['user' => 1]);
// http://local.test/unsubscribe?user=1&signature=753c19...6d18baa59c3b5851

// You can choose to generate a relative path instead
URL::signedRoute('unsubscribe', ['user' => 1], absolute: false);
// unsubscribe?user=1&signature=45cb85167c31085...05641509acb600

// You can also make it temporary
URL::temporarySignedRoute(
    'unsubscribe', now()->addMinutes(30), ['user' => 1], absolute: false
);
// unsubscribe?expires=1727640587&user=1&signature=f71fa139fe2...515fe111a8
```

## Tip #251 💡: Get the Age of a Date

Since Laravel uses Carbon under the hood, you can easily get the age of a parsed date 🚀

```php
<?php

use Illuminate\Support\Carbon;

// Get the age from a given date
$laravelsAge = Carbon::parse('01-06-2011')->age;

// This also works with date columns
$age = User::first()->birthday->age;
```

## Tip #252 💡: Latest of Many

Have you ever needed to get the latest record from a one to many relationship? While you can use subqueries to do this, Laravel already ship with the "latestOfMany" method to do exactly that 🚀

```php
<?php

class User extends Model
{
    public function latestOrder()
    {
        return $this->hasOne(Order::class)->latestOfMany('created_at');
    }
}

// You can now get the latest order
$latestOrder = $user->latestOrder;

// It can also be eager loaded
$users = User::with('latestOrder')->get();
```

## Tip #253 💡: A Cleaner Eager Loading Syntax

Sometimes, when need to eager load nested relationships, and for that we the use dot notation. But did you know you can also pass nested arrays? 🚀

```php
<?php

// Instead of this
$books = Book::with([
    'author.contacts',
    'author.publisher'
])->get();

// You can pass a nested array
$books = Book::with([
    'author' => [
        'contacts',
        'publisher'
    ]
])->get();
```

## Tip #254 💡: Get the Closest and Farthest Dates

Ever needed to get the closest or farthest of two dates compared to a given date? Since Laravel uses Carbon under the hood, you can do that with the "closest" and "farthest" methods 🚀

```php
<?php

use Illuminate\Support\Carbon;

$date = Carbon::parse('2024-05-15');
$date1 = Carbon::parse('2024-01-01');
$date2 = Carbon::parse('2024-05-16');

// You can now check the closest or farthest date
$date->closest($date1, $date2); // 2024-05-16
$date->farthest($date1, $date2); // 2024-01-01
```

## Tip #255 💡: Define Command Aliases

We've all created custom Artisan commands for different purposes. While it's great to have an expressive signature, if you're using the command often, you can define an alias for it 🚀

```php
<?php

class BillReminder extends Command
{
    protected $signature = 'reminder:bill';

    protected $aliases = ['rb']; // Alias here

    // ...
}

// You can run the command with:
// php artisan reminder:bill

// But now you can also use the alias
// php artisan rb
```

## Tip #256 💡: File Checksum

Ever needed to generate a checksum for a file to check if it has been tampered with or simply to track changes over time? Laravel ships with the "checksum" method to do exactly that! 🚀

```php
<?php

// You can choose any other algo (returned by hash_algos())
Storage::checksum('/path/to/file', ['checksum_algo' => 'sha1']);
```

## Tip #257 💡: Validate Image Dimensions

Ever needed to validate the dimensions of an image, like an avatar? Laravel comes with built-in validation rules for this. You can use the "dimensions" rule to build your validation logic 🚀

```php
<?php

use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;
 
Validator::make($data, [
    'avatar' => [
        'required',
        Rule::dimensions()->maxWidth(1000)->maxHeight(500)->ratio(3 / 2),
    ],
]);
```

## Tip #258 💡: The "shortRelativeDiffForHumans" Method

I'm sure you've used the "diffForHumans" method to get a human-readable date. But did you know you can get a shorter version using the "shortRelativeDiffForHumans" method? 🚀

```php
<?php

now()->subDays(5)->diffForHumans(); // 5 days ago

// Need a shorter version? No problem
now()->subDays(5)->shortRelativeDiffForHumans(); // 5d ago
```

## Tip #259 💡: The "finish" Helper

Sometimes, you might need to ensure that a string ends with a specific character, like a slash or a dot. Laravel ships with the "finish" helper to do exactly that 🚀

```php
<?php

use Illuminate\Support\Str;
 
Str::finish('this/string', '/'); // this/string/

Str::finish('this/string/', '/'); // this/string/

```

## Tip #260 💡: The "diffInDaysFiltered" Method

Ever needed to count the days between 2 dates while filtering some based on a condition? Since Laravel uses Carbon under the hood, you can use "diffInDaysFiltered" to doexactly that 🚀

```php
<?php

$start = now();
$end = now()->addDays(10);

// This will count only weekdays between the two dates 🔥
$weekdays = $start->diffInDaysFiltered(fn (Carbon $date) => !$date->isWeekend(), $end);

$weekdays; // 8
```

## Tip #261 💡: The "prohibitedIf" Rule

Sometimes, you may want to "prohibit" a field from having data based on a condition, such as the presence of another field. Laravel ships with the "prohibitedIf" rule to do exactly that 🚀

```php
<?php

use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;
 
// You can pass a bool
Validator::make($request->all(), [
    'role_id' => Rule::prohibitedIf($request->user()->is_admin),
]);
 
// Or a closure if the logic is more complex
Validator::make($request->all(), [
    'role_id' => Rule::prohibitedIf(fn () => $request->user()->is_admin),
]);
```

## Tip #262 💡: URI Templates

Since Laravel uses Guzzle under the hood, you can use URI templates with Laravel's HTTP Client by calling the "withUrlParameters" method 🚀

```php
<?php

// This follows the URI Template specification (RFC 6570).
Http::withUrlParameters([
    'endpoint' => 'https://laravel.com',
    'page' => 'docs',
    'version' => '11.x',
    'topic' => 'validation',
])->get('{+endpoint}/{page}/{version}/{topic}');
```

## Tip #263 💡: The "isBirthday" Method

Ever needed to check whether a date is someone's birthday? Since Laravel uses Carbon under the hood, you can use the "isBirthday" method to do exactly that 🚀

```php
<?php

use Illuminate\Support\Carbon;

$born = Carbon::createFromDate(1987, 4, 23);

$noCake = Carbon::createFromDate(2014, 9, 26);
$yesCake = Carbon::createFromDate(2014, 4, 23);

$born->isBirthday($noCake); // false
$born->isBirthday($yesCake); // true
```

## Tip #264 💡: Global Request Middleware

When consuming APIs, you may want to apply a specific user agent to all your outgoing requests. This can make debugging easier later on. Laravel ships with global request middleware to do exactly that 🚀

```php
<?php

use Illuminate\Support\Facades\Http;
 
Http::globalRequestMiddleware(fn ($request) => $request->withHeader(
    'User-Agent', 'Example Application/1.0'
));
 
Http::globalResponseMiddleware(fn ($response) => $response->withHeader(
    'X-Finished-At', now()->toDateTimeString()
));
```

## Tip #265 💡: The "headline" Method

Ever needed to convert a string to a title? Laravel ships with the "headline" method to do exactly that 🚀

```php
<?php

Str::headline('a_cool_title'); // A Cool Title

Str::headline('EmailNotificationSent'); // Email Notification Sent

// Also works with fluent strings 🔥
str('a_cool_title')->headline();
```

## Tip #266 💡: The New "optimizes" Method

Laravel v11.27.1 introduced a new service provider method called "optimizes". You can now define commands to run with the "optimize" and "optimize:clear" commands, such as "filament:optimize" or other custom commands 🚀

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        $this->optimizes(
            optimize: 'filament:optimize',
            // Defaults to the service provider name without "ServiceProvider" suffix
            key: 'filament' 
        );

        // ...
    }
}
```

## Tip #267 💡: The "insertGetId" Method

Have you ever found yourself needing the ID of a newly inserted row? Laravel ships with the "insertGetId" method to do exactly that 🚀

```php
<?php

$id = DB::table('users')->insertGetId(
    ['email' => 'john@example.com', 'votes' => 0]
);

dd($id); // The ID of the newly inserted row
```

## Tip #268 💡: The "ddRawSql" Method

When debugging queries, we often use "dd()" or "toSql()", but did you know you can use "ddRawSql" to get the raw SQL with all bindings substituted? 🚀

```php
<?php

DB::table('users')->where('votes', '>', 100)->ddRawSql();
// "select * from `users` where `votes` > 100"
```

## Tip #269 💡: The "expectsJson" Method

If you have multiple clients and some of them only expect JSON, instead of manually checking the Accept header, you can use Laravel's built-in "expectsJson" method 🚀

```php
<?php

// True when the "Accept: application/json" header is present
if ($request->expectsJson()) {
    // ...
}
```

## Tip #270 💡: The "mergeIfMissing" Method

Sometimes you may want to add additional input to the current request. While you can merge it manually, Laravel already ships with the "mergeIfMissing" method to do exactly that 🚀

```php
<?php

$request->mergeIfMissing(['votes' => 0]);
```

## Tip #271 💡: Email Command Failures Automatically

Did you know that Laravel ships with the "emailOutputOnFailure" method which automatically sends the output of a failed command to your email? 🚀

```php
<?php

use Illuminate\Support\Facades\Schedule;

Schedule::command('report:generate')
         ->daily()
         ->emailOutputOnFailure(config('contact.support'));
```

## Tip #272 💡: Avoid Duplicate Queries

We often eager load relationships manually using the "load" method. While this works, it can result in duplicate queries when the relationship is already loaded. This can be avoided by using the 'loadMissing' method 🚀

```php
<?php

// If the relationships are already loaded, this will result in duplicate queries
$users->load(['comments', 'posts']);

// This will not query the DB if the relationships are already there
$users->loadMissing(['comments', 'posts']);
```

## Tip #273 💡: The New "CollectedBy" Attribute

As of Laravel v11.28, instead of overriding the "newCollection" method, you can now use the new "CollectedBy" attribute to specify a custom collection for your model 🚀

```php
<?php

use Illuminate\Database\Eloquent\Attributes\CollectedBy;

#[CollectedBy(PostCollection::class)]
class Post
{
    public function newCollection(array $models = [])
    {
        return new PostCollection($models);
    }
}
```

## Tip #274 💡: Check if a User is a Guest

We often need to check if a user is authenticated, and for that, we use the "check" method. But did you know that when you need to check if a user is a guest, you can use the "guest" method instead? 🚀

```php
<?php

// This is good
if (! Auth::check()) {
    // The user is a guest
}

// This is more readable
if (Auth::guest()) {
    // The user is a guest
}
```

## Tip #275 💡: Aggregate Functions

We often use "withCount" when working with relationships, but did you know other aggregate functions are available out of the box? For example, you can also use "sum", "min", and "max" functions 🚀

```php
<?php

Post::withSum('comments', 'votes')->first(); // $post->comments_sum_votes

Post::withMax('comments', 'votes')->first(); // $post->comments_max_votes

Post::withAvg('comments', 'votes')->first(); // $post->comments_avg_votes

Post::withMin('comments', 'votes')->first(); // $post->comments_min_votes

Post::withExists('comments')->first(); // $post->comments_exists_votes
```

## Tip #276 💡: The "sometimes" Validation Rule

Have you ever needed to validate a field only if it's present, but skip it when it’s not? Laravel ships with the "sometimes" validation rule to do exactly that 🚀

```php
<?php

$validator = Validator::make($data, [
    // The email will only be validated if it is present in the $data array
    'email' => ['sometimes', 'email'],
]);
```

## Tip #277 💡: Generate Fake Credit Card Details

Since Laravel uses FakerPHP to generate fake data, you can use the "creditCardDetails" method to generate fake credit cards for your tests 🚀

```php
<?php

fake()->creditCardDetails();

// [
//   "type" => "Visa Retired"
//   "number" => "4485338826091"
//   "name" => "Ron Effertz"
//   "expirationDate" => "07/25"
// ]

// If you want to generate potentially expired cards you can pass false
fake()->creditCardDetails(valid: false);

// [
//   "type" => "MasterCard"
//   "number" => "5125073907573220"
//   "name" => "Dr. Thelma Koelpin PhD"
//   "expirationDate" => "08/22"
// ]
```

## Tip #278 💡: Sort Arrays Recursively

Have you ever needed to sort an array recursively, including all its sub arrays? Laravel ships with the "sortRecursive" method to do exactly that 🚀

```php
<?php

use Illuminate\Support\Arr;
 
$array = [
    ['PHP', 'Ruby', 'JavaScript'],
    ['one' => 1, 'two' => 2, 'three' => 3],
];
 
Arr::sortRecursive($array);
 
/*
    [
        ['one' => 1, 'three' => 3, 'two' => 2],
        ['JavaScript', 'PHP', 'Ruby'],
    ]
*/

// You can also sort in a desc order
Arr::sortRecursiveDesc($array);
```

## Tip #279 💡: The "data_forget" Helper

Have you ever needed to unset data from nested arrays? It can get messy (and ugly) quickly. Laravel ships with the "data_forget" helper to do exactly that using the dot notation 🚀

```php
<?php
 
$data = ['products' => ['desk' => ['price' => 100]]];
 
data_forget($data, 'products.desk.price'); // ['products' => ['desk' => []]]
```

## Tip #280 💡: The "hasHeader" Method

Have you ever needed to check for a header? While you can do it manually, Laravel ships with the "hasHeader" method to do exactly that 🚀

```php
<?php
 
if ($request->hasHeader('X-Header-Name')) {
    // ...
}
```

## Tip #281 💡: The "aware" Blade Directive

Sometimes you might want to make parent props available to child components. While you could explicitly redefine the props for child component, Laravel ships with the "aware" directive to do exactly that 🚀

```html
<x-menu color="purple">
    <x-menu.item>...</x-menu.item>
    <x-menu.item>...</x-menu.item>
</x-menu>

<!-- /resources/views/components/menu/index.blade.php -->

@props(['color' => 'gray']) <!-- Color property is not accessible to child components -->
@aware(['color' => 'gray']) <!-- Color property is accessible to child components -->

<ul {{ $attributes->merge(['class' => 'bg-'.$color.'-200']) }}>
    {{ $slot }}
</ul>
```

## Tip #282 💡: The "readonly" Blade Directive

Often, we need to conditionally mark an input as readonly. While this can be done manually, Laravel provides a cool blade directive "readonly" to do exactly that 🚀

```html
<input
    type="email"
    name="email"
    {{ $user->isNotAdmin() ? 'readonly' : '' }}
    @readonly($user->isNotAdmin())
/>
```

## Tip #283 💡: The "includeWhen" Blade Directive

Have you ever needed to conditionally include a Blade view? While you could use "if" and "include" together, Laravel ships with the "includeWhen" and "includeUnless" directives to do exactly that 🚀

```php
// Instead of this 🥱
@if ($isAdmin)
    @include('components.impersonate')
@endif

// You can do this 🔥
@includeWhen($isAdmin, 'components.impersonate')

// even this
@includeUnless(! $isAdmin, 'components.impersonate')
```

## Tip #284 💡: Deferred Providers

If you have a service provider that only registers some bindings, you can mark it as deferred by implementing the "DeferrableProvider" interface. This way, it will load only when one of its bindings is needed 🚀

```php
<?php
 
namespace App\Providers;
 
use App\Services\Riak\Connection;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Contracts\Support\DeferrableProvider;
use Illuminate\Support\ServiceProvider;

// This service provider will only load when the Connection is needed,
// improving your apps performance.
class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
{
    public function register(): void
    {
        $this->app->singleton(Connection::class, function (Application $app) {
            return new Connection($app['config']['riak']);
        });
    }
}
```

## Tip #285 💡: The "pipe" Collection Method

Did you know that Laravel collections ship with a "pipe" method? It passes the collection to a given callback and returns the result. It can be useful when you want to wrap your collection or perform calculations 🚀

```php
<?php

use Illuminate\Support\Collection;

$collection = collect([1, 2, 3]);
 
$collection->pipe(fn (Collection $collection) => $collection->sum());  // 6

$collection->pipe(fn (Collection $collection) => ['numbers' => $collection->toArray()]);

// ['numbers' => [1, 2, 3]]
```

## Tip #286 💡: Render Inline Blade Templates

Did you know you can render Blade templates inline? This is great for compiling Blade to HTML, like adding help texts in Nova or Filament or generating emails outside Laravel projects since it can work as a standalone package 🚀

```php
<?php

use Illuminate\Support\Facades\Blade;

return Blade::render('Hello, {{ $name }}', ['name' => 'Laravel']); // Hello, Laravel
```

## Tip #287 💡: The "toggle" method

At some point, we all needed to toggle a value, for example, a like feature that switches between states. While you can do it manually, Laravel ships with a "toggle" method to do exactly that 🚀

```php
<?php

// Instead of this 😑
if ($user->likes()->where('tweet_id', $tweet->id)->exists()) {
    $user->likes()->detach($tweet->id);
} else {
    $user->likes()->attach($tweet->id);
}

// You can do this 🔥
$user->likes()->toggle($tweet->id);

// It also works with multiple IDs
$user->products()->toggle([1, 2, 3]);
```

## Tip #288 💡: The New "doesntContain" String Method

Sometimes, you might need to check if a string does not contain a given value. Previously, you could negate the "contains" helper, but things just got even better with the new "doesntContain" method 🚀

```php
<?php

use Illuminate\Support\Str;

if (!Str::contains('Larvel forever', 'Laravel')) {
    // string does not contain Laravel, fix it
}

if (Str::doesntContain('Larvel forever', 'Laravel')) {
    // string does not contain Laravel, fix it
}
```

## Tip #289 💡: Skip Exception Reporting

There are some exceptions that you may not want to report to your monitoring tool. While you could manually register them in "app.php", you can just mark the exception with the "ShouldntReport" interface 🚀

```php
<?php

namespace App\Exceptions;

use Exception;
use Illuminate\Contracts\Debug\ShouldntReport;

class PodcastProcessingException extends Exception implements ShouldntReport
{
    //
}
```

## Tip #290 💡: Get the Full Query Log

Ever needed to dump the full query log executed in a method? You can enable the query log at the very beginning and dump it at the end by using "enableQueryLog" and "getRawQueryLog" 🚀

```php
<?php

use Illuminate\Support\Facades\DB;

DB::enableQueryLog();

User::whereNotNull('email_verified_at')->get();

DB::getRawQueryLog();
// select * from `users` where `email_verified_at` is not null
// time: 0.33
```

## Tip #291 💡:  Autocompletion in PestPHP

When writing Pest tests, you will likely use "$this", which is not good for IDE autocompletion and may require adding a PHPDoc. To avoid this, use the "test()" helper instead, which returns the current Test Case instance 🚀

```php
<?php

test('the array has the specified key', function () {
    $array = [
        'name' => 'John Doe',
        'email' => 'john@example.com',
    ];

    // No autocompletion 😞
    $this->assertArrayHasKey('email', $array);

    // autocompletion works perfectly fine, and the IDE is happy 🔥
    test()->assertArrayHasKey('email', $array);
});
```

## Tip #292 💡:  Global Middleware for HTTP Client

Sometimes you may want to apply global headers to all outgoing requests. For instance, a global user agent can help you identify your app's requests in other services or third-party APIs. Laravel already supports request and response middleware  to do exactly that 🚀

```php
<?php

use Illuminate\Support\Facades\Http;

Http::globalRequestMiddleware(fn ($request) => $request->withHeader(
    'User-Agent', 'Example Application/1.0'
));

Http::globalResponseMiddleware(fn ($response) => $response->withHeader(
    'X-Finished-At', now()->toDateTimeString()
));
```

## Tip #293 💡:  The New "rawColumn" Method

Laravel v11.32 introduces a new "rawColumn" method. Now, instead of having to use a DB statement when the grammar does not support updating or creating the column, you can use the "rawColumn" method 🚀

```php
<?php

new class extends Migration {
    public function up()
    {
        // Instead of this
        Schema::create('posts', function (Blueprint $table) {
            $table->id();
        });

        DB::statement('alter table `posts` add `legacy_boolean` int(1) default 0 not null');

        Schema::table('posts', function (Blueprint $table) {
            $table->index(['id', 'legacy_boolean']);
        });

        // You can now do this
        Schema::create('table', function (Blueprint $table) {
            $table->id();
            $table->rawColumn('legacy_boolean', 'int(1)')->default(0);
            $table->index(['id', 'legacy_boolean']);
        });
    }
};
```

## Tip #294 💡:  Explain Eloquent Queries

Have you ever needed to run an EXPLAIN on an Eloquent query to check if an index is being used? While you could manually extract the raw query and run EXPLAIN on it, you can just chain the "explain" method to do exactly that 🚀

```php
<?php

User::query()
    ->where('email', 'john@example.com')
    ->explain()
    ->dd();

array:1 [
  0 => {#3368
    +"id": 1
    +"select_type": "SIMPLE"
    +"table": null
    +"partitions": null
    +"type": null
    +"possible_keys": null
    +"key": null
    +"key_len": null
    +"ref": null
    +"rows": null
    +"filtered": null
    +"Extra": "no matching row in const table"
  }
]
```

## Tip #295 💡:  Force HTTPS for URLs

Since Laravel v11.31, you can enforce HTTPS for all generated URLs without needing the HTTPS schema specified in the request 🚀

```php
<?php

use Illuminate\Support\Facades\URL;

URL::forceHttps(app()->isProduction());
```

## Tip #296 💡:  The Prohibitable Trait

Most Laravel apps often have local-only or environment-dependent commands that shouldn't run elsewhere. To prevent accidents, use the "Prohibitable" trait and call the "prohibit" method 🚀

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Console\Prohibitable;

class CleanLogs extends Command
{
    use Prohibitable;
}

// Now in the boot method of the AppServiceProvider
CleanLogs::prohibit(app()->isProduction());
```

## Tip #297 💡:  Prohibit DB Destructive Commands

Running migrations or wiping the DB in production can be, well, disastrous. Since Laravel v11, you can prohibit all DB destructive commands by calling "prohibitDestructiveCommands" method 🚀

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\DB;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Prevents 'migrate:fresh', 'migrate:refresh', 'migrate:reset', and 'db:wipe'
        DB::prohibitDestructiveCommands(
            $this->app->isProduction()
        );
    }
}
```

## Tip #298 💡: Conditionally Hide Console Commands

Sometimes, you may want to hide console commands, such as legacy commands, from being listed. While you can manually hide them using the ["setHidden()"](#tip-55--hide-console-commands), you can do this with the "isHidden()" method 🚀

```php
<?php
 
namespace App\Console\Commands;
 
use Illuminate\Console\Command;
 
class LegacyCommand extends Command
{
    // ...

    public function isHidden(): bool
    {
        // This could check a configuration flag
        return true;
    }
}
```

## Tip #299 💡:  The New "RouteParameter" Attribute

Laravel v11.28 introduced a new attribute `RouteParameter`, which provides an elegant way to access route parameters. While you can use the [`route()`](#tip-3--model-binding-in-form-requests) method on form requests, with the new attribute you also get proper type hints 🚀

```php
<?php

use App\Models\Post;
use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Container\Attributes\RouteParameter;

class UpdatePost extends FormRequest
{
    public function authorize(#[RouteParameter('post')] Post $post): bool
    {
        return $this->user()->can('update', $post);
    }

    // ...
}

// Route definition
Route::put('/post/{post}', [PostsController::class, 'update']);
```

## Tip #300 💡: On-Demand Storage Disks

Have you ever needed to quickly create a disk, whether for tests or temporary files, but had to define it in the filesystem configuration? Well, Laravel ships with on-demand disks so that you can define disks at runtime instead 🚀

```php
<?php

use Illuminate\Support\Facades\Storage;
 
$disk = Storage::build([
    'driver' => 'local',
    'root' => '/path/to/root',
]);
 
$disk->put('image.jpg', $content);
```

## Tip #301 💡: Spell Ordinal Numbers

Starting from Laravel v11.34, you can now spell ordinal numbers using the newly introduced "spellOrdinal" method 🚀

```php
<?php

use Illuminate\Support\Number;
 
$position = Number::ordinal(3); // 3rd

// Now you can spell it 🔥
$position = Number::spellOrdinal(3); // third
```

## Tip #302 💡: Keeping Jobs Unique Until Processing

Sometimes, you may have jobs where only one job can be queued at a time, but once it starts processing, you can queue more jobs. Laravel ships with the "ShouldBeUniqueUntilProcessing" contract to do exactly this 🚀

```php
<?php
 
use App\Models\Product;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUniqueUntilProcessing;
 
class UpdateSearchIndex implements ShouldQueue, ShouldBeUniqueUntilProcessing
{
    // ...
}
```

## Tip #303 💡: The "firstOrNew" Method

Sometimes you may want to check if a model exists, and if not, instantiate it without saving it to the database right away. Laravel ships with the "firstOrNew" to do exactly that 🚀

```php
<?php

use App\Models\User;

// Find the user by email, or save it to the database if it doesn't exist
User::firstOrCreate(['email' => 'name@example.com']);

// Find the user by email, or instantiate a new User instance without saving it
User::firstOrNew(['email' => 'name@example.com']);

// Do something with $user

// Save it to the database
$user->save();
```

## Tip #304 💡: The "keyBy" Method

Have you ever needed to key your eloquent collection by an attribute from its items? While you can hack your way with pluck, the "keyBy" method does exactly that 🚀

```php
<?php

$collection = collect([
    ['product_id' => 'prod-100', 'name' => 'Desk'],
    ['product_id' => 'prod-200', 'name' => 'Chair'],
]);

// Instead of doing this or any manual transformation
$keyed = $collection->pluck(null, 'product_id');

// You can simply do this 🔥
$keyed = $collection->keyBy('product_id');

$keyed->all();

/*
    [
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]
*/
```

## Tip #305 💡: The "remove" Str Method

Have you ever needed to remove some characters from a text? While you can do it manually with built-in PHP functions, Laravel provides a much more readable wrapper "remove", which you can also chain with other methods 🚀

```php
<?php

use Illuminate\Support\Str;

$string = 'Peter Piper picked a peck of pickled peppers.';

// This works
str_replace('e', '', $string); // Ptr Pipr pickd a pck of pickld ppprs.

// But this reads much better 🔥 and supports case insensitive as well
Str::remove('e', $string, caseSensitive: false); // Ptr Pipr pickd a pck of pickld ppprs.

// You can also remove multiple letters at once
Str::remove(['e', 'i', 't'], $string); // Pr Ppr pckd a pck of pckld ppprs.
```

## Tip #306 💡: The "withWhereHas" Method

Have you ever needed to eager load a relationship but also constrain it with relationship existence? While you can do that manually with 2 methods, Laravel ships with the "withWhereHas" method to do exactly that 🚀

```php
<?php

// Instead of this
User::query()
    ->whereHas('posts', function (Builder $query) {
        $query->where('featured', true);
    })
    ->with('posts')
    ->get();

// You can simply use withWhereHas 🔥
User::query()
    ->withWhereHas('posts', function (Builder $query) {
        $query->where('featured', true);
    })
    ->get();
```
