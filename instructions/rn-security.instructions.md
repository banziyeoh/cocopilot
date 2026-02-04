````markdown
# GitHub Copilot Instructions for React Native Security

## Core Security Principles

When working with React Native code, prioritize security at every level. Always assume user input is malicious, apply defense-in-depth strategies, and follow the principle of least privilege.

## Secure Data Storage

### DO: Use React Native Keychain for Sensitive Data

```javascript
import * as Keychain from "react-native-keychain";

// Store credentials securely
await Keychain.setGenericPassword("username", "password", {
  service: "com.myapp.auth",
  accessible: Keychain.ACCESSIBLE.WHEN_UNLOCKED_THIS_DEVICE_ONLY,
});
```
````

### DON'T: Store Sensitive Data in AsyncStorage

```javascript
// NEVER do this - AsyncStorage is not encrypted
await AsyncStorage.setItem("password", userPassword);
await AsyncStorage.setItem("apiKey", apiKey);
await AsyncStorage.setItem("creditCard", cardNumber);
```

### DO: Encrypt Sensitive Data Before Storage

```javascript
import CryptoJS from "crypto-js";
import EncryptedStorage from "react-native-encrypted-storage";

const encryptData = (data, key) => {
  return CryptoJS.AES.encrypt(JSON.stringify(data), key).toString();
};

await EncryptedStorage.setItem(
  "user_session",
  encryptData(sessionData, encryptionKey),
);
```

### DON'T: Store Tokens in Plain Text

```javascript
// ANTI-PATTERN: Storing JWT in plain AsyncStorage
const token = await loginAPI();
await AsyncStorage.setItem("jwt_token", token);
```

## API Security and Network Communication

### DO: Implement Certificate Pinning

```javascript
import axios from "axios";
import { certificatePinning } from "react-native-ssl-pinning";

const secureAPI = axios.create({
  baseURL: "https://api.example.com",
  httpsAgent: certificatePinning({
    certificates: [
      {
        host: "api.example.com",
        hash: "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=",
      },
    ],
  }),
});
```

### DON'T: Disable SSL Certificate Validation

```javascript
// NEVER do this in production
const api = axios.create({
  baseURL: "https://api.example.com",
  httpsAgent: new https.Agent({
    rejectUnauthorized: false, // SECURITY RISK
  }),
});
```

### DO: Sanitize and Validate API Responses

```javascript
import { z } from "zod";

const UserSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  role: z.enum(["user", "admin"]),
  createdAt: z.string().datetime(),
});

const fetchUser = async (userId) => {
  const response = await api.get(`/users/${userId}`);
  // Validate response structure
  const validatedUser = UserSchema.parse(response.data);
  return validatedUser;
};
```

### DON'T: Trust API Responses Blindly

```javascript
// ANTI-PATTERN: No validation
const fetchUser = async (userId) => {
  const response = await api.get(`/users/${userId}`);
  // Directly using unvalidated data
  setUserData(response.data);
};
```

### DO: Implement Request Rate Limiting

```javascript
import { RateLimiter } from "limiter";

const limiter = new RateLimiter({ tokensPerInterval: 10, interval: "minute" });

const makeAPIRequest = async (endpoint) => {
  await limiter.removeTokens(1);
  return api.get(endpoint);
};
```

### DO: Use Secure Headers

```javascript
const api = axios.create({
  baseURL: "https://api.example.com",
  headers: {
    "X-Content-Type-Options": "nosniff",
    "X-Frame-Options": "DENY",
    "Strict-Transport-Security": "max-age=31536000; includeSubDomains",
  },
});
```

## Authentication and Authorization

### DO: Implement Secure Token Management

```javascript
import * as Keychain from "react-native-keychain";
import jwtDecode from "jwt-decode";

class SecureAuthManager {
  static async storeTokens(accessToken, refreshToken) {
    await Keychain.setGenericPassword(
      "tokens",
      JSON.stringify({ accessToken, refreshToken }),
      {
        service: "com.myapp.auth",
        accessible: Keychain.ACCESSIBLE.WHEN_UNLOCKED_THIS_DEVICE_ONLY,
      },
    );
  }

  static async getAccessToken() {
    const credentials = await Keychain.getGenericPassword({
      service: "com.myapp.auth",
    });
    if (!credentials) return null;

    const { accessToken } = JSON.parse(credentials.password);
    const decoded = jwtDecode(accessToken);

    // Check if token is expired
    if (decoded.exp * 1000 < Date.now()) {
      return this.refreshAccessToken();
    }

    return accessToken;
  }

  static async refreshAccessToken() {
    // Implement token refresh logic
  }

  static async clearTokens() {
    await Keychain.resetGenericPassword({ service: "com.myapp.auth" });
  }
}
```

