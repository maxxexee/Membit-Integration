# 🧠 Membit – n8n Integration Guide
This repository provides a clear, reliable method for integrating Membit into n8n using only the Membit REST API, designed specifically for users who do not have access to the MCP Tool but still need full Membit functionality.

Membit’s documentation historically referenced “MCP Tools,” but many users only have access to MCP Trigger, which is not the same and cannot be used to send data to Membit.
This guide documents the correct and stable approach.

---

## 📌 What This Documentation Covers

- The difference between MCP Tool and MCP Trigger
- Why you don’t need the MCP Tool to integrate Membit
- How to call Membit using HTTP Request nodes
- A clear n8n node-by-node workflow architecture
- Reusable examples for your own automations  

---

## 🚀 Prerequisites
Make sure you have:

- **n8n running locally or on server**
- Membit **API key** (or endpoint token)
- Basic workflow knowledge
- The Membit HTTP endpoint you want to call

---
# 1. Understanding MCP Tool vs MCP Trigger

### **MCP Tool**
- Callable from within n8n  
- Lets your workflow **send** data to Membit  
- Not available to all users  

### **MCP Trigger**
- Fires **only when Membit sends events to you**  
- One-way: cannot be used to send messages to Membit  
- Available to all users by default  

> **Note:** If you only have **MCP Trigger**, you cannot push messages into Membit with it.  
> Fortunately, **Membit’s REST API can be used directly in n8n with standard HTTP Request nodes.**  
> This provides full capability without needing any MCP Tool.

---

# 2. Integration Approach

This workflow uses a simple and robust architecture:

1. Accept incoming text (Telegram, Webhook, etc.)  
2. Format the message for Membit  
3. Send it to Membit using an HTTP Request node  
4. Receive the processed response from Membit  
5. Deliver the result to your destination (Telegram, Slack, etc.)  

This structure is portable, predictable, and easy to extend.







---
## 3. Workflow Structure/Integration (Nodes Needed)
Below is the **exact node setup** required to call Membit inside n8n.

### Open the Node Panel
In your n8n workflow editor, open the node panel to add a new node to your workflow.
<p align="left">
  <img src="https://github.com/user-attachments/assets/49fdda79-8575-48a0-a4e7-7e2a011858b3" alt="node-panel" width="376" height="315">
</p>

### Search for the HTTP Request Node
In the node search bar, type “HTTP REQUEST” to find the Http Request Node and add to your workflow...


<img width="376" height="315" alt="http-request" src="https://github.com/user-attachments/assets/e8241729-13ef-4247-9d0b-2c6cc8e8dee5" />

> **Note:** This is our replacement for the MCP tool.  
> This node communicates directly with Membit’s API.  
> You can make GET or POST requests depending on your choice of query.

### Configuring the Membit Integration
Once you’ve added the HTTP Request node to your workflow, click on it and select Import cURL. Then paste the code below, replace "api-key" with your actual Membit API key.

```bash
curl --request GET \
  --url 'https://api.membit.ai/v1/clusters/search?limit=10&format=json&q=dog' \
  --header 'X-Membit-Api-Key: <api-key>'
```


  <img width="539" height="229" alt="cURL" src="https://github.com/user-attachments/assets/a4aab9b0-1288-41cb-bb8f-74d154d043db" />

> **Note:** Replace "api-key" with your actual api key. do not remove the single qoute at the end of the api key.
> Example: X-Membit-Api-Key:123456789'.

---
### Building Your First Workflow
Now that Membit is configured via HTTP, let’s create a complete workflow that uses real-time context.

> **Add the Required Nodes**  
> Create a complete workflow by adding and connecting the following nodes:

1. Chat Trigger - To receive user input
2. HTTP Request - To query Membit

> **Connect the Required Nodes**  
> Connect them in this order:  
> - Chat Trigger → HTTP Request
<img width="431" height="233" alt="flow" src="https://github.com/user-attachments/assets/d2f70fba-59a8-48ed-85f5-f056fd301046" />

### Testing the Workflow
In the code above, the cluster query is set to "dog". This means only chat triggers that match "dog" will return results. Any other messages won’t trigger anything.

Want different results? Double-click your HTTP Request Node and change the keyword in the parameters to whatever you like. Boom your workflow now responds to your custom query.
<img width="911" height="440" alt="query" src="https://github.com/user-attachments/assets/99312c37-9adb-4fdb-8b51-53e421c3cb2f" />

### Authentication
Currently, the workflow queries Membit without authentication, which is risky. To fix this:

- Open your HTTP Request Node.
- Set Authentication to Generic Credential.
- Set Generic Authentication Type to Header Auth.
- Create a new Header Auth credential with your API key.

<img width="423" height="400" alt="authentication" src="https://github.com/user-attachments/assets/d1bd13b4-005f-408d-9146-170e99e7f262" />

---
### Setting the Header Auth Credential:
- Name: X-Membit-Api-Key

- Value: Your actual Membit API key (keep the quotes if required)

This will attach your API key to every HTTP request, ensuring secure access to Membit.

<img width="665" height="365" alt="AuthAccount" src="https://github.com/user-attachments/assets/dc408c42-1e28-4fb2-bca4-f43f050dd957" />

---

## 🎉 You Made It!

Congrats! If you made it till the end, you’re officially ready to integrate Membit.  

Go forth, automate like a wizard, and may your workflows always return exactly what you asked for. 🚀  




