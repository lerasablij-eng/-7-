#include <iostream>
#include <stdexcept>
#include <string>
#include <Windows.h>

// Власні виключення
class EmptyException {};
class CustomException {
public:
    std::string message;
    int h, m;
    CustomException(const std::string& msg, int hours, int minutes)
        : message(msg), h(hours), m(minutes) {
    }
};
class DerivedException : public std::out_of_range {
public:
    int value;
    DerivedException(const std::string& msg, int val)
        : std::out_of_range(msg), value(val) {}
};

// 1. Без специфікації
int toSeconds1(int h, int m) {
    if (h < 0) throw std::invalid_argument("Години не можуть бути від'ємними");
    if (m < 0 || m >= 60) throw std::out_of_range("Хвилини повинні бути від 0 до 59");
    return h * 3600 + m * 60;
}

// 2. Зі специфікацією throw()
int toSeconds2(int h, int m) throw() {
    if (h < 0 || m < 0) throw std::invalid_argument("Негативне значення");
    if (m >= 60) throw std::out_of_range("Занадто багато хвилин");
    return h * 3600 + m * 60;
}

// 3. З конкретною специфікацією
int toSeconds3(int h, int m) noexcept(false) {
    if (h < 0) throw std::invalid_argument("Некоректні години: " + std::to_string(h));
    if (m < 0 || m >= 60) throw std::out_of_range("Некоректні хвилини: " + std::to_string(m));
    return h * 3600 + m * 60;
}

// 4. З власними виключеннями
int toSeconds4(int h, int m) {
    if (h < 0) throw EmptyException();
    if (m < 0) throw CustomException("Негативні хвилини", h, m);
    if (m >= 60) throw DerivedException("Хвилини поза діапазоном", m);
    return h * 3600 + m * 60;
}
// Головна функція
int main() {
    SetConsoleOutputCP(1251);
    SetConsoleCP(1251);
    int testCases[][2] = { {2, 30}, {-1, 30}, {2, -5}, {1, 75} };

    for (int i = 0; i < 4; ++i) {
        std::cout << "\nТест " << i + 1 << ": " << testCases[i][0] << " год " << testCases[i][1] << " хв\n";

        for (int func = 1; func <= 4; ++func) {
            std::cout << "Функція " << func << ": ";
            try {
                int result = 0;
                switch (func) {
                case 1: result = toSeconds1(testCases[i][0], testCases[i][1]); break;
                case 2: result = toSeconds2(testCases[i][0], testCases[i][1]); break;
                case 3: result = toSeconds3(testCases[i][0], testCases[i][1]); break;
                case 4: result = toSeconds4(testCases[i][0], testCases[i][1]); break;
                }
                std::cout << result << " секунд\n";
            }
            catch (const std::invalid_argument& e) { std::cout << e.what() << "\n"; }
            catch (const std::out_of_range& e) { std::cout << e.what() << "\n"; }
            catch (const EmptyException&) { std::cout << "EmptyException\n"; }
            catch (const CustomException& e) { std::cout << e.message << " h:" << e.h << " m:" << e.m << "\n"; }
            catch (const DerivedException& e) { std::cout << e.what() << " value:" << e.value << "\n"; }
            catch (...) { std::cout << "Невідоме виключення\n"; }
        }
    }
    // Демонстрація коректної роботи
    std::cout << "\nКоректні обчислення";
    try {
        std::cout << "2 год 15 хв = " << toSeconds1(2, 15) << " секунд\n";
        std::cout << "0 год 45 хв = " << toSeconds2(0, 45) << " секунд\n";
        std::cout << "1 год 0 хв = " << toSeconds3(1, 0) << " секунд\n";
        std::cout << "3 год 30 хв = " << toSeconds4(3, 30) << " секунд\n";
    }
    catch (...) {
        std::cout << "Неочікувана помилка при коректних даних\n";
    }

    return 0;
}
