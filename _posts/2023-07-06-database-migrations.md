---
layout: post
title: "Running Database Migrations: Best Practices"
date: 2023-07-05
categories: [migrations, databases, db-migration, mongo]
---

![Databases](/assets/images/database.jpeg)

Database migrations are a crucial part of software development. They allow developers to make changes to a database schema over time, as their application evolves. In TypeScript, database migrations can be implemented using a tool like Typegoose, a TypeScript ORM for MongoDB. In this post, we'll cover best practices for running database migrations in TypeScript with Typegoose.

You're right, I apologize for the mistake. Here's the corrected version of the first section:

## 1. Provide Up and Down Functions and Implement Migration Logic

Each database migration should contain two functions: an `up` function that applies the migration changes to the database, and a `down` function that rolls back the changes. The `up` function should contain the logic for making the changes to the schema, such as creating or modifying collections or documents. The `down` function should reverse those changes, returning the schema to its previous state.

Here's an example of a simple migration that creates a new collection:

```typescript
import { QueryRunner } from 'typeorm';

export async function up(queryRunner: QueryRunner): Promise<void> {
  await queryRunner.createCollection('users');
}

export async function down(queryRunner: QueryRunner): Promise<void> {
  await queryRunner.dropCollection('users');
}
```

In this example, we're using TypeORM's `QueryRunner` to execute database queries. The `up` function creates a new collection called `users`, while the `down` function drops the `users` collection.

## 2. Add Unit Tests for the Migrations

Each migration should have a corresponding unit test that ensures the migration logic works as expected. The tests should cover both the `up` and `down` functions, and should be run as part of the automated testing suite.

Here's an example of a unit test for the migration above:

```typescript
import { QueryRunner } from 'typeorm';
import { up, down } from './1625489128534-create-user-collection.migration';

describe('CreateUserCollection migration', () => {
  let queryRunner: QueryRunner;

  beforeEach(() => {
    queryRunner = {
      createCollection: jest.fn(),
      dropCollection: jest.fn(),
    } as any;
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  describe('up', () => {
    it('should create the users collection', async () => {
      await up(queryRunner);
      expect(queryRunner.createCollection).toHaveBeenCalledWith('users');
    });
  });

  describe('down', () => {
    it('should drop the users collection', async () => {
      await down(queryRunner);
      expect(queryRunner.dropCollection).toHaveBeenCalledWith('users');
    });
  });
});
```

Here we're mocking the `QueryRunner` object with a Jest mock, and we're calling the `up` and `down` functions instead of creating a migration instance. We're also using `jest.clearAllMocks()` in the `afterEach` block to clear the mock calls between tests.

## 3. Store the State of the Migrations in the Database

The state of the migrations should be stored in a collection in the database, so that the system knows which migrations have already been run. This collection can be created using a model class provided by Typegoose.

Here's an example of the migration state collection:

```typescript
import { prop, getModelForClass } from '@typegoose/typegoose';

class MigrationState {
  @prop({ required: true })
  name: string;

  @prop({ required: true })
  service: string;

  @prop({ required: true })
  timestamp: Date;

  @prop({ required: true })
  success: boolean;
}

export const MigrationStateModel = getModelForClass(MigrationState);
```

In this example, we're using Typegoose's `prop` decorator to define the properties of the `MigrationState` class, and we're using the `getModelForClass` function to create a model class based on `MigrationState`.

## 4. Deploying Migrations with a CD System

When deploying to Kubernetes, it's a good practice to run a separate job for each service's migration. At the start of each job, it will run the `up` command, which will check the database to find which migrations didn't complete successfully. 

## 5. Running Migrations on Batches

When running migrations on large databases, you may face timeout issues or other performance problems. In this case, you can break up the migration into smaller batches and run them sequentially.

One way to do this is to use a tool like TypeORM's QueryRunner to batch your migration changes into smaller transactions. This can help reduce the risk of timeout issues or other performance problems.

By breaking up the migration into smaller batches, you can also easily roll back the migration if an error occurs. For example, if you're running a migration that modifies a large number of records, you may want to run the migration in batches of 1000 records at a time. If an error occurs during one of the batches, you can simply roll back the transaction and start again.

In general, it's a good idea to test your migration scripts on a development or staging environment first, to ensure that they can handle the size and complexity of your production database. You should also monitor the migration progress and performance during the migration process, to ensure that it's running smoothly.


## 6. Adding Retry Logic

In some cases, database migrations may fail due to network issues or other transient errors. To handle these situations, you can add retry logic to your migration scripts.

One way to add retry logic is to use a library like `retry` to retry the migration function a certain number of times before giving up. Here's an example of how you might add retry logic to a migration:

```typescript
import { QueryRunner } from 'typeorm';
import retry from 'async-retry';

export async function up(queryRunner: QueryRunner): Promise<void> {
  await retry(async () => {
    await queryRunner.createCollection('users');
  }, { retries: 3 });
}

export async function down(queryRunner: QueryRunner): Promise<void> {
  await retry(async () => {
    await queryRunner.dropCollection('users');
  }, { retries: 3 });
}
```

In this example, we're using the `retry` library to retry the `createCollection` and `dropCollection` functions up to three times before giving up.

Adding retry logic can help make your migration scripts more robust and resilient to transient errors.
## Conclusion
That's it for our best practices for running database migrations in TypeScript with Typegoose. By following these practices, you can ensure that your database schema evolves smoothly with your application, while minimizing the risk of data loss or corruption.
