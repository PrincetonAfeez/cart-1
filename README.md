# PHP Cart

[![Build Status](https://img.shields.io/travis/riesenia/cart/master.svg?style=flat-square)](https://travis-ci.org/riesenia/cart)
[![Latest Version](https://img.shields.io/packagist/v/riesenia/cart.svg?style=flat-square)](https://packagist.org/packages/riesenia/cart)
[![Total Downloads](https://img.shields.io/packagist/dt/riesenia/cart.svg?style=flat-square)](https://packagist.org/packages/riesenia/cart)
[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE)

PHP library providing basic shopping cart functionality.

## Installation

Install the latest version using `composer require riesenia/cart`

Or add to your *composer.json* file as a requirement:

```json
{
    "require": {
        "riesenia/cart": "~1.0"
    }
}
```

## Usage

Constructor takes three configuration parameters:
 * context data that are passed to each added cart item (you can pass i.e. customer id to resolve custom price)
 * true when listing gross prices, false for net prices (see [nice explanation](http://makandracards.com/makandra/1505-invoices-how-to-properly-round-and-calculate-totals))
 * number of decimals for rounding

All of them can be set separately.

```php
use Riesenia\Cart\Cart; 

// default is (null, true, 2)
$cart = new Cart();

$cart->setContext(['customer_id' => $_SESSION['customer_id']]);
$cart->setPricesWithVat(false);
$cart->setRoundingDecimals(4);
```

### Manipulating cart items

Items can be accessed by their cart id (provided by *getCartId* method).

```php
// adding item to cart ($product class has to implement CartItemInterface)
$cart->addItem($product);

// set quantity of the item
$cart->addItem($anotherProduct, 3);

// when $product->getCartId() returns i.e. 'abc'
$cart->setItemQuantity('abc', 7);

// removing item
$cart->removeItem('abc');
```

### Batch cart items manipulation

Cart can be cleared using *clear()* method. Items can be set using *setItems()* method. Please note
that *setItems* will call *clear*. All items have to implement *CartItemInterface*.

### Getting items

Items can be fetched using *getItems* or by type using *getItemsByType*.

### Getting totals

Cart works with *Decimal* class (see [litipk/php-bignumbers](https://github.com/Litipk/php-bignumbers/wiki/Decimal)).
You can access subtotal (without VAT), taxes (array of amounts for all rates) and total (subtotal + taxes).

```php
// item 1 [price: 1.00, tax rate: 10]
// item 2 [price: 2.00, tax rate: 20]

// 3.00
echo $cart->getSubtotal();

// 0.10
echo $cart->getTaxes()[10];

// 0.40
echo $cart->getTaxes()[20];

// 3.50
echo $cart->getTotal();
```

Totals can be also count by type:

```php
// get totals of type 'product'
echo $cart->getTotal('product');

// get totals of type 'product' and 'service'
echo $cart->getTotal('product,service');

// get totals of all items except type 'product' and 'service'
echo $cart->getTotal('~product,service');
```

### Getting cart weight

Item implementing *WeightedCartItemInterface* can be added to cart, so cart can count total weight.
Weight can be counted by type using the same format as for counting totals.

```php
// get weight of type 'product'
echo $cart->getWeight('product');
```

### Bound cart items

Item implementing *BoundCartItemInterface* can be added to cart. When the target item is removed from
the cart, bound item is removed automatically too. If *updateCartQuantityAutomatically* method returns
true, bound item also reflects quantity changes of target item.

### Multiple bound cart items

Item implementing *MultipleBoundCartItemInterface* can be added to cart. When any of target items is
removed from the cart, bound item is removed automatically too.

## Tests

You can run the unit tests with the following command:

    $ cd path/to/riesenia/cart
    $ composer install
    $ vendor/bin/phpspec run