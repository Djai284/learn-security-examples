# Repudiation

The example demonstrates a vulnerability that can lead to repudiation by malicious users attempting to access the services provided by a server.

## Steps to reproduce

1. Install all dependencies

    `$ npm install`

2. Run the server __insecure.ts__.

3. Pretend to be a malicous user and interact with the services by sending requests from the browser.

4. Do you think your actions can be repudiated?

## For you to do

1. Briefly explain the vulnerability.
The vulnerability in insecure.ts centers around its lack of proper logging and audit trail mechanisms. The application allows users to send and retrieve messages without maintaining any detailed records of these actions. While it does use console.log for basic request logging, this approach is inadequate for several reasons. First, console logs are ephemeral and typically don't persist between server restarts. Second, the logging is minimal and doesn't capture important details like timestamps, IP addresses, or the full context of actions being performed. This lack of comprehensive logging means that if a malicious user performs harmful actions, there would be no reliable way to trace these actions back to them, effectively allowing them to deny (repudiate) their actions.

2. Briefly explain why the vulnerability is addressed in __secure.ts__.
The secure.ts version addresses these vulnerabilities by implementing a robust logging system that creates an audit trail of all system activities. It uses the Observer design pattern to monitor and log all significant events that occur within the application. The implementation includes detailed logging of every request with timestamps, IP addresses, and specific actions performed. The logs are written to a persistent file using a write stream (server.log), ensuring that all activities are permanently recorded. Additionally, the secure version includes error logging and maintains contextual information about user actions, making it much more difficult for users to repudiate their actions since there's a clear, timestamped record of everything that happens in the system.

3. Which design pattern is used in the secure version to address the vulnerability? Briefly explain how it works?
The secure version implements the Observer design pattern through its middleware and logging system. In this pattern, the logger acts as an observer that monitors all activities in the application (the subject). The pattern is implemented using Express middleware that intercepts all requests before they reach their handlers. The middleware function observes and logs each request, capturing details like timestamp, HTTP method, URL, and IP address. This pattern is particularly effective because it separates the logging logic from the business logic, making the code more maintainable while ensuring that no significant action goes unrecorded. The implementation using fs.createWriteStream ensures that logs are written asynchronously without blocking the main application flow, while the error handling middleware ensures that even application errors are properly logged and tracked.