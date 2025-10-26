# @codixverse/totp

Universal TOTP library for React Native, websites, Chrome/Firefox extensions, and Node.js.

## Features

- Works everywhere: React Native, Expo Go, websites, Node.js
- No native modules needed (pure JavaScript)
- TOTP and HOTP support
- SHA1, SHA256, SHA512
- Rate limiting and throttling
- TypeScript support
- Encrypted secret storage
- Memory safe cleanup
- QR code URI generation
- Progress tracking

## Installation

```bash
npm install @codixverse/totp
```

## Usage

### Basic Usage

```typescript
import { generateTOTP, verifyTOTP, generateSecret } from '@codixverse/totp';

const secret = generateSecret();
const code = generateTOTP(secret);
console.log('Your TOTP code:', code); // "123456"

const isValid = verifyTOTP(secret, code);
console.log('Code is valid:', isValid); // true
```

### Advanced

```typescript
import { 
  generateTOTPAdvanced, 
  verifyTOTPAdvanced,
  generateSecureSecret,
  getRemainingTime,
  getProgress 
} from '@codixverse/totp';

const secret = generateSecureSecret();

const result = generateTOTPAdvanced(secret);
console.log('Code:', result.token);
console.log('Time remaining:', result.timeRemaining, 'seconds');

const verifyResult = verifyTOTPAdvanced(secret, userInput, undefined, {}, 'user123');
if (verifyResult.isValid) {
  console.log('Success!');
} else {
  console.log('Invalid code. Attempts remaining:', verifyResult.attemptsRemaining);
}
```

## Configuration

```typescript
const options = {
  timeStep: 30,      // seconds (default: 30)
  digits: 6,         // default: 6
  algorithm: 'sha1', // 'sha1' | 'sha256' | 'sha512'
  window: 1,         // verification tolerance
  maxAttempts: 5,    // before lockout
  lockoutDuration: 300 // seconds
};

const code = generateTOTP(secret, undefined, options);
const isValid = verifyTOTP(secret, userInput, undefined, options, 'user123');

// Check time sync
const result = verifyTOTPAdvanced(secret, userInput);
if (result.isValid && result.timeDelta && Math.abs(result.timeDelta) > 60) {
  console.warn('Clock out of sync by', result.timeDelta, 'seconds');
}
```

## React Native Example

```typescript
import React, { useState, useEffect } from 'react';
import { View, Text, TextInput, Button, StyleSheet } from 'react-native';
import { 
  generateTOTPAdvanced, 
  verifyTOTPAdvanced, 
  generateSecret 
} from '@codixverse/totp';

export default function TOTPAuthenticator() {
  const [secret, setSecret] = useState('');
  const [token, setToken] = useState('');
  const [timeRemaining, setTimeRemaining] = useState(0);
  const [progress, setProgress] = useState(0);
  const [userInput, setUserInput] = useState('');

  useEffect(() => {
    setSecret(generateSecret());
  }, []);

  useEffect(() => {
    if (!secret) return;
    
    const interval = setInterval(() => {
      const result = generateTOTPAdvanced(secret);
      setToken(result.token);
      setTimeRemaining(result.timeRemaining);
    }, 1000);

    return () => clearInterval(interval);
  }, [secret]);

  const handleVerify = () => {
    const result = verifyTOTPAdvanced(secret, userInput, undefined, {}, 'user123');
    
    if (result.isValid) {
      alert('Code verified!');
    } else {
      alert(`Invalid code. Attempts remaining: ${result.attemptsRemaining}`);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>TOTP Authenticator</Text>
      
      <View style={styles.tokenContainer}>
        <Text style={styles.token}>{token}</Text>
        <Text style={styles.timer}>{timeRemaining}s</Text>
      </View>
      
      <TextInput
        style={styles.input}
        placeholder="Enter code"
        value={userInput}
        onChangeText={setUserInput}
        keyboardType="numeric"
        maxLength={6}
      />
      
      <Button title="Verify" onPress={handleVerify} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 20, justifyContent: 'center' },
  title: { fontSize: 24, fontWeight: 'bold', marginBottom: 30 },
  tokenContainer: { padding: 20, alignItems: 'center', marginBottom: 20 },
  token: { fontSize: 48, fontWeight: 'bold' },
  timer: { marginTop: 10 },
  input: { padding: 15, fontSize: 24, textAlign: 'center', marginBottom: 20 }
});
```

## Website Example

Works in any website or React app:

```typescript
import { generateTOTP, verifyTOTP } from '@codixverse/totp';

function LoginForm() {
  const [code, setCode] = useState('');
  const secret = 'JBSWY3DPEHPK3PXP'; // from database

  const handleSubmit = () => {
    const isValid = verifyTOTP(secret, code);
    if (isValid) {
      // login success
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input 
        value={code} 
        onChange={e => setCode(e.target.value)} 
        placeholder="Enter 6-digit code"
      />
      <button type="submit">Verify</button>
    </form>
  );
}
```

## Node.js Example

```typescript
import { generateTOTP, verifyTOTP } from '@codixverse/totp';

const secret = 'JBSWY3DPEHPK3PXP';
const code = generateTOTP(secret);
console.log('TOTP code:', code);

// Verify on server
const isValid = verifyTOTP(secret, userInput);
if (isValid) {
  console.log('Code is valid');
}
```

## Chrome/Firefox Extension Example

```typescript
// background.js or content script
import { generateTOTP, verifyTOTP, generateSecret } from '@codixverse/totp';

// Generate secret
const secret = generateSecret();

// Generate code
const code = generateTOTP(secret);

// Verify
chrome.storage.local.set({ secret });
const isValid = verifyTOTP(secret, userInput);
```

## HOTP

```typescript
import { generateHOTP, verifyHOTP } from '@codixverse/totp';

let counter = 0;
const hotpCode = generateHOTP(secret, counter);

const isValid = verifyHOTP(secret, userInput, counter);
```

## QR Codes

```typescript
import { generateTOTPURI, parseTOTPURI } from '@codixverse/totp';

const uri = generateTOTPURI(secret, 'user@example.com', 'MyApp');
// otpauth://totp/MyApp:user@example.com?secret=...

const parsed = parseTOTPURI(uri);
```

## Security Features

### Encrypted Secrets

```typescript
import { encryptSecret, decryptSecret } from '@codixverse/totp';

// Encrypt
const encrypted = encryptSecret(secret, 'password');
await AsyncStorage.setItem('secret', JSON.stringify(encrypted));

// Decrypt
const data = JSON.parse(await AsyncStorage.getItem('secret'));
const decrypted = decryptSecret(data, 'password');
```

### Memory Cleanup

```typescript
import { zeroMemory } from '@codixverse/totp';

const buffer = new Uint8Array(secret);
// ... use buffer
zeroMemory(buffer); // cleanup
```

### Rate Limiting

```typescript
const isValid = verifyTOTP(secret, userInput, undefined, {
  maxAttempts: 5,
  lockoutDuration: 300
}, 'user123');

// Check attempts
import { getAttemptsInfo } from '@codixverse/totp';
const attempts = getAttemptsInfo('user123');
```

## Progress Tracking

```typescript
import { getRemainingTime, getProgress } from '@codixverse/totp';

const remaining = getRemainingTime(); // seconds
const progress = getProgress(); // percentage
```

## Advanced Features

### Global Config

```typescript
import { setGlobalTOTPConfig } from '@codixverse/totp';

setGlobalTOTPConfig({
  timeStep: 45,
  digits: 8
});
```

### Hooks

```typescript
import { onVerifySuccess, onVerifyFailure } from '@codixverse/totp';

const unsub = onVerifySuccess((info) => {
  console.log('Success', info);
});

onVerifyFailure((error) => {
  console.log('Failed', error);
});

// Cleanup
unsub();
```

### Error Handling

```typescript
import { TOTPError } from '@codixverse/totp';

try {
  generateTOTP(invalid);
} catch (error) {
  if (error instanceof TOTPError) {
    console.log(error.code); // 'INVALID_SECRET'
  }
}
```

## Base32

```typescript
import { encodeBase32, decodeBase32 } from '@codixverse/totp';

const encoded = encodeBase32(bytes);
const decoded = decodeBase32(encoded);
```

## API Reference

**Core Functions:**
- `generateTOTP(secret, time?, options?)` - Generate code
- `verifyTOTP(secret, token, time?, options?, identifier?)` - Verify code
- `generateHOTP(secret, counter, options?)` - Generate HOTP
- `verifyHOTP(secret, token, counter, options?)` - Verify HOTP

**Security:**
- `encryptSecret(rawSecret, password)` - Encrypt with password
- `decryptSecret(encryptedSecret, password)` - Decrypt
- `zeroMemory(buffer)` - Cleanup memory
- `checkThrottle(identifier, config)` - Rate limiting

**Utilities:**
- `getRemainingTime(options?)` - Time left
- `getProgress(options?)` - Progress %
- `generateTOTPURI(secret, account, issuer)` - QR URI
- `parseTOTPURI(uri)` - Parse URI

## Compatibility

Works in any JavaScript environment:

- **React Native** 0.60+ (Expo included)
- **Websites** (Chrome, Firefox, Safari, Edge)
- **Chrome Extensions** (Manifest V2 & V3)
- **Firefox Add-ons** (WebExtensions API)
- **Node.js** 14+
- **Web Workers**
- **Service Workers**

Pure JavaScript - no native modules needed. Auto-detects available crypto APIs (WebCrypto, Expo, Node.js, Extensions).

## License

MIT

---

[GitHub](https://github.com/codixverse/totp) | [Issues](https://github.com/codixverse/totp/issues) | [Codixverse](https://codixverse.xyz)
