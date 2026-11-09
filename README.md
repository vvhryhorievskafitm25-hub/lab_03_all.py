# lab_03_all.py
"""
Лабораторна робота №3: Реалізація обробки виключень, модульності та документації в проектах Python
Виконано студенткою групи КБ-25: Григор'євська
"""

import os
import sys
import time
from typing import List, Dict, Optional


# =====================================================================
# ЧАСТИНА 2. Ієрархія доменних виключень
# =====================================================================
class AppError(Exception):
    """Базовий клас для всіх доменних виключень застосунку."""

    def __init__(self, message: str, error_code: int = 1000):
        super().__init__(message)
        self.error_code = error_code

    def __str__(self) -> str:
        return f"[Код помилки {self.error_code}]: {super().__str__()}"


class ValidationError(AppError):
    """Виключення, що виникає при невідповідності вхідних даних чи форматів."""

    def __init__(self, message: str, field_name: str, error_code: int = 1001):
        super().__init__(message, error_code)
        self.field_name = field_name

    def __str__(self) -> str:
        return f"[ValidationError | Поле: '{self.field_name}'] {super().__str__()}"


class ResourceNotFoundError(AppError):
    """Виключення для випадків, коли файл, ресурс або запис не знайдено."""

    def __init__(self, message: str, resource_id: str, error_code: int = 1002):
        super().__init__(message, error_code)
        self.resource_id = resource_id

    def __str__(self) -> str:
        return f"[ResourceNotFoundError | Ресурс: '{self.resource_id}'] {super().__str__()}"


class BusinessLogicError(AppError):
    """Виключення для порушення бізнес-правил (наприклад, неможлива операція)."""

    def __init__(self, message: str, operation: str, error_code: int = 1003):
        super().__init__(message, error_code)
        self.operation = operation

    def __str__(self) -> str:
        return f"[BusinessLogicError | Операція: '{self.operation}'] {super().__str__()}"


# =====================================================================
# ЧАСТИНА 4. Утиліти та валідація (з анотаціями типів та docstrings)
# =====================================================================
def calculate_average_grade(grades: List[int]) -> float:
    """Обчислює середній арифметичний бал студента.

    Args:
        grades (List[int]): Список числових оцінок студента.

    Returns:
        float: Середнє значення оцінки, округлене до 2 знаків.

    Raises:
        ValueError: Якщо передано порожній список оцінок.
    """
    if not grades:
        raise ValueError("Список оцінок порожній — неможливо обчислити середнє.")
    return round(sum(grades) / len(grades), 2)


def find_student_record(database: Dict[str, int], student_name: str) -> Optional[int]:
    """Шукає оцінку студента у словнику-базі даних.

    Args:
        database (Dict[str, int]): Словник у форматі {ім'я: оцінка}.
        student_name (str): Ім'я шуканого студента.

    Returns:
        Optional[int]: Оцінка студента або None, якщо студента не знайдено.

    Raises:
        TypeError: Якщо введено некоректний тип бази даних.
    """
    if not isinstance(database, dict):
        raise TypeError("База даних має бути словником.")
    return database.get(student_name)


# =====================================================================
# ЧАСТИНА 1 & 2. Бізнес-логіка та обробка виключень
# =====================================================================
def process_numeric_input() -> None:
    """Сценарій 1: Базова конструкція try/except для обробки числових помилок."""
    print("\n--- [Сценарій 1] Базовий try/except (Числові операції) ---")
    try:
        raw_val = "100"
        divisor = 0
        result = int(raw_val) / divisor
        print(f"Результат: {result}")
    except (ValueError, ZeroDivisionError) as e:
        print(f"Перехоплено очікувану помилку (ValueError / ZeroDivisionError): {e}")


def process_file_data(file_path: str) -> None:
    """Сценарій 2: Повна конструкція try/except/else/finally з контекстним менеджером (with)."""
    print("\n--- [Сценарій 2] Повний try/except/else/finally та with ---")
    try:
        if not os.path.exists(file_path):
            raise FileNotFoundError(f"Файл '{file_path}' відсутній у системі.")
            
        with open(file_path, "r", encoding="utf-8") as f:
            content = f.read()
            
    except FileNotFoundError as fnf_err:
        print(f"[ERROR] Файл не знайдено: {fnf_err}")
    except PermissionError as perm_err:
        print(f"[ERROR] Недостатньо прав доступу: {perm_err}")
    else:
        print(f"[SUCCESS] Файл успішно прочитано. Довжина контенту: {len(content)} символів.")
    finally:
        print("[FINALLY] Завершення операції читання (ресурси закриті контекстним менеджером).")


def validate_student_grade(grade: int) -> bool:
    """Сценарій 3: Демонстрація ланцюжка виключень (raise ... from ...)."""
    print("\n--- [Сценарій 3] Ланцюжки виключень (raise ... from ...) ---")
    try:
        if not (0 <= grade <= 100):
            raise ValueError(f"Недопустиме значення оцінки: {grade}")
    except ValueError as orig_err:
        print(f"[LOG] Зафіксовано низькорівневу помилку: {orig_err}")
        # Ланцюжок виключень
        raise ValidationError(
            message="Оцінка має бути в межах від 0 до 100 балів.",
            field_name="grade"
        ) from orig_err
    return True


# =====================================================================
# ГОЛОВНИЙ БЛОК ВИКОНАННЯ ТА ІНТРОСПЕКЦІЇ
# =====================================================================
def main() -> None:
    print("=================================================================")
    print(" ЛАБОРАТОРНА РОБОТА №3 (Григор'євська, група КБ-25) — ОДИН ФАЙЛ")
    print("=================================================================")

    # 1. Завдання 1: Тестування сценаріїв обробки виключень
    process_numeric_input()
    
    # Хибний шлях (файл не існує)
    process_file_data("missing_config.json")
    
    # Успішний шлях (створення та читання тимчасового файлу)
    temp_filename = "test_config.txt"
    with open(temp_filename, "w", encoding="utf-8") as tmp:
        tmp.write("Settings: Enabled")
    process_file_data(temp_filename)
    if os.path.exists(temp_filename):
        os.remove(temp_filename)

    # 2. Завдання 2: Тестування доменних виключень і ланцюжків
    try:
        validate_student_grade(120)
    except ValidationError as ve:
        print(f"Перехоплено доменне виключення: {ve}")
        print(f"Первинна причина через __cause__: {ve.__cause__}")

    # 3. Завдання 4: Демонстрація інтроспекції
    print("\n--- [Завдання 4] Демонстрація інтроспекції коду ---")
    target_func = calculate_average_grade
    print(f"Назва функції (__name__): {target_func.__name__}")
    print(f"Анотації типів (__annotations__): {target_func.__annotations__}")
    print(f"Документація (__doc__):\n{target_func.__doc__}")


if __name__ == "__main__":
    main()
