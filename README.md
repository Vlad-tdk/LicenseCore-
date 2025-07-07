# LicenseCore++ Documentation

## 🚀 Overview

Welcome to LicenseCore++ - the professional commercial software licensing solution with military-grade HMAC-SHA256 security, hardware fingerprinting, and WebAssembly support. This documentation site provides comprehensive guides, examples, and interactive demos.

## 📚 Documentation Structure

### 🏠 **Main Documentation**
- **[Interactive Demo](index.html)** - Try LicenseCore++ live in your browser
- **[API Documentation](docs.html)** - Complete API reference and examples
- **[Quickstart Guide](../QUICKSTART.md)** - Get started in 5 minutes

### 📖 **Specialized Guides**
- **[Embedded Guide](EMBEDDED_GUIDE.md)** - Lightweight integration for embedded systems
- **[Obfuscated Guide](OBFUSCATED_GUIDE.md)** - Security hardening and anti-tampering
- **[GitHub Pages Security](GITHUB_PAGES_SECURITY.md)** - Deployment security considerations

### 🛠️ **Core Library Guides**
- **[Usage Guide](../USAGE_GUIDE.md)** - Complete C++ API documentation
- **[Examples Collection](../EXAMPLES.md)** - Practical usage examples
- **[Cross-Platform Guide](../CROSS_PLATFORM_GUIDE.md)** - Windows, macOS, Linux support
- **[Production Build Guide](../PRODUCTION_BUILD_GUIDE.md)** - Release optimization

### ⚡ **Advanced Features**
- **[WebAssembly Guide](../WASM_GUIDE.md)** - Browser integration with WASM
- **[Caching Implementation](../CACHING_IMPLEMENTATION.md)** - Performance optimization
- **[Error Handling Guide](../ERROR_HANDLING_GUIDE.md)** - Exception management

## 🎯 Quick Navigation

### For Developers
```cpp
#include <license_core/license_manager.hpp>

LicenseManager manager("your-secret-key");
auto result = manager.load_and_validate(license_json);

if (result.valid && manager.has_feature("premium")) {
    // License OK, premium features available
}
```

### For Integration
1. **C++ Library**: Full-featured library with exceptions and caching
2. **Embedded Edition**: Lightweight C API for minimal footprint
3. **WebAssembly**: Browser-compatible WASM module
4. **Obfuscated**: Hardened version with anti-tampering protection

### For Deployment
- **Desktop Apps**: Static linking with hardware binding
- **Web Applications**: Real-time validation with WASM
- **IoT/Embedded**: Minimal footprint with C API
- **Enterprise**: Scalable licensing with server validation

## 🔐 Security Features

| Feature | Description | Implementation |
|---------|-------------|----------------|
| **HMAC-SHA256** | Cryptographic signatures | OpenSSL + Web Crypto API |
| **Hardware Binding** | Device fingerprinting | CPU ID, MAC, Volume Serial |
| **Time Validation** | Expiry checking | ISO 8601 timestamps |
| **Feature Control** | Granular permissions | JSON feature arrays |
| **Tamper Detection** | Integrity verification | Signature validation |

## 🌐 Platform Support

### Operating Systems
- ✅ **Windows** (MSVC 2019+, MinGW)
- ✅ **macOS** (Xcode 12+, Intel & Apple Silicon)
- ✅ **Linux** (GCC 9+, Ubuntu 20.04+)
- ✅ **WebAssembly** (All modern browsers)

### Architectures
- ✅ **x86/x64** (Intel, AMD)
- ✅ **ARM/ARM64** (Apple Silicon, Raspberry Pi)
- ✅ **WASM** (Browser environments)

### Build Systems
- ✅ **CMake** 3.16+ (recommended)
- ✅ **Make** (traditional builds)
- ✅ **MSVC** (Visual Studio)
- ✅ **Emscripten** (WebAssembly)

## 🎭 Interactive Demo Features

The documentation site includes a comprehensive interactive demo:

### **Live License Generation**
- Real-time license creation with HMAC-SHA256
- Hardware fingerprint simulation
- Configurable features and expiry dates
- JSON output with cryptographic signatures

### **Security Demonstrations**
- **Hardware Change Simulation** - Shows device binding
- **Expired License Testing** - Validates time restrictions
- **Tamper Detection** - Demonstrates signature verification
- **Performance Benchmarks** - WASM vs JavaScript comparison

### **Technology Showcase**
- **WebAssembly Mode** - Real C++ code in browser
- **JavaScript Fallback** - Compatible simulation
- **Hybrid Architecture** - Automatic fallback system
- **Cross-Platform** - Same code, multiple platforms

## 🔧 API Quick Reference

### Core Classes

#### LicenseManager
```cpp
class LicenseManager {
public:
    explicit LicenseManager(const std::string& secret_key);
    
    // Core validation
    LicenseInfo load_and_validate(const std::string& license_json);
    bool has_feature(const std::string& feature) const;
    
    // Utility methods
    std::string generate_license(const LicenseInfo& info) const;
    std::string get_current_hwid() const;
    
    // Configuration
    void set_hardware_config(const HardwareConfig& config);
    void set_strict_validation(bool strict = true);
};
```

#### HardwareFingerprint
```cpp
class HardwareFingerprint {
public:
    explicit HardwareFingerprint(const HardwareConfig& config = {});
    
    // Fingerprint generation
    std::string get_fingerprint() const;       // May throw
    std::string get_fingerprint_safe() const; // No exceptions
    
    // Individual components
    std::string get_cpu_id() const;
    std::string get_mac_address() const;
    
    // Cache management
    void clear_cache() const;
    CacheStats get_cache_stats() const;
};
```

#### Exception Hierarchy
```cpp
// Base exception
class LicenseException : public std::exception {};

// Specific exceptions
class InvalidSignatureException : public LicenseException {};
class ExpiredLicenseException : public LicenseException {};
class HardwareMismatchException : public LicenseException {};
class MalformedLicenseException : public LicenseException {};
```

## 📊 Performance Characteristics

| Operation | Time | Memory | Notes |
|-----------|------|--------|-------|
| License Validation | < 1ms | < 10KB | Cached fingerprint |
| Hardware Fingerprint | < 5ms | < 5KB | First generation |
| HMAC-SHA256 | < 0.1ms | < 1KB | OpenSSL optimized |
| JSON Parsing | < 0.5ms | < 2KB | Simple JSON only |

## 🛡️ Security Best Practices

### For Production Deployment
1. **Secret Key Management**
   - Use strong, unique keys (32+ characters)
   - Store keys securely (HSM, key vault)
   - Rotate keys periodically

2. **Hardware Fingerprinting**
   - Configure components based on stability
   - Enable caching for performance
   - Handle hardware changes gracefully

3. **License Distribution**
   - Generate licenses server-side only
   - Use secure channels (HTTPS, TLS)
   - Implement license revocation

4. **Application Protection**
   - Obfuscate critical code paths
   - Implement anti-debugging measures
   - Use binary packing (UPX, etc.)

### Common Security Pitfalls
❌ **Don't embed secret keys in client code**
❌ **Don't rely solely on client-side validation**
❌ **Don't use predictable license IDs**
❌ **Don't ignore hardware fingerprint changes**

✅ **Do generate licenses server-side**
✅ **Do use strong secret keys**
✅ **Do implement proper error handling**
✅ **Do consider obfuscation for sensitive code**

## 🎓 Learning Path

### Beginner (15 minutes)
1. Read [Quickstart Guide](../QUICKSTART.md)
2. Try [Interactive Demo](index.html)
3. Build [Simple Example](../examples/simple_example.cpp)

### Intermediate (1 hour)
1. Study [Usage Guide](../USAGE_GUIDE.md)
2. Explore [Examples Collection](../EXAMPLES.md)
3. Configure hardware fingerprinting
4. Implement exception handling

### Advanced (half day)
1. Deploy [WebAssembly integration](../WASM_GUIDE.md)
2. Optimize with [Caching Guide](../CACHING_IMPLEMENTATION.md)
3. Harden with [Obfuscation Guide](OBFUSCATED_GUIDE.md)
4. Scale with production architecture

### Expert (full day)
1. Custom hardware fingerprinting
2. Server-side license generation
3. License revocation systems
4. Enterprise deployment patterns

## 📞 Support & Resources

### Documentation
- **Website**: [licensecore.tech](https://licensecore.tech)
- **API Docs**: [docs.licensecore.tech](https://docs.licensecore.tech)
- **Downloads**: [licensecore.tech/downloads](https://licensecore.tech/downloads)

### Community
- **Forum**: [forum.licensecore.tech](https://forum.licensecore.tech)
- **Discord**: [LicenseCore++ Community](https://discord.gg/licensecore)
- **Stack Overflow**: Tag `licensecore`

### Commercial Support
- **Email**: [enterprise@licensecore.tech](mailto:enterprise@licensecore.tech)
- **Sales**: [sales@licensecore.tech](mailto:sales@licensecore.tech)
- **Priority Support**: Available with Professional+ licenses

## 📄 License & Pricing

### Commercial Licenses Required
All usage requires a commercial license:

- **Developer**: $299 (single developer)
- **Professional**: $899 (up to 5 developers)
- **Enterprise**: $1,999/year (unlimited developers)

All commercial licenses include:
- ✅ Production deployment rights
- ✅ Obfuscated source access
- ✅ Priority email support
- ✅ 1-year free updates
- ✅ Commercial use license

### No Open Source License
This is a commercial software product. All components are proprietary and require a valid license for any use.

---

**🚀 Ready to get started?** Try our [Interactive Demo](index.html) or read the [Quickstart Guide](../QUICKSTART.md)!

**💼 Need enterprise features?** Contact [sales@licensecore.tech](mailto:sales@licensecore.tech) for custom solutions.

---

*© 2024 LicenseCore Technologies. Professional software licensing made simple.*
