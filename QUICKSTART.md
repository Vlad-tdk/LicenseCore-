# LicenseCore++ Quick Start

## 🚀 Build & Test (5 minutes)

```bash
# Download LicenseCore++ Professional from licensecore.tech
# Extract the package
unzip LicenseCore-Professional.zip
cd LicenseCore

# Build everything
chmod +x build.sh
./build.sh

# Test the examples
cd build
./examples/simple_example
./examples/hwid_tool
./examples/license_generator --help
```

## 📦 Integration

### CMake Project
```cmake
find_package(LicenseCore REQUIRED)
target_link_libraries(your_app LicenseCore::licensecore)
```

### Manual Integration
```cpp
#include <license_core/license_manager.hpp>
// Link: -llicensecore -lssl -lcrypto
```

## 💡 Basic Usage

```cpp
#include <license_core/license_manager.hpp>

using namespace license_core;

int main() {
    try {
        // 1. Initialize
        LicenseManager manager("your-secret-key");

        // 2. Generate license (server-side)
        LicenseInfo info;
        info.user_id = "customer-123";
        info.hardware_hash = manager.get_current_hwid();
        info.features = {"basic", "premium"};
        info.expiry = std::chrono::system_clock::now() + std::chrono::hours(24 * 365); // 1 year
        info.issued_at = std::chrono::system_clock::now();
        info.license_id = "lic-" + std::to_string(std::time(nullptr));
        info.version = 1;
        
        std::string license = manager.generate_license(info);

        // 3. Validate license (client-side)
        auto result = manager.load_and_validate(license);
        if (result.valid && manager.has_feature("premium")) {
            // License OK, premium features available
            std::cout << "✅ Welcome, " << result.user_id << "!" << std::endl;
        }

    } catch (const LicenseException& e) {
        std::cerr << "❌ License error: " << e.what() << std::endl;
    }

    return 0;
}
```

## 🔧 Dependencies

- **OpenSSL** (for HMAC-SHA256)
- **C++17** compiler
- **CMake 3.16+**

### Install Dependencies

**macOS:**
```bash
brew install openssl cmake
```

**Ubuntu/Debian:**
```bash
sudo apt install libssl-dev cmake build-essential
```

**Windows:**
```bash
vcpkg install openssl
```

## 📋 Next Steps

1. **Purchase a license** at [licensecore.tech](https://licensecore.tech)
2. **Customize hardware fingerprinting** - see `HardwareConfig`
3. **Integrate into your app** - see `examples/simple_example.cpp`
4. **Deploy license server** - use `license_generator` tool
5. **Add obfuscation** - protect your secret key
6. **Consider enterprise features** - RSA signatures, online validation

## 🆘 Common Issues

**Build fails with OpenSSL not found:**
```bash
# macOS
export PKG_CONFIG_PATH="/opt/homebrew/lib/pkgconfig"

# Linux
sudo apt install pkg-config

# Windows
set OpenSSL_ROOT_DIR=C:\vcpkg\installed\x64-windows
```

**Hardware fingerprint empty:**
- Check platform-specific permissions
- Some components require admin rights
- Virtual machines may have limited HW info

**License Required:**
- This is commercial software
- All usage requires a valid license
- Purchase at [licensecore.tech](https://licensecore.tech)

## 💰 Commercial Licensing

### Developer License ($299)
- Single developer
- Commercial deployment rights
- 1 year updates & support

### Professional License ($899)
- Up to 5 developers
- OEM embedding rights
- Advanced obfuscation
- Priority support

### Enterprise License ($1,999/year)
- Unlimited developers
- Custom integrations
- Source modifications
- SLA guarantees

### Purchase
- **Website**: [licensecore.tech](https://licensecore.tech)
- **Sales**: [sales@licensecore.tech](mailto:sales@licensecore.tech)

---

**Ready to deploy professional licensing?** Get your commercial license at [licensecore.tech](https://licensecore.tech)

*© 2024 LicenseCore Technologies*
