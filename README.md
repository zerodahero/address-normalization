# Basic Address Normalizer

[![Build Status]https://github.com/zerodahero/address-normalization/actions/workflows/php.yml/badge.svg]

## Purpose

The main purpose of this package is as the first layer of address normalization and standardization. Recommended use is to pre-parse/normalize an address and compare to an existing cache/record set using the hash functions.

A way to normalize US mailing addresses without the need for an external service. This is a port of the perl module Geo::StreetAddress::US originally written by Schuyler D. Erle.

This is a fork from khartnett/address-normalization -- kudos for the original work!

## Limitations

This is a very basic normalizer. It realistically only handles US-based addresses, and should not be considered dependable for strict address-to-address comparison. **This normalizer does not verify the validity of the address!** If you are dependent on _accurate_ addresses, you **need** to be using some other means (3rd party service, most likely) to verify an address.

## Why?

I forked and added features to this package because I needed a decent first-layer to pre-normalize addresses _before_ sending them our standardization service. This helps us limit the number of calls and strict dependence on the service, but also lets us catch a few easy-to-match scenarios here and there, which is a better user experience.

## Alternatives

[Libpostal](https://github.com/openvenues/libpostal) is probably the best of its class in this area. I decided not to use Libpostal because: (1) It requires a few Gbs of space, which is undesirable in my current environment, and (2) it's probably overkill, since I consider our 3rd party service to be authoritative in the matter anyway.

## Installation

`composer require zerodahero/address-normalization`

## Usage

### Normalizing

```php
<?php
use ZeroDaHero\Normalizer;
$normalizer = new Normalizer();

// This returns a \ZeroDaHero\Address object with the parsed components
$address = $normalizer->parse('204 southeast Smith Street Harrisburg, or 97446');

$address->getAddressComponents();
/* output:
[
    "number" => "204",
    "street" => "Smith",
    "street_type" => "St",
    "unit" => "",
    "unit_prefix" => "",
    "suffix" => "",
    "prefix" => "SE",
    "city" => "Harrisburg",
    "state" => "OR",
    "postal_code" => "97446",
    "postal_code_ext" => null,
    "street_type2" => null,
    "prefix2" => null,
    "suffix2" => null,
    "street2" => null,
] */

$address->toString();
/* string_result:
"204 SE Smith St, Harrisburg, OR 97446"
*/
```

### Comparing

```php
<?php
use ZeroDaHero\Normalizer;
$normalizer = new Normalizer();

$address1 = $normalizer->parse('204 southeast Smith Street Harrisburg, or 97446');
$address2 = $normalizer->parse('204 SE Smith St. Harrisburg, Oregon 97446');
// Same street, different number
$address3 = $normalizer->parse('207 SE Smith St. Harrisburg, Oregon 97446');

$address1->is($address2); // true
$address2->is($address3); // false
$address1->isSameStreet($address3); // true

// or can compare hashes directly
$address1->getFullHash() === $address2->getFullHash(); // true
```

### Formatting

```php
<?php
use ZeroDaHero\Normalizer;
$normalizer = new Normalizer();

$address = $normalizer->parse('204 southeast Smith Street Harrisburg, or 97446');

$address->toString();
// or
(string) $address;
/* string:
  "204 SE Smith St, Harrisburg, OR 97446"
*/

$address->toArray();
/* array:
  [
    '204 SE Smith St',
    'Harrisburg, OR 97446'
  ]
```

### Hashing

If you only need to make use of a consistent way of hashing (e.g. if you're starting with a dependable 5-part address, such as from a 3rd party service), you can build a `SimpleAddress`.

```php
<?php
use ZeroDaHero\SimpleAddress;

$address = new SimpleAddress('1234 Main St NE', null, 'Minneapolis', 'MN', '55401');
$address->getHash(); // full hash minus zip
$address->getFullHash(); // full hash including zip

// or do it all with the factory method:
SimpleAddress::hashFromParts('1234 Main St NE', null, 'Minneapolis', 'MN', '55401');

// CANNOT hash street, since the component parts don't exist
$address->getStreetHash(); // throws exception
```
