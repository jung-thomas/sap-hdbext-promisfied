# sap-hdbext-promisfied
Promisfied wrapper around @sap/hdbext

## Example

With the standard @sap/hdbext you use nested events and callbacks like this:

```js
    let client = req.db;
		client.prepare(
			`SELECT SESSION_USER, CURRENT_SCHEMA 
				             FROM "DUMMY"`,
			(err, statement) => {
				if (err) {
					return res.type("text/plain").status(500).send(`ERROR: ${err.toString()}`);
				}
				statement.exec([],
					(err, results) => {
						if (err) {
							return res.type("text/plain").status(500).send(`ERROR: ${err.toString()}`);
						} else {
							var result = JSON.stringify({
								Objects: results
							});
							return res.type("application/json").status(200).send(result);
						}
					});
				return null;
			});
```

However this module wraps the major features of @sap/hdbext in a ES6 class and returns promises. Therefore you could re-write the above block using the easier to read and maintain promise based approach:

```js
    const dbClass = require(global.__base + "utils/dbPromises");
		let db = new dbClass(req.db);
		db.preparePromisified(`SELECT SESSION_USER, CURRENT_SCHEMA 
				            	 FROM "DUMMY"`)
			.then(statement => {
				db.statementExecPromisified(statement, [])
					.then(results => {
						let result = JSON.stringify({
							Objects: results
						});
						return res.type("application/json").status(200).send(result);
					})
					.catch(err => {
						return res.type("text/plain").status(500).send(`ERROR: ${err.toString()}`);
					});
			})
			.catch(err => {
				return res.type("text/plain").status(500).send(`ERROR: ${err.toString()}`);
			});
```

Or better yet if you are running Node.js 8.x or higher you can use the new AWAIT feature and the code is even more streamlined:

```js
    try {
			const dbClass = require(global.__base + "utils/dbPromises");
			let db = new dbClass(req.db);
			const statement = await db.preparePromisified(`SELECT SESSION_USER, CURRENT_SCHEMA 
				            								 FROM "DUMMY"`);
			const results = await db.statementExecPromisified(statement, []);
			let result = JSON.stringify({
				Objects: results
			});
			return res.type("application/json").status(200).send(result);
		} catch (e) {
			return res.type("text/plain").status(500).send(`ERROR: ${e.toString()}`);
		}
```

### Methods
The following @sap/hdbext functions are exposed as promise-based methods

```js
prepare = preparePromisified(query) 
statement.exec = statementExecPromisified(statement, parameters) 
loadProcedure = loadProcedurePromisified(hdbext, schema, procedure)
storedProc = callProcedurePromisified(storedProc, inputParams)
```
