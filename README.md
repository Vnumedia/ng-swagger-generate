ng-swagger-generate: A Swagger 2.0 codegen for Angular 2+
---

originally forked from https://github.com/cyclosproject/ng-swagger-gen

Difference from `ng-swagger-gen`:
- supporting multiple swagger.json files
- generating stubs client (for development and testing purpose)


## How to use it:
In your project, run:
```bash
cd <your_angular2+_app_dir>
npm install ng-swagger-generate --save-dev
```

Configure `ng-swagger-generate.json` (see _Configuration file_)

Run to generate HTTP client
```
node_modules/.bin/ng-swagger-generate
```
OR

Run to generate Stubs client
```
node_modules/.bin/ng-swagger-generate --stubs
```

Params:

- `--stubs` will use stubs client. Stubs will be copied from '/root/stubs` directory


## Configuration file

```json
{
  "$schema": "./node_modules/ng-swagger-generate/ng-swagger-generate-schema.json",
  "sources": ["my-swagger.json", "swagger2.json"],
  "apiModule": false
}
```