### DON'T: Store Tokens in Redux or State

```javascript
// ANTI-PATTERN: Token in global state
const authSlice = createSlice({
  name: "auth",
  initialState: {
    token: null, // SECURITY RISK - accessible via React DevTools
  },
});
```

### DO: Implement Biometric Authentication

```javascript
import ReactNativeBiometrics from "react-native-biometrics";

const authenticateWithBiometrics = async () => {
  const rnBiometrics = new ReactNativeBiometrics();

  const { available, biometryType } = await rnBiometrics.isSensorAvailable();

  if (!available) {
    throw new Error("Biometrics not available");
  }

  const { success } = await rnBiometrics.simplePrompt({
    promptMessage: "Confirm your identity",
    cancelButtonText: "Cancel",
  });

  if (!success) {
    throw new Error("Biometric authentication failed");
  }

  return true;
};
```

### DO: Implement Session Timeout

```javascript
import { AppState } from "react-native";

class SessionManager {
  static timeout = 15 * 60 * 1000; // 15 minutes
  static lastActivity = Date.now();

  static initializeSessionTimeout() {
    AppState.addEventListener("change", this.handleAppStateChange);
    this.setupActivityListeners();
  }

  static handleAppStateChange = (nextAppState) => {
    if (nextAppState === "background") {
      this.startBackgroundTimer();
    } else if (nextAppState === "active") {
      this.checkSessionValidity();
    }
  };

  static checkSessionValidity() {
    if (Date.now() - this.lastActivity > this.timeout) {
      this.logout();
    }
  }

  static updateActivity() {
    this.lastActivity = Date.now();
  }
}
```

## Input Validation and Sanitization

### DO: Validate All User Inputs

```javascript
import { z } from "zod";

const LoginSchema = z.object({
  email: z.string().email().max(255),
  password: z.string().min(8).max(128),
});

const handleLogin = (email, password) => {
  try {
    const validated = LoginSchema.parse({ email, password });
    // Proceed with validated data
  } catch (error) {
    // Handle validation errors
    console.error("Validation failed:", error.errors);
  }
};
```

### DON'T: Use Unvalidated Input Directly

```javascript
// ANTI-PATTERN: No input validation
const handleLogin = (email, password) => {
  // Directly using user input
  loginAPI(email, password);
};
```

### DO: Sanitize Data for Display

```javascript
import DOMPurify from "isomorphic-dompurify";

const SafeHTMLView = ({ html }) => {
  const sanitized = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ["b", "i", "em", "strong", "p"],
    ALLOWED_ATTR: [],
  });

  return <WebView source={{ html: sanitized }} />;
};
```

### DON'T: Render Unescaped User Content

```javascript
// ANTI-PATTERN: XSS vulnerability
const UserComment = ({ comment }) => {
  return <WebView source={{ html: comment.text }} />;
};
```

### DO: Validate File Uploads

```javascript
const validateImageUpload = (file) => {
  const allowedTypes = ["image/jpeg", "image/png", "image/webp"];
  const maxSize = 5 * 1024 * 1024; // 5MB

  if (!allowedTypes.includes(file.type)) {
    throw new Error("Invalid file type");
  }

  if (file.size > maxSize) {
    throw new Error("File too large");
  }

  return true;
};
```

## Deep Linking and URI Handling

### DO: Validate Deep Link Parameters

```javascript
import { Linking } from "react-native";
import { z } from "zod";

const DeepLinkSchema = z.object({
  userId: z.string().uuid(),
  action: z.enum(["view", "edit"]),
});

const handleDeepLink = (url) => {
  const params = parseURLParams(url);

  try {
    const validated = DeepLinkSchema.parse(params);
    navigateToScreen(validated);
  } catch (error) {
    console.error("Invalid deep link:", error);
    navigateToHome();
  }
};

Linking.addEventListener("url", ({ url }) => handleDeepLink(url));
```

