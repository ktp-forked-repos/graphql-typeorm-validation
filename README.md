# Graphql typeorm validation

Map GraphQL type definitions `@constaints` directives to validations in typeorm entities.
This allows you to kep validations in sync across app layers (mutations, entities) from a single source of truth (GraphQL type definitions)

Also allows decorating any type of entity model (not just typeorm) with `class-validator` validators, via GraphQL type definition constraints, converted to object form using [graphSchemaToJson](https://github.com/jjwtay/graphSchemaToJson)

## Status

This library has not been published yet and is still a WIP.
Has not even been tested. At this point just a collection of functions which might be useful and should be composable enough to fit scenarios described below.

For more on this, see [this typeorm issue](https://github.com/typeorm/typeorm/issues/3135#issuecomment-445998746) the objective is fully fleshed out.

## TypeORM Entity validation

This library lets your wrap TypeORM `@Entity` classes with validation logic.

The validation constraints are extracted via [graphGenTypeorm](https://github.com/jjwtay/graphGenTypeorm) from a GraphQL type definition, defined using a `@constraints` directive.

`graphGenTypeorm` generates entity metadata used by typeorm to define the ORM for the entities.
This metadata also contains all the directives applied to each field, including `@constraints`

The function `buildEntityClassMap(connection)` can load this entity metadata from a given `connection`, extract the constraint metadata and apply it on each class property as [class-validator](https://github.com/typestack/class-validator) decorators.

In effect taking a typedef like:

```graphql
type Person {
  name: String! @constraints(minLength: 2, maxLength: 60)
}
```

And applying it via the typeorm connection "backdoor" to achieve the following entity (along with any DB related metadata)

```js
@Entity
class Person extends BaseEntity {
  @MinLength(2)
  @MaxLength(60)
  name: string;
}
```

When calling `buildEntityClassMap` you can pass a custom `decorator` function to apply custom validation logic as you see fit.

## Usage

```js
import { buildEntityClassMap } from 'graphql-typeorm-validation';

const entityClassMap = buildEntityClassMap(connection);
const { Post } = entityClassMap;
// ...
let postRepository = connection.getRepository('Post');

let post = new Post();
post.title = 'A new Post';
post.text = 'bla bla bla';
await validate(post, {
  // ... optional validation options
});
await postRepository.save(post);
```

This means that you can have your application generate GraphqL resolver logic to apply validation both on the input object (arguments) of a mutation and on the entity created to be saved in the DB. You can then add additional validation on the entity as you see fit.
This ensures that if you access the entities in other contexts, such as via a different API (f.ex REST), the validation defined in the GraphQL type definition still applies.

No more manual sync across your entire codebase!

The default decoration of `buildEntityClassMap` makes `async save()` and `async validate(opts)` available as instance methods on the Entity class, so that you can simplify it to:

```js
import { buildEntityClasses } from 'graphql-typeorm-validation';
const entityClassMap = buildEntityClassMap(connection);
const { Post } = entityClassMap;

let post = new Post();
post.title = 'A new Post';
post.text = 'bla bla bla';
const errors = await post.validate();
errors ? handleErrors(error) : await post.save();
```

## Package generator

This project (package) was bootstrapped using [typescript-starter](https://github.com/bitjson/typescript-starter)

## TypeORM Resources on entities metadata

- [Generate entity classes via connection entities metadata](https://github.com/typeorm/typeorm/issues/3141)
- [Entity configuration via JSON Schema](https://github.com/typeorm/typeorm/issues/1818)

Also see discusions on (old) issues in [graphGenTypeorm](https://github.com/jjwtay/graphGenTypeorm/issues)

- [Reuse @constraint directive for adding typeorm entity field validations](https://github.com/jjwtay/graphGenTypeorm/issues/1)
- [Building Entity classes from connection.entities metadata](https://github.com/jjwtay/graphGenTypeorm/issues/2)

## License

MIT
