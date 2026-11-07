# Membit and n8n: integration guide

Membit ships two MCP primitives that look alike and behave nothing alike. Most of the
vendor documentation assumes you have the first one. A lot of accounts only have the
second, and that is where integrations get stuck.

This repo documents the path that works with nothing but an API key and a stock n8n
install: call the Membit REST API directly from an HTTP Request node.

No custom node. No MCP Tool. No upgrade.

| | MCP Tool | MCP Trigger |
|---|---|---|
| Direction | outbound, your workflow sends data to Membit | inbound, fires when Membit sends to you |
| Can push a message to Membit | yes | no |
| Available by default | no | yes |

If you have the Tool, use it. If all you have is the Trigger, the rest of this page is
the workaround, and it is fully functional.

## Prerequisites

- n8n running locally or on a server
- A Membit API key
- The Membit endpoint you intend to call
- Enough n8n familiarity to add and connect nodes

## How the flow is shaped

```
Chat Trigger  ->  HTTP Request (Membit REST)  ->  Response  ->  Telegram / Slack
```

One node does the real work. Everything else is input and delivery.

---

## Step 1. Add an HTTP Request node

Open the node panel from the workflow editor and add a fresh node.

<img src="https://github.com/user-attachments/assets/49fdda79-8575-48a0-a4e7-7e2a011858b3" alt="n8n node panel open on a new workflow" width="376" height="315">

*Figure 1: the node panel, starting point for every step below.*

Search for **HTTP Request** and add it to the canvas.

<img src="https://github.com/user-attachments/assets/e8241729-13ef-4247-9d0b-2c6cc8e8dee5" alt="HTTP Request node selected in the n8n node search results" width="376" height="315">

*Figure 2: the HTTP Request node. This replaces the MCP Tool for our purposes and speaks
to Membit directly over HTTPS.*

---

## Step 2. Point it at Membit

The fastest way to get the request shape right is to paste a cURL command and let n8n
fill in the fields. Open the node, choose **Import cURL**, and paste this:

```bash
curl --request GET \
  --url 'https://api.membit.ai/v1/clusters/search?limit=10&format=json&q=dog' \
  --header 'X-Membit-Api-Key: <api-key>'
```

<img src="https://github.com/user-attachments/assets/a4aab9b0-1288-41cb-bb8f-74d154d043db" alt="Import cURL dialog inside the n8n HTTP Request node" width="539" height="229">

*Figure 3: the Import cURL dialog. Paste, import, and the method, URL and headers are
populated for you.*

Replace `<api-key>` with your own key. Keep the closing `'` where it appears, since that
quote closes the shell string around the header rather than belonging to the key.

The query is a search for the cluster `dog`. Whatever you put in `q` is what Membit
answers about, so this is the field you will end up templating from your trigger.

---

## Step 3. Wire a trigger to it

Connect a trigger node to the request. A Chat Trigger is the quickest way to test, since
you can type at it and see the response immediately.

<img src="https://github.com/user-attachments/assets/d2f70fba-59a8-48ed-85f5-f056fd301046" alt="n8n canvas with a Chat Trigger connected to the HTTP Request node" width="431" height="233">

*Figure 4: two nodes, one connection. Trigger on the left, Membit call on the right.*

---

## Step 4. Test it

Run the workflow and message the trigger. With `q=dog`, only a message about dogs returns
useful clusters, which makes it an easy way to confirm the pipe is live before you build
anything real.

<img src="https://github.com/user-attachments/assets/99312c37-9adb-4fdb-8b51-53e421c3cb2f" alt="n8n execution panel showing the query param and the returned response" width="720" height="348">

*Figure 5: the execution panel. The incoming query on one side, the Membit response on the
other. If the body is empty, check `q` before you check anything else.*

To make it respond to real input, open the node and replace the hardcoded `dog` with an
expression that reads the incoming message.

---

## Step 5. Move the key out of the URL

Query strings end up in logs, workflow exports and screenshots, so the key does not belong
in the node parameters. n8n has a credential type for exactly this.

In the HTTP Request node:

1. Set **Authentication** to `Generic Credential`
2. Set **Generic Authentication Type** to `Header Auth`
3. Create a new Header Auth credential

<img src="https://github.com/user-attachments/assets/d1bd13b4-005f-408d-9146-170e99e7f262" alt="HTTP Request node authentication settings showing Generic Credential and Header Auth" width="423" height="400">

*Figure 6: the authentication block. Generic Credential, then Header Auth.*

In the credential itself:

| Field | Value |
|---|---|
| Name | `X-Membit-Api-Key` |
| Value | your Membit API key |

<img src="https://github.com/user-attachments/assets/dc408c42-1e28-4fb2-bca4-f43f050dd957" alt="Header Auth credential form with the X-Membit-Api-Key header name and value fields filled in" width="665" height="365">

*Figure 7: the credential editor. Set it once, and every request using this node carries
the header.*

The parameter name is case sensitive in some proxies, so copy `X-Membit-Api-Key` exactly
as written.

---

## Checking the result

The workflow is working when all of these hold:

- The execution returns HTTP 200
- The response body is JSON
- The returned clusters relate to your `q` value
- Removing the credential produces a 401, which proves the header is what authorises you

That last one is worth doing once, because it confirms you are not silently relying on an
unauthenticated endpoint that could change behaviour later.

## Things worth knowing

- Some endpoints answer without authentication. Configure header auth anyway, so the
  workflow keeps working when that stops being true.
- `limit` caps results per call. For anything dashboard shaped, paginate rather than
  raising `limit` past what you will actually render.
- This guide covers the REST path only. If your account has the MCP Tool, that route is
  shorter and worth preferring.

## Scope

Written from a real setup against Membit's REST API in November 2025. No affiliation with
either vendor. If an endpoint or credential name changes upstream, this page will be wrong
in exactly one place, and that place is worth a pull request.