### DON'T: Execute Deep Links Without Validation

```javascript
// ANTI-PATTERN: Open redirect vulnerability
Linking.addEventListener("url", ({ url }) => {
  const { redirect } = parseURLParams(url);
  // Directly navigating to user-controlled URL
  Linking.openURL(redirect);
});
```

### DO: Implement Allowlist for External URLs

```javascript
const ALLOWED_DOMAINS = ["example.com", "trusted-site.com"];

const openExternalURL = (url) => {
  try {
    const urlObj = new URL(url);

    if (!ALLOWED_DOMAINS.includes(urlObj.hostname)) {
      throw new Error("URL not in allowlist");
    }

    Linking.openURL(url);
  } catch (error) {
    Alert.alert("Error", "Cannot open this URL");
  }
};
```

## WebView Security

### DO: Implement Secure WebView Configuration

```javascript
import { WebView } from "react-native-webview";

const SecureWebView = ({ uri }) => {
  return (
    <WebView
      source={{ uri }}
      javaScriptEnabled={false} // Disable unless absolutely necessary
      domStorageEnabled={false}
      allowsInlineMediaPlayback={false}
      allowFileAccess={false}
      allowUniversalAccessFromFileURLs={false}
      mixedContentMode="never"
      onShouldStartLoadWithRequest={(request) => {
        // Validate navigation requests
        return request.url.startsWith("https://trusted-domain.com");
      }}
    />
  );
};
```

### DON'T: Enable Unnecessary WebView Features

```javascript
// ANTI-PATTERN: Overly permissive WebView
<WebView
  source={{ uri }}
  javaScriptEnabled={true}
  domStorageEnabled={true}
  allowFileAccess={true}
  allowUniversalAccessFromFileURLs={true} // SECURITY RISK
  mixedContentMode="always" // SECURITY RISK
/>
```

### DO: Sanitize JavaScript Messages

```javascript
const SecureWebView = () => {
  const handleMessage = (event) => {
    try {
      const data = JSON.parse(event.nativeEvent.data);

      // Validate message structure
      if (!data.type || !ALLOWED_ACTIONS.includes(data.type)) {
        throw new Error("Invalid message type");
      }

      // Process validated message
      processMessage(data);
    } catch (error) {
      console.error("Invalid message from WebView:", error);
    }
  };

  return <WebView onMessage={handleMessage} />;
};
```

## Environment Variables and Secrets

### DO: Use Environment-Specific Configuration

```javascript
import Config from "react-native-config";

// .env file (never commit to version control)
// API_URL=https://api.example.com
// API_KEY=secret_key_here

const api = axios.create({
  baseURL: Config.API_URL,
  headers: {
    "X-API-Key": Config.API_KEY,
  },
});
```

### DON'T: Hardcode Secrets in Code

```javascript
// ANTI-PATTERN: Hardcoded secrets
const API_KEY = "sk_live_51ABC123xyz";
const DATABASE_PASSWORD = "mySecretPassword123";
const ENCRYPTION_KEY = "1234567890abcdef";
```

### DO: Use Different Keys for Debug and Release

```javascript
// config.js
export const getAPIConfig = () => {
  if (__DEV__) {
    return {
      apiUrl: Config.DEV_API_URL,
      apiKey: Config.DEV_API_KEY,
    };
  }

  return {
    apiUrl: Config.PROD_API_URL,
    apiKey: Config.PROD_API_KEY,
  };
};
```

### DON'T: Use Production Credentials in Development

```javascript
// ANTI-PATTERN: Same credentials for all environments
const API_KEY = "sk_live_production_key";
```

## Code Obfuscation and Protection

### DO: Enable Code Obfuscation for Release Builds

```javascript
// android/app/build.gradle
android {
  buildTypes {
    release {
      minifyEnabled true
      shrinkResources true
      proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
  }
}
```

### DO: Implement Root/Jailbreak Detection

