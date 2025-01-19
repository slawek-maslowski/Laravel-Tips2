# Authentication & Authorization Tips ([cd ..](./README.md))

- [Hook into Authentication Events](#laravel-tip--hook-into-authentication-events-️)
- [Deny As Not Found](#laravel-tip--deny-as-not-found-️)
- [Authenticate the User Once](#laravel-tip--authenticate-the-user-once-️)
- [Log Out Other Devices](#laravel-tip--log-out-other-devices-️)
- [Log Out The Current Device Only](#laravel-tip--log-out-the-current-device-only-️)
- [Check if a User is a Guest](#laravel-tip--check-if-a-user-is-a-guest-️)

## Laravel Tip 💡: Hook into Authentication Events ([⬆️](#authentication--authorization-tips-cd-))

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

## Laravel Tip 💡: Deny As Not Found ([⬆️](#authentication--authorization-tips-cd-))

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

## Laravel Tip 💡: Authenticate the User Once ([⬆️](#authentication--authorization-tips-cd-))

Did you know that Laravel ships with the "once" method for authenticating the user only for the current request? This can be handy in various scenarios, such as building single-use links or RESTful APIs 🚀

```php
<?php

if (Auth::once($credentials)) {
    // ...
}

```

## Laravel Tip 💡: Log Out Other Devices ([⬆️](#authentication--authorization-tips-cd-))

When users log out, you might want to ask them if they want to log out from other devices while keeping the current one. Luckily, Laravel ships with the "logoutOtherDevices" method that does exactly that 🚀

```php
<?php

use Illuminate\Support\Facades\Auth;
 
Auth::logoutOtherDevices($currentPassword);
```

## Laravel Tip 💡: Log Out The Current Device Only ([⬆️](#authentication--authorization-tips-cd-))

Did you know that Laravel ships with the "logoutCurrentDevice" method, which allows you to log out only the currently authenticated device? 🚀

```php
<?php

use Illuminate\Support\Facades\Auth;
 
Auth::logoutCurrentDevice();
```

## Laravel Tip 💡: Check if a User is a Guest ([⬆️](#authentication--authorization-tips-cd-))

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
