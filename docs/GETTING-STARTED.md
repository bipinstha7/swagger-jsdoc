# Getting started

`swagger-jsdoc` returns the validated OpenAPI specification as JSON or YAML.

```javascript
const swaggerJsdoc = require('swagger-jsdoc');

const options = {
  swaggerDefinition: {
    info: {
      title: 'Hello World',
      version: '1.0.0',
    },
  },
  apis: ['./src/routes*.js'],
};

const swaggerSpecification = swaggerJsdoc(options);
```

- `options.definition` is also acceptable. Pass an [oasObject](https://swagger.io/specification/#oasObject)
- `options.apis` are resolved with [node-glob](https://github.com/isaacs/node-glob). Construct these patterns carefully in order to reduce the number of possible matches speeding up files' discovery. Values are relative to the current working directory.

Use any of the [swagger tools](https://swagger.io/tools/) to get the benefits of your `swaggerSpecification`.

## Specification version

`swagger-jsdoc` was created in 2015. The OpenAPI as a concept did not exist, and thus the naming of the package itself.

The default target specification is 2.0. This provides backwards compatibility for many APIs written in the last couple of years.

In order to create a specification compatibile with 3.0 or higher, i.e. the so called OpenAPI, set this information in the `swaggerDefinition`:

```diff
const swaggerJsdoc = require('swagger-jsdoc');

const options = {
  swaggerDefinition: {
+   openapi: '3.0.0',
    info: {
      title: 'Hello World',
      version: '1.0.0',
    },
  },
  apis: ['./src/routes*.js'],
};

const swaggerSpecification = swaggerJsdoc(options);
```

## Annotating source code

Place `@swagger` or `@openapi` on top of YAML-formatted specification parts:

```javascript
/**
 * @swagger
 *
 * /login:
 *   post:
 *     produces:
 *       - application/json
 *     parameters:
 *       - name: username
 *         in: formData
 *         required: true
 *         type: string
 *       - name: password
 *         in: formData
 *         required: true
 *         type: string
 */
app.post('/login', (req, res) => {
  // Your implementation comes here ...
});
```

## Re-using Model Definitions

The examples below are targeting specification 2.0. Please keep in mind that since 3.x, you can use [components](https://swagger.io/docs/specification/components/) in order to define and reuse resources.

A model may be the same for multiple endpoints: `POST`, `PUT` responses, etc.

Duplicating parts of your code into multiple locations is error prone and requires more attention when maintaining your code base. To solve these, you can define a model and re-use it across multiple endpoints. You can also reference another model and add fields.

```javascript
/**
 * @swagger
 *
 * definitions:
 *   NewUser:
 *     type: object
 *     required:
 *       - username
 *       - password
 *     properties:
 *       username:
 *         type: string
 *       password:
 *         type: string
 *         format: password
 *   User:
 *     allOf:
 *       - $ref: '#/definitions/NewUser'
 *       - required:
 *         - id
 *       - properties:
 *         id:
 *           type: integer
 *           format: int64
 */

/**
 * @swagger
 * /users:
 *   get:
 *     description: Returns users
 *     produces:
 *      - application/json
 *     responses:
 *       200:
 *         description: users
 *         schema:
 *           type: array
 *           items:
 *             $ref: '#/definitions/User'
 */
app.get('/users', (req, res) => {
  // Your implementation logic comes here ...
});

/**
 * @swagger
 *
 * /users:
 *   post:
 *     description: Creates a user
 *     produces:
 *       - application/json
 *     parameters:
 *       - name: user
 *         description: User object
 *         in:  body
 *         required: true
 *         type: string
 *         schema:
 *           $ref: '#/definitions/NewUser'
 *     responses:
 *       200:
 *         description: users
 *         schema:
 *           $ref: '#/definitions/User'
 */
app.post('/users', (req, res) => {
  // Your implementation logic comes here ...
});
```

## Using YAML

It's possible to source parts of your specification through YAML files.

Imagine having a file `x-amazon-apigateway-integrations.yaml` with the following contents:

```yaml
x-amazon-apigateway-integrations:
  default-integration: &default-integration
    type: object
    x-amazon-apigateway-integration:
      httpMethod: POST
      passthroughBehavior: when_no_match
      type: aws_proxy
      uri: 'arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/arn:aws:lambda:us-east-1:123456789:function:helloworldlambda/invocations'
```

The following is an acceptable reference to information from `x-amazon-apigateway-integrations.yaml` when it's defined within the `apis` input array.

```javascript
  /**
   * @swagger
   * /aws:
   *   get:
   *     description: contains a reference outside this file
   *     x-amazon-apigateway-integration: *default-integration
   */
  app.get('/aws', (req, res) => {
    // Your implementation comes here ...
  });
};
```

## Further resources

Additional materials to inspire you:

- [Document your Javascript code with JSDoc](https://dev.to/paulasantamaria/document-your-javascript-code-with-jsdoc-2fbf) - 20/08/2019
- [Swagger: Time to document that Express API you built!](https://levelup.gitconnected.com/swagger-time-to-document-that-express-api-you-built-9b8faaeae563) - 25/05/2019
  [Express API with autogenerated OpenAPI doc through Swagger](https://www.acuriousanimal.com/blog/2018/10/20/express-swagger-doc) - 20/10/2018
- [Express에 Swagger 붙이기](https://gongzza.github.io/javascript/nodejs/swagger-node-express/) - 18/07/2018
- [Swaggerize your API Documentation](http://imaginativethinking.ca/swaggerize-your-api-documentation/) - 01/06/2018
- [Swagger and NodeJS](https://mherman.org/blog/swagger-and-nodejs/) 20/11/2017
- [Agile documentation for your API-driven project](https://kalinchernev.github.io/agile-documentation-api-driven-project) - 21/01/2017

Suggestions for extending this helpful list are welcome! [Submit your article](https://github.com/Surnet/swagger-jsdoc/issues/new)