```javascript
import JailMonkey from "jail-monkey";

const checkDeviceSecurity = () => {
  if (JailMonkey.isJailBroken()) {
    Alert.alert(
      "Security Warning",
      "This device appears to be jailbroken/rooted. The app cannot run on compromised devices.",
      [{ text: "Exit", onPress: () => BackHandler.exitApp() }],
    );
    return false;
  }

  if (JailMonkey.isDebuggedMode()) {
    // Handle debug mode detection
  }

  return true;
};
```

### DO: Implement Tampering Detection

```javascript
import { NativeModules } from "react-native";

const checkAppIntegrity = async () => {
  try {
    const { isOriginal } = await NativeModules.IntegrityChecker.check();

    if (!isOriginal) {
      Alert.alert("Security Alert", "App integrity check failed");
      return false;
    }

    return true;
  } catch (error) {
    console.error("Integrity check failed:", error);
    return false;
  }
};
```

## Logging and Error Handling

### DO: Implement Secure Logging

```javascript
class SecureLogger {
  static log(message, data = {}) {
    if (__DEV__) {
      console.log(message, this.sanitizeData(data));
    } else {
      // Send to secure logging service
      this.sendToRemoteLogger({
        level: "info",
        message,
        timestamp: new Date().toISOString(),
      });
    }
  }

  static sanitizeData(data) {
    const sensitiveKeys = [
      "password",
      "token",
      "apiKey",
      "secret",
      "ssn",
      "creditCard",
    ];
    const sanitized = { ...data };

    Object.keys(sanitized).forEach((key) => {
      if (sensitiveKeys.some((k) => key.toLowerCase().includes(k))) {
        sanitized[key] = "***REDACTED***";
      }
    });

    return sanitized;
  }
}
```

### DON'T: Log Sensitive Information

```javascript
// ANTI-PATTERN: Logging sensitive data
console.log("User logged in:", {
  email: user.email,
  password: user.password, // NEVER log passwords
  token: authToken, // NEVER log tokens
  ssn: user.ssn, // NEVER log PII
});
```

### DO: Implement Global Error Boundary

```javascript
import { ErrorBoundary } from "react-error-boundary";

const ErrorFallback = ({ error, resetErrorBoundary }) => {
  // Don't expose error details to users
  return (
    <View>
      <Text>Something went wrong. Please try again.</Text>
      <Button onPress={resetErrorBoundary} title="Retry" />
    </View>
  );
};

const App = () => (
  <ErrorBoundary
    FallbackComponent={ErrorFallback}
    onError={(error, errorInfo) => {
      // Log error securely without exposing to user
      SecureLogger.error("App error", { error: error.message });
    }}
  >
    <MainApp />
  </ErrorBoundary>
);
```

### DON'T: Display Stack Traces to Users

```javascript
// ANTI-PATTERN: Exposing error details
const MyComponent = () => {
  try {
    // code
  } catch (error) {
    Alert.alert("Error", error.stack); // SECURITY RISK
  }
};
```

## Dependency Security

### DO: Regularly Audit Dependencies

```bash
# Run these commands regularly
npm audit
npm audit fix

# Use Snyk or similar tools
npx snyk test
```

### DO: Pin Dependency Versions

```json
{
  "dependencies": {
    "react-native": "0.73.2",
    "axios": "1.6.5",
    "react-native-keychain": "8.1.2"
  }
}
```

### DON'T: Use Wildcard Versions

```json
// ANTI-PATTERN: Unpredictable dependencies
{
  "dependencies": {
    "react-native": "*",
    "axios": "^1.0.0",
    "some-package": "latest"
  }
}
```

### DO: Verify Package Integrity

```bash
# Use lock files
npm ci # instead of npm install

# Verify package checksums
npm audit signatures
```

## Cryptography Best Practices

### DO: Use Strong Encryption Algorithms

```javascript
import CryptoJS from "crypto-js";

const encryptData = (data, masterKey) => {
  // Use AES-256
  const encrypted = CryptoJS.AES.encrypt(JSON.stringify(data), masterKey, {
    mode: CryptoJS.mode.GCM,
    padding: CryptoJS.pad.Pkcs7,
  });

  return encrypted.toString();
};

const decryptData = (encryptedData, masterKey) => {
  const decrypted = CryptoJS.AES.decrypt(encryptedData, masterKey);
  return JSON.parse(decrypted.toString(CryptoJS.enc.Utf8));
};
```

### DON'T: Use Weak Encryption or Custom Crypto

