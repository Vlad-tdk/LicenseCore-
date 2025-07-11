# 💻 Примеры использования основной библиотеки LicenseCore++

## 📁 Структура примеров

Все примеры для основной библиотеки находятся в папке `examples/`:
- `simple_example` - базовое использование
- `license_generator` - создание лицензий
- `hwid_tool` - получение Hardware ID

## 🚀 Базовое использование

### Простейший пример

```cpp
#include <iostream>
#include "license_core/hardware_fingerprint.hpp"
#include "license_core/license_manager.hpp"

using namespace license_core;

int main() {
    try {
        // 1. Получение Hardware ID
        HardwareConfig config;
        HardwareFingerprint fingerprint(config);
        std::string hardware_id = fingerprint.get_fingerprint();
        
        std::cout << "Hardware ID: " << hardware_id << std::endl;
        
        // 2. Проверка лицензии из JSON
        LicenseManager manager("your-secret-key");
        
        std::string license_json = R"({
            "user_id": "customer-123",
            "hardware_hash": ")" + hardware_id + R"(",
            "features": ["basic", "premium"],
            "expiry": "2025-12-31T23:59:59Z",
            "issued_at": "2024-01-01T00:00:00Z",
            "license_id": "lic-456",
            "version": 1,
            "hmac_signature": "c4ef45e6..."
        })";
        
        // 3. Загрузка и проверка лицензии
        auto info = manager.load_and_validate(license_json);
        
        if (info.valid) {
            std::cout << "✅ Лицензия валидна!" << std::endl;
            std::cout << "👤 Пользователь: " << info.user_id << std::endl;
            
            if (manager.has_feature("premium")) {
                std::cout << "🌟 Премиум функции доступны!" << std::endl;
            }
        } else {
            std::cout << "❌ Лицензия невалидна: " << info.error_message << std::endl;
        }
        
    } catch (const LicenseException& e) {
        std::cerr << "Ошибка: " << e.what() << std::endl;
        return 1;
    }
    
    return 0;
}
```

## 🔧 Продвинутое использование

### Кастомная конфигурация оборудования

```cpp
#include "license_core/hardware_fingerprint.hpp"

HardwareConfig create_secure_config() {
    HardwareConfig config;
    
    // Компоненты для отпечатка
    config.use_cpu_id = true;              
    config.use_mac_address = true;         
    config.use_motherboard_serial = false; // Отключено для совместимости
    config.use_volume_serial = true;         
    
    // Настройки кэширования
    config.enable_caching = true;             
    config.cache_lifetime = std::chrono::minutes(30);
    config.thread_safe_cache = true;                
    
    return config;
}

void hardware_example() {
    try {
        HardwareConfig config = create_secure_config();
        HardwareFingerprint fingerprint(config);
        
        // Получить полный отпечаток
        std::string hardware_id = fingerprint.get_fingerprint();
        std::cout << "Full Hardware ID: " << hardware_id << std::endl;
        
        // Получить отдельные компоненты
        std::string cpu_id = fingerprint.get_cpu_id();
        std::string mac_addr = fingerprint.get_mac_address();
        
        std::cout << "CPU ID: " << cpu_id << std::endl;
        std::cout << "MAC Address: " << mac_addr << std::endl;
        
        // Безопасные версии (не выбрасывают исключения)
        std::string safe_hardware_id = fingerprint.get_fingerprint_safe();
        std::cout << "Safe Hardware ID: " << safe_hardware_id << std::endl;
        
        // Статистика кэша
        auto stats = fingerprint.get_cache_stats();
        std::cout << "Cache hit rate: " << (stats.hit_rate() * 100) << "%" << std::endl;
        
    } catch (const HardwareDetectionException& e) {
        std::cerr << "Ошибка оборудования: " << e.what() << std::endl;
    }
}
```

### Работа с лицензиями из файла

