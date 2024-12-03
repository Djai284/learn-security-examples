# Tampering

This example demonstrates tampering through script injection.

## Steps to reproduce

1. Install all dependencies

    `npm install`

2. Start the **insecure.ts** server

    `npx ts-node insecure.ts`

3. In the browser, type a potentially malicious script in the name field of the form

    ```
        <script> document.body.innerHTML = "<a href='https://google.com'> Gotcha </a>"</script>
    ```

4. Do you see the potentially malicious hyperlink being injected into the form?

## For you to do

Answer the following:

1. Briefly explain the potential vulnerabilities in **insecure.ts**

The main vulnerability is Cross-Site Scripting (XSS) due to unsanitized user input. When displaying the user's name in the welcome message, the raw user input is directly inserted into the HTML. The application stores the raw user input from req.body.name directly into the session without any sanitization. This creates a stored XSS vulnerability since malicious scripts can be persisted in the session and executed whenever the page loads

2. Briefly explain how a malicious attacker can exploit them.

An attacker can input malicious JavaScript code in the name field, like the example given:
```html
<script>document.body.innerHTML = "<a href='https://google.com'> Gotcha </a>"</script>
```
This script would execute when the page renders, allowing various attacks like:
- Modifying the page content (as shown in the example)
- Stealing sensitive information from the page
- Making unauthorized requests using the user's session
- Redirecting users to malicious websites
- Capturing user input like passwords
Since the malicious script is stored in the session, it will execute every time the user visits the page until they log out

3. Briefly explain why **secure.ts** does not have the same vulnerabilties?

It implements the `escapeHTML()` function that sanitizes user input by escaping HTML special characters:
```ts
function escapeHTML(input: string): string {
  return input
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&#39;");
}
```

The register endpoint uses this function to sanitize input before storing it:
```ts
const sanitizedName = escapeHTML(req.body.name.trim());
req.session.user = sanitizedName;
```

This ensures that any HTML or JavaScript in the user input is escaped and rendered as plain text rather than being executed as code. Even if an attacker inputs malicious scripts, they will be displayed as text rather than being executed