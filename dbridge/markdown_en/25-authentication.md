# Authentication & Subscription

## 1. Overview

DBridge supports two authentication modes: OAuth third-party authentication (User Center) and local account password authentication. It also provides subscription management functionality, displaying current subscription status, entitlement information, and network connectivity status.

**Core Capabilities**:
- OAuth third-party authentication (multiple Providers supported)
- Local account password authentication
- Subscription status viewing
- Entitlement information queries
- Network connectivity monitoring

---

## 2. Login

### 2.1 Login Page

After accessing the system URL, unauthenticated users are automatically redirected to the login page. The login page displays different login methods based on system configuration.

### 2.2 OAuth Login

When OAuth Providers are configured, the login page displays the OAuth tab.

**Steps**:

1. Switch to the **OAuth Login** tab on the login page
2. The page displays available OAuth Provider buttons (e.g., GitHub, Google)
3. Click the corresponding Provider button
4. Browser redirects to the Provider's authorization page
5. After user authorization, automatically redirects back to DBridge and completes login

> **Tip**: If both OAuth and local login are configured, a prompt at the bottom of the page indicates you can switch to local login.

### 2.3 Local Account Login

When local login is allowed, the login page displays the local login tab.

**Steps**:

1. Switch to the **Local Login** tab on the login page
2. Enter username and password
3. Click the **Login** button

> **Note**: Local accounts are managed through dbridge-desktop's security settings.

### 2.4 Login Error Handling

| Error Status | Description |
|-------------|-------------|
| missing_code | OAuth callback missing authorization code |
| device_limit_reached | Device limit reached; manage devices |
| login_failed | Login failed; check credentials |

---

## 3. Authentication Mode Configuration

Authentication mode is configured in the system configuration file `config.yaml`:

```yaml
subscription:
  enabled: true
  auth_mode: "usercenter"  # "password" or "usercenter"
  base_url: "http://localhost:8080"
  app_id: "your_app_id"
  product_id: "your_product_id"
```

### 3.1 auth_mode Description

| Mode | Description |
|------|-------------|
| `usercenter` | Uses User Center OAuth authentication, supports multiple Providers |
| `password` | Uses local account password authentication |

### 3.2 Authentication Flow

**OAuth Mode**:
1. Frontend requests `/api/v1/auth/login-config` to get available Providers
2. Click Provider button, requests `/api/v1/auth/oauth/start?provider=xxx`
3. Backend returns `authorize_url`, frontend redirects
4. After user authorization, callback to `/auth/callback`
5. Frontend carries code to complete backend verification and login

**Local Mode**:
1. Frontend submits username and password to `/api/v1/auth/local-login`
2. Backend validates credentials
3. Returns JWT Token

---

## 4. Subscription Management

### 4.1 Accessing Subscription Page

In the left navigation bar, click **Subscription** to enter the subscription page.

### 4.2 Subscription Status

The page displays detailed subscription information:

| Field | Description |
|-------|-------------|
| Status | Subscription status tag |
| End Date | Subscription expiration time |
| Effective Expiry | Actual effective expiration (including grace period) |
| Grace Period | Grace period information (if applicable) |
| Readonly Mode | Whether in read-only mode |
| Network Status | Online/offline status, last sync time |

### 4.3 Subscription Status Descriptions

| Status | Color | Description |
|--------|-------|-------------|
| Active | Green | Subscription valid, all features working |
| Grace | Orange | Subscription expired, in grace period, limited features |
| Expired | Red | Subscription expired, entered read-only mode |
| Canceled | Gray | Subscription canceled |
| Refunded | Gray | Subscription refunded |

### 4.4 Entitlement Information

The entitlement card displays the benefits included in the current subscription:

- **Source**: Entitlement source
- **Product ID**: Subscribed product identifier
- **Expiration**: Entitlement expiration time
- **Version Series**: Applicable version series

### 4.5 Refreshing Subscription

Click the **Refresh** button to re-fetch the latest subscription data from the server.

### 4.6 Renewal Prompt

When subscription status is expired, grace, or canceled, the page displays renewal information and provides a **Renew** button.

---

## 5. Network Connectivity Status

The system periodically syncs status with the license server.

| Field | Description |
|-------|-------------|
| Online Status | Connection status with the license server |
| Last Sync Time | Time of last successful sync |
| Consecutive Failures | Number of consecutive sync failures |

---

## 6. Device Management

In OAuth mode, the system identifies different device instances through device fingerprints.

### 6.1 Device Limits

When the subscription reaches its device limit, login returns a `device_limit_reached` error.

### 6.2 Managing Devices

Users can view and remove registered devices on the User Center's device management page. After removing old devices, new devices can log in normally.

---

## 7. Session Management

After successful login, the system issues a JWT Token with a default validity of 7200 seconds (2 hours).

```yaml
jwt:
  expires_in: 7200
  secret: change-me-in-production
```

After the Token expires, re-login is required. The frontend automatically detects Token expiration and redirects to the login page.

---

## 8. Security Recommendations

1. **Change JWT Secret in production**: The default `change-me-in-production` is insecure
2. **Enable HTTPS**: Protect login credentials during transmission
3. **Update passwords regularly**: If using local account authentication
4. **Monitor device count**: Clean up unused devices promptly
5. **Configure appropriate Token validity**: Adjust `expires_in` based on security requirements
