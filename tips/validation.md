# Validation Tips ([cd ..](../README.md))

- [Inline Validation](#laravel-tip--inline-validation-️)
- [Validate Nested Arrays Easily](#laravel-tip--validate-nested-arrays-easily-️)
- [Stop On First Failure](#laravel-tip--stop-on-first-failure-️)
- [The "bail" Validation Rule](#laravel-tip--the-bail-validation-rule-️)
- [Extract Validated Input](#laravel-tip--extract-validated-input-️)
- [Conditionally Adding Rules](#laravel-tip--conditionally-adding-rules-️)
- [Normalize Validated Data](#laravel-tip--normalize-validated-data-️)
- [Validate Dates Elegantly](#laravel-tip--validate-dates-elegantly-️)
- [Exclude Validated Input](#laravel-tip--exclude-validated-input-️)
- [Filter Only Real Emails](#laravel-tip--filter-only-real-emails-️)
- [Better Error Messages for Arrays](#laravel-tip--better-error-messages-for-arrays-️)
- [The Required If Accepted Rule](#laravel-tip--the-required-if-accepted-rule-️)
- [Validate Image Dimensions](#laravel-tip--validate-image-dimensions-️)
- [The "prohibitedIf" Rule](#laravel-tip--the-prohibitedif-rule-️)
- [The "sometimes" Validation Rule](#laravel-tip--the-sometimes-validation-rule-️)
- [The "distinct" Validation Rule](#laravel-tip--the-distinct-validation-rule-️)
- [Confirm User Password](#laravel-tip--confirm-user-password-️)
- [Conditional Validation](#laravel-tip--conditional-validation-️)
- [Customize the Default Password Rules](#laravel-tip--customize-the-default-password-rules-️)

## Laravel Tip 💡: Inline Validation ([⬆️](#validation-tips-cd-))

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

## Laravel Tip 💡: Validate Nested Arrays Easily ([⬆️](#validation-tips-cd-))

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

## Laravel Tip 💡: Stop On First Failure ([⬆️](#validation-tips-cd-))

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

## Laravel Tip 💡: The "bail" Validation Rule ([⬆️](#validation-tips-cd-))

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

## Laravel Tip 💡: Extract Validated Input ([⬆️](#validation-tips-cd-))

When working with validated input, we often need to extract only a few items from the request. Instead of manually unsetting or filtering, use Laravel's "safe()" method to do this elegantly 🚀

```php
<?php

$validated = $request->safe()->only(['name', 'email']);
 
$validated = $request->safe()->except(['name', 'email']);
 
$validated = $request->safe()->all();
```

## Laravel Tip 💡: Conditionally Adding Rules ([⬆️](#validation-tips-cd-))

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

## Laravel Tip 💡: Normalize Validated Data ([⬆️](#validation-tips-cd-))

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

## Laravel Tip 💡: Validate Dates Elegantly ([⬆️](#validation-tips-cd-))

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

## Laravel Tip 💡: Exclude Validated Input ([⬆️](#validation-tips-cd-))

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

## Laravel Tip 💡: Filter Only Real Emails ([⬆️](#validation-tips-cd-))

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

## Laravel Tip 💡: Better Error Messages for Arrays ([⬆️](#validation-tips-cd-))

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

## Laravel Tip 💡: The Required If Accepted Rule ([⬆️](#validation-tips-cd-))

When validating forms, sometimes you want to conditionally require a field if another one was filled. Laravel ships with "required_if_accepted" to do exactly that 🚀

```php
<?php
// subscribe_to_newsletter must be true for the email field to be required
$validator = Validator::make($data, [
    'subscribe_to_newsletter' => 'boolean',
    'email' => 'required_if_accepted:subscribe_to_newsletter|email',
]);
```

## Laravel Tip 💡: Validate Image Dimensions ([⬆️](#validation-tips-cd-))

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

## Laravel Tip 💡: The "prohibitedIf" Rule ([⬆️](#validation-tips-cd-))

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

## Laravel Tip 💡: The "sometimes" Validation Rule ([⬆️](#validation-tips-cd-))

Have you ever needed to validate a field only if it's present, but skip it when it’s not? Laravel ships with the "sometimes" validation rule to do exactly that 🚀

```php
<?php

$validator = Validator::make($data, [
    // The email will only be validated if it is present in the $data array
    'email' => ['sometimes', 'email'],
]);
```

## Laravel Tip 💡: The "distinct" Validation Rule ([⬆️](#validation-tips-cd-))

Have you ever needed to check if an array contains duplicate values? While you can do this manually, it can get slightly messy. Instead, you can use the "distinct" validation rule to do exactly this 🚀

```php
<?php

use Illuminate\Support\Facades\Validator;

$posts = [
    ['title' => 'First Post'],
    ['title' => 'Second Post'],
    ['title' => 'First Post'], // Duplicate title here
    ['title' => 'Third Post'],
];

// Instead of manually checking if the data contains duplicates
$filteredCount = collect($posts)->pluck('title')->unique();
$filteredCount->count() !== collect($posts)->count(); // true

// You can simply use built in validation rules 🔥
Validator::make($posts, ['*.title' => 'distinct:strict'])->fails(); // true
```

## Laravel Tip 💡: Confirm User Password ([⬆️](#validation-tips-cd-))

Certain actions like deleting an account might require users to confirm their password. While you could implement this logic manually, Laravel ships with a built in validation rule, current_password, to do exactly that 🚀

```php
<?php

public function destroy(Request $request): RedirectResponse
{
    $request->validate([
        'password' => ['required', 'current_password:web'],
    ]);
 
    // ...
 
    return to_route('home');
}
```

## Laravel Tip 💡: Conditional Validation ([⬆️](#validation-tips-cd-))

Have you ever needed to apply a validation rule only in specific cases? For example, rejecting discount coupons for certain plans. While you could implement this manually, you can make use of the "sometimes" method to do exactly that 🚀

```php
<?php

use Illuminate\Support\Fluent;
use Illuminate\Support\Facades\Validator;
 
$validator = Validator::make($request->all(), [
    'email' => 'required|email',
    'games' => 'required|numeric',
]);

// The "reason" field will be validated with the "required" and "max:500" rules if  
// there are 100 or more games 🔥
$validator->sometimes('reason', 'required|max:500', function (Fluent $input) {
    return $input->games >= 100;
});
```

## Laravel Tip 💡: Customize the Default Password Rules ([⬆️](#validation-tips-cd-))

Laravel ships with default password rules that suit most use cases. However, if you need to use specific rules, you don't have to discard the default ones. Instead, customize them, and keep your rules in one single place 🚀

```php
<?php

use Illuminate\Validation\Rules\Password;

public function boot(): void
{
    Password::defaults(function () {
        $rule = Password::min(8);

        return $this->app->isProduction()
            ? $rule->mixedCase()->uncompromised()
            : $rule;
    });
}

// Now you can use it like this
'password' => ['required', Password::defaults()]

// Instead of defining custom rules everywhere in your app
'password' => ['required', Password::min(8)->mixedCase()->uncompromised()]
```
