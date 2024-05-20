# Mongodb migrations

Dead simple non transactional migration service for mongodb.

## Usage:

```yml
  migrations:
    image: nbittich/mongodb-migration
    volumes:
      - ./config/migrations:/migrations
    environment: 
      MONGO_HOST: mongo
      MONGO_PORT: 27017
      MONGO_USERNAME: root
      MONGO_PASSWORD: root
    restart: "no"
```


## Environment variables:

- `MONGO_HOST` (default: `localhost`)
- `MONGO_PORT` (default: `27017`)
- `MONGO_USERNAME` (required)
- `MONGO_PASSWORD` (required)
- `MIGRATIONS_DIR` (default: `migrations`)
- `MIGRATIONS_DB` (default: `_migration`)

## Example

Add this in a file named e.g `202405220900-add-position.js`  under `/migrations` folder:

```js
const INSERTS = [
    {
      name: "Software engineer",
      description: `
        A software engineer is a person who applies the principles of software engineering to design, develop, maintain, test, and evaluate computer software. 
        The term programmer is sometimes used as a synonym, but may also lack connotations of engineering education or skills. 
        `,
      level: "OPERATIONAL",
    }
]
const execute = async (db, context = {}) => {
  const { now, uuid } = context;
  INSERTS.forEach((i) => {
    i.creationDate = now();
    i._id = uuid();
  });
  const positionCollection = await db.collection("position");
  await positionCollection.insertMany(INSERTS);
};

const rollback = async (db, _context = {}) => {
  const positionCollection = await db.collection("position");
  for (const { name } of INSERTS) {
    await positionCollection.deletOne({ name });
  }
};

module.exports = {
  targetDatabases: null, // force to run on all db
  description: "Add default positions",
  rollback,
  execute,
};
```
