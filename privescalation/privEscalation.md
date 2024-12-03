# Privilege Escalation

The example demonstrates a privilege escalation vulnerability and how to exploit it.

## Steps to reproduce

1. Install all dependencies

    `$ npm install`

2. Start the **insecure.ts** server

    `$ npx ts-node insecure.ts`

3. In the browser, send a GET request

    ```
        http://localhost:3000/send-form
    ```

4. Try different UserIds and see which one gives you authorized access to change the role of that user.

## For you to do

Answer the following:

1. Briefly explain the potential vulnerabilities in **insecure.ts**
The insecure.ts application contains several critical vulnerabilities related to privilege escalation. The most significant issue lies in its authentication and authorization mechanism. The code only checks if a user exists and has an admin role based on the user ID provided in the request body, without verifying the identity of the requesting user through any session or token-based authentication. This means the application trusts the user ID sent in the request without verifying if the sender actually has the authority to perform the requested action. Additionally, there's no validation of the newRole parameter, allowing any role value to be set without restrictions.

2. Briefly explain how a malicious attacker can exploit them.
A malicious attacker can exploit these vulnerabilities in several ways. The most straightforward attack involves manipulating the userId parameter in the request body. Since the application doesn't verify the authenticity of the request, an attacker can simply try different user IDs until they find one that has admin privileges. Once they find a valid admin ID, they can use it to modify any user's role in the system. For example, they could escalate their own privileges by changing their role to 'admin', or they could downgrade other administrators' privileges by changing their roles to 'user'. The lack of session management means these attacks can be performed without even needing to authenticate properly into the system.

3. Briefly explain the defensive techniques used in **secure.ts** to prevent the privilege escalation vulnerability?
The secure.ts implementation addresses these vulnerabilities through several defensive techniques. First, it implements proper session management using express-session middleware, which maintains user authentication state securely. The session cookie is configured with httpOnly and strict sameSite flags for additional security. The application verifies the user's identity through their session before processing any role update requests, ensuring that only authenticated users can attempt such operations. Furthermore, it implements a proper authorization check by verifying that the logged-in user (identified by the session) actually has admin privileges before allowing role modifications. This creates a proper separation between authentication (verifying identity) and authorization (verifying permissions), making it much more difficult for attackers to escalate privileges through unauthorized access.