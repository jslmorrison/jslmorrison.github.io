+++
title = "Doctrine ORM embeddables"
date = 2023-11-19
tags = ["doctrine", "orm"]
categories = ["symfony"]
draft = false
+++

When you have more than one entity having common properties such as address or geolocation coordinates for example, rather than duplicate those properties on each entity they can be split out into reusable value objects and embedded into those entities for better separation of concerns.

<!--more-->

> Embeddables are classes which are not entities themselves, but are embedded in entities and can also be queried in DQL. You'll mostly want to use them to reduce duplication or separating concerns. Value objects such as date range or address are the primary use case for this feature.


## Example entity {#example-entity}

```php
<?php

declare(strict_types=1);

namespace App\Entity;

use App\ValueObject\GeoLocation;
use Symfony\Component\Uid\Ulid;
use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity]
final class Property
{
    #[ORM\Id]
    #[ORM\Column(type: UlidType::NAME)]
    private Ulid $id;

    // N.B. annotation marking this as embedded
    #[ORM\Embedded(class: GeoLocation::class, columnPrefix: false)]
    private GeoLocatable $geoLocation;
}
```


## Embeddable {#embeddable}

A reusable value object.

```php
<?php

declare(strict_types=1);

namespace App\ValueObject;

use App\Interface\GeoLocatable;
use Doctrine\ORM\Mapping as ORM;

// N.B. annotation marking this as an embeddable
#[ORM\Embeddable]
final class GeoLocation implements GeoLocatable
{
    #[ORM\Column(type: "float")]
    private float $latitude;

    #[ORM\Column(type: "float")]
    private float $longitude;

    public function __construct(float $latitude, float $longitude)
    {
        $this->latitude = $latitude;
        $this->longitude = $longitude;
    }

    public function latitude(): float
    {
        return $this->latitude;
    }

    public function longitude(): float
    {
        return $this->longitude;
    }
}
```

At the database level the value object properties will be in-lined as columns, prefixed with the value object name, in the `property` table. The prefix can be removed by setting the `columnPrefix` attribute to false, as above.

Their values can be accessed via the entity as follows:

```php
/** @var \App\Entity\Property
$property->geoLocation()->latitude();
```