```cpp
#include <fstream>
#include <chrono>

class LicenseFileHandler {
private:
    LicenseManager manager_;
    
public:
    LicenseFileHandler(const std::string& secret_key) 
        : manager_(secret_key) {}
    
    bool loadLicenseFromFile(const std::string& file_path) {
        try {
            std::ifstream file(file_path);
            if (!file.is_open()) {
                std::cerr << "Не удалось открыть файл: " << file_path << std::endl;
                return false;
            }
            
            std::string license_json((std::istreambuf_iterator<char>(file)),
                                    std::istreambuf_iterator<char>());
            
            // Загружаем и проверяем лицензию
            auto info = manager_.load_and_validate(license_json);
            
            if (info.valid) {
                std::cout << "✅ Лицензия загружена успешно!" << std::endl;
                std::cout << "👤 Пользователь: " << info.user_id << std::endl;
                std::cout << "🆔 ID лицензии: " << info.license_id << std::endl;
                
                // Выводим время истечения
                auto expiry_time = std::chrono::system_clock::to_time_t(info.expiry);
                std::cout << "⏰ Истекает: " << std::ctime(&expiry_time);
                
                return true;
            } else {
                std::cout << "❌ Ошибка лицензии: " << info.error_message << std::endl;
                return false;
            }
            
        } catch (const std::exception& e) {
            std::cerr << "Ошибка при загрузке лицензии: " << e.what() << std::endl;
            return false;
        }
    }
    
    std::vector<std::string> getAvailableFeatures() {
        return manager_.get_available_features();
    }
    
    bool hasFeature(const std::string& feature) {
        return manager_.has_feature(feature);
    }
};
```

### Обработка ошибок

```cpp
void comprehensive_error_handling() {
    try {
        HardwareConfig config;
        HardwareFingerprint fingerprint(config);
        
        std::string hardware_id = fingerprint.get_fingerprint();
        
        LicenseManager manager("secret-key");
        std::string license_json = "..."; // JSON лицензии
        
        auto info = manager.load_and_validate(license_json);
        
        if (info.valid) {
            std::cout << "Лицензия действительна" << std::endl;
        }
        
    } catch (const InvalidSignatureException& e) {
        // Неверная подпись HMAC
        std::cerr << "❌ Неверная подпись лицензии: " << e.what() << std::endl;
        
    } catch (const ExpiredLicenseException& e) {
        // Лицензия истекла
        std::cerr << "⏰ Лицензия истекла: " << e.what() << std::endl;
        
    } catch (const HardwareMismatchException& e) {
        // Hardware ID не совпадает
        std::cerr << "🖥️ Лицензия для другого оборудования: " << e.what() << std::endl;
        
    } catch (const MalformedLicenseException& e) {
        // Поврежденный JSON или неверный формат
        std::cerr << "📄 Поврежденная лицензия: " << e.what() << std::endl;
        
    } catch (const HardwareDetectionException& e) {
        // Ошибки определения оборудования
        std::cerr << "🔧 Ошибка определения оборудования: " << e.what() << std::endl;
        
    } catch (const CryptographicException& e) {
        // Ошибки криптографии
        std::cerr << "🔐 Криптографическая ошибка: " << e.what() << std::endl;
        
    } catch (const LicenseException& e) {
        // Базовый класс - все остальные ошибки библиотеки
        std::cerr << "⚠️ Общая ошибка LicenseCore: " << e.what() << std::endl;
        
    } catch (const std::exception& e) {
        // Стандартные исключения C++
        std::cerr << "💥 Стандартная ошибка: " << e.what() << std::endl;
    }
}
```

### Многопоточное использование

