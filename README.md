# WordMaster (ТЗ)

**Краткое описание:**
WordMaster — WPF-приложение для изучения иностранных слов. Пользователь может создавать словари, добавлять слова, проходить тесты, отслеживать прогресс и использовать систему повторений (SRS).


## Содержание


1. Архитектура проекта
2. Структура репозитория
3. Сущности (БД)
4. Формы (окна) и элементы UI
5. Навигация
6. Бизнес-логика и правила






## 2. Архитектура проекта

Проект разделён на 3 основных проекта внутри решения `WordMaster.sln`:

* `WordMaster.Domain` — классы сущностей, бизнес-логика (интерфейсы сервисов)
* `WordMaster.Data` — реализация доступа к данным 
* `WordMaster.UI` — WPF приложение, ViewModels, XAML-формы
* `WordMaster.Tests` — модульные тесты
* `WordMaster.Services` — внешние интеграции (если понадобится)


## 3. Сущности (таблицы БД)

### Word

* Id (int, PK)
* ForeignWord (string, required)
* Translation (string, required)
* PartOfSpeech (string?)
* Example (string?)
* SRSLevel (int) — уровень по SRS (0..n)
* NextReviewDate (DateTime?)
* CreatedAt (DateTime)

### Dictionary

* Id (int, PK)
* Name (string, required)
* Description (string?)

### DictionaryWord (many-to-many)

* DictionaryId (int, FK → Dictionary.Id)
* WordId (int, FK → Word.Id)

### TestResult

* Id (int, PK)
* Date (DateTime)
* TotalQuestions (int)
* CorrectAnswers (int)
* DurationSeconds (int?)

### Payment

* Id (int, PK)
* Tariff (string)
* CardLast4 (string?)
* Amount (decimal)
* CreatedAt (DateTime)
* Status (string) — Pending / Success / Failed


## 5. Формы (окна) и ViewModels

### MainWindow

**ViewModel:** MainViewModel
**Элементы:** DataGrid (список слов), ComboBox (фильтр словарей), кнопки: Add, Edit, Delete, Start Test, Dictionaries, Stats, Subscription.

### WordEditWindow

**ViewModel:** WordEditViewModel
**Поля:** ForeignWord (TextBox), Translation (TextBox), PartOfSpeech (ComboBox), Example (TextBox), DictionaryPicker (ListBox / MultiSelect), NextReviewDate (DatePicker), Save, Cancel.

### DictionaryEditWindow

**ViewModel:** DictionaryEditViewModel
**Поля:** Name, Description, Save, Cancel

### TestWindow

**ViewModel:** TestViewModel
**Поведение:** Последовательно показывает вопросы. Варианты ответов формируются (один правильный + 3 случайных). Поддержка разных режимов: Multiple Choice, Write (optional).

### StatisticsWindow

**ViewModel:** StatisticsViewModel
**Элементы:** таблица TestResults, summary (total words, learned words, accuracy), график прогресса (минимум — линейный график по дате теста).

### PaymentWindow

**ViewModel:** PaymentViewModel
**Поля:** Tariff (ComboBox), CardNumber (Masked), Expiry, CVV, Pay. (Реальная интеграция с платежными шлюзами не требуется — симуляция.)

---

## 6. Навигация

* Приложение запускается с `MainWindow`.
* Основная навигация реализуется через команды в MainViewModel, открывающие окна (`ShowDialog`) или меняющие текущую Page в Frame.
* Возврат к MainWindow после закрытия модальных окон.

---

## 7. Бизнес-логика и правила

* При добавлении слова: `CreatedAt = DateTime.UtcNow`, `SRSLevel = 0`.
* После теста: для каждого слова обновляется `SRSLevel` и `NextReviewDate` (алгоритм: простая версия SM-2 или ступенчатое увеличение интервала: +1, +3, +7, +30 дней и т.д.).
* Подписка: при успешной оплате записывается Payment и устанавливается флаг Premium в локальном профиле (в таблице Users/Settings при необходимости).
* Валидация: обязательные поля — ForeignWord, Translation.



