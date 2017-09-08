# KOEX.JS
#### [Knex.js](http://knexjs.org/), [Objection.js](vincit.github.io/objection.js), [EXpress](https://expressjs.com/)

In the beginnig of every project you have to do boring work like create basic entity with some basic attributes (e.g. `created_at`, `updated_at`),
implement CRUD operations and endpoints for every entity or for example implement hashing user password.

This helper library provides all of this and much more for every project! Just take a look..

The best on all of this is, that you can use any part of this library and than change, override or inject you own implementation.
For example: you want just CRUD router - so use CRUDRouter as express-middleware and inject your own DAO instance - no problem :) 

* Models to extend
    * BasicModel (`id`, `created_at`, `updated_at`, `updated_by`, `created_by`)
    * PublicModel (all from BasicModel + `uuid` - public identifier)
* DAO to extend
    * BasicDAO with methods: 
        * `create(object)`
        * `update(object)`
        * `remove(id)`
        * `getAll()`
        * `getById(id)`
        * `getByField(field)`
        * `getByCriteria`
* Routers to extends
    * EndpointRouter - contains express.Router instance with '/ping' endpoint for testing
    * CRUDRouter - contains CRUD routes connected with DAO
        * `POST '/'` create entity
        * `PUT '/:id'` update entity
        * `DELETE '/:id'` delete entity
        * `GET '/'` get all entities
        * `GET '/:id'` get entity


##### Requirements
* `npm install --save knex`
* `npm install --save objection`
* `npm install --save express`

This library provides implementation of basic classes and common parts for each backend.

Everything in API part is based on Express.js (Router). Everything in DB part is designed for Objection.js models and Knex.js queries (model implementation, query services - DAO or migrations).

## How to use?

## Crete database connection object
```javascript
//example for PostgreSql - use you favourite DB
module.exports = {
    test: {
        connection: 'postgres://postgres:test@localhost:5432/koex-test',
        client: 'pg',
        migrations: {
            directory: __dirname + "/src/api/db/migrations",
            tableName: "version"
        }
    },
};
```

## Init - initialize knex database connection
```javascript
//insert path to knex cofiguration file
require('adane-fw').API.DBFactory(__dirname+'/knexfile.js')
```
     
### Custom model, dao and router implementation
```javascript
//Class
// This custom model has created_at, created_by, updated_at, updated_by, uuid attributes
class CustomModel extends PublicModel {
    
    static get tableName() {
        return 'CUSTOM_MODEL';
    }
    
    static get jsonSchema() {
        return _merge(super.jsonSchema, {
            properties: {
                myCustomAttribyte: {type: 'string'}
            }
        })
    }
}
module.exports = CustomModel;

//DAO
// This DAO has standard CRUD operations implemented
class CustomDAO extends BasicDao {
    
}
module.exports = new CustomDAO(CustomModel);

//Router
// This Router has standard CRUD endpoints implemented
class CustomRouter extends CRUDRouter {
    initAdditionalEndpoints() {
        this.getRouter().get('/custom-endpoint', (req, res) => {
            res.status(200).send('Hello from custom endpoint');
        })
    }
}
module.exports = new CustomRouter(CustomDAO);

//Add router as middleware for express app
const CustomRouter = require('./CustomRouter').getRouter();
const app = new Express();
app.use('/custom-entity', CustomRouter);


//Migration - knex migration
const up = (kn, Promise) => {
    return kn.transaction(knex => {
        return () =>  knex.schema.createTable('CUSTOM_MODEL', table => {
            initPublicModelTable(table);
            table.string('customAttribute');
        });
    })
};
```

### Use pre-defined models (e.g. User, Permission, Account, ...)
```javascript
//UserModel, DAO and Router are done - we need just use migrations and router
const UserRouter = require('adane-fw').API.UserRouter.getRouter();

const app = new Express();
app.use('/user', UserRouter);

// Migrations - you can use own or pre-defined migrations
// There is standard knex migrations for User, Account, Role and Permission entities

//file: 001_user.js
module.exports = require('adane-fw').API.migrations.user;

//file: 002_account.js
module.exports = require('adane-fw').API.migrations.account;
//...
```
That's all :)