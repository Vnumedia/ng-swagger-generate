nd-swagger-generate-stubs: A Swagger 2.0 codegen for Angular 2+
---

originally forked from https://github.com/cyclosproject/ng-swagger-gen

This project is a NPM module that takes a [Swagger 2.0](http://swagger.io/)
JSON [specification](http://swagger.io/specification/) and generates two differen clients: 1: HTTPS: services
and model classes for an Angular 2+ project. 2. MOCK: http client + returning stubs instead of making real http calls.

This generator may not cover all usages of the Swagger 2.0 specifications.

Here are a few notes:

- The descriptor must be in JSON format. If you have your Swagger file in
  YAML format, use the [online swagger editor](http://editor.swagger.io) to
  export the descriptor as JSON;
- Each operation is assumed to have a single tag. If none is declared, a default
  of `Api`(configurable) is assumed. If multiple tags are declared, the first
  one is used;
- Each tag generates a service class;
- Operations that don't declare an id have an id generated. However, it is
  recommended that all operations define an id;
- File uploads are not supported;

## How to use it:
In your project, run:
```bash
cd <your_angular2+_app_dir>
npm install ng-swagger-generate --save-dev
```

Configure `ng-swagger-generate.json` (see _Using a configuration file_)

```
node_modules/.bin/ng-swagger-generate
```
Params:

- `--stubs` will use stubs client. Stubs will be copied from '/root/stubs` directory

The folder `src/app/api` (or your custom folder) will contain the following
structure:

```
project_root
+- src
   +- app
      +- api
         +- models
         |  +- model1.ts
         |  +- ...
         |  +- modeln.ts
         +- services
         |  +- tag1.service.ts
         |  +- ...
         |  +- tagn.service.ts
         +- stubs
         |  +- OperationName1.stub.ts
         |  +- ...
         |  +- OperationNameN.stub.ts
         +- api-configuration.ts
         +- api-response.ts
         +- api.module.ts
         +- models.ts
         +- services.ts
```

The files are:

- **api/models/model*n*.ts**: One file per model file is generated here.
  Enumerations are also correctly generated;
- **api/models.ts**: An index script which exports all model classes. It is
  used to make it easier for application classes to import models, so they can
  use `import { Model1, Model2 } from 'api/models'` instead of
  `import { Model1 } from 'api/models/model1'` and
  `import { Model2 } from 'api/models/model2'`;
- **api/services/tag*n*.service.ts**: One file per Swagger tag is generated
  here;
- **api/services.ts**: An index script which exports all service classes,
  similar to the analog file for models;
- **api/api-configuration.ts**: A configuration class that holds the following
  public static properties, which can be set directly in your `AppModule` (or
  some other imported module):
  - *rootUrl*: A string pointing to the root URL for the API. The default value
    is read from the Swagger description, from the `schemes`, `host` and
    `basePath` definitions;
  - *handleError*: A function that takes the error as input, and works as a
    general error handler for any error returned by the API. A global error
    handler can be disabled in the configuration file. In that case, this
    property is not generated;
  - *prepareRequestOptions*: A function that takes a `RequestOptions` object
    before any actual request. It can be used, for example, to set authorization
    headers, additional search parameters, etc.
- **api/api-response.ts**: A wrapper class that holds both the original response
  object, which has a type variable: `ApiResponse<T>`, where `T` is the type of
  the first operation response in the 2xx range. The following properties are
  available:
  - *response*: The original HTTP response;
  - *data*: The data returned by the operation, according to the operation
    successful response.
- **api/api.module.ts**: A module that declares an `NgModule` that provides all
  services. If this module is imported by your `AppModule` (or some other
  shared / core module) automatically all services are provided for dependency
  injection in your component constructors.

## Using a configuration file
If you place a file called `ng-swagger-generate.json` in the root folder of your
project, or in the current directory if `ng-swagger-generate` is installed globally,
the script parameters can be omitted. It is recommended to use a configuration
file, because it grants greater control over the generation.

If you have installed and saved the `ng-swagger-generate` module in your node
project, you can use a JSON schema in your configuration file pointing to
`./node_modules/ng-swagger-generate/ng-swagger-generate-schema.json`.
It is also possible to use the online version.

The supported properties in the JSON file are:

- `swagger`: The location of the swagger descriptor in JSON format.
  May be either a local file or URL.
- `output`: Where generated files will be written to. Defaults to `src/app/api`.
- `includeTags`: When specified, filters the generated services to be only
  those corresponding to this list of tags.
- `ignoreUnusedModels`: Indicates whether or not to ignore model files that are
  not referenced by any operation. Defaults to true.
- `minParamsForContainer`: Indicates the minimum number of parameters to wrap
  operation parameters in a container class. Defaults to 2.
- `defaultTag`: The assumed tag for operations that don't define any.
  Defaults to `Api`.
- `removeStaleFiles`: Indicates whether or not to remove any files in the
  output folder that were not generated by ng-swagger-generate. Defaults to true.
- `modelIndex`: Indicates whether or not to generate the file which exports all
  models. Defaults to true.
- `errorHandler`: Indicates whether or not to generate all service calls with a
  global error handler. Defaults to true.
- `serviceIndex`: Indicates whether or not to generate the file which exports
  all services. Defaults to true.
- `apiModule`: Indicates whether or not to generate the Angular module which
  provides all services. Defaults to true.
- `templates`: Path to override the Mustache templates used to generate files.

The following is an example of a configuration file which will choose a few
tags to generate, and chose not to generate the ApiModule class:
```json
{
  "$schema": "./node_modules/ng-swagger-generate/ng-swagger-generate-schema.json",
  "swagger": "my-swagger.json",
  "includeTags": [
    "Blogs",
    "Comments",
    "Users"
  ],
  "apiModule": false
}
```

This will not only generate only the services for the chosen tags, but models
which are not referenced by any of the generated services are skipped,
preventing the generation of unused classes.

## Setting up a node script
Regardless If your Angular project was generated or is managed by
[Angular CLI](https://cli.angular.io/), or you have started your project with
some other seed (for example, using [webpack](https://webpack.js.org/)
directly), you can setup a script to make sure the generated API classes are
consistent with the swagger descriptor.

To do so, create the `ng-swagger-generate.json` configuration file and add the
following in your `package.json`:
```json
{
  ...
  "scripts": {
    "ng": "ng",|
    "ng-swagger-generate": "ng-swagger-generate",|
    "start": "ng-swagger-generate --stubs && ng serve",
    "build": "ng-swagger-generate && ng build -prod",
    "lint": "ng lint"
  },
  ...
}
```
This way whenever you run `npm run start` or `npm run build`, the API classes
will be generated before actually serving / building your application.

## Swagger extensions
The swagger specification doesn't allow referencing an enumeration to be used
as an operation parameter. Hence, `ng-swagger-generate` supports the vendor
extension `x-type` in operations, whose value could either be a model name
representing an enum or `Array<EnumName>` or `List<EnumName>` (both are
equivallents) to use an array of models.
