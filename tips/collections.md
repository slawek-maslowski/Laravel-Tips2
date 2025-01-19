# Laravel Collections Tips ([cd ..](../README.md))

- [The "containsOneItem" Method](#laravel-tip--the-containsoneitem-method-️)
- [The "dot" and "undot" Methods](#laravel-tip--the-dot-and-undot-methods-️)
- [The "times" Methods](#laravel-tip--the-times-methods-️)
- [Check Collection Item Types](#laravel-tip--check-collection-item-types-️)
- [The "every" Collection Method](#laravel-tip--the-every-collection-method-️)
- [The "forget" Collection Method](#laravel-tip--the-forget-collection-method-️)
- [Skip Collection Items Until a Condition is Met](#laravel-tip--skip-collection-items-until-a-condition-is-met-️)
- [The "zip" Collection Method](#laravel-tip--the-zip-collection-method-️)
- [The "WhenNotEmpty" Collection Method](#laravel-tip--the-whennotempty-collection-method-️)
- [Search Collection Items](#laravel-tip--search-collection-items-️)
- [The "sole" Method](#laravel-tip--the-sole-method-️)
- [Filter Falsy Values](#laravel-tip--filter-falsy-values-️)
- [A Better Implode](#laravel-tip--a-better-implode-️)
- [Higher Order Messages](#laravel-tip--higher-order-messages-️)
- [A Better Pluck](#laravel-tip--a-better-pluck-️)
- [The "pipe" Collection Method](#laravel-tip--the-pipe-collection-method-️)
- [Finding Duplicates](#laravel-tip--finding-duplicates-️)

## Laravel Tip 💡: The "containsOneItem" Method ([⬆️](#laravel-collections-tips-cd-))

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

## Laravel Tip 💡: The "dot" and "undot" Methods ([⬆️](#laravel-collections-tips-cd-))

When working with Laravel collections, you may want to flatten a multi-dimensional collection into a single-level collection, or vice versa. Luckily, there are 2 methods just for that, "dot" and "undot" 🚀

```php
$collection = collect(['products' => ['desk' => ['price' => 100]]]);

$dotted = $collection->dot();    // ['products.desk.price' => 100]
$undotted = $collection->undot(); // ['products' => ['desk' => ['price' => 100]]]
```

## Laravel Tip 💡: The "times" Methods ([⬆️](#laravel-collections-tips-cd-))

Did you know that Laravel ships with a cool collection method "times," which allows you to create a collection by invoking a closure N times? This could be helpful when working with days or generating random strings 🚀

```php
$collection = Collection::times(10, function (int $number) {
    return $number * 9;
});

$collection->all(); 
// [9, 18, 27, 36, 45, 54, 63, 72, 81, 90]
```

## Laravel Tip 💡: Check Collection Item Types ([⬆️](#laravel-collections-tips-cd-))

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

## Laravel Tip 💡: The "every" Collection Method ([⬆️](#laravel-collections-tips-cd-))

Sometimes you may want to check if every element in the collection passes a condition. Luckily, Laravel ships with the "every" method to do exactly that 🚀

```php
<?php

$result = collect([1, 2, 3])->every(fn (int $value, int $key) => $value > 2);
// $result will be false

$result = collect([])->every(fn (int $value, int $key) => $value > 2);
// Since the collection is empty, $result will be true
```

## Laravel Tip 💡: The "forget" Collection Method ([⬆️](#laravel-collections-tips-cd-))

Sometimes, when working with collections, you may want to remove an element by its key. Luckily, collections come with the "forget" method to do exactly that 🚀

```php
<?php

$collection = collect(['name' => 'John Doe', 'framework' => 'laravel']);

$collection->forget('name');

$collection->all(); // ['framework' => 'laravel']
```

## Laravel Tip 💡: Skip Collection Items Until a Condition is Met ([⬆️](#laravel-collections-tips-cd-))

Sometimes, when working with collections, you may want to skip all the elements until a condition is met. Laravel comes with the "skipUntil" method to do exactly that 🚀

```php
<?php

$collection = collect([1, 2, 3, 4]);

$subset = $collection->skipUntil(function (int $item) {
    return $item >= 3;
});

$subset->all(); // [3, 4]
```

## Laravel Tip 💡: The "zip" Collection Method ([⬆️](#laravel-collections-tips-cd-))

When working with collections, you might want to merge two collections by their index, combining the values of the first index, then the second, and so on. Luckily, Laravel includes the "zip" method to do exactly that 🚀

```php
<?php

$collection = collect(['Chair', 'Desk']);

// This will merge values by index, so "Chair" with 100, and "Desk" with 200
$zipped = $collection->zip([100, 200]);

$zipped->all(); // [['Chair', 100], ['Desk', 200]]
```

## Laravel Tip 💡: The "WhenNotEmpty" Collection Method ([⬆️](#laravel-collections-tips-cd-))

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

## Laravel Tip 💡: Search Collection Items ([⬆️](#laravel-collections-tips-cd-))

Did you know that Laravel allows you to search collection items? You can even pass a condition to search for the first element that meets it 🚀

```php
<?php
$collection = collect([2, 4, 6, 8]);

$collection->search('4'); // 1 (the index)

$collection->search('4', strict: true); // false (not found)

$collection->search(fn (int $item, int $key) => $item > 5); // 2 (the index)
```

## Laravel Tip 💡: The "sole" Method ([⬆️](#laravel-collections-tips-cd-))

When working with collections, whether regular or Eloquent, if you want to get the first item that matches the condition and ensure it is the only one, use the "sole" method 🚀

```php
<?php

// Returns 2
collect([1, 2, 3, 4])->sole(fn (int $value, int $key) => $value === 2);

// Throws: Illuminate\Support\MultipleItemsFoundException 2 items were found.
collect([1, 2, 2, 4])->sole(fn (int $value, int $key) => $value === 2);
```

## Laravel Tip 💡: Filter Falsy Values ([⬆️](#laravel-collections-tips-cd-))

We've all used the "filter" method on collections, but did you know that if no callback is passed, it will filter out all the falsy values? 🚀

```php
<?php

$collection = collect([1, 2, 3, null, false, '', 0, []]);
$collection->filter()->all(); // [1, 2, 3]
```

## Laravel Tip 💡: A Better Implode ([⬆️](#laravel-collections-tips-cd-))

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

## Laravel Tip 💡: Higher Order Messages ([⬆️](#laravel-collections-tips-cd-))

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

## Laravel Tip 💡: A Better Pluck ([⬆️](#laravel-collections-tips-cd-))

We often need to get the IDs of some models. While you can use the "pluck()" method to do this, you can also use "modelKeys()", which reads better and won't break if you change the primary key at any point 🚀

```php
<?php

use App\Models\User;

// Instead of this
$ids = User::all()->pluck('id');

// You can do this 🔥
$ids = User::all()->modelKeys();
```

## Laravel Tip 💡: The "pipe" Collection Method ([⬆️](#laravel-collections-tips-cd-))

Did you know that Laravel collections ship with a "pipe" method? It passes the collection to a given callback and returns the result. It can be useful when you want to wrap your collection or perform calculations 🚀

```php
<?php

use Illuminate\Support\Collection;

$collection = collect([1, 2, 3]);
 
$collection->pipe(fn (Collection $collection) => $collection->sum());  // 6

$collection->pipe(fn (Collection $collection) => ['numbers' => $collection->toArray()]);

// ['numbers' => [1, 2, 3]]
```

## Laravel Tip 💡: Finding Duplicates ([⬆️](#laravel-collections-tips-cd-))

Sometimes you may need to find duplicate values, such as when cleaning up data. While you could do this manually, Laravel already ships with the "duplicates" method to do exactly that 🚀

```php
<?php

$collection = collect(['a', 'b', 'a', 'c', 'b', 100, '100']);

// Loose comparison
$collection->duplicates(); // ['a', 'b', '100']

// Strict comparison
$collection->duplicatesStrict(); // ['a', 'b']
```
