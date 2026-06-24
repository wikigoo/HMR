# Slack MCP

## 1. Prerequisites

Before using the Slack MCP node you need:

* **A Slack account**: sign up at [https://slack.com/](https://slack.com/)
* **A Slack workspace**
* **An OAuth client app**: generated inside your Slack workspace. This gives you a **Client ID** and **Client Secret**.

***

## 2. Setting Up Slack Credentials

### 2.1 Create a new app in Slack

1. Go to [https://api.slack.com/apps/new](https://api.slack.com/apps/new)
2. Create a new app, either via manifest or from scratch.

<figure><img src="../../../.gitbook/assets/image (174).png" alt=""><figcaption></figcaption></figure>

3. After an app is created, grab the Client ID and Client Secret.

<figure><img src="../../../.gitbook/assets/image (310).png" alt=""><figcaption></figcaption></figure>

### 2.2 Add Credentials in Flowise

1. In Flowise, navigate to **Credentials** from the sidebar.
2. Click **Add Credential** and search for **Slack User Token OAuth2**.
3. Fill in the following fields:

<table><thead><tr><th>Field</th><th>Description</th><th width="250">Example</th></tr></thead><tbody><tr><td><strong>Client ID</strong></td><td>The OAuth Client ID from Slack</td><td><code>wBSGhxxxx</code></td></tr><tr><td><strong>Client Secret</strong></td><td>The OAuth Client Secret (stored securely)</td><td><code>••••••••</code></td></tr><tr><td><strong>Scopes</strong></td><td><em>(Optional)</em> Space-separated scopes.</td><td><p><code>search:read.public search:read.private search:read.mpim search:read.im search:read.files search:read.users groups:history</code></p><p><code>mpim:history</code></p><p><code>im:history</code></p><p><code>channels:history</code></p><p><code>chat:write</code></p><p><code>canvases:read canvases:write</code></p><p><code>users:read</code></p><p><code>users:read.email</code></p></td></tr></tbody></table>

4. Copy OAuth Redirect URL, then click **Save**.

<figure><img src="../../../.gitbook/assets/image (338).png" alt="" width="548"><figcaption></figcaption></figure>

**Tip:** For production environments, use the narrowest scopes you need. See available [scopes](https://docs.slack.dev/reference/scopes/).

### 2.3 Add Redirect URL to Slack App

1. From the left side nav bar, select OAuth & Permissions:

<figure><img src="../../../.gitbook/assets/image (340).png" alt=""><figcaption></figcaption></figure>

2. Scroll down and you will see Redirect URLs section. Add the copied redirect URL from previous step. Then click Save URLs.

<figure><img src="../../../.gitbook/assets/image (341).png" alt="" width="563"><figcaption></figcaption></figure>

***

## 3. Adding Slack MCP

1. Drag and drop an Agent node.
2. Add a new Slack MCP tool.

<figure><img src="../../../.gitbook/assets/image (345).png" alt="" width="500"><figcaption></figcaption></figure>

3. Select preconfigured credential, and click the edit button. Click Authenticate.

<figure><img src="../../../.gitbook/assets/image (342).png" alt="" width="538"><figcaption></figcaption></figure>

3. A slack pop-up window will appear, review the permissions and click Allow.

<figure><img src="../../../.gitbook/assets/image (347).png" alt="" width="542"><figcaption></figcaption></figure>

4. After authenticated, click the refresh button to load the available actions.

<figure><img src="../../../.gitbook/assets/image (348).png" alt=""><figcaption></figcaption></figure>

5. Select the actions. _Tip: Select only the actions your agent needs. Fewer tools help the LLM make better decisions and reduce token usage._

<figure><img src="../../../.gitbook/assets/image (339).png" alt="" width="452"><figcaption></figcaption></figure>

6. Voila! You can start chatting with the Agent and see how its calling Slack MCP tools.

<figure><img src="../../../.gitbook/assets/image (349).png" alt=""><figcaption></figcaption></figure>
