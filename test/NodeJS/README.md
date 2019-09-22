# Starting Tests

## Create User test

Create a migration
Ex:

`yarn sequelize migration:create --name=create-users`

Change migration to create table users:

```js
return queryInterface.createTable("users", {
  id: {
    type: Sequelize.INTEGER,
    primaryKey: true,
    autoIncrement: true,
    allowNull: false
  },
  name: {
    type: Sequelize.STRING,
    allowNull: false
  },
  email: {
    type: Sequelize.STRING,
    allowNull: false
  },
  password_hash: {
    type: Sequelize.STRING,
    allowNull: false
  },
  creates_at: {
    type: Sequelize.DATE,
    allowNull: false
  },
  updated_at: {
    type: Sequelize.DATE,
    allowNull: false
  }
});
```

Alter package.json to load tests variables:

```json
"scripts":{
"pretest": "NODE_ENV=test sequelize db:migrate",
"test": "NODE_ENV=test jest",
"posttest": "NODE_ENV=test sequelize db:migrate:undo:all"
}
```

Run test:

`yarn test`

Create a model:

e.g.

```js
import Sequelize, { Model } from "sequelize";

class User extends Model {
  static init(sequelize) {
    super.init(
      {
        name: Sequelize.STRING,
        email: Sequelize.STRING,
        password_hash: Sequelize.STRING
      },
      {
        sequelize
      }
    );

    return this;
  }
}

export default User;
```

Create a file to write tests:

```
__test__
    └── integration
        └── user.test.js

```

Install supertest library. Use requests in tests (like axios but better in tests)

`yarn add supertest -D`

Writing a test:

```js
import request from "supertest";
import app from "../../src/app";
//Used to categorize tests
describe("User", () => {
  //similar to test() but with a better read
  it("should be able to resgister", async () => {
    const response = await request(app)
      .post("/users")
      .send({
        name: "Diego Fernandes",
        email: "diego@rocketseat.com.br",
        password_hash: "12345678"
      });

    expect(response.body).toHaveProperty("id");
  });
});
```

This test must fail because we don`t have the route to create a user.

Then we need to create the route in routes.js:

```js
routes.post("/users", (req, res) => {
  return res.json({ id: 1 });
});
```

If we run test again the test will pass because now the route returns an id.

Following TDD. Now we need to refactor our code to satisfy our code:

1. Create UserController.js:

```js
import User from "../models/User";

class UserController {
  async store(req, res) {
    const user = await User.create(req.body);

    return res.json(user);
  }
}

export default new UserController();
```

2. Update routes:

```js
import UserController from "./app/controllers/UserController";

routes.post("/users", UserController.store);
```

3. Run test:

`yarn test`

must return same result

## Code coverage

Just open coverage>lcov-report>index.html

## Duplicated email test

TDD

1. Create test:

```js
it("shouldn`t be able to register with a duplicated email", async () => {
  await request(app)
    .post("/users")
    .send({
      name: "Diego Fernandes",
      email: "diego@rocketseat.com.br",
      password_hash: "12345678"
    });

  const response = await request(app)
    .post("/users")
    .send({
      name: "Diego Fernandes",
      email: "diego@rocketseat.com.br",
      password_hash: "12345678"
    });

  expect(response.status).toBe(400);
});
```

2. Change UserController to match test:

```js
//Add following lines in store method
const { email } = req.body;

const checkEmail = await User.findOne({ where: { email } });

if (checkEmail) {
  return res.status(400).json({ error: "Duplicated e-mail" });
}
```

## Password hash test

For this test we need to install the following dependencies:

`yarn add bcryptjs`

1. Create test:

```js
it("should encrypt user password when new user created", async () => {
  const user = await User.create({
    name: "Diego Fernandes",
    email: "diego@rocketseat.com.br",
    password: "12345678"
  });

  const compareHash = await bcrypt.compare("12345678", user.password_hash);

  expect(compareHash).toBe(true);
});
```

Test must fail.

2. Alter User model to pass:

```js
import bcrypt from 'bcryptjs';

class User extends Model {
  static init(sequelize) {
    super.init(
      {
        ...
        password: Sequelize.VIRTUAL,
        ...
      },
      {
        sequelize,
      }
    );

    this.addHook('beforeSave', async user => {
      if (user.password) {
        user.password_hash = await bcrypt.hash(user.password, 8);
      }
    });

    return this;
  }
}
```

test passed!

## Generate random data

To do the following steps we need to install the denpendencies below:

`yarn add factory-girl faker -D`

Faker Documentation
https://github.com/marak/Faker.js/

1. Create factories.js:

```
__test__
    └── factories.js
```

```js
import faker from "faker";
import { factory } from "factory-girl";

import User from "../src/app/models/User";

factory.define("User", User, {
  name: faker.name.findName(),
  email: faker.internet.email(),
  password: faker.internet.password()
});

export default factory;
```

2. Change import on user.test.js:

```js
- import User from '../../src/app/models/User';
+ import factory from '../factories';
```

3. Change create method:

```js
const user = await factory.create("User", {
  // if we need to compare hash for example.
  // we want to know the password created
  // This avoid to create a random password
  password: "12345678"
});
```

```js
// Create just user data not instance of User
  const user = await factory.attrs('User');

  const response = await request(app)
      .post('/users')
      .send(user);
});
```

For each model create a factory.

```
projeto
├── package.json
├── src
│ ├── app.js
│ ├── routes.js
│ └── server.js
└── yarn.lock
```