```cpp
#include <thread>
#include <vector>
#include <mutex>

class ThreadSafeLicenseManager {
private:
    HardwareFingerprint fingerprint_;
    LicenseManager license_manager_;
    mutable std::mutex results_mutex_;
    std::vector<bool> validation_results_;
    
public:
    ThreadSafeLicenseManager(const std::string& secret_key) 
        : fingerprint_(HardwareConfig{.thread_safe_cache = true}),
          license_manager_(secret_key) {}
    
    void validateLicenseAsync(const std::string& license_json, size_t thread_id) {
        bool result = false;
        
        try {
            auto info = license_manager_.load_and_validate(license_json);
            result = info.valid;
            
        } catch (const std::exception& e) {
            std::cerr << "Ошибка в потоке " << thread_id << ": " << e.what() << std::endl;
        }
        
        // Потокобезопасное сохранение результата
        std::lock_guard<std::mutex> lock(results_mutex_);
        if (validation_results_.size() <= thread_id) {
            validation_results_.resize(thread_id + 1);
        }
        validation_results_[thread_id] = result;
    }
    
    void runConcurrentValidation(const std::vector<std::string>& licenses) {
        std::vector<std::thread> threads;
        
        for (size_t i = 0; i < licenses.size(); ++i) {
            threads.emplace_back(&ThreadSafeLicenseManager::validateLicenseAsync,
                               this, std::ref(licenses[i]), i);
        }
        
        // Ожидаем завершения всех потоков
        for (auto& thread : threads) {
            thread.join();
        }
        
        // Выводим результаты
        std::lock_guard<std::mutex> lock(results_mutex_);
        for (size_t i = 0; i < validation_results_.size(); ++i) {
            std::cout << "Лицензия " << i << ": " 
                      << (validation_results_[i] ? "✅ Валидна" : "❌ Невалидна") 
                      << std::endl;
        }
    }
};
```

### Создание собственного генератора лицензий

```cpp
#include <chrono>
#include <ctime>
#include <iomanip>
#include <sstream>

class LicenseGenerator {
private:
    LicenseManager manager_;
    
public:
    LicenseGenerator(const std::string& secret_key) : manager_(secret_key) {}
    
    LicenseInfo createLicense(const std::string& customer_id,
                             const std::string& hardware_id,
                             const std::vector<std::string>& features,
                             int validity_days = 365) {
        LicenseInfo license;
        
        license.user_id = customer_id;
        license.license_id = "lic-" + std::to_string(std::time(nullptr));
        license.hardware_hash = hardware_id;
        license.features = features;
        license.version = 1;
        
        // Устанавливаем время выдачи
        license.issued_at = std::chrono::system_clock::now();
        
        // Устанавливаем срок действия
        license.expiry = license.issued_at + std::chrono::hours(24 * validity_days);
        
        return license;
    }
    
    std::string generateSignedLicense(const LicenseInfo& license) {
        return manager_.generate_license(license);
    }
    
    bool validateGeneratedLicense(const std::string& license_json) {
        try {
            auto info = manager_.load_and_validate(license_json);
            return info.valid;
        } catch (const std::exception&) {
            return false;
        }
    }
};

// Пример использования генератора
void license_generation_example() {
    LicenseGenerator generator("my-secret-key");
    
    // Получаем Hardware ID клиента
    HardwareFingerprint fingerprint;
    std::string customer_hardware = fingerprint.get_fingerprint_safe();
    
    // Генерируем лицензию
    auto license = generator.createLicense(
        "enterprise-customer-001",
        customer_hardware,
        {"basic", "premium", "enterprise"},
        365  // 1 год
    );
    
    // Создаем подписанную лицензию
    std::string signed_license = generator.generateSignedLicense(license);
    
    std::cout << "Сгенерированная лицензия:" << std::endl;
    std::cout << signed_license << std::endl;
    
    // Проверяем сгенерированную лицензию
    bool is_valid = generator.validateGeneratedLicense(signed_license);
    std::cout << "Лицензия " << (is_valid ? "✅ валидна" : "❌ невалидна") << std::endl;
}
```

## 🎯 Практические сценарии

### Desktop приложение с активацией

