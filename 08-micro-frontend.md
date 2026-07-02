# ⊞ How to Embed the Assessment Service

Want to show your assessments inside your own website, Learning Management System (LMS), or app? You're in the right place! 

This guide explains the easiest ways to seamlessly blend the Assessment Service right into your own platform so your users never feel like they left your website.

---

## 💡 Method 1: The Easy Way (iFrames)

The absolute easiest and most common way to embed an assessment is by using a simple HTML `<iframe>`. 

When you use an iframe, our system is smart enough to detect it! It will automatically hide our top navigation bars and make our background transparent so that the assessment blends perfectly into your website's design.

### How to do it securely in 3 easy steps:

**Step 1: Whitelist your website**
To keep your data safe, we block random websites from embedding your assessments. 
* Go to your Workspace Settings.
* Find the `Allowed Origins` setting.
* Type in your website's URL (for example: `https://my-awesome-lms.com`).

**Step 2: Get an Embed Token (Securely)**
For security, we now use a token-based approach. Your backend server should request a one-time token from our API using your Client ID and Secret. 

Make a `POST` request to `https://api.molika.app/api/v1/auth/embed-token` from your backend:
```json
{
  "clientId": "YOUR_CLIENT_ID",
  "clientSecret": "YOUR_CLIENT_SECRET",
  "origin": "https://your-website.com"
}
```
This will return an `access_token` that you pass to your frontend.

**Step 3: Render the iframe**
Now that your frontend has the secure token, you can pass it to the iframe URL using the `?token=` parameter.

```html
<iframe 
  src="https://assessment-service.molika.app/assessments/12345?token=YOUR_SECURE_TOKEN" 
  width="100%" 
  height="600px" 
  frameborder="0">
</iframe>
```

### Talking to the iFrame (Advanced but easy!)
Your website can actually "talk" to the assessment! We use a standard browser feature called `window.postMessage`.

* **Knowing when they finish:** When a user completes the test, the iframe will shout out `ASSESSMENT_COMPLETED` to your website, along with their score. You can listen for this to unlock the next lesson in your LMS!
* **Theme & Customization Sync:** You can tell the iframe to switch themes or inject CSS by sending a `SYNC_THEME` message when the iframe sends `EMBED_READY`.
```javascript
// Example of passing theme colors to the iframe
iframeRef.current.contentWindow.postMessage({
  type: "SYNC_THEME",
  mode: "light",
  theme: {
    "--primary": "210 100% 50%", // Use your own HSL primary color
    "--radius": "0.5rem"
  }
}, "*");
```

---

## 💡 Method 2: The Native Way (Next.js Multi Zones)

If you are building your own website using **Next.js**, you can use a cool feature called **Multi Zones**. This makes the Assessment Service feel like it is literally part of your own codebase.

Instead of an iframe, you tell your website to secretly fetch the Assessment Service whenever someone goes to a specific URL.

### Why is this awesome?
* No more iframe borders or weird scrolling issues.
* The URL looks like it's yours! (e.g. `https://your-website.com/assessments/123`).
* It shares your cookies perfectly, so users stay logged in without any hiccups.

### How to do it:
In your main website's `next.config.js` file, you just add a "rewrite" rule. 

```javascript
module.exports = {
  async rewrites() {
    return [
      {
        // When a user goes to your website's /assessments path...
        source: '/assessments/:path*',
        // ...secretly show them the Assessment Service!
        destination: 'https://assessment-service.molika.app/assessments/:path*',
      },
    ]
  },
}
```

---

## 💡 What about Admin Logins? (SSO)

If you want your *teachers* or *admins* to log into your main website, and automatically be logged into the Assessment Service workspace without typing their password again, you can use **Single Sign-On (SSO)**.

1. Your main website logs the user in.
2. Your main website creates a secret JWT token for them.
3. You pass that token to the Assessment Service in the background.

*Tip: This works best if your main website and the Assessment Service share the same root domain (like `app.your-website.com` and `assessments.your-website.com`), because they can share the same login cookie!*
