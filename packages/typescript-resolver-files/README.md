# @eddeee888/gcg-typescript-resolver-files

This [GraphQL Code Generator](https://www.the-guild.dev/graphql/codegen) plugin creates resolvers given GraphQL schema.

This relies on types generated from [typescript](https://the-guild.dev/graphql/codegen/plugins/typescript/typescript) and [@graphql-codegen/typescript-resolvers](https://the-guild.dev/graphql/codegen/plugins/typescript/typescript-resolvers) plugins.

```bash
yarn add -D @eddeee888/gcg-typescript-resolver-files
yarn add graphql-scalars
```

## Features

- Generates opinionated folder/file structure with type-safety in mind to help finding query, mutation or object types easily
- Does _NOT_ overwrite existing resolver logic
- Detects missing / wrong resolver exports and adds them to resolver files
- Automatically sets up GraphQL Scalars with types and config from [graphql-scalars](https://github.com/Urigo/graphql-scalars) with ability to opt-out
- Automatically extracts object type mapper interfaces and types (marked with `Mapper` suffix) from accompanying `.mappers.ts` files in each module. For example, if the schema file is `/path/to/schema.graphql`, the mapper file is `/path/to/schema.mappers.ts`.

## Example

### Setup

```graphql
# src/graphql/modules/base/schema.graphqls
type Query
```

```graphql
# src/graphql/modules/user/schema.graphqls
extend type Query {
  user(id: ID!): User
}

type User {
  id: ID!
  fullName: String!
}

type Address {
  id: ID!
  address: String!
}
```

````ts
// src/graphql/modules/user/schema.mappers.graphqls

// Exporting the following Mapper interfaces and types is the equivalent of this codegen config:
// ```yml
// mappers:
//   Address: './user/schema.mappers#AddressMapper'
//   User: './user/schema.mappers#UserMapper'
// ```

export { Address as AddressMapper } from 'address-package';

export interface UserMapper {
  id: string;
  firstName: string;
  lastName: string;
}
````

```yml
# codegen.yml
schema: '**/*.graphqls'
generates:
  src/graphql/modules:
    preset: '@eddeee888/gcg-typescript-resolver-files'
```

### Result

Running codegen will generate the following files:

- `src/graphql/modules/user/resolvers/Query/user.ts`
- `src/graphql/modules/user/resolvers/User.ts`
- `src/graphql/modules/user/resolvers/Address.ts`
- `src/graphql/modules/resolvers.generated.ts`
- `src/graphql/modules/types.generated.ts`

## Config

| Name                        | Type                                                                                                                             | Description                                                                                                                                                                                                                                                                  |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `mode`                      | `merged` or `modules`                                                                                                            | (Default: `modules`) How files are collocated. `modules` detects containing dir of a schema file as "modules", then split resolvers into those modules. `merged` treats `baseOutputDir` as the one and only module and generates resolvers.                                  |
| `resolverTypesPath`         | `string`                                                                                                                         | (Default: `./types.generated.ts`) Relative path to type file generated by `typescript-resolvers` plugin.                                                                                                                                                                     |
| `resolverRelativeTargetDir` | `string`                                                                                                                         | (Default: `resolvers`) Relative path to target dir. For `config.mode=merged`, files will be generated into `<baseOutputDir>/<resolverRelativeTargetDir>`. For `config.mode=modules`, files will be generated into `<baseOutputDir>/<moduleName>/<resolverRelativeTargetDir>` |
| `resolverMainFile`          | `string`                                                                                                                         | (Default: `resolvers.generated.ts`) File that puts all generated resolvers together. Relative from `baseOutputDir`                                                                                                                                                           |
| `whitelistedModules`        | `Array<string>`                                                                                                                  | (Only works with `config.mode=modules`) Whitelists modules to generate files and entries in main file. By default all modules are whitelisted. Useful for gradual migrations.                                                                                                |
| `blacklistedModules`        | `Array<string>`                                                                                                                  | (Only works with `config.mode=modules`) Blacklists modules to avoid generate files and entries in main file. Useful for gradual migrations.                                                                                                                                  |
| `externalResolvers`         | `Record<string, string>`                                                                                                         | Map of relative or absolute path (prefixed with `~`) to external or existing resolvers. e.g. `DateTime: ~graphql-scalars#DateTimeResolver`, `Query.me: '~@org/meResolver#default as meResolver'`, `User: 'otherResolvers#User as UserResolver'`.                             |
| `typesPluginsConfig`        | `(@graphql-codegen/typescript).TypeScriptPluginConfig & (@graphql-codegen/typescript-resolvers).TypeScriptResolversPluginConfig` | Takes [typescript config](https://www.the-guild.dev/graphql/codegen/plugins/typescript/typescript) and [typescript-resolvers config](https://www.the-guild.dev/graphql/codegen/plugins/typescript/typescript-resolvers) to override the defaults                             |
| `mappersFileExtension`      | `string`                                                                                                                         | (Default: `.mappers.ts`) The files with this extension provides mappers interfaces and types for the schema files in the same dir.                                                                                                                                           |
| `mappersSuffix`             | `string`                                                                                                                         | (Default: `Mapper`) Exported interfaces and types with this suffix from `mappersFile` in each module are put into the mappers object of [@graphql-codegen/typescript-resolvers](https://the-guild.dev/graphql/codegen/plugins/typescript/typescript-resolvers) .             |