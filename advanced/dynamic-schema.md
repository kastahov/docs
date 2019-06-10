# Dynamic Entity Schema
Cycle ORM does not limit developer ability to carry entity information using unique classes, it is possible to use specifically crafted model
or models to represent entity data. ORM support an abiity to carry information using StdClass objects out of the box, it is also 
possible to combine classic annotated entities and StdClasses within one ORM scope.

## StdMapper
In order to represent entity as StdClass you have to change it's mapper from default `Cycle\ORM\Mapper\Mapper` to `Cycle\ORM\Mapper\StdMapper`.
This can be done by either defining your schema using schema builder or by writing schema manually.

## Using Schema Builder
We can build our entities using entity definitions:

```php
$e = new Entity();
$e->setRole('user');
$e->setMapper(Cycle\ORM\Mapper\StdMapper);

$entity->getFields()->set(
    'id', (new Field())->setType('primary')->setColumn('id')->setPrimary(true)
);

$r = new Registry($this->dbal);
$r->register($e)->linkTable($e, 'default', 'user');

$c = new Compiler();
$schema = $c->compile($r, [new RenderTables()]);

print_r($schema);

$orm = $orm->withSchema(new Schema($schema));
```

## Manual Schema Definition
You can also define StdClass schema manually using set of constants exposed by `Cycle\ORM\Schema` and `Cycle\ORM\Relation` classes:

```php
$orm = $orm->withSchema(new Schema([
  'user'    => [
        Schema::MAPPER      => StdMapper::class,
        Schema::DATABASE    => 'default',
        Schema::TABLE       => 'user',
        Schema::PRIMARY_KEY => 'id',
        Schema::COLUMNS     => ['id', 'email', 'balance'],
        Schema::SCHEMA      => [],
        Schema::RELATIONS   => [
            'comments' => [
                Relation::TYPE   => Relation::HAS_MANY,
                Relation::TARGET => 'comment',
                Relation::SCHEMA => [
                    Relation::CASCADE   => true,
                    Relation::INNER_KEY => 'id',
                    Relation::OUTER_KEY => 'user_id'
                ],
            ]
        ]
    ],
    'comment' => [
        Schema::MAPPER      => StdMapper::class,
        Schema::DATABASE    => 'default',
        Schema::TABLE       => 'comment',
        Schema::PRIMARY_KEY => 'id',
        Schema::COLUMNS     => ['id', 'user_id', 'message'],
        Schema::SCHEMA      => [],
        Schema::RELATIONS   => [],
    ]
]));
```

> More examples can be found in https://github.com/cycle/orm/tree/master/tests/ORM/Classless

## Usage
The only difference in usage of StdClass objects to carry your entity information is the need to identify entities using their roles.

To get access to entity repository:

```php
$userRepository = $orm->getRepository('user');
```

It is also required to create new entities using role specification instead of `new Class()`:

```php
$user = $orm->make('user', [/* fields* /]);
```

> You can freely assign custom repositories and constrains to your entities.