+++
title = "State management in Api Platform"
date = 2023-11-21
tags = ["symfony", "api-platform"]
draft = false
+++

Fetch data and mutate state for custom api resources when using the [Api Platform framework](https://api-platform.com/).

<!--more-->


## Define an api resource {#define-an-api-resource}

The following defines an api resource for a product. Note the provider classes set for the operations annotation.

```php
<?php

declare(strict_types=1);

namespace App\ApiResource;

use App\Entity\Product;
use ApiPlatform\Metadata\Get;
use App\State\ProductProvider;
use ApiPlatform\Metadata\ApiResource;
use ApiPlatform\Metadata\GetCollection;

#[ApiResource(
    shortName: 'Product',
    operations: [
        new Get(provider: ProductProvider::class),
        new GetCollection(provider: ProductProvider::class)
    ]
)]
final class ProductResource
{
    public string $id;
    public string $name;
    public int $price;

    public static function fromEntity(Product $entity): self
    {
        $productResource = new self();
        $productResource->id = (string) $entity->id();
        $productResource->name = $entity->name();
        $productResource->price = $entity->price();

        return $productResource;
    }
}
```


## State Providers {#state-providers}

The following defines a state provider for the above resource.

```php
<?php

declare(strict_types=1);

namespace App\State;

use ApiPlatform\Metadata\Operation;
use App\ApiResource\ProductResource;
use App\Repository\ProductRepository;
use ApiPlatform\State\ProviderInterface;
use ApiPlatform\Metadata\CollectionOperationInterface;
use Symfony\Component\Uid\Ulid;

final class ProductProvider implements ProviderInterface
{
    public function __construct(private readonly ProductRepository $productRepository)
    {
    }

    public function provide(Operation $operation, array $uriVariables = [], array $context = []): object|array|null
    {
        if ($operation instanceof CollectionOperationInterface) {
            $products = [];

            foreach ($this->productRepository->findAll() as $product) {
                $products[] = ProductResource::fromEntity($product);
            }

            return $products;
        }

        return ProductResource::fromEntity($this->productRepository->find(Ulid::fromString($uriVariables['id'])));
    }
}
```


## State Processors {#state-processors}

Continuing with the above resource, in order to mutate its state a new operation should be configured. E.g to enable post request to create new product:

```php
#[ApiResource(
    shortName: 'Product',
    operations: [
        new Get(provider: ProductProvider::class),
        new GetCollection(provider: ProductProvider::class),
        new Post(processor: ProductProcessor::class), // new operation defining processor
    ]
)]
final class ProductResource
```

And the corresponding processor:

```php
// App\State\ProductProcessor.php

final class ProductProcessor implements ProcessorInterface
{
    public function process($data, Operation $operation, array $uriVariables = [], array $context = [])
    {
        // mutate and persist data as you like here, e.g:
        $this->productRepository->add(ProductResource::toEntity($data));

        return $data;
    }
}
```

Read the documentation for more info on [state providers](https://api-platform.com/docs/v3.2/core/state-providers/) and [state processors](https://api-platform.com/docs/v3.2/core/state-processors/).