```javascript
// ANTI-PATTERN: Weak encryption
const encrypt = (data, key) => {
  // Custom encryption algorithm - NEVER do this
  return data
    .split("")
    .map((c, i) =>
      String.fromCharCode(c.charCodeAt(0) ^ key.charCodeAt(i % key.length)),
    )
    .join("");
};
```

### DO: Generate Secure Random Values

```javascript
import { randomBytes } from "react-native-randombytes";

const generateSecureToken = async () => {
  const bytes = await randomBytes(32);
  return bytes.toString("hex");
};

const generateNonce = async () => {
  const bytes = await randomBytes(16);
  return bytes.toString("base64");
};
```

### DON'T: Use Math.random() for Security

```javascript
// ANTI-PATTERN: Predictable random values
const generateToken = () => {
  return Math.random().toString(36).substring(2); // NOT SECURE
};
```

## Permission Handling

### DO: Request Minimum Necessary Permissions

```javascript
import { PermissionsAndroid, Platform } from "react-native";

const requestCameraPermission = async () => {
  if (Platform.OS === "android") {
    const granted = await PermissionsAndroid.request(
      PermissionsAndroid.PERMISSIONS.CAMERA,
      {
        title: "Camera Permission",
        message: "App needs access to your camera to take photos",
        buttonNeutral: "Ask Me Later",
        buttonNegative: "Cancel",
        buttonPositive: "OK",
      },
    );

    return granted === PermissionsAndroid.RESULTS.GRANTED;
  }

  return true;
};
```

### DON'T: Request All Permissions Upfront

```javascript
// ANTI-PATTERN: Requesting unnecessary permissions
const requestAllPermissions = async () => {
  await PermissionsAndroid.requestMultiple([
    PermissionsAndroid.PERMISSIONS.CAMERA,
    PermissionsAndroid.PERMISSIONS.READ_CONTACTS,
    PermissionsAndroid.PERMISSIONS.ACCESS_FINE_LOCATION,
    PermissionsAndroid.PERMISSIONS.READ_SMS, // Why?
    PermissionsAndroid.PERMISSIONS.RECORD_AUDIO, // Unnecessary
  ]);
};
```

### DO: Explain Permission Usage

```javascript
const requestLocationPermission = async () => {
  Alert.alert(
    "Location Access",
    "We need your location to show nearby stores. Your location is never shared with third parties.",
    [
      {
        text: "Cancel",
        style: "cancel",
      },
      {
        text: "Allow",
        onPress: async () => {
          const granted = await PermissionsAndroid.request(
            PermissionsAndroid.PERMISSIONS.ACCESS_FINE_LOCATION,
          );
          return granted === PermissionsAndroid.RESULTS.GRANTED;
        },
      },
    ],
  );
};
```

## Secure Communication Patterns

### DO: Implement Request/Response Validation

```javascript
import { z } from "zod";

const APIResponseSchema = z.object({
  success: z.boolean(),
  data: z.unknown(),
  timestamp: z.number(),
  signature: z.string(),
});

const verifyResponse = (response, expectedSignature) => {
  const validated = APIResponseSchema.parse(response);

  // Verify response signature
  const calculatedSignature = calculateHMAC(validated.data, SECRET_KEY);

  if (calculatedSignature !== validated.signature) {
    throw new Error("Response signature mismatch");
  }

  // Verify timestamp (prevent replay attacks)
  const now = Date.now();
  if (Math.abs(now - validated.timestamp) > 300000) {
    // 5 minutes
    throw new Error("Response timestamp too old");
  }

  return validated.data;
};
```

### DO: Implement Request Signing

```javascript
import CryptoJS from "crypto-js";

const signRequest = (payload, secretKey) => {
  const timestamp = Date.now();
  const nonce = generateNonce();

  const message = `${timestamp}.${nonce}.${JSON.stringify(payload)}`;
  const signature = CryptoJS.HmacSHA256(message, secretKey).toString();

  return {
    ...payload,
    timestamp,
    nonce,
    signature,
  };
};

const makeSecureRequest = async (endpoint, data) => {
  const signedData = signRequest(data, SECRET_KEY);
  return api.post(endpoint, signedData);
};
```

## Memory Management and Data Cleanup

### DO: Clear Sensitive Data from Memory

