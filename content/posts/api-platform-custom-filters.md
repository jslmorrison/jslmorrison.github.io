+++
title = "Api platform custom filters"
date = 2023-11-27
tags = ["symfony", "api-platform"]
draft = false
+++

When not using entities as the api resources, in order to filter collections one or more custom filters can be applied to the classes annotated with the `[#ApiResource]` attribute.

<!--more-->

Define a class as an `ApiResource` and annotate it with the desired filter(s), using the `[#ApiFilter]` attribute:

```php
// src/ApiResource/ProductResource.php
<?php

declare(strict_types=1);

namespace App\ApiResource;

use App\Filter\ProductInStockFilter;
use ApiPlatform\Metadata\ApiFilter;
use ApiPlatform\Metadata\ApiResource;

[#ApiResource()]
[#ApiFilter(ProductInStockFilter::class)]
final class ProductResource
```

The custom filters should implement the `\ApiPlatform\Doctrine\Orm\Filter\FilterInterface`

```php
<?php

declare(strict_types=1);

namespace App\Filter;

use Symfony\Component\HttpFoundation\Request;
use ApiPlatform\Doctrine\Orm\Filter\FilterInterface;

final class ProductInStockFilter implements FilterInterface
{
    public function apply(
        QueryBuilder $queryBuilder,
        QueryNameGeneratorInterface $queryNameGenerator,
        string $resourceClass,
        Operation $operation = null,
        array $context = [],
    ): void {
        if (!isset($context['filters']['product_in_stock'])) {
            return;
        }

        if ($context['filters']['product_in_stock'] === 'true') {
            $queryBuilder->andWhere('o.inStock > 1');
        }
    }

    // This function is only used to hook in documentation generators (supported by Swagger and Hydra)
    public function getDescription(string $resourceClass): array
    {
        return [
            'product_in_stock' => [
                'property' => 'inStock',
                'type' => 'bool',
                'required' => false,
            ],
        ];
    }
}
```

If utilising the default collection provider in your custom provider class then the query will be automatically altered to include the additional `andWhere` clause.
Alternatively you can conditionally add the logic to your database lookup (repository method) to apply in your provider.
