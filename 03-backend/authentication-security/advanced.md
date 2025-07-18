
# 🔐 Advanced Authentication & Security

## Table of Contents
- [Multi-Factor Authentication](#multi-factor-authentication)
- [Single Sign-On (SSO)](#single-sign-on-sso)
- [Zero Trust Architecture](#zero-trust-architecture)
- [Advanced Threat Protection](#advanced-threat-protection)
- [Biometric Authentication](#biometric-authentication)
- [Blockchain-Based Authentication](#blockchain-based-authentication)
- [Security Monitoring](#security-monitoring)
- [Compliance & Auditing](#compliance--auditing)
- [Advanced Encryption](#advanced-encryption)

---

## 🔐 Multi-Factor Authentication

### TOTP (Time-based One-Time Password) Implementation

```javascript
const speakeasy = require('speakeasy');
const QRCode = require('qrcode');

// Generate MFA secret for user
const generateMFASecret = async (user) => {
    const secret = speakeasy.generateSecret({
        name: `${user.email} (YourApp)`,
        issuer: 'YourApp',
        length: 32
    });
    
    // Store secret in database (encrypted)
    await User.findByIdAndUpdate(user.id, {
        mfaSecret: encrypt(secret.base32),
        mfaEnabled: false
    });
    
    // Generate QR code for authenticator app
    const qrCodeUrl = await QRCode.toDataURL(secret.otpauth_url);
    
    return {
        secret: secret.base32,
        qrCode: qrCodeUrl,
        manualEntryKey: secret.base32
    };
};

// Verify MFA token
const verifyMFAToken = async (userId, token) => {
    const user = await User.findById(userId);
    
    if (!user.mfaSecret) {
        throw new Error('MFA not set up for this user');
    }
    
    const decryptedSecret = decrypt(user.mfaSecret);
    
    const verified = speakeasy.totp.verify({
        secret: decryptedSecret,
        encoding: 'base32',
        token: token,
        window: 2 // Allow 2 time steps before/after
    });
    
    return verified;
};

// MFA Setup Endpoint
app.post('/api/auth/mfa/setup', verifyToken, async (req, res) => {
    try {
        const mfaSetup = await generateMFASecret(req.user);
        res.json(mfaSetup);
    } catch (error) {
        res.status(500).json({ message: 'MFA setup failed' });
    }
});

// MFA Verification Endpoint
app.post('/api/auth/mfa/verify', verifyToken, async (req, res) => {
    const { token } = req.body;
    
    try {
        const isValid = await verifyMFAToken(req.user.userId, token);
        
        if (isValid) {
            await User.findByIdAndUpdate(req.user.userId, { mfaEnabled: true });
            res.json({ message: 'MFA enabled successfully' });
        } else {
            res.status(400).json({ message: 'Invalid MFA token' });
        }
    } catch (error) {
        res.status(500).json({ message: 'MFA verification failed' });
    }
});
```

### SMS-based MFA

```javascript
const twilio = require('twilio');
const client = twilio(process.env.TWILIO_ACCOUNT_SID, process.env.TWILIO_AUTH_TOKEN);

// SMS OTP storage (use Redis in production)
const smsOTPStore = new Map();

// Generate and send SMS OTP
const sendSMSOTP = async (phoneNumber, userId) => {
    const otp = Math.floor(100000 + Math.random() * 900000); // 6-digit OTP
    const expiresAt = Date.now() + 5 * 60 * 1000; // 5 minutes
    
    smsOTPStore.set(userId, { otp, expiresAt, phoneNumber });
    
    try {
        await client.messages.create({
            body: `Your verification code is: ${otp}. Valid for 5 minutes.`,
            from: process.env.TWILIO_PHONE_NUMBER,
            to: phoneNumber
        });
        
        return true;
    } catch (error) {
        console.error('SMS sending failed:', error);
        return false;
    }
};

// Verify SMS OTP
const verifySMSOTP = (userId, providedOTP) => {
    const storedData = smsOTPStore.get(userId);
    
    if (!storedData) {
        return false;
    }
    
    const { otp, expiresAt } = storedData;
    
    if (Date.now() > expiresAt) {
        smsOTPStore.delete(userId);
        return false;
    }
    
    if (parseInt(providedOTP) === otp) {
        smsOTPStore.delete(userId);
        return true;
    }
    
    return false;
};
```

---

## 🌐 Single Sign-On (SSO)

### SAML 2.0 Implementation

```javascript
const saml = require('passport-saml');
const passport = require('passport');

// SAML Strategy Configuration
passport.use(new saml.Strategy({
    entryPoint: process.env.SAML_ENTRY_POINT,
    issuer: process.env.SAML_ISSUER,
    cert: process.env.SAML_CERT,
    privateCert: process.env.SAML_PRIVATE_CERT,
    decryptionPvk: process.env.SAML_DECRYPTION_KEY,
    signatureAlgorithm: 'sha256',
    digestAlgorithm: 'sha256',
    callbackUrl: process.env.SAML_CALLBACK_URL,
    additionalParams: {},
    additionalAuthorizeParams: {},
    identifierFormat: null,
    acceptedClockSkewMs: -1,
    attributeConsumingServiceIndex: false,
    disableRequestedAuthnContext: false,
    authnContext: 'http://schemas.microsoft.com/ws/2008/06/identity/authenticationmethod/password',
    forceAuthn: false,
    skipRequestCompression: false,
    authnRequestBinding: 'HTTP-POST',
    validateInResponseTo: false,
    requestIdExpirationPeriodMs: 28800000,
    cacheProvider: 'memory'
}, async (profile, done) => {
    try {
        // Extract user information from SAML assertion
        const userInfo = {
            email: profile.email || profile.nameID,
            firstName: profile.firstName,
            lastName: profile.lastName,
            department: profile.department,
            role: profile.role
        };
        
        // Find or create user
        let user = await User.findOne({ email: userInfo.email });
        
        if (!user) {
            user = await User.create({
                email: userInfo.email,
                firstName: userInfo.firstName,
                lastName: userInfo.lastName,
                department: userInfo.department,
                role: userInfo.role,
                authProvider: 'saml'
            });
        }
        
        return done(null, user);
    } catch (error) {
        return done(error, null);
    }
}));

// SAML Routes
app.get('/auth/saml',
    passport.authenticate('saml', { failureRedirect: '/login', failureFlash: true }),
    (req, res) => {
        res.redirect('/dashboard');
    }
);

app.post('/auth/saml/callback',
    passport.authenticate('saml', { failureRedirect: '/login', failureFlash: true }),
    (req, res) => {
        const token = generateToken(req.user.id, req.user.email, req.user.role);
        res.redirect(`/dashboard?token=${token}`);
    }
);
```

---

## 🛡️ Zero Trust Architecture

### Device Fingerprinting

```javascript
const crypto = require('crypto');

// Generate device fingerprint
const generateDeviceFingerprint = (req) => {
    const userAgent = req.get('User-Agent') || '';
    const acceptLanguage = req.get('Accept-Language') || '';
    const acceptEncoding = req.get('Accept-Encoding') || '';
    const xForwardedFor = req.get('X-Forwarded-For') || '';
    
    const fingerprint = crypto
        .createHash('sha256')
        .update(userAgent + acceptLanguage + acceptEncoding + xForwardedFor)
        .digest('hex');
    
    return fingerprint;
};

// Device trust middleware
const deviceTrustMiddleware = async (req, res, next) => {
    const deviceFingerprint = generateDeviceFingerprint(req);
    const userId = req.user.userId;
    
    // Check if device is trusted
    const trustedDevice = await TrustedDevice.findOne({
        userId,
        fingerprint: deviceFingerprint
    });
    
    if (!trustedDevice) {
        // New device detected - require additional verification
        return res.status(403).json({
            message: 'New device detected. Additional verification required.',
            requiresDeviceVerification: true
        });
    }
    
    // Update last seen
    await TrustedDevice.findByIdAndUpdate(trustedDevice._id, {
        lastSeen: new Date(),
        ipAddress: req.ip
    });
    
    next();
};

// Trust new device
app.post('/api/auth/trust-device', verifyToken, async (req, res) => {
    const { verificationCode } = req.body;
    const deviceFingerprint = generateDeviceFingerprint(req);
    
    // Verify the code (sent via email/SMS)
    const isValidCode = await verifyDeviceCode(req.user.userId, verificationCode);
    
    if (!isValidCode) {
        return res.status(400).json({ message: 'Invalid verification code' });
    }
    
    // Trust the device
    await TrustedDevice.create({
        userId: req.user.userId,
        fingerprint: deviceFingerprint,
        deviceInfo: {
            userAgent: req.get('User-Agent'),
            ipAddress: req.ip,
            location: await getLocationFromIP(req.ip)
        },
        trustedAt: new Date()
    });
    
    res.json({ message: 'Device trusted successfully' });
});
```

---

## 🔍 Advanced Threat Protection

### Behavioral Analytics

```javascript
const ml = require('ml-matrix');

// User behavior tracking
const trackUserBehavior = async (userId, action, metadata) => {
    const behaviorData = {
        userId,
        action,
        timestamp: new Date(),
        metadata: {
            ...metadata,
            ipAddress: metadata.ipAddress,
            userAgent: metadata.userAgent,
            sessionDuration: metadata.sessionDuration,
            clickPattern: metadata.clickPattern
        }
    };
    
    await UserBehavior.create(behaviorData);
    
    // Analyze for anomalies
    const isAnomalous = await analyzeUserBehavior(userId, behaviorData);
    
    if (isAnomalous) {
        await handleAnomalousActivity(userId, behaviorData);
    }
};

// Anomaly detection
const analyzeUserBehavior = async (userId, currentBehavior) => {
    // Get historical behavior
    const historicalBehavior = await UserBehavior.find({
        userId,
        timestamp: { $gte: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000) }
    });
    
    if (historicalBehavior.length < 10) {
        return false; // Not enough data
    }
    
    // Simple anomaly detection based on patterns
    const anomalies = [];
    
    // Check for unusual login times
    const currentHour = new Date(currentBehavior.timestamp).getHours();
    const usualHours = historicalBehavior
        .filter(b => b.action === 'login')
        .map(b => new Date(b.timestamp).getHours());
    
    const isUnusualHour = !usualHours.includes(currentHour);
    if (isUnusualHour) anomalies.push('unusual_login_time');
    
    // Check for unusual location
    const currentIP = currentBehavior.metadata.ipAddress;
    const usualIPs = historicalBehavior.map(b => b.metadata.ipAddress);
    const isUnusualIP = !usualIPs.includes(currentIP);
    if (isUnusualIP) anomalies.push('unusual_location');
    
    // Check for rapid successive actions
    const recentActions = historicalBehavior
        .filter(b => Date.now() - new Date(b.timestamp).getTime() < 60000) // Last minute
        .length;
    
    if (recentActions > 10) anomalies.push('rapid_actions');
    
    return anomalies.length > 0;
};

// Handle anomalous activity
const handleAnomalousActivity = async (userId, behaviorData) => {
    // Create security alert
    await SecurityAlert.create({
        userId,
        alertType: 'anomalous_behavior',
        severity: 'medium',
        details: behaviorData,
        timestamp: new Date()
    });
    
    // Require step-up authentication
    await User.findByIdAndUpdate(userId, {
        requiresStepUp: true,
        stepUpReason: 'anomalous_behavior'
    });
    
    // Send notification to user
    await sendSecurityNotification(userId, 'suspicious_activity', behaviorData);
};
```

---

## 🧬 Biometric Authentication

### WebAuthn Implementation

```javascript
const { generateRegistrationOptions, verifyRegistrationResponse, 
        generateAuthenticationOptions, verifyAuthenticationResponse } = require('@simplewebauthn/server');

// WebAuthn Registration
app.post('/api/auth/webauthn/register/begin', verifyToken, async (req, res) => {
    const user = await User.findById(req.user.userId);
    
    const options = await generateRegistrationOptions({
        rpName: 'YourApp',
        rpID: process.env.WEBAUTHN_RP_ID,
        userID: user.id,
        userName: user.email,
        userDisplayName: user.name,
        attestationType: 'none',
        excludeCredentials: user.authenticators?.map(auth => ({
            id: auth.credentialID,
            type: 'public-key',
            transports: auth.transports,
        })),
        authenticatorSelection: {
            residentKey: 'discouraged',
            userVerification: 'preferred',
        },
        supportedAlgorithmIDs: [-7, -257],
    });
    
    // Store challenge for verification
    await User.findByIdAndUpdate(user.id, {
        currentChallenge: options.challenge
    });
    
    res.json(options);
});

app.post('/api/auth/webauthn/register/finish', verifyToken, async (req, res) => {
    const { body } = req;
    const user = await User.findById(req.user.userId);
    
    const verification = await verifyRegistrationResponse({
        response: body,
        expectedChallenge: user.currentChallenge,
        expectedOrigin: process.env.WEBAUTHN_ORIGIN,
        expectedRPID: process.env.WEBAUTHN_RP_ID,
    });
    
    if (verification.verified) {
        const { registrationInfo } = verification;
        
        const newAuthenticator = {
            credentialID: registrationInfo.credentialID,
            credentialPublicKey: registrationInfo.credentialPublicKey,
            counter: registrationInfo.counter,
            transports: body.response.transports,
        };
        
        await User.findByIdAndUpdate(user.id, {
            $push: { authenticators: newAuthenticator },
            $unset: { currentChallenge: 1 }
        });
        
        res.json({ verified: true });
    } else {
        res.status(400).json({ verified: false });
    }
});
```

---

## 🔐 Advanced Encryption

### End-to-End Encryption

```javascript
const crypto = require('crypto');

// Key derivation for E2E encryption
const deriveKey = (password, salt) => {
    return crypto.pbkdf2Sync(password, salt, 100000, 32, 'sha512');
};

// Encrypt data for E2E
const encryptE2E = (data, key) => {
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipher('aes-256-gcm', key);
    cipher.setAAD(Buffer.from('additional-data'));
    
    let encrypted = cipher.update(data, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    const authTag = cipher.getAuthTag();
    
    return {
        iv: iv.toString('hex'),
        encrypted,
        authTag: authTag.toString('hex')
    };
};

// Decrypt data for E2E
const decryptE2E = (encryptedData, key) => {
    const { iv, encrypted, authTag } = encryptedData;
    
    const decipher = crypto.createDecipher('aes-256-gcm', key);
    decipher.setAAD(Buffer.from('additional-data'));
    decipher.setAuthTag(Buffer.from(authTag, 'hex'));
    
    let decrypted = decipher.update(encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return decrypted;
};

// Secure message storage
app.post('/api/messages/secure', verifyToken, async (req, res) => {
    const { recipientId, message, userKey } = req.body;
    
    // Derive encryption key from user's key
    const salt = crypto.randomBytes(16);
    const encryptionKey = deriveKey(userKey, salt);
    
    // Encrypt message
    const encryptedMessage = encryptE2E(message, encryptionKey);
    
    // Store encrypted message
    await Message.create({
        senderId: req.user.userId,
        recipientId,
        encryptedContent: encryptedMessage,
        salt: salt.toString('hex'),
        timestamp: new Date()
    });
    
    res.json({ success: true });
});
```

---

## 📊 Security Monitoring

### Real-time Security Dashboard

```javascript
const SecurityMetrics = {
    // Track security events
    trackSecurityEvent: async (eventType, severity, details) => {
        await SecurityEvent.create({
            eventType,
            severity,
            details,
            timestamp: new Date()
        });
        
        // Real-time notification for high severity events
        if (severity === 'high') {
            await notifySecurityTeam(eventType, details);
        }
    },
    
    // Get security metrics
    getSecurityMetrics: async (timeRange) => {
        const events = await SecurityEvent.find({
            timestamp: { $gte: new Date(Date.now() - timeRange) }
        });
        
        return {
            totalEvents: events.length,
            highSeverityEvents: events.filter(e => e.severity === 'high').length,
            eventsByType: events.reduce((acc, event) => {
                acc[event.eventType] = (acc[event.eventType] || 0) + 1;
                return acc;
            }, {}),
            timeSeriesData: events.map(e => ({
                timestamp: e.timestamp,
                eventType: e.eventType,
                severity: e.severity
            }))
        };
    }
};

// Security dashboard endpoint
app.get('/api/admin/security/dashboard', verifyToken, requireRole('admin'), async (req, res) => {
    const timeRange = req.query.timeRange || 24 * 60 * 60 * 1000; // 24 hours
    const metrics = await SecurityMetrics.getSecurityMetrics(timeRange);
    res.json(metrics);
});
```

This advanced authentication guide covers multi-factor authentication, SSO, zero trust architecture, threat protection, biometric authentication, advanced encryption, and security monitoring - providing enterprise-level security implementations.
