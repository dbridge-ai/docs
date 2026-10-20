# License Management

## 1. Overview

DBridge uses a device fingerprint + Ed25519 certificate signing license management mechanism, supporting both online and offline activation methods. The system provides a 14-day trial period; after expiration, it enters read-only mode and requires license activation to continue using.

**Core Capabilities**:
- Device fingerprint identification
- Online activation (one-click authentication)
- Offline activation (CSR certificate exchange)
- 14-day free trial
- License status monitoring

---

## 2. License Status

### 2.1 Viewing License

In the left navigation bar, click **License** to enter the license management page.

### 2.2 Status Information

| Field | Description |
|-------|-------------|
| Current Phase | License status |
| License ID | Unique license identifier |
| Trial Remaining | Remaining trial days |
| Expiry Time | License expiration time |
| Certificate Path | Certificate file storage path |

### 2.3 Phase Descriptions

| Phase | Color | Description |
|-------|-------|-------------|
| Active | Green | License valid, all features working |
| Grace | Orange | License expiring soon, in grace period |
| Readonly | Gold | License expired, system in read-only mode |
| Locked | Red | System locked, unusable |

### 2.4 Device Information

The page displays the current device fingerprint (Machine Hash), used for license binding. The device fingerprint is read-only and cannot be modified.

---

## 3. Trial

### 3.1 Starting Trial

When using DBridge for the first time, the system is in Locked state. Click the **Start Trial (14 days)** button to activate a 14-day free trial.

### 3.2 Trial Benefits

During the trial period, system features are identical to a full license with no feature restrictions.

### 3.3 Trial Expiration

After the trial expires, the system enters Locked state and requires activation of a full license.

---

## 4. Online Activation

Online activation is suitable for scenarios where the DBridge server can access the internet.

### 4.1 Steps

1. Go to the **License** page
2. Click the **Activate Device** button to enter the activation wizard
3. Select **Online Authentication**
4. Click the **Online Authentication** button
5. The system automatically communicates with the license server to complete activation
6. After successful activation, automatically redirects to the home page

### 4.2 Prerequisites

- DBridge server can access the license server
- Device fingerprint is not occupied by another license
- Valid license quota available

---

## 5. Offline Activation

Offline activation is suitable for secure environments where the DBridge server cannot access the internet.

### 5.1 Steps

1. Go to the **License** page
2. Click the **Activate Device** button to enter the activation wizard
3. Select **Offline Authentication**
4. Click the **Generate Request** button
5. The system generates a CSR (Certificate Signing Request), displayed in a text box
6. Click **Copy Request** to copy the CSR to clipboard
7. Send the CSR to the license administrator (via email, USB drive, etc.)
8. The license administrator uses the CSR to generate a signed certificate
9. Return the signed certificate to the DBridge server
10. Paste the signed certificate in the second text box
11. Click **Submit Response**
12. After successful activation, automatically redirects to the home page

### 5.2 CSR Format

The CSR is text containing device fingerprint and public key information, in the following format:

```
-----BEGIN CERTIFICATE REQUEST-----
[Base64 encoded request data]
-----END CERTIFICATE REQUEST-----
```

---

## 6. Certificate Status

### 6.1 Viewing Certificate

At the top of the activation wizard page, the certificate status is displayed:

- **Valid** (green): Certificate is valid, showing expiration date
- **Invalid** (yellow): Certificate is invalid or does not exist

### 6.2 Refreshing Certificate

When the license is in Grace or Readonly state, click the **Refresh Certificate** button to attempt updating the certificate from the license server.

---

## 7. License Lifecycle

```
First Installation
    │
    ▼
  Locked ──── Start Trial ────► Trial (14 days)
    │                              │
    │                              ▼
    │                       Trial Expired ────► Locked
    │                              │
    ▼                              ▼
  Activate Device ◄──────── Get License
    │
    ▼
  Active ──── Expiry ────► Grace ────► Readonly ────► Locked
    │
    ▼
  Refresh Certificate (Renewal)
```

---

## 8. FAQ

### 8.1 License Invalid After Server Change

Licenses are bound to device fingerprints. After changing servers, the device fingerprint changes and re-activation is required. Contact the license administrator to migrate the license.

### 8.2 How to Get the Signed Certificate for Offline Activation

Send the CSR to the DBridge license administrator. The administrator uses the license management tool to generate a signed certificate and returns it.

### 8.3 Can the Trial Period Be Extended?

The trial period is 14 days and cannot be extended. To continue using, activate a full license.

### 8.4 Will Data Be Lost After License Expiration?

No. After license expiration, the system enters read-only mode and all existing data is fully preserved. Normal use resumes after activating a license.
