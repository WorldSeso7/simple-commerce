---
title: Upgrade Guide
---

When upgrading from v2.2 to v2.3, please read through this guide in case there's anything you need to change. Most of the updates around the configuration file and blueprint changes have been automated to help speed up the process.

You may also [review the changes](https://github.com/doublethreedigital/simple-commerce/compare/v2.3...v2.2) manually if you wish.

In your `composer.json` file, change the `doublethreedigital/simple-commerce` version constraint:

```json
"doublethreedigital/simple-commerce": "2.3.*"
```

Then run:

```
composer update doublethreedigital/simple-commerce --with-dependencies
```

## High: Gateways

### Built-in gateway namespace changes

The namespace of all built-in gateways (Dummy, Stripe & Mollie) have been updated from `Gateways\GatewayName` to `Gateways\Builtin\GatewayName`.

If you're using one of these gateways, you must update the namespace in your `simple-commerce.php` config file.

**Note: this is one of the upgrade steps that's automated for you.**

```php
/*
|--------------------------------------------------------------------------
| Gateways
|--------------------------------------------------------------------------
|
| You can setup multiple payment gateways for your store with Simple Commerce.
| Here's where you can configure the gateways in use.
|
| https://simple-commerce.duncanmcclean.com/gateways
|
*/

'gateways' => [
    \DoubleThreeDigital\SimpleCommerce\Gateways\Builtin\DummyGateway::class => [
        'display' => 'Card',
    ],
],
```

### Custom gateway namespace changes

If you're using a custom gateway, please update the namespaces of the Data Transfer Objects (DTOs) provided by SC.

```
// Before
use DoubleThreeDigital\SimpleCommerce\Data\Gateways\BaseGateway;
use DoubleThreeDigital\SimpleCommerce\Data\Gateways\GatewayPrep;
use DoubleThreeDigital\SimpleCommerce\Data\Gateways\GatewayPurchase;
use DoubleThreeDigital\SimpleCommerce\Data\Gateways\GatewayResponse;

// Now
use DoubleThreeDigital\SimpleCommerce\Gateways\BaseGateway;
use DoubleThreeDigital\SimpleCommerce\Gateways\Prepare;
use DoubleThreeDigital\SimpleCommerce\Gateways\Purchase;
use DoubleThreeDigital\SimpleCommerce\Gateways\Response;
```

## High: Email notifications

Previously, Mailables were used to send email notifications to customers and store owners.

However, in order to easily support more notification types in the future, I've changed the way email notifications work. They now use Laravel's [Notifications feature](https://laravel.com/docs/master/notifications).

### Updated configuration

You may now specifiy multiple notifications for an 'event'. You can also specify who you'd like each of the notifications to be sent to. You can choose from

```
/*
|--------------------------------------------------------------------------
| Notifications
|--------------------------------------------------------------------------
|
| Simple Commerce can automatically send notifications after events occur in your store.
| eg. a cart being completed.
|
| Here's where you can toggle if certain notifications are enabled/disabled.
|
| https://simple-commerce.duncanmcclean.com/email
|
*/

'notifications' => [
    'order_paid' => [
        \DoubleThreeDigital\SimpleCommerce\Notifications\CustomerOrderPaid::class   => ['to' => 'customer'],
        \DoubleThreeDigital\SimpleCommerce\Notifications\BackOfficeOrderPaid::class => ['to' => 'duncan@example.com'],
    ],
],
```

**Note: this is one of the upgrade steps that's automated for you.**

## Medium: Events

I've updated the names of events, the parameters available in events and there's a few events which have also been removed. A full list of changes is available below:

### `CartUpdated`

This event has been removed.

### `CouponRedeemed`

Previously the `$coupon` was an `Entry` instance, this has been changed to a `Coupon` instance.

```php
public function handle(CouponRedeemed $event)
{
	$event->coupon; // is now a Coupon
}
```

### `CustomerAddedToCart`

This event has been removed. In place, I'd recommend listening for order entries being updated and checking if the `customer` field has been changed.

### `CartCompleted` -> `OrderPaid`

This event has been renamed `OrderPaid`. Additionally, the `$cart` parameter has been removed, `$order` should now be used (all of the same methods are available).

```php
public function handle(OrderPaid $event)
{
	$event->order; // will return an Order, should be used in place of $cart
}
```

### `CartSaved` -> `OrderSaved`

This event has been renamed `OrderSaved`. Additionally, the `$cart` parameter has been removed and a `$order` parameter has been added.

The `$order` parameter will return an `Order` instance.

```php
public function handle(OrderSaved $event)
{
	$event->order; // will return an Order, should be used in place of $cart
}
```

### `PostCheckout`

Previously, only an order's data array was passed into this event. However, now the full `$order` is available.

```php
public function handle(PostCheckout $event)
{
	$event->order; // will return an Order
}
```

### `PreCheckout`

Previously, only an order's data array was passed into this event. However, now the full `$order` is available.

```php
public function handle(PreCheckout $event)
{
	$event->order; // will return an Order
}
```

## Medium: Translations

I've simplified the translations into a single file, `messages.php`. If you were previously using your own translations, please [review the updated file](https://github.com/doublethreedigital/simple-commerce/blob/2.3/resources/lang/en/messages.php).

## Medium: Custom Data Classes

To help with future features I've got planned (👀), I've updated a few things around custom data classes.

Note: You'll now find me referring to these as 'content drivers'.

### Updates to binding your custom data class

Instead of directly binding to the service container in your `AppServiceProvider.php` file, please bind your data class from inside the `simple-commerce.php` config file.

```php
/*
|--------------------------------------------------------------------------
| Content Drivers
|--------------------------------------------------------------------------
|
| Simple Commerce stores all products, orders, coupons etc as flat-file entries.
| This works great for store stores where you want to keep everything simple. But
| sometimes, for more complex stores, you may want use a database instead. To do so,
| just swap out the 'content driver' in place below.
|
*/

'content' => [
    'orders' => [
        'driver' => \DoubleThreeDigital\SimpleCommerce\Orders\Order::class,
        'collection' => 'orders',
    ],

    'products' => [
        'driver' => \DoubleThreeDigital\SimpleCommerce\Products\Product::class,
        'collection' => 'products',
    ],

    'coupons' => [
        'driver' => \DoubleThreeDigital\SimpleCommerce\Coupons\Coupon::class,
        'collection' => 'coupons',
    ],

    'customers' => [
        'driver' => \DoubleThreeDigital\SimpleCommerce\Customers\Customer::class,
        'collection' => 'customers',
    ],
],
```

When updating, we will **automate** the change of configuration format. However, it will not automatically switch it to your custom driver/data class.

### Contract changes

I've also made various changes to the contracts implemented by custom drivers/data classes. You may [review the changes on GitHub](https://github.com/doublethreedigital/simple-commerce/tree/2.3/src/Contracts).

## Low: Cart facade

Like mentioned in the [v2.2 upgrade guide](https://simple-commerce.duncanmcclean.com//update-guide#cart-facade-being-phased-out), the `Cart` facade has now been completley removed.

You should now use the `Order` facade which provides all of the same methods.

```php
// Before
Cart::find('abc-123');

// After
Order::find('abc-123');
```

---

Please feel free to [reach out](mailto:duncan@doublethree.digital) if you've got any questions about upgrading! I'm always happy to help.