```javascript
class SecureComponent extends React.Component {
  state = {
    sensitiveData: null,
  };

  componentWillUnmount() {
    // Clear sensitive data
    this.setState({ sensitiveData: null });

    // Clear from closure
    this.sensitiveData = null;
  }

  handleLogout = async () => {
    // Clear all sensitive data
    await Keychain.resetGenericPassword();
    this.setState({ sensitiveData: null });

    // Clear navigation stack
    navigation.reset({
      index: 0,
      routes: [{ name: "Login" }],
    });
  };
}
```

### DO: Implement Screenshot Protection

```javascript
import {
  preventScreenCapture,
  allowScreenCapture,
} from "react-native-screen-capture";

const SecureScreen = () => {
  useEffect(() => {
    preventScreenCapture();

    return () => {
      allowScreenCapture();
    };
  }, []);

  return <View>{/* Sensitive content */}</View>;
};
```

## Platform-Specific Security

### DO: Configure Android Security Settings

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<application
  android:allowBackup="false"
  android:usesCleartextTraffic="false"
  android:networkSecurityConfig="@xml/network_security_config">
</application>
```

```xml
<!-- android/app/src/main/res/xml/network_security_config.xml -->
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
  <base-config cleartextTrafficPermitted="false">
    <trust-anchors>
      <certificates src="system" />
    </trust-anchors>
  </base-config>
  <domain-config cleartextTrafficPermitted="false">
    <domain includeSubdomains="true">api.example.com</domain>
    <pin-set>
      <pin digest="SHA-256">XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX=</pin>
    </pin-set>
  </domain-config>
</network-security-config>
```

### DO: Configure iOS Security Settings

```xml
<!-- ios/YourApp/Info.plist -->
<key>NSAppTransportSecurity</key>
<dict>
  <key>NSAllowsArbitraryLoads</key>
  <false/>
  <key>NSExceptionDomains</key>
  <dict>
    <key>api.example.com</key>
    <dict>
      <key>NSExceptionAllowsInsecureHTTPLoads</key>
      <false/>
      <key>NSIncludesSubdomains</key>
      <true/>
      <key>NSExceptionRequiresForwardSecrecy</key>
      <true/>
      <key>NSExceptionMinimumTLSVersion</key>
      <string>TLSv1.2</string>
    </dict>
  </dict>
</dict>
```

## Testing Security

### DO: Write Security-Focused Tests

```javascript
import { render, fireEvent } from "@testing-library/react-native";

describe("LoginScreen Security", () => {
  it("should not expose password in DOM", () => {
    const { getByTestId } = render(<LoginScreen />);
    const passwordInput = getByTestId("password-input");

    expect(passwordInput.props.secureTextEntry).toBe(true);
  });

  it("should clear password on unmount", () => {
    const { unmount, getByTestId } = render(<LoginScreen />);
    const passwordInput = getByTestId("password-input");

    fireEvent.changeText(passwordInput, "secretPassword");
    unmount();

    // Verify password is cleared
  });

  it("should validate input before submission", async () => {
    const { getByTestId } = render(<LoginScreen />);
    const submitButton = getByTestId("submit-button");

    fireEvent.press(submitButton);

    // Should show validation errors without making API call
  });
});
```

## Security Checklist

Before deploying to production, verify:

- [ ] All sensitive data is encrypted at rest
- [ ] Certificate pinning is implemented
- [ ] No hardcoded secrets or API keys
- [ ] All user inputs are validated and sanitized
- [ ] Authentication tokens are stored securely
- [ ] Biometric authentication is implemented where appropriate
- [ ] Deep links are validated
- [ ] WebViews are configured securely
- [ ] Root/jailbreak detection is active
- [ ] Code obfuscation is enabled
- [ ] Logging does not expose sensitive data
- [ ] Dependencies are audited and up to date
- [ ] Strong encryption algorithms are used
- [ ] Permissions are minimally scoped
- [ ] Screenshot protection is enabled for sensitive screens
- [ ] Network security configuration is properly set
- [ ] Session timeout is implemented
- [ ] Error messages don't leak information

## Additional Resources

- OWASP Mobile Security Testing Guide
- React Native Security Best Practices
- iOS Security Guide
- Android Security Best Practices

```

```
