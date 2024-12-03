# Information Disclosure

This example demonstrates information disclosure by injecting malicious query objects to a NoSQL database.

## Steps to reproduce

1. Install all dependencies

    `$ npm install`

2. Insert test data in the MongoDB database. Make sure the mongod is up and running by typing the `mongosh` command in the termainal. If mongod process is up then you will see that the connection was successful. Command to insert test data:

    `$ npx ts-node insert-test-users.ts`

This will create a database in MongoDB called __infodisclosure__. Verify its presence by connecting with mongosh and running the command `show dbs;`.

2. Start the **insecure.ts** server

    `$ npx ts-node insecure.ts`

3. In the browser, pretend to be a hacker and type a malicious request

    ```
        http://localhost:3000/userinfo?username[$ne]=
    ```

4. Do you see user information being displayed despite the malicious request not having a valid username in the request?

## For you to do

Answer the following:

1. Briefly explain the potential vulnerabilities in **insecure.ts**

The main vulnerability is NoSQL injection due to unsanitized user input in database queries. The application directly uses the query parameter in MongoDB's findOne operation without any validation or sanitization:
```ts
const { username } = req.query;
const user = await User.findOne({ username: username as string }).exec();
```
There's no type checking to ensure the username is actually a string. The code doesn't sanitize special MongoDB operators in the input. Error handling is minimal, which could lead to information leakage through error messages

2. Briefly explain how a malicious attacker can exploit them.

An attacker can use NoSQL injection techniques by manipulating the query parameters. The example query shown demonstrates this:
```
http://localhost:3000/userinfo?username[$ne]=
```
The `$ne` (not equal) operator tricks the query into matching ALL users where username is not equal to an empty string. This effectively bypasses authentication and returns all user records. 

Other MongoDB operators could be used for different attacks:
- `$gt`, `$lt` for range-based attacks
- `$regex` for pattern matching attacks
- `$where` for executing arbitrary JavaScript

The attacker could potentially extract sensitive user information including passwords

3. Briefly explain the defensive techniques used in **secure.ts** to prevent the information disclosure vulnerability?

Input validation to ensure username is a string:
```ts
if (typeof username !== 'string') {
  return res.status(400).send('Invalid username format');
}
```

Input sanitization to remove special characters that could be used for NoSQL injection:
```ts
const sanitizedUsername = username.replace(/[^\w\s]/gi, '');
```

Proper error handling with try-catch blocks:
```ts
try {
  // Query code
} catch (error) {
  console.error('Error querying database:', error);
  res.status(500).send('Internal server error');
}
```

Use of the sanitized username in the database query:
```ts
const user = await User.findOne({ username: sanitizedUsername }).exec();
```

The key difference is that secure.ts implements proper input validation, sanitization, and error handling to prevent NoSQL injection attacks and information disclosure, while insecure.ts directly uses unvalidated user input in database queries.