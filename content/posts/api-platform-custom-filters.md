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

The custom filters should implement the `\ApiPlatform\Serializer\Filter\FilterInterface`

```php
<?php

declare(strict_types=1);

namespace App\Filter;

use Symfony\Component\HttpFoundation\Request;
use ApiPlatform\Serializer\Filter\FilterInterface;

final class ProductInStockFilter implements FilterInterface
{
    // This function adds the filter context, available in the api resource providers
    public function apply(Request $request, bool $normalization, array $attributes, array &$context): void
    {
        $productInStock = $request->query->get('product_in_stock');
        if (!$productInStock) {
            return;
        }

        $context['product_in_stock'] = $productInStock;
    }

    // This function is only used to hook in documentation generators (supported by Swagger and Hydra)
    public function getDescription(string $resourceClass): array
    {
        return [
            'product_in_stock' => [
                'property' => null,
                'type' => 'string',
                'required' => false,
            ],
        ];
    }
}
```

The provider defined for the `ProductResource` class will now have access to the filters within the `$context` argument of the `provide` method and these can be used when filtering the collection, in your repository for example:

```php
public function provide(Operation $operation, array $uriVariables = [], array $context = []): object|array|null
{
    if ($operation instanceof CollectionOperationInterface) {
        // e.g. $context['filters']['product_in_stock'] = 1
        if (isset($context['filters'])) {
            $collection = $this->productRepository->filterBy($context['filters']);
        } else {
            $collection = $this->productRepository->findAll();
        }
        // transform your collection to your api resource and return
        return $collection;
    }
}
```