```cpp
class DesktopAppLicensing {
private:
    std::unique_ptr<HardwareFingerprint> fingerprint_;
    std::unique_ptr<LicenseManager> manager_;
    bool is_activated_;
    
public:
    DesktopAppLicensing(const std::string& secret_key) 
        : is_activated_(false) {
        HardwareConfig config;
        config.enable_caching = true;
        config.cache_lifetime = std::chrono::hours(1);
        
        fingerprint_ = std::make_unique<HardwareFingerprint>(config);
        manager_ = std::make_unique<LicenseManager>(secret_key);
    }
    
    std::string getActivationCode() {
        return fingerprint_->get_fingerprint_safe();
    }
    
    bool activateWithLicense(const std::string& license_json) {
        try {
            auto info = manager_->load_and_validate(license_json);
            
            if (!info.valid) {
                std::cout << "❌ Лицензия невалидна: " << info.error_message << std::endl;
                return false;
            }
            
            // Проверяем привязку к оборудованию
            std::string current_hardware = fingerprint_->get_fingerprint();
            if (info.hardware_hash != current_hardware) {
                std::cout << "❌ Лицензия не подходит для этого оборудования" << std::endl;
                return false;
            }
            
            // Проверяем срок действия
            if (manager_->is_expired()) {
                std::cout << "❌ Срок действия лицензии истёк" << std::endl;
                return false;
            }
            
            is_activated_ = true;
            std::cout << "✅ Приложение активировано успешно!" << std::endl;
            return true;
            
        } catch (const std::exception& e) {
            std::cerr << "Ошибка активации: " << e.what() << std::endl;
            return false;
        }
    }
    
    bool hasFeature(const std::string& feature) {
        if (!is_activated_) {
            return false;
        }
        return manager_->has_feature(feature);
    }
    
    void showFeatureStatus() {
        if (!is_activated_) {
            std::cout << "❌ Приложение не активировано" << std::endl;
            return;
        }
        
        std::cout << "🎯 Доступные функции:" << std::endl;
        auto features = manager_->get_available_features();
        for (const auto& feature : features) {
            bool available = hasFeature(feature);
            std::cout << "  " << (available ? "✅" : "❌") << " " << feature << std::endl;
        }
    }
};
```

### Использование HMAC валидатора

```cpp
#include "license_core/hmac_validator.hpp"

void hmac_validation_example() {
    try {
        HMACValidator validator("my-secret-key");
        
        // Подписываем данные
        std::string data = "important license data";
        std::string signature = validator.sign(data);
        
        std::cout << "Данные: " << data << std::endl;
        std::cout << "Подпись: " << signature << std::endl;
        
        // Проверяем подпись
        bool is_valid = validator.verify(data, signature);
        std::cout << "Подпись " << (is_valid ? "✅ действительна" : "❌ недействительна") << std::endl;
        
        // Работа с JSON
        std::string json_without_signature = R"({
            "user_id": "test-user",
            "features": ["basic", "premium"],
            "expiry": "2025-12-31T23:59:59Z"
        })";
        
        std::string signed_json = validator.sign_json(json_without_signature);
        std::cout << "Подписанный JSON:" << std::endl << signed_json << std::endl;
        
        // Проверяем JSON
        bool json_valid = validator.verify_json(signed_json);
        std::cout << "JSON подпись " << (json_valid ? "✅ действительна" : "❌ недействительна") << std::endl;
        
    } catch (const CryptographicException& e) {
        std::cerr << "Криптографическая ошибка: " << e.what() << std::endl;
    }
}
```

## 🔍 Отладка и диагностика

### Система логирования

```cpp
#include <fstream>

class LicenseLogger {
private:
    std::ofstream log_file_;
    
public:
    LicenseLogger(const std::string& log_path) : log_file_(log_path, std::ios::app) {}
    
    void logHardwareInfo() {
        HardwareFingerprint fingerprint;
        
        log_file_ << "=== Hardware Information ===" << std::endl;
        log_file_ << "Full ID: " << fingerprint.get_fingerprint_safe() << std::endl;
        log_file_ << "CPU ID: " << fingerprint.get_cpu_id_safe() << std::endl;
        log_file_ << "MAC Address: " << fingerprint.get_mac_address_safe() << std::endl;
        log_file_ << "Volume Serial: " << fingerprint.get_volume_serial() << std::endl;
        log_file_ << "Timestamp: " << std::time(nullptr) << std::endl;
        log_file_ << std::endl;
    }
    
    void logLicenseValidation(const LicenseInfo& info) {
        log_file_ << "=== License Validation ===" << std::endl;
        log_file_ << "License ID: " << info.license_id << std::endl;
        log_file_ << "User ID: " << info.user_id << std::endl;
        log_file_ << "Valid: " << (info.valid ? "YES" : "NO") << std::endl;
        if (!info.valid) {
            log_file_ << "Error: " << info.error_message << std::endl;
        }
        
        // Логируем время истечения
        auto expiry_time = std::chrono::system_clock::to_time_t(info.expiry);
        log_file_ << "Expiry: " << std::ctime(&expiry_time);
        
        log_file_ << "Features: ";
        for (const auto& feature : info.features) {
            log_file_ << feature << " ";
        }
        log_file_ << std::endl << std::endl;
    }
};
```

