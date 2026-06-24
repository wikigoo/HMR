# A breakdown of the key API settings for connecting Flowise assistant to a site.

---

## 1. The Core Endpoint

All chat interactions go through the **Prediction API**:

```
POST /api/v1/prediction/{chatflow-id}
```

Your `chatflow-id` is visible in the URL when you open your assistant in the Flowise dashboard. [1](#0-0) 

---

## 2. Access Control (API Key)

By default, your assistant is **public** — anyone with the chatflow ID can call it.

To restrict access:
1. Go to **API Keys** in the dashboard and create a key.
2. Open your assistant's **Settings → Configuration**, and assign that API key to it.
3. All API calls must then include the header:

```json
"Authorization": "Bearer <your-api-key>"
```
 [2](#0-1) 
---

## 3. Request Body Parameters

| Parameter | Required | Description |
|---|---|---|
| `question` | Yes | The user's message |
| `streaming` | No | Set `true` for real-time token streaming |
| `overrideConfig` | No | Override flow settings at runtime |
| `history` | No | Pass prior conversation messages |
| `uploads` | No | Images/audio as base64 or URL |
| `sessionId` | No (in overrideConfig) | Maintain conversation memory per user | [3](#0-2) 

---

## 4. Security Tab — What to Enable

In your assistant's **Settings → Configuration → Security** tab, you will find options to enable:

- **Override Config** — disabled by default. Enable this if you want to pass `overrideConfig` in API calls (e.g., to set `sessionId`, `temperature`, or `vars` dynamically).
- **Override Variables** — similarly must be enabled before you can pass `vars` through the API.

> If you don't enable these, any `overrideConfig` values sent in the request body will be ignored. [4](#0-3) [5](#0-4) 

---

## 5. Embedding on a Site (Alternative to Raw API)

Instead of calling the API directly, you can embed a chat widget:

```html
<script type="module">
  import Chatbot from 'https://cdn.jsdelivr.net/npm/flowise-embed/dist/web.js';
  Chatbot.init({
    chatflowid: 'your-chatflowid-here',
    apiHost: 'https://your-flowise-instance.com',
  })
</script>
```
 [6](#0-5) 

If you call the API from a different domain, you must set CORS environment variables on your Flowise server:

```
CORS_ORIGINS=*
IFRAME_ORIGINS=*
```
 [7](#0-6) 

---

## Summary: Recommended Checklist

- [ ] Copy your **Chatflow ID** from the dashboard URL
- [ ] Create an **API Key** and assign it to your assistant (if you want to restrict access)
- [ ] Enable **Override Config** in Security tab (if you need per-request session IDs or config)
- [ ] Set `CORS_ORIGINS` on your server if calling from a browser on a different domain
- [ ] Use `sessionId` inside `overrideConfig` to keep each user's conversation separate

### Citations

**File:** en/using-flowise/prediction.md (L15-17)
```markdown
**Base URL**: `http://localhost:3000` (or your Flowise instance URL)

**Endpoint**: `POST /api/v1/prediction/:id`
```

**File:** en/using-flowise/prediction.md (L38-46)
```markdown
| Parameter        | Type    | Required                    | Description                                 |
| ---------------- | ------- | --------------------------- | ------------------------------------------- |
| `question`       | string  | Yes                         | The message/question to send to the flow    |
| `form`           | object  | Either `question` or `form` | The form object to send to the flow         |
| `streaming`      | boolean | No                          | Enable streaming responses (default: false) |
| `overrideConfig` | object  | No                          | Override flow configuration                 |
| `history`        | array   | No                          | Previous conversation messages              |
| `uploads`        | array   | No                          | Files to upload (images, audio, etc.)       |
| `humanInput`     | object  | No                          | Return human feedback and resume execution  |
```

**File:** en/using-flowise/prediction.md (L440-444)

### Configuration Override

Override chatflow settings dynamically.

Override config is **disabled** by default for security reasons. Enable it from the top right: **Settings** → **Configuration** → **Security** tab:

**File:** en/configuration/authorization/chatflow-level.md (L9-29)

After you have a chatflow / agentflow constructed, by default, your flow is available to public. Anyone that has access to the Chatflow ID is able to run prediction through Embed or API.

In cases where you might want to allow certain people to be able to access and interact with it, you can do so by assigning an API key for that specific chatflow.

## API Key
In dashboard, navigate to API Keys section, and you should be able to see a DefaultKey created. You can also add or delete any keys.

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Chatflow

Navigate to the chatflow, and now you can select the API Key you want to use to protect the chatflow.

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

After assigning an API key, one can only access the chatflow API when the Authorization header is provided with the correct API key specified during a HTTP call.

```json
"Authorization": "Bearer <your-api-key>"
```

**File:** en/using-flowise/variables.md (L23-31)

### Override or setting variable through API

In order to override variable value, user must explicitly enable it from the top right button:

**Settings** -> **Configuration** -> **Security** tab:

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

If there is an existing variable created, variable value provided in the API will override the existing value.

**File:** en/using-flowise/embed.md (L9-31)
You can easily add the chat widget to your website. Just copy the provided widget script and paste it anywhere between the `<body>` and `</body>` tags of your HTML file.

<figure><img src="../.gitbook/assets/image (8) (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Widget Setup
The following video shows how to inject the widget script into any webpage.

{% embed url="https://github.com/FlowiseAI/Flowise/assets/26460777/c128829a-2d08-4d60-b821-1e41a9e677d0" %}

## Using Specific Version

You can specify the version of flowise-embed's `web.js` to use. For full list of versions: [https://www.npmjs.com/package/flowise-embed](https://www.npmjs.com/package/flowise-embed)

```html
<script type="module">
  import Chatbot from 'https://cdn.jsdelivr.net/npm/flowise-embed@<some-version>/dist/web.js';
  Chatbot.init({
    chatflowid: 'your-chatflowid-here',
    apiHost: 'your-apihost-here',
  })
</script>
```
**File:** en/using-flowise/embed.md (L271-284)

## CORS
When using embedded chat widget, there's chance that you might face CORS issue like:

{% hint style="danger" %}
Access to fetch at 'https://\<your-flowise.com>/api/v1/prediction/' from origin 'https://\<your-flowise.com>' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
{% endhint %}

To fix it, specify the following environment variables:

CORS_ORIGINS=*
IFRAME_ORIGINS=*
```
