# IM Bots

## 1. Overview

The IM Bot feature connects DBridge's AI Assistant to enterprise instant messaging platforms, allowing users to interact directly with AI in DingTalk, Feishu (Lark), and WeCom for data queries, analysis, and other tasks.

**Core Capabilities**:
- Support for DingTalk, Feishu, and WeCom platforms
- AI Agent binding for intelligent conversations
- Real-time connection status monitoring
- One-click bot start/stop

---

## 2. Accessing Bot Management

In the left navigation bar, click **IM Bots** to enter the bot management page.

---

## 3. Creating a Bot

### 3.1 Basic Information

Click the **Create Bot** button and fill in the dialog:

| Field | Description |
|-------|-------------|
| Name | Bot name for identification in the list |
| IM Type | Select platform: WeCom / Feishu / DingTalk |
| Bind Agent | Select associated AI Agent (optional) |

### 3.2 Platform Configuration

Depending on the selected IM type, different platform configuration fields are required.

**WeCom**:

| Field | Description |
|-------|-------------|
| Bot ID | WeCom bot ID |
| Secret | WeCom bot secret key |

**DingTalk**:

| Field | Description |
|-------|-------------|
| App Key | DingTalk application App Key |
| App Secret | DingTalk application App Secret |

**Feishu (Lark)**:

| Field | Description |
|-------|-------------|
| App ID | Feishu application App ID |
| App Secret | Feishu application App Secret |
| Encrypt Key | Encryption key (optional, for message encryption) |
| Verification Token | Verification token (optional, for event verification) |

---

## 4. Managing Bots

### 4.1 Bot List

| Column | Description |
|--------|-------------|
| Name | Bot name |
| IM Type | Platform type |
| Enabled | Toggle to enable/disable the bot |
| Connection Status | Current status (Connected / Disconnected / Connecting / Failed) |
| Last Connected | Time of last successful connection |
| Actions | Edit, Delete |

### 4.2 Enable / Disable

Control bot connections via the enable toggle:

- **Enabled**: System automatically connects to the corresponding IM platform
- **Disabled**: System disconnects from the IM platform

### 4.3 Editing Bots

Click **Edit** to modify the bot's name, IM type, bound Agent, and platform configuration.

### 4.4 Deleting Bots

Click **Delete**, confirm to remove the bot. The system automatically disconnects before deletion.

---

## 5. Platform Integration Guide

### 5.1 WeCom Integration

1. Log in to the WeCom admin console
2. Go to "App Management" > "Self-built"
3. Create a new app and obtain the Bot ID and Secret
4. Configure the callback URL to point to DBridge's webhook endpoint
5. Create a bot in DBridge and fill in the Bot ID and Secret

### 5.2 DingTalk Integration

1. Log in to the DingTalk Open Platform
2. Create an enterprise internal application
3. Obtain the App Key and App Secret from "Credentials and Basic Info"
4. Configure the bot message receiving address
5. Create a bot in DBridge and fill in the App Key and App Secret

### 5.3 Feishu Integration

1. Log in to the Feishu Open Platform
2. Create an enterprise self-built application
3. Obtain the App ID and App Secret from "Credentials and Basic Info"
4. Configure the event subscription address
5. (Optional) Configure message encryption and event verification
6. Create a bot in DBridge and fill in the relevant configuration

---

## 6. Agent Binding

Bots can be bound to an AI Agent for intelligent conversation capabilities.

### 6.1 Binding Method

When creating or editing a bot, select an Agent from the "Bind Agent" dropdown.

### 6.2 Conversation Flow

1. User @mentions the bot or sends a message in the IM platform
2. IM platform forwards the message to DBridge
3. DBridge uses the bound Agent to process the message
4. Agent calls tools (e.g., query database, analyze data)
5. Results are returned to the IM platform
6. User sees the reply in IM

### 6.3 Without Agent Binding

If a bot is not bound to an Agent, it can only receive messages but cannot perform intelligent conversations.

---

## 7. Connection Status

| Status | Description |
|--------|-------------|
| Connected | Bot is running normally, can receive and reply to messages |
| Disconnected | Bot is disabled or connection not started |
| Connecting | Attempting to connect to the IM platform |
| Failed | Connection failed; check if configuration is correct |

---

## 8. Notes

1. **Network Requirements**: DBridge server must be able to access IM platform API endpoints
2. **Callback URL**: IM platforms must be able to callback to DBridge's webhook endpoint
3. **Permission Configuration**: Configure appropriate permissions for the application in the IM platform
4. **Message Limits**: Each IM platform has limits on message length and sending frequency
5. **Security**: It is recommended to configure Encrypt Key and Verification Token (Feishu) for enhanced security
