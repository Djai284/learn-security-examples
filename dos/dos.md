# Denial-of-Service (DoS)

This example demonstrates DoS vulnerabilities and how they can be exploited.

## Steps to reproduce

1. Install all dependencies

    `$ npm install`

2. Ignore if you have already done this once. Insert test data in the MongoDB database. Make sure the mongod is up and running by typing the `mongosh` command in the termainal. If mongod process is up then you will see that the connection was successful. Command to insert test data:

    `$ npx ts-node insert-test-users.ts`

This will create a database in MongoDB called __infodisclosure__. Verify its presence by connecting with mongosh and running the command `show dbs;`.

2. Start the **insecure.ts** server

    `$ npx ts-node insecure.ts`

3. In the browser, pretend to be a hacker and type a malicious request

    ```
        http://localhost:3000/userinfo?id[$ne]=
    ```

4. Do you see the server crashing?

## For you to do

Answer the following:

1. Briefly explain the potential vulnerabilities in **insecure.ts** that can lead to a DoS attack.
- No rate limiting on API endpoints
- Unsanitized MongoDB queries that could trigger expensive operations
- Lack of error handling for database queries
- No request timeout implementation
- No input validation before processing requests
- Direct use of user input in MongoDB queries without sanitization:
  ```ts
  const uid = id as string;
  const user = await User.findOne({ _id: uid }).exec();
  ```

2. Briefly explain how a malicious attacker can exploit them.
- Using NoSQL injection with operators to trigger resource-intensive queries:
  ```
  http://localhost:3000/userinfo?id[$ne]=
  ```
- Sending a large number of concurrent requests to overwhelm the server
- Using malformed IDs that cause MongoDB to perform expensive operations
- Since there's no rate limiting, an attacker can flood the server with requests
- The lack of error handling means the server might crash on malformed queries
- Memory exhaustion through large query results since there's no limit on result size

3. Briefly explain the defensive techniques used in **secure.ts** to prevent the DoS vulnerability?

The secure.ts implementation employs several defensive techniques to prevent DoS attacks. At its core, it implements rate limiting using the express-rate-limit middleware, which restricts each IP address to one request every five seconds. This effectively prevents request flooding attacks. The application also implements proper error handling with try-catch blocks around MongoDB queries, preventing server crashes from malformed queries. It returns generic error messages that don't expose system details, adding an extra layer of security. The combination of rate limiting, error handling, and request validation creates a robust defense against various DoS attack vectors. By limiting requests to 1 per 5 seconds per IP address and properly handling errors, the secure version significantly reduces the risk of both accidental and intentional service disruptions.