## Config project

Install rocketseat cli to help create projects with a common structure:

`yarn global add @rocketseat/omni`

`omni init [name] --only=server`

Configure .env file:

```
APP_URL=http://localhost:3333
NODE_ENV=development

# Auth

APP_SECRET=templatenoderocketseat

# Database

DB_HOST=
DB_USER=
DB_PASS=
DB_NAME=
```

`yarn add jest -D`

`yarn jest --init`

Questions:

```
Would you like to use Jest when running "test" script in "package.json"? … yes
Choose the test environment that will be used for testing › node
Do you want Jest to add coverage reports? … yes
Automatically clear mock calls and instances between every test? … yes
```

Create `__tests__` folder:

## Config jest

Configurando jest.config.js:

```js
bail: 1,
clearMocks: true,
collectCoverage: true,
collectCoverageFrom: ['src/app/**/*.js'],
coverageDirectory: '__tests__/coverage',
coverageReporters: ['text', 'lcov'],
testEnvironment: 'node',
testMatch: ['**/__tests__/**/*.test.js'],
transform: {
'.(js|jsx|ts|tsx)': '@sucrase/jest-plugin',
},
```

Add sucrase feature to jest:

`yarn add --dev @sucrase/jest-plugin`

Add to nodemon.json:

```json
"ignore":["__tests__"]
```

Evita que o servidor restarte caso o `__tests__` seja alterado.

Habilitar auto complete nos arquivos de teste:

`yarn add -D @types/jest`

Criando testes:

Create example.test.js file in `__test__` folder

Example:

```javascript
function soma(a, b) {
  return a + b;
}

test("if I call soma funtion with 4 and 5 should return 9", () => {
  const result = soma(4, 5);

  expect(result).toBe(9);
});
```

Run tests:

`yarn test`

!>Tests can`t interfere on production data to avoid this we need to create
other database for tests

`yarn add sqlite3 -D`

Create .env.test file:

```
APP_URL=http://localhost:3333
NODE_ENV=development

# Auth

APP_SECRET=templatenoderocketseat

# Database

DB_DIALECT=sqlite
```

Alter database config file:

```js
dialect: process.env.DB_DIALECT || 'postgres',
storage: './__tests__/database.sqlite',
//reducing logs
logging:false
```

## Environment variables

Config import environment_variables depending of environment:

1. Create bootstrap.js file in src folder

```javascript
const dotenv = require("dotenv");

dotenv.config({
  path: process.env.NODE_ENV === "test" ? ".env.test" : ".env"
});
```

I had an error using import syntax. I changed to require.

2. Add in app.js:

```javascript
import "./bootstrap";
```

3. Alter require in database.js:

```javascript
require("../bootstrap");
```

4. Alter package.json to set NODE_ENV variable

```json
"scripts": {
    "test": "NODE_ENV=test jest"
  },
```

## Separated tests environments

Because jest is mult-thread tests can run simultaneously with others
and cause some unexpected errors. Then we need to create separated
environments to avoid this.

```
__test__
    └── util
        └── truncate.js

```

1. Create util folder;
2. Create truncate.js;

```js
import database from "../../src/database";
//force deletion on all table records
export default function truncate() {
  return Promise.all(
    Object.keys(database.connection.models).map(key => {
      return database.connection.models[key].destroy({
        truncate: true,
        force: true
      });
    })
  );
}
```

3. Import truncate.js

```js
import truncate from "../util/truncate";
```

4. Call truncate:

```js
describe('User', () => {
  // Called before any it execution
  beforeEach(async () => {
    await truncate();
  });

  it()...

  it()...
});
```
