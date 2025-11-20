[[Start Page|На главную]]

---
# Оглавление
```table-of-contents
```

---
# Что такое патчгард и как он работает
Механизм ОС Windows для защиты ядра от изменений. Он может проверять:
- IDT/GDT
- Debug routines
- Loaded module list
- PathGuard code and structure itself
- etc.
Основная идея в том, что patchguard регулярно проверяет контрольную сумму важных структур во время исполнения системы. Он будет сравнивать эту сумму с той, которую получил на этапе загрузки ОС перед загрузкой любого пользовательского драйвера. Как только он обнаружит модификацию, то сработает BSOD с кодом `0x109` (`CRITICAL_STRUCTURE_CORRUPTION`).

---
# Инициализация
## Вызов `KiFilterFiberContext`
`KiFilterFiberContext` - это функция, которая отвечает за инициализацию patchguard и названа так, чтобы сбить столку аналитиков.
Есть два способа инициализации:
- Exception in `KiAmd64SpecificState`. Инициализация внутри обработчика исключений, который срабатывает при делении на ноль в функции `KiAmd64SpecificState`. Если подключен отладчик, то деления на ноль не происходит.
- `ExpLicenseWatchInitWorker`. Вызов происходит в функции, которая отвечает за верификацию лицензии. Инициализация патчгарда происходит с вероятностью в 4%. Это не шутка. В функцию `KiFilterFiberContext` подаётся параметр.
Во втором случае он вызывает `KiLockServiceTable`, которая заполняет `prcb.HalReserveed` поля `[5]` и `[6]`. В `[6]` хранится структура `KiServiceTablesLocked`, которая выступает параметром для `KiFilterFiberContext`, которая находится в `[5]`.

## Обзор `KiFilterFiberContext`

## Контекст PatchGuard


---
# Триггер события

---
# Рутина верификации

---
# Отключение PatchGuard

---
# Notes
- PG проверяет постоянно контрольные суммы структур на основе записанных при загрузке ОС значений.
- Основные направления обхода PG: Direct Kernel Object Manipulation (DKOM), SSDT hooking using indirect modifications, race condition, Zero-Day эксплойты, Kernel Code injection.