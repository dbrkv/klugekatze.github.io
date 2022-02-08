---
title: "Symfony routing from yaml to attributes"
date: "2022-02-08"
tags:
    - symfony
    - rector
    - refactoring
    - php
---

> It's Over 9000!

Given: Symfony application with 99 controllers and 457 unique route definitions in YAML.
PhpStorm Symfony plugin helps a lot with navigation but still have limited refactoring capabilities. When all definitions are in one place it's simply better.
With new PHP Attributes syntax, decided to refactor YAML routing to attributes for easier maintenance.

Each route definition have minimum 2 config parameters: `path` and `name`. Many routes have `methods`,  `_defaults` and `requirements` defined, some contain `options` parameter. It seems quite some copy paste per route work.

Manual migration of one controller with 50 actions didn't go well: I had to restart it a few times due to human errors: forgot to copy/paste or paste into the wrong place. Wrong, computers should do all hard work: [Rector](https://github.com/rectorphp/rector) with custom rule to the rescue.

Refactoring consists of 3 steps:

* extract routes
* deduplicate and group them by a controller action (i18n)
* run rector to add `#[Route]` attributes.

As this one-off task, I did not spend too much time making migration scripts and rector rule beautiful.

Luckily, routes can be printed in machine-readable format with option `--format json`.

```shell
php bin/console debug:router --format json
```

It's big Symfony application with [JMSI18nRoutingBundle](https://github.com/schmittjoh/JMSI18nRoutingBundle), `debug:router` prints routes in all locales, need to get routes only from default locale and group them by a controller action:

```php
<?php

$json = json_decode(file_get_contents('php://stdin'), true, 50, JSON_THROW_ON_ERROR);

$routes = [];

function cleanupRouteDefinition(&$def): void {
    unset(
        $def['defaults']['_locale'],
        $def['defaults']['version'],
        $def['defaults']['_app'],
        $def['options']['compiler_class'],
        $def['options']['utf8'],
        $def['options']['i18n'],
        $def['options']['version'],
    );
}

foreach ($json as $k => $v) {
    $parts = explode('__RG__', $k);

    if (count($parts) === 1) {
        cleanupRouteDefinition($v);
        $routes[$k] = $v;

        continue;
    }

    // Use only default locale
    if (count($parts) === 2 && $parts[0] === 'en_US') {
        cleanupRouteDefinition($v);
        $routes[$parts[1]] = $v;
    }
}

$groups = [];

foreach ($routes as $k => $v) {


    if (!isset($v['defaults']['_controller'])) {
        continue;
    }

    $controller = $v['defaults']['_controller'];

    /*
     * controllers starting from "\"
     * Correct this:
     * \App\Controller\Work\BillingController::checkoutSubscription
     * Into This:
     * App\Controller\Work\BillingController::checkoutSubscription
     */

    $controller = preg_replace("#^\\\\#", '', $controller);

    /*
     * Remove path prefixes for api: /v1.4/auth/ => /auth/
     */
    $v['path'] = preg_replace("#^/v1.4#", '', $v['path']);

    $v['____name'] = $k;
    unset($v['defaults']['_controller']);
    $groups[$controller][] = $v;
}

file_put_contents('./routing-groups.json', json_encode($groups, JSON_THROW_ON_ERROR));
```

Run commands

```shell
php bin/console debug:router --format json | php removei18nroutes.php -
```

The result of execution is a file `routing-groups.json` with routes grouped by a controller action. Nice. Now need rector rule to get controller class public methods, try to find action in `routing-groups.json` and add `#[Route]` attribute with named arguments.

```php
<?php

declare(strict_types=1);

namespace Utils\Rector\Rector;


use PhpParser\Node;
use Rector\Core\Contract\Rector\ConfigurableRectorInterface;
use Rector\Core\Rector\AbstractRector;
use Symfony\Component\Routing\Annotation\Route;
use Symplify\RuleDocGenerator\ValueObject\CodeSample\CodeSample;
use Symplify\RuleDocGenerator\ValueObject\RuleDefinition;

class SymfonyYamlToAttributeRoutingRector extends AbstractRector implements ConfigurableRectorInterface
{
    public const ROUTES = 'ROUTES';

    private array $routes = [];

    public function configure(array $configuration): void
    {
        $this->routes = $configuration[self::ROUTES];
    }

    public function getRuleDefinition(): RuleDefinition
    {
        return new RuleDefinition(
            'Replace routing yaml to attributes',
            [
                new CodeSample(
                    'a',
                    'b',
                ),
            ]
        );
    }

    public function getNodeTypes(): array
    {
        return [
            Node\Stmt\ClassMethod::class,
        ];
    }

    /**
     * @param Node\Stmt\ClassMethod $node
     * @return Node|null
     */
    public function refactor(Node $node): ?Node
    {
        if (!$node->isPublic() || $node->isMagic()) {
            return null;
        }

        $class = $this->betterNodeFinder->findParentType($node, Node\Stmt\ClassLike::class);
        $className = $this->getName($class);
        $methodName = $this->getName($node);
        $controllerPath = $className . '::' . $methodName;
        $routeConfigs = $this->routes[$controllerPath] ?? null;

        if (!$routeConfigs) {
            return null;
        }

        $attrGroups = [];

        foreach ($routeConfigs as $routeConfig) {

            $args = [
                new Node\Arg(
                    value: new Node\Scalar\String_($routeConfig['path']),
                    name: new Node\Identifier('path'),
                ),
                new Node\Arg(
                    value: new Node\Scalar\String_($routeConfig['____name']),
                    name: new Node\Identifier('name'),
                ),
            ];

            if ($routeConfig['method'] !== 'ANY') {
                $method = array_map(
                    static fn(string $m) => new Node\Expr\ArrayItem(new Node\Scalar\String_($m)),
                    explode('|', $routeConfig['method'])
                );

                $args[] = new Node\Arg(
                    value: new Node\Expr\Array_($method),
                    name: new Node\Identifier('methods')
                );
            }

            if ($routeConfig['defaults']) {

                $defaultsArrayItems = [];

                foreach ($routeConfig['defaults'] as $defaultsKey => $defaultsValue) {
                    
                    $defaultsArrayItems[] = $this->getArrayItem(
                        is_string($defaultsKey) ? $defaultsKey : null,
                        $defaultsValue
                    );
                }

                $args[] = new Node\Arg(
                    value: new Node\Expr\Array_($defaultsArrayItems),
                    name: new Node\Identifier('defaults'),
                );
            }

            if ($routeConfig['options']) {
                $optionsArrayItems = [];

                foreach ($routeConfig['options'] as $optionsKey => $optionsValue) {
                    $optionsArrayItems[] = $this->getArrayItem(
                        is_string($optionsKey) ? $optionsKey : null,
                        $optionsValue,
                    );
                }

                $args[] = new Node\Arg(
                    value: new Node\Expr\Array_($optionsArrayItems),
                    name: new Node\Identifier('options')
                );
            }

            if (is_array($routeConfig['requirements'])) {
                $requirementsArrayItems = [];

                foreach ($routeConfig['requirements'] as $requirementKey => $requirementValue) {
                    $requirementsArrayItems[] = $this->getArrayItem(
                        is_string($requirementKey) ? $requirementKey : null,
                        $requirementValue,
                    );
                }

                $args[] = new Node\Arg(
                    value: new Node\Expr\Array_($requirementsArrayItems),
                    name: new Node\Identifier('requirements')
                );
            }

            $attrGroups[] = new Node\AttributeGroup([
                new Node\Attribute(
                    new Node\Name(Route::class),
                    $args,
                ),
            ]);
        }

        $node->attrGroups = $attrGroups;

        return $node;
    }

    private function getArrayItem(?string $k, string|array|float|bool $v): Node\Expr\ArrayItem
    {
        if (is_array($v)) {

            $valueFromArray = [];

            foreach ($v as $subK => $subV) {
                $valueFromArray[] = $this->getArrayItem($subK, $subV);
            }

            return new Node\Expr\ArrayItem(
                value: new Node\Expr\Array_($valueFromArray),
                key: $k === null ? null : new Node\Scalar\String_($k)
            );
        }

        $finalValue = match(true) {
            is_float($v) => new Node\Scalar\DNumber($v),
            is_string($v) => new Node\Scalar\String_($v),
            is_bool($v) => $v === true
                ? new Node\Expr\ConstFetch(new Node\Name('true'))
                : new Node\Expr\ConstFetch(new Node\Name('false')),
        };

        return new Node\Expr\ArrayItem(
            value: $finalValue,
            key: $k === null ? null : new Node\Scalar\String_($k)
        );
    }
}
```

And `rector.php` config

```php
<?php

declare(strict_types=1);

use Rector\Core\Configuration\Option;
use Symfony\Component\DependencyInjection\Loader\Configurator\ContainerConfigurator;
use Utils\Rector\Rector\SymfonyYamlToAttributeRoutingRector;

return static function (ContainerConfigurator $containerConfigurator): void {
    // get parameters
    $parameters = $containerConfigurator->parameters();
    $parameters->set(Option::PATHS, [
        __DIR__ . '/src/App/Controller'
    ]);
    $parameters->set(Option::IMPORT_SHORT_CLASSES, true);
    $parameters->set(Option::AUTO_IMPORT_NAMES, true);

    $services = $containerConfigurator->services();
    $services->set(SymfonyYamlToAttributeRoutingRector::class)
        ->configure([
            SymfonyYamlToAttributeRoutingRector::ROUTES => json_decode(file_get_contents('./routing-groups.json'), true),
        ]);
};
```

As this done, dry run and run refactor

```shell
vendor/bin/rector process --dry-run --clear-cache
vendor/bin/rector process --clear-cache
```

All refactoring is done in 27 seconds!
Need just to clean up old YAML routing files, resolve priority route conflicts.

Rector is a true lifesaver for repeating boring work.
