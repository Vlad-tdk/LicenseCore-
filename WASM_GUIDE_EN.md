# WebAssembly Integration Guide for LicenseCore++

**Professional license validation in browsers with enterprise-grade HMAC-SHA256 cryptography.**

## 🚀 Quick Start (5 minutes)

### Option 1: Native JavaScript Implementation (Ready Now)
✅ **Production-ready** - Works in all browsers without additional setup
- Full LicenseCore++ API compatibility
- Web Crypto API for HMAC-SHA256 signatures
- Hardware fingerprinting simulation
- Perfect for client demos and prototypes

### Option 2: C++ WebAssembly Module (Build Required)
🔧 **Enhanced version** - Real C++ code compiled to WebAssembly
- Identical behavior to native LicenseCore++
- Performance benchmarking capabilities
- Maximum compatibility demonstration
- Enterprise client showcase

---

## 📦 Integration Methods

### Method 1: JavaScript Implementation (Immediate)

**Current live demo:** [https://username.github.io/LicenseCore/](docs/index.html)

```javascript
// Include the JavaScript version
<script src="demo.js"></script>

// Initialize LicenseCore
const licenseCore = new LicenseCoreDemo();

// Generate license
const license = await licenseCore.generateLicense(
    "customer-123", 
    ["basic", "premium", "api"], 
    365 // days
);

// Validate license
const result = await licenseCore.validateLicense(license);
console.log("Valid:", result.valid);
console.log("Features:", result.features);
```

**What's included:**
- ✅ HMAC-SHA256 signatures via Web Crypto API
- ✅ Hardware fingerprinting (browser-based)
- ✅ Full license validation logic
- ✅ Feature checking and expiry validation
- ✅ Cross-browser compatibility (Chrome, Firefox, Safari, Edge)

### Method 2: WebAssembly Module (Advanced)

**Build the WASM module:**

```bash
# Prerequisites: Emscripten SDK
git clone https://github.com/emscripten-core/emsdk.git
cd emsdk
./emsdk install latest
./emsdk activate latest
source ./emsdk_env.sh

# Build LicenseCore++ WASM
cd /path/to/LicenseCore
./wasm/build_wasm.sh
```

**Integration:**

```javascript
// Load WebAssembly module
import LicenseCoreModule from './license_core.js';

LicenseCoreModule().then(Module => {
    // Initialize with your secret key
    const manager = new Module.LicenseCoreWasm("your-secret-key");

    // Get hardware fingerprint
    const hwid = manager.getCurrentHwid();
    console.log("Hardware ID:", hwid);

    // Generate license
    const features = new Module.VectorString();
    features.push_back("basic");
    features.push_back("premium");
    features.push_back("api");

    const license = manager.generateLicense("user-123", features, 365);
    console.log("Generated license:", license);

    // Validate license
    const result = manager.validateLicense(license);
    console.log("Valid:", result.valid);
    console.log("User ID:", result.user_id);
    console.log("Features:", result.features);
    console.log("Error:", result.error_message);

    // Check specific feature
    const hasFeature = manager.hasFeature("premium", result.features);
    console.log("Has premium:", hasFeature);
});
```

---

## 🔐 API Reference

### LicenseCore JavaScript API

#### Constructor
```javascript
const licenseCore = new LicenseCoreDemo();
// Initializes with demo secret key and generates browser fingerprint
```

#### Methods

**`generateLicense(userId, features, expiryDays)`**
- **userId**: `string` - Unique user identifier
- **features**: `string[]` - Array of feature names
- **expiryDays**: `number` - License validity period (-1 for permanent)
- **Returns**: `Promise<string>` - JSON license with HMAC signature

```javascript
const license = await licenseCore.generateLicense(
    "customer-001", 
    ["basic", "premium"], 
    365
);
```

**`validateLicense(licenseJson)`**
- **licenseJson**: `string` - License JSON to validate
- **Returns**: `Promise<ValidationResult>` - Validation result object

```javascript
const result = await licenseCore.validateLicense(license);
// {
//   valid: true,
//   error: null,
//   features: ["basic", "premium"],
//   userId: "customer-001",
//   expiresAt: "2025-07-12T10:30:00Z"
// }
```

**`getCurrentHwid()`**
- **Returns**: `string` - Current hardware fingerprint

**`generateNewHwid()`**
- Generates new hardware fingerprint (for testing)

### LicenseCore WebAssembly API

#### Constructor
```javascript
const manager = new Module.LicenseCoreWasm(secretKey);
```

#### Methods

**`generateLicense(userId, features, expiryDays)`**
- **userId**: `string` - User identifier
- **features**: `Module.VectorString` - Features collection
- **expiryDays**: `number` - Days until expiry
- **Returns**: `string` - JSON license

**`validateLicense(licenseJson)`**
- **licenseJson**: `string` - License to validate
- **Returns**: `ValidationResult` - Result object

**`getCurrentHwid()`**
- **Returns**: `string` - Hardware fingerprint

**`hasFeature(feature, features)`**
- **feature**: `string` - Feature to check
- **features**: `string[]` - Available features
- **Returns**: `boolean`

---

## 🏗️ License JSON Format

Both implementations use the same license format:

```json
{
  "user_id": "customer-123",
  "license_id": "lic-1721036400",
  "expiry": "2025-07-12T10:30:00Z",
  "issued_at": "2024-07-15T10:30:00Z",
  "hardware_hash": "a1b2c3d4e5f6789...",
  "features": ["basic", "premium", "api"],
  "version": 1,
  "hmac_signature": "c4ef45e6d7a8b9..."
}
```

### Field Descriptions

- **`user_id`**: Unique customer identifier
- **`license_id`**: Unique license identifier
- **`expiry`**: ISO 8601 expiration timestamp
- **`issued_at`**: ISO 8601 issuance timestamp
- **`hardware_hash`**: Hardware fingerprint for device binding
- **`features`**: Array of enabled feature names
- **`version`**: License format version (currently 1)
- **`hmac_signature`**: HMAC-SHA256 signature for integrity

---

## 🔧 Build Instructions

### Prerequisites

1. **Emscripten SDK** (for WebAssembly build)
```bash
git clone https://github.com/emscripten-core/emsdk.git
cd emsdk
./emsdk install latest
./emsdk activate latest
source ./emsdk_env.sh
```

2. **Web Server** (for testing)
```bash
# Python
python -m http.server 8000

# Node.js
npx serve .

# PHP
php -S localhost:8000
```

### Building WebAssembly Module

```bash
# Navigate to LicenseCore directory
cd /path/to/LicenseCore

# Execute build script
./wasm/build_wasm.sh

# Output files generated:
# - wasm/build/license_core.js     (JavaScript loader)
# - wasm/build/license_core.wasm   (WebAssembly binary)
```

### Build Configuration

The build script uses these Emscripten flags:

```bash
emcc \
    -std=c++17 \
    -O3 \                           # Maximum optimization
    --bind \                        # Enable Embind bindings
    -s WASM=1 \                     # Generate WebAssembly
    -s MODULARIZE=1 \              # Create ES6 module
    -s EXPORT_NAME="LicenseCoreModule" \
    -s ENVIRONMENT=web \           # Web-only environment
    -s ALLOW_MEMORY_GROWTH=1 \     # Dynamic memory
    -s NO_EXIT_RUNTIME=1 \         # Keep runtime alive
    -s ASYNCIFY=1 \                # Enable async operations
    license_core_wasm.cpp \
    -o build/license_core.js
```

---

## 🌐 Deployment

### GitHub Pages (Recommended)

1. **Upload files to `docs/` directory:**
```bash
# Copy WebAssembly files (if using WASM)
cp wasm/build/license_core.js docs/
cp wasm/build/license_core.wasm docs/

# Commit and push
git add docs/
git commit -m "Add LicenseCore++ WebAssembly demo"
git push origin main
```

2. **Enable GitHub Pages:**
   - Go to repository Settings
   - Navigate to Pages section
   - Select "Deploy from branch: main /docs"
   - Save settings

3. **Access demo:**
   - URL: `https://username.github.io/repository-name/`
   - Demo ready in 2-3 minutes

### Custom Web Server

```bash
# Ensure HTTPS (required for Web Crypto API)
# Copy files to web root
cp docs/* /var/www/html/

# Configure proper MIME types
# Add to .htaccess or server config:
AddType application/wasm .wasm
AddType application/javascript .js
```

---

## 🔍 Testing & Validation

### Cross-Platform Compatibility Test

```javascript
// Test in different browsers
const testSuite = async () => {
    const licenseCore = new LicenseCoreDemo();
    
    // 1. Generate license
    const license = await licenseCore.generateLicense(
        "test-user", 
        ["basic"], 
        30
    );
    
    // 2. Validate same license
    const result = await licenseCore.validateLicense(license);
    
    // 3. Verify results
    console.assert(result.valid === true, "License should be valid");
    console.assert(result.userId === "test-user", "User ID should match");
    console.assert(result.features.includes("basic"), "Should have basic feature");
    
    console.log("✅ All tests passed!");
};

testSuite();
```

### HMAC Signature Verification

```javascript
// Verify that HMAC signatures are consistent
const verifySignatures = async () => {
    const licenseCore = new LicenseCoreDemo();
    
    // Generate same license data twice
    const license1 = await licenseCore.generateLicense("user1", ["test"], 365);
    const license2 = await licenseCore.generateLicense("user1", ["test"], 365);
    
    const parsed1 = JSON.parse(license1);
    const parsed2 = JSON.parse(license2);
    
    // Signatures should be different (due to timestamps)
    // but both should validate correctly
    const result1 = await licenseCore.validateLicense(license1);
    const result2 = await licenseCore.validateLicense(license2);
    
    console.assert(result1.valid && result2.valid, "Both licenses should be valid");
    console.log("✅ HMAC signature verification passed!");
};

verifySignatures();
```

---

## ⚡ Performance Comparison

### JavaScript vs WebAssembly Benchmarks

```javascript
// Benchmark license generation
const benchmark = async () => {
    const iterations = 1000;
    
    // JavaScript implementation
    const jsStart = performance.now();
    const licenseCore = new LicenseCoreDemo();
    
    for (let i = 0; i < iterations; i++) {
        await licenseCore.generateLicense(`user-${i}`, ["basic"], 365);
    }
    
    const jsTime = performance.now() - jsStart;
    
    // WebAssembly implementation (if available)
    if (typeof LicenseCoreModule !== 'undefined') {
        const wasmStart = performance.now();
        const Module = await LicenseCoreModule();
        const manager = new Module.LicenseCoreWasm("secret");
        
        for (let i = 0; i < iterations; i++) {
            const features = new Module.VectorString();
            features.push_back("basic");
            manager.generateLicense(`user-${i}`, features, 365);
        }
        
        const wasmTime = performance.now() - wasmStart;
        
        console.log(`JavaScript: ${jsTime.toFixed(2)}ms`);
        console.log(`WebAssembly: ${wasmTime.toFixed(2)}ms`);
        console.log(`Speedup: ${(jsTime / wasmTime).toFixed(2)}x`);
    }
};

benchmark();
```

### Expected Performance

| Operation | JavaScript | WebAssembly | Improvement |
|-----------|------------|-------------|-------------|
| License generation | ~2ms | ~0.5ms | 4x faster |
| License validation | ~1ms | ~0.3ms | 3x faster |
| HMAC computation | ~0.8ms | ~0.2ms | 4x faster |
| Hardware fingerprint | ~0.1ms | ~0.05ms | 2x faster |

---

## 🛡️ Security Features

### Cryptographic Implementation

**JavaScript Version:**
- Web Crypto API for HMAC-SHA256
- Browser's native cryptographic functions
- Hardware-accelerated when available
- Same security level as native implementation

**WebAssembly Version:**
- Identical C++ cryptographic code
- Compiled with optimizations
- Memory-safe execution
- Cross-platform consistency

### Hardware Fingerprinting

**Browser-based fingerprinting includes:**
- User agent string hash
- Screen resolution and color depth
- Timezone and language settings
- Available fonts and plugins
- WebGL renderer information
- Canvas fingerprinting

```javascript
// Hardware fingerprint components
const generateFingerprint = () => {
    const components = [
        navigator.userAgent,
        screen.width + 'x' + screen.height,
        screen.colorDepth,
        new Date().getTimezoneOffset(),
        navigator.language,
        // Additional entropy sources...
    ];
    
    return components.join('|');
};
```

### Tampering Protection

**License integrity verification:**
1. HMAC-SHA256 signature validation
2. Hardware fingerprint matching
3. Expiration date checking
4. Feature list validation
5. JSON structure verification

---

## 📋 Integration Checklist

### Pre-deployment
- [ ] Choose implementation (JavaScript or WebAssembly)
- [ ] Test in target browsers (Chrome, Firefox, Safari, Edge)
- [ ] Verify HTTPS deployment (required for Web Crypto API)
- [ ] Configure proper MIME types for .wasm files
- [ ] Test license generation and validation

### Production Setup
- [ ] Replace demo secret key with production key
- [ ] Implement proper error handling
- [ ] Add loading states for async operations
- [ ] Configure content security policy (CSP)
- [ ] Set up monitoring and analytics

### Security Validation
- [ ] Verify HMAC signatures match server-side
- [ ] Test license tampering detection
- [ ] Validate hardware fingerprint binding
- [ ] Check expiration date enforcement
- [ ] Test feature access control

---

## 🚨 Troubleshooting

### Common Issues

**"Web Crypto API not available"**
- **Cause**: HTTP instead of HTTPS
- **Solution**: Deploy over HTTPS or use localhost for testing

**"WebAssembly module failed to load"**
- **Cause**: Incorrect MIME type for .wasm files
- **Solution**: Configure server to serve .wasm as `application/wasm`

**"HMAC computation timeout"**
- **Cause**: Web Crypto API not responding
- **Solution**: Check browser support and network connectivity

**"License validation failed"**
- **Cause**: Hardware fingerprint mismatch
- **Solution**: Check device binding requirements

### Debug Mode

```javascript
// Enable detailed logging
const licenseCore = new LicenseCoreDemo();
licenseCore.debug = true;

// Generates detailed console output for troubleshooting
const result = await licenseCore.validateLicense(license);
```

### Browser Compatibility Check

```javascript
// Check Web Crypto API availability
if (!window.crypto || !window.crypto.subtle) {
    console.error("Web Crypto API not supported");
    // Fallback to server-side validation
}

// Check WebAssembly support
if (!window.WebAssembly) {
    console.warn("WebAssembly not supported, using JavaScript fallback");
}
```

---

## 📚 Additional Resources

### Documentation
- **Complete API Reference**: [docs/docs.html](docs/docs.html)
- **Usage Examples**: [EXAMPLES_EN.md](EXAMPLES_EN.md)
- **Error Handling**: [ERROR_HANDLING_GUIDE.md](ERROR_HANDLING_GUIDE.md)
- **Cross-Platform Guide**: [CROSS_PLATFORM_GUIDE.md](CROSS_PLATFORM_GUIDE.md)

### Live Examples
- **Interactive Demo**: [https://username.github.io/LicenseCore/](docs/index.html)
- **API Playground**: [docs/product.html](docs/product.html)
- **Performance Comparison**: [docs/comparison.html](docs/comparison.html)

### Support
- **Technical Support**: support@licensecore.tech
- **Sales Inquiries**: sales@licensecore.tech
- **Documentation**: [licensecore.tech/docs](https://licensecore.tech/docs)

---

## 🎯 Next Steps

### For Evaluation
1. **Test the live demo**: [GitHub Pages URL]
2. **Download evaluation package**: Complete with examples
3. **Schedule technical consultation**: 30-minute demo call
4. **Request custom integration**: Tailored to your needs

### For Production
1. **Purchase commercial license**: Choose appropriate tier
2. **Receive production package**: Source code and documentation
3. **Technical onboarding**: Integration support included
4. **Deploy and monitor**: Launch with confidence

---

**LicenseCore++** - Professional software licensing made simple.

*© 2024 LicenseCore Technologies. All rights reserved.*