### Проверка производительности

```cpp
#include <chrono>

void performance_test() {
    // Тест производительности Hardware Fingerprint
    auto start = std::chrono::high_resolution_clock::now();
    
    HardwareFingerprint fingerprint;
    for (int i = 0; i < 1000; ++i) {
        fingerprint.get_fingerprint_safe(); // Должен использовать кэш
    }
    
    auto end = std::chrono::high_resolution_clock::now();
    auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start);
    
    std::cout << "1000 вызовов get_fingerprint_safe: " 
              << duration.count() << " микросекунд" << std::endl;
    
    auto stats = fingerprint.get_cache_stats();
    std::cout << "Cache hit rate: " << (stats.hit_rate() * 100) << "%" << std::endl;
    std::cout << "Cache hits: " << stats.cache_hits << std::endl;
    std::cout << "Cache misses: " << stats.cache_misses << std::endl;
}
```

## 📁 Файлы примеров

Смотрите готовые примеры в папке `examples/`:

1. **`simple_example.cpp`** - базовое использование
2. **`license_generator.cpp`** - создание лицензий  
3. **`hwid_tool.cpp`** - получение Hardware ID
4. **`error_handling_example.cpp`** - обработка ошибок
5. **`caching_example.cpp`** - работа с кэшированием
6. **`performance_test.cpp`** - тесты производительности

Компиляция примеров:
```bash
cd build
./examples/simple_example
./examples/license_generator
./examples/hwid_tool
```

## 📝 Заметки по интеграции

### Инициализация менеджера лицензий

```cpp
// Правильная инициализация
class MyApplication {
private:
    std::unique_ptr<LicenseManager> license_manager_;
    
public:
    bool initialize() {
        try {
            // Создаем менеджер с секретным ключом
            license_manager_ = std::make_unique<LicenseManager>("secure-secret-key");
            
            // Загружаем лицензию при запуске
            return loadLicense();
            
        } catch (const std::exception& e) {
            std::cerr << "Ошибка инициализации: " << e.what() << std::endl;
            return false;
        }
    }
    
    bool loadLicense() {
        // Попытка загрузить лицензию из разных источников
        std::vector<std::string> license_paths = {
            "license.json",
            "config/license.json", 
            getAppDataPath() + "/license.json"
        };
        
        for (const auto& path : license_paths) {
            if (tryLoadLicenseFromFile(path)) {
                return true;
            }
        }
        
        return false; // Лицензия не найдена
    }
};
```

### Безопасное хранение секретного ключа

```cpp
// Не храните секретный ключ прямо в коде!
// ❌ Плохо:
// const char* SECRET_KEY = "my-secret-key-123";

// ✅ Хорошо - загрузка из переменной окружения:
std::string getSecretKey() {
    const char* key = std::getenv("LICENSECORE_SECRET_KEY");
    if (!key) {
        throw std::runtime_error("Secret key not found in environment");
    }
    return std::string(key);
}

// ✅ Хорошо - загрузка из зашифрованного файла:
std::string loadEncryptedSecretKey(const std::string& encrypted_file) {
    // Загрузка и расшифровка секретного ключа
    // Реализация зависит от выбранного метода шифрования
    return decryptSecretKey(encrypted_file);
}
```
