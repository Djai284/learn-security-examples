# Spoofing

This example demonstrates spoofind through two ways -- Stealing cookies programmatically and cross site request forgery (CSRF).

## Steps to reproduce the vulnerability

1. Install dependencies

    `$ npx install`

2. Start the **insecure.ts** server

    `npx ts-node insecure.ts`

3. Start the malicious server **mal.ts**

    `npx ts-node mal.ts`

4. Open __http://localhost:8000__ in a browser, type a name and Submit.

5. Open the __Application__ tab in the Browser's inspect pane. Find the __Cookies__ under __Storage__. You should see a __connect.sid__ cookie being set.

6. Open the HTML file __mal-steal-cookie.html__ file in the same browser (different tab). Open inspect and view the console.

7. Click the link in the HTML file. Do you see the cookie being stolen in the console?

8. Open the HTML file __mal-csrf.html__ file in the same browser (different tab). What do you see if the user has not logged out of **insecure.ts**? What do you see if the user has logged out? 


## For you to answer

1. Briefly explain the spoofing vulnerability in **insecure.ts**.

The main vulnerability stems from insecure session cookie configuration in the express-session middleware. Specifically:
- httpOnly: false allows client-side JavaScript to access the session cookie
- No sameSite attribute is set, making it vulnerable to cross-site requests
- Using a hardcoded secret ("SOMESECRET") which is not secure for production

2. Briefly explain different ways in which vulnerability can be exploited.

- Cookie Theft (shown in mal-steal-cookie.html):
  Since httpOnly: false, malicious JavaScript can access document.cookie
  The malicious site can steal the session cookie through JavaScript and send it to an attacker's server
  Attacker can then use this cookie to impersonate the legitimate user

- Cross-Site Request Forgery (CSRF) (shown in mal-csrf.html):
  Without sameSite protection, a malicious site can make POST requests to http://localhost:8000/sensitive
  If user is already authenticated (has valid session cookie), the browser will automatically include it
  The attacker can trick the user into performing unwanted actions while logged in
  For example, if the user is an Admin, the attacker could trigger the sensitive operation without their knowledge

3. Briefly explain why **secure.ts** does not have the spoofing vulnerability in **insecure.ts**.

The key difference is that secure.ts implements proper security headers and cookie configurations that follow security best practices, while insecure.ts leaves these protections disabled or misconfigured.

- Sets httpOnly: true which prevents JavaScript from accessing the cookie
- Enables sameSite: true which prevents the cookie from being sent in cross-site requests
- Takes secret as a command line argument instead of hardcoding it
- These security configurations prevent both cookie theft (through JavaScript) and CSRF attacks (through cross-site requests)

