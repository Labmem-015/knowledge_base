# Вопросы
1) ~~Как вызвать код, внутри блока `__except`?~~
2) ~~Как вызвать код после блока `__except`?~~
3) ~~Почему сигнатура выражения-фильтра отличается от сигнатуры callback-функции обработчика исключений?~~
4) ~~Как выражение-фильтр вызывается операционной системой?~~
5) ~~Как выглядит структура `_ThrowInfo`, которую нужно подать в `__CxxThrowException`~~
6) ~~Почему при использовании try-catch мы имеем разные обработчики в exception registration?~~
7) ~~Как получить доступ к funcinfo изнутри обработчика помимо обращения к регистру eax?~~
8) Как вызвать деструкторы объектов в try блоке (unwinding)?
9) Как вызвать catch-блок?
10) Как работает state?
11) Как заменить `dispatch_context`?
12) Что делает атрибут `frame` в директиве `proc`? И что делает аргумент этого атрибута?
# Возможные ответы
1) Вызывается `lpfnHandler` из `scopetable` 
2) С помощью сохранённого ESP
3) Потому, что они имеют разное предназначение и фильтр вызывается из callback-функции `__except_handler3` по указателю в `scopetable`
4) Не вызывается. Вызывается callback-функция, а из неё выражение-фильтр. Все указатели на обработчики в связном списке `EXCEPTION_REGISTRATION` указывает на одну и ту же функцию, которая в RTL определена как `__except_handler3`.
5) Описание структуры см. далее
6) Возможно он генерит доп код, где передаёт разные экземпляры funcinfo в `__CxxFrameHandler3`
7) Видимо никак.
8) ва
9) ыва 
10) ыва
11) ыва
12) [Про /SAFESEHпш](https://learn.microsoft.com/en-us/cpp/build/reference/safeseh-image-has-safe-exception-handlers?view=msvc-170)

# Оглавление
* [[#CRT (Documentation) Frontier]]
	* [[#ThrowInfo]]
* [[#Reverse Engineering Frontier]]
	* [[#x32 reversing]]
	* [[#x64 reversing]]
* [[#KTL research frontier]]
	* [[#Martin Vejnar's capture.asm file]]
	* [[#throw.hpp and throw.cpp]]
	* [[#frame_handlers]]
	* [[#x86 error handling by Martin Vejnar's implementation]]
	* [[#Общие сведения об ассемблере]]
* [[#Structured Exception Handling (C/C++) Frontier]]
	* [[#EXCEPTION_REGISTRATION Список структур с указателями на обработчиков исключений]]
	* [[#__CxxFrameHandler3 Обработчик исключений]]
	* [[#Win32 API функции для SEH]]
	* [[#Функлеты и реализация исключений в Microsoft]]
* [[#SEH в Visual C]]
	* [[#Расширенный фрейм обработки исключений]]
	* [[#SCOPETABLE]]
	* [[#EXCEPTION_POINTERS]]
	* [[#Раскрутка]]
	* [[#Защита от переполнения]]
* [[#VC ++ Exceptions]]
	* [[#FUNCINFO структура]]
* [[#CONTEXT and DISPATCH_CONTEXT]]
	* [[#x86 CPU Context]]
	* [[#x64 CPU Context]]
	* [[#x64 DISPATCHER_CONTEXT]]
* [[#Источники]]
# CRT (Documentation) Frontier
[Анатомия C Run-Time, или Как сделать программу немного меньшего размера](https://rsdn.org/article/cpp/crt.xml)
Использование механизмов обработки исключений и RTTI (Run-Time Type Information) требует инициализации в стартовом коде CRT.
Для использования конструкции try-catch в x64 нужно определить функцию `__CxxFrameHandler4`, а для x32 `__CxxFrameHandler3`.
`_CxxThrowException` принимает указатель на объект исключения и указатель на throw info

## `ThrowInfo`
`ThrowInfo — информация, описывающая брошенный объект, статически построенная
на месте выброса.
Указатель функции `pForwardCompat` предназначен для заполнения будущими версиями, так что если, скажем, DLL, собранная с более новой версией (например, `C10`), выдает исключение, а кадр `C9` пытается выполнить перехват, обработчик кадра, пытающийся выполнить перехват (`C9`), может позволить версии, которая знает все последние данные, выполнить работу.
```cpp
typedef const struct _s_ThrowInfo {
	unsigned int attributes;	// Throw Info attributes (Bit field)
	PMFN pmfnUnwind;    // Destructor to call when exception has been handled or aborted
	int	(__cdecl * pForwardCompat)(...);	// Forward compatibility frame handler
	CatchableTypeArray* pCatchableTypeArray;// Pointer to list of pointers to types
} ThrowInfo;
```
`CatchableTypeArray` содержит RTTI о типах, которые можно перехватить.

Информация о выбрасываемом объекте разбита на три уровня, чтобы обеспечить максимальное свертывание `comdat` (за счет некоторых дополнительных указателей). `ThrowInfo` — это заголовок описания, содержащий информацию о конкретном выброшенном варианте исключения.
`CatchableTypeArray` — это массив указателей на дескрипторы типов. Он будет совместно использоваться объектами, выброшенными по ссылке, но с различными квалификаторами.
`CatchableType` — это описание отдельного типа и того, как выполнить преобразование из заданного типа.

---
# Reverse Engineering Frontier

Я создаю проект в Visual Studio 17 2022. Реализую простой пример выбрасывания исключений. Смотрю, на дизассемблированный код.

В прологе функции берётся эффективная ссылка на `..._common_exceptions@cpp` и записывается в rcx (регистр аргумента функции, которая будет вызвана?)

Порядок вызова функций:
1) Вызов `__CheckForDebuggerJustMyCode` с передачей `..._common_exceptions@cpp`
2) Создание объекта `runtime_error` (конструктор, принимающий `const char*`)
3) Перемещение объекта `runtime_error` по `rvalue` в функцию `_CxxThrowException` и её вызов 
4) Catch

Bruh

## x32 reversing
Я заметил, что в C++ исключениях в exception registration отличаются обработчики на каждом узле.
## x64 reversing
Context Frame используется для создания фрейма для доставки вызова асинхронного вызова процедуры (APC). 

---
# KTL research frontier
## Martin Vejnar's capture.asm file:
У нас есть 4 функции, которые мы должны реализовать:
`__cxx_dispatch_exception`
`__cxx_destroy_exception`
`__cxx_seh_frame_handler`
`__cxx_call_catch_frame_handler`

Говорится, что кадр стека `_CxxThrowException` всегда содержит следующую структуру (может это описание аргумента функции?). Ну или содержание кадра стека этой функции всегда соответствует этой структуре:
```masm
catch_info_t struct
    cont_addr_0              qword ?
    cont_addr_1              qword ?
    primary_frame_ptr        qword ?
    exception_object_or_link qword ?
    throw_info_if_owner      qword ?
    unwind_context           qword ?
catch_info_t ends
```
В этой структуре мы можем хранить ссылку на объект исключения, текущий кадр стека и контекст для размотки стека. Вообще как я понял, мы при размотке меняем обработчики, следовательно убирая кадровые стеки и вставляя их обратно. Контекст позволяет сохранить некоторые данные и менять не-волатильные регистры функлета. Bruh, прочитай про [О передаче управления при вызове исключения](https://habr.com/ru/articles/267771/).

Аналогично и с `throw_fr` и `catch_fr`.

В этой реализации мы храним только одну копию контекста во время разматывания, причём только его неизменную часть. Как только мы находим обработчик, мы освобождаем контекст из стека. Процесс разматывания осуществляется в один проход, без использования локального хранилища потока.

По сравнению с SEH, реализация запрещает совмещение C++ исключений и SEH исключений. Также We do not support throwing across module boundaries. Ещё дебагер ничего не увидит. Ну и ещё если вызывается исключение при копирование объекта исключения, то мы нарушаем стандарт и позволяем исключениями распространяться дальше, не вызывая std::terminate.

## `throw.hpp and throw.cpp`
Пока не разобрал

## `frame_handlers`
`DISPATCH_CONTEXT`, там юзается `image_base`, `fn->unwind_info.value()`, `cookie`, `throw_frame`, `pdata->image_base`, `extra_data`, `fn->begin`, `pdata`, `handler`, `fn`.

## x86 error handling by Martin Vejnar's implementation
Как только мы вызываем оператор `throw`, компилятор подставляет вызов функции `_CxxThrowException`, с передачей указателями на объект исключения и структуру `ThrowInfo`.
```cpp
extern "C" void __stdcall
_CxxThrowException( void* pExceptionObject, _ThrowInfo* pThrowInfo );
```
* Найти ближайший кадр вызова функции с обработчиком `catch`, который соответствует типу объекта исключения;
* развернуть все кадры выше него в стеке;
* частично развернуть целевой кадр, чтобы локальные переменные в блоке `try`, который поймал исключение, были уничтожены;
* Вызвать конструктор копирования для объекта исключения в `catch`, или, если переменная имеет ссылочный тип, привязать ее к объекту исключения;
* вызвать функцию обработчика `catch`;
* уничтожить объект исключения;
* возобновить выполнение по адресу, возвращенному функцией обработчика `catch`.
Порядок деструкции объектов:
1) локальные переменные в `catch`;
2) объект исключения (который во владении текущего фрейма);
3) остальные локальные переменные.
## Общие сведения об ассемблере
Адрес растёт вверх. Стек растёт вниз. База стека находится на старших адресах. Вершина стека на нижних адресах.
`qword` - (quad word) означает 8-ми байтовое слово.
`?` - означает неинициализированное значение.
`.allocstack` - используется для указания того, как выполнять размотку функции. Используется только в прологе функции.
`$` - помогает различить регистр от переменной. Использование не обязательно.
`.code` - указывает MASM сгруппировать операторы, следующие за ней, в специальный раздел в памяти, зарезервированный для машинных инструкций.
Нам нужно учитывать, что при вызове функций WinAPI нужно иметь стек, кратный 16. Ещё выделить 32 байта на shadow storage. И только потом на параметры, начиная с 5-го (первые четыре подаются через регистр). Ещё при вызове функции в стек помещается адрес возврата размером в 8 байт.

---
# Structured Exception Handling (C/C++) Frontier
[Основной Источник](https://www.codeproject.com/KB/cpp/exceptionhandler.aspx)
[Win32 SEH изнутри](https://qxov.narod.ru/articles/seh/seh.html)
[MSDN](https://learn.microsoft.com/en-us/cpp/cpp/structured-exception-handling-c-cpp?view=msvc-170)
[Руководство YASM (см. главы 15.2 и 16.2)](https://www.tortall.net/projects/yasm/manual/html/index.html)
SEH - это расширение языка от Microsoft для обработки исключений. Благодаря SEH мы можем убедиться, что корректно отпускаем все ресурсы (блоки памяти, файлы) и правильно деаллоцируем память, если возникло исключение.

Когда происходит исключение, система обходит список структур **EXCEPTION_REGISTRATION**, до тех пор пока не находит обработчик исключения. Как только он найдётся, ОС снова обходит список, до узла, который будет обрабатывать исключение. (**Примечание переводчика**: _это утверждение может ввести вас в заблуждение! Лично у меня, после его прочтения, сложилось впечатление, что процессом раскрутки полностью управляет ОС. На самом деле, это не так. Процесс раскрутки инициируется обработчиком, взявшимся за обработку исключения. Более того, для проведения раскрутки, обработчику могут вообще не понадобиться услуги ОС, т.к. у него может иметься собственный код для ее реализации_.) Во время этого второго прохода ОС вызывает каждый обработчик ещё раз, но на этот раз значение флагов исключения равно 2, что соответствует **EH_UNWINDING** (определение **EH_UNWINDING** есть в файле **EXCEPT.INC**, который находится в исходниках **RTL Visual C++**, но в **Win32 SDK** его эквивалента нет).

У нас есть два механизма SEH:
* Обработчики исключений (try-except)
* Обработчики завершения (try-finally)
Во время выбрасывания исключения, ОС Windows ищет ближайший активный обработчик исключений. Есть три варианта, как обработчик себя поведёт: не распознает исключение и передаст следующему обработчику (EXCEPTION_CONTINUE_SEARCH), распознает и отклонит исключение (EXCEPTION_CONTINUE_EXECUTION), распознает и обработает (EXCEPTION_EXECUTE_HANDLER). 
Разматывание стека (unwinding of stack) - процесс терминирования и очищения всего того, что было в стеке до места обработки исключения.

Если отключить исключения в проекте и убрать default libs (/NODEFAULTLIB), то нам потребуется имплементировать функцию `__C_specific_handler`. Он находится в заголовочном файле Wdm.h (библиотека NtosKrnl.lib).

Когда вызывается исключение, управление передаётся операционной системе. Она вызывает обработчик исключений. Обработчик исключений проверяет последовательность вызова, начиная с текущей функции, при этом выполняя разматывание стека, и в конце передаёт управление программе (на том месте, где исключение было поймано и обработано?).

```cpp
__try
{
  // guarded body of code
}
__except (filter_expression)
{
  // exception handler block
}

__try
{
  // guarded body of code
}
__finally
{
  // termination handler block
}
```
## `EXCEPTION_REGISTRATION` Список структур с указателями на обработчиков исключений
Чтобы использовать своё исключение, его надо зарегистрировать. Создаём структуру с указателем на обработчик событий и предыдущую такую же структуру. Таким образом можно создавать цепочку обработчиков. Адрес последней структуры с актуальным обработчиком доступен через регистр `FS:[0]`. Вызов обработчиков мы начинаем с начала списка, пока кто-либо не вернёт значение `ExceptionContinueExecution(0)`.  Конец списка находится наверху на старших адресах. Для последнего элемента в списке, `prev` будет указывать на 0xFFFFFFFF.
```cpp
struct MY_EXCEPTION_REGISTRATION {
	MY_EXCEPTION_REGISTRATION* prev;
	DWORD handler;
};
```
Кладём указатель на нашу структуру в сегментный регистр (`FS:[0]`?). Он должна располагаться в стеке потока, который вызвал исключение с выравниванием адреса структуры и её компонентов на адрес, кратный 4 (биты 0 и 1 адреса равны нулю).

Эта список регистрируется в блоке информации потока (Thread Information Block, TIB). На платформе Intel Win32 регистр `FS` всегда указывает на TIB текущего потока, а `FS:[0]` указывает на этот список callback-функций (обработчиков исключений). ОС вставляет в конец списка дефолтный обработчик.
На начало этого списка всегда указывает первый **DWORD** в **TIB** (**`FS:[0]`** в машинах на основе **Intel**).

Эта структура соотносится с блоком `__try` и говорят, что код внутри блока `__try` защищён структурой `EXCEPTION_REGISTRATION`.

## __CxxFrameHandler3 Обработчик исключений
Сам обработчик событий представлен call-back функцией. Он не должен располагаться в стеке потока, который вызвал исключение. Обработчик имеет следующую сигнатуру из `excpt.h`:
```cpp
_VCRTIMP EXCEPTION_DISPOSITION __cdecl __C_specific_handler(
	_In_    struct _EXCEPTION_RECORD*   ExceptionRecord,
	_In_    void*                       EstablisherFrame,
	_Inout_ struct _CONTEXT*            ContextRecord,
	_Inout_ struct _DISPATCHER_CONTEXT* DispatcherContext
);
```
`_EXCEPTION_RECORD*` содержит информацию об исключении.
`void* EstablisherFrame` адрес структуры EXCEPTION_REGISTRATION в стеке потока, вызвавшего исключение.
`_CONTEXT*`  адрес структуры содержащей все регистры процессора в момент возникновения исключения.
`_DISPATCHER_CONTEXT*` служебная информация.

Обычно структура `EXCEPTION_REGISTRATION` создаётся в прологе функции (перед базой стека функции, т.е. сначала идут параметры функции, потом адрес возврата управления, потом сама структура и затем начало кадра стека функции с локальной областью видимости)

Как только нужный (заинтересованный в обработке) обработчик исключений начинает свою работу, он делегирует очистку кадра стека обработчику, ассоциированному с этим кадром. Начинается с `FS:[0]`. Тем самым мы очищаем все кадры стека по цепочке вызовов функций до места, где исключение было поймано.

Необходимо активировать кадр стека, где исключение должно быть поймано, прежде чем произойдёт перехват. Перед вызовом блока-перехвата объект исключения должен быть скопирован в кадр стека этого блока. Он возвращает адрес того места, где нужно продолжить выполнение функции. Ещё нужно не забыть очистить кадр стека блока перехвата.

Если обработчик решает, что исключение должно быть обработано, то он:
* выполняет размотку
* вызывает вызывает except блок
* очищает стек и устанавливает указатели на кадры
* передаёт управления обратно пользовательскому коду сразу после except блока.

Во время размотки исключения, C++ SEH Exception Callback Function обходит связный список обработчиков, пока не найдёт нужный. Затем ещё раз, но уже с флагом `EXCEPTION_UNWINDING`. Во время второго прохода, обработчик вызывает пользовательский код (финальный блок), чтобы выполнить очистку ресурсов.

По терминологии Win32, callback-функция обработчика исключений соответствует коду выражения-фильтра. Возвращаемое значение указывает операционной системе, что делать дальше. Будет ли исключение обработано в `__except`-блоке. `__try` добавляет в начало списка `EXCEPTION_REGISTRATION`, а `__except` добавляет выражение-фильтр в качестве callback-функции в `EXCEPTION_REGISTRATION`.

Код выражения-фильтра вызывается из callback-функции `__except_handler3`. Код выражения-фильтра не вызывается из операционной системы. Все поля exception handler в списке обработчиков указывает на одну и ту же функцию, которая в RTL определена как `__except_handler3`.

## Возможная реализация `__except_handler3`
Ветвление на режим размотки и режим обработки.
Режим обработки. Сохраняем значения из аргумента функции в расширенный SEH фрейм. Затем производим обход всех SEH фреймов (всех функций содержащих блок обработки исключений try-except), а также всех scopetable в пределах каждой функции

## Win32 API функции для SEH
* `GetExceptionCode()` - интринсик, который возвращает код, идентифицирующий исключение. Может быть вызван только из `filter_expression` или `exception handler block`
* `GetExceptionInformation()` - интринсик, который возвращает указатель на `ExceptionRecord` и указатель на структуру контекста исключения. Вызывается только из `filter_expression`
* `AbnormalTermination()` - указывает, завершился ли try-блок обработчика завершения нормально. Может быть вызван только из обработчика завершения.
* `RaiseException()` - генерирует исключение в вызываемом потоке (синхронное программное исключение)
`_CxxThrowException` передаёт управление операционной системе через программное прерывание (см. `RaiseException`). Её аргументы передаются заносятся в `EXCEPTION_RECORD`.
## Функлеты и реализация исключений в Microsoft
Функлет - это кусок кода, который мы можем вызвать, и который создаёт свой кадр стека. При компиляции для каждой функции создаётся первичный функлет и catch-функлеты. Чтобы catch-блок функлеты имели доступ к скопу функции, где блок сам и находится, мы в кадре стека каждого catch функлета добавляем указатель на кадр стека первичного функлета, где располагаются все переменные функции. Этот указатель на AMD64 передаётся в catch функлет через регистр rdx. Microsoft передаёт указатель на сам функлет через второй аргумент, а именно регистр rcx. Объект исключения живёт в кадре первичного функлета.

В Microsoft CRT функция `_CxxThrowExpression` вызывает `RaiseException`, который проходит по стеку и ищет нужный обработчик. После того, как он найдёт нужный обработчик, он начинает размотку стека (очистку) путём указания флага `EXCEPTION_UNWIND`. То есть обработка исключений идёт в две фазы.

Каждый обработчик из-за того, что содержит копии контекстов CPU, может занимать 8КБ пространства. Так что, gg. Удачи юзать в kernel-моде.

Зарегистрированный нами обработчик вызывается операционной системой по обращению к регистру `FS:[0]`. Обработчик возвращает значение, по которому ОС решает, как действовать дальше.

# SEH в Visual C
Нас надули! ОБМАНУЛИ! 
> Другое искажение, сделанное мной для облегчения восприятия, заключается в том, что структура **`EXCEPTION_REGISTRATION`** не создаётся и не разрушается каждый раз при входе и выходе из любого **`_try`**-блока. Вместо этого, в каждой функции использующей **`SEH`** создается только один **`EXCEPTION_REGISTRATION`**. Другими словами, вы можете иметь множество **`_try/_except`** конструкций в одной функции, но в стеке будет создан только один **`EXCEPTION_REGISTRATION`**. Аналогично, вы могли бы сделать в функции вложенные блоки **`_try`**. Однако, при этом, **`Visual C++`** создаст только один **`EXCEPTION_REGISTRATION`**. Но это всё в рамках одной функции. В цепочке вызовов функций, `EXCEPTION_REGISTRATION` столько же, сколько  вызовов этих функций.
> 
> Если единственный обработчик исключений (это упоминавшийся ранее **`__except_handler3`**) покрывает весь **`EXE`** или **`DLL`**, и если единственный **`EXCEPTION_REGISTRATION`** разделяет множество **`_try`**-блоков, то там очевидно делается более сложная работа, чем кажется на первый взгляд. Этот фокус реализован с помощью таблицы данных, которую вы обычно не видите. Однако, так как общая цель этой статьи состоит в том, чтобы разобрать **`SEH`** по винтикам, давайте взглянем, как это делается.
## Расширенный фрейм обработки исключений
Расширенная структура позволяет единственной `__except_handler3` обработать все исключения в программе и управлять вызовом соответствующих выражений-фильтров и `_except`-блоков. Тут 3 новых поля: `scopetable`, `trylevel` и `_ebp`
```cpp
struct _EXCEPTION_REGISTRATION{
    struct _EXCEPTION_REGISTRATION *prev;
    void (*handler)(PEXCEPTION_RECORD,
                    PEXCEPTION_REGISTRATION,
                    PCONTEXT,
                    PEXCEPTION_RECORD);
    struct scopetable_entry *scopetable;
    int trylevel;
    int _ebp;
};
```
После `EXCEPTION_REGISTRATION` в стек кладётся `DWORD` указатель на `EXCEPTION_POINTERS`. Мы можем получить её с помощью `GetExceptionInformation`, которая раскрывается компилятором в : `MOV EAX,DWORD PTR [EBP-14]`. Аналогично и с `GetExeptionCode`, которая находит и возвращает значение поля, принадлежащего одной из структур данных, возвращённых `GetExceptionInformation`. Короче он возвращает первое поле структуры `_EXCEPTION_RECORD`, а именно `ExceptionCode`.
Вторым в стек кладётся ESP

стандартный **`SEH`**-фрейм, который создаёт **`Visual C++`** для функции, использующей **`SEH`**:

>  EBP-00 `_ebp`
>  EBP-04 `trylevel`
>  EBP-08 указатель на таблицу `scopetable`
>  EBP-0C адрес функции обработчика
>  EBP-10 Указатель на предыдущую `EXCEPTION_REGISTRATION`
>  EBP-14 `PEXCEPTION_POINTERS`
>  EBP-18 Обычное значение регистра `ESP` в фрейме

С точки зрения ОС, существуют только два поля, из которых состоит базовая структура **`EXCEPTION_REGISTRATION`**: указатель `prev` в **`[EBP-10]`** и указатель функцию обработчика в **`[EBP-0Ch]`**. Остальное содержимое фрейма является специфическим расширением **`Visual C++`**.
## `SCOPETABLE`
```cpp
typedef struct _SCOPETABLE
 {
     DWORD       previousTryLevel;
     DWORD       lpfnFilter;    // выражение-фильтр
     DWORD       lpfnHandler;   // _except-обработчик
} SCOPETABLE, *PSCOPETABLE;
```
## `EXCEPTION_POINTERS`
```cpp
typedef struct _EXCEPTION_POINTERS {
	PEXCEPTION_RECORD ExceptionRecord;
	PCONTEXT ContextRecord;
} EXCEPTION_POINTERS, *PEXCEPTION_POINTERS;

// Exception info
typedef struct _EXCEPTION_RECORD {
  DWORD                    ExceptionCode;
  DWORD                    ExceptionFlags;
  struct _EXCEPTION_RECORD *ExceptionRecord;
  PVOID                    ExceptionAddress;
  DWORD                    NumberParameters;
  ULONG_PTR                ExceptionInformation[EXCEPTION_MAXIMUM_PARAMETERS];
} EXCEPTION_RECORD;

// All registers
typedef struct _CONTEXT {
  DWORD64 P1Home;
  DWORD64 P2Home;
  // ...
  DWORD64 Rax;
  DWORD64 Rcx;
  DWORD64 Rdx;
  DWORD64 Rbx;
  DWORD64 Rsp;
  DWORD64 Rbp;
  // ...
} CONTEXT, *PCONTEXT;
```

## Раскрутка

## Защита от переполнения
`GSCookieOffset` = -2 означает, что GS cookie не используется. EH cookie всегда присутствует. Смещения являются относительными к `ebp`. Проверка выполняется следующим образом: `(ebp + CookieXOROffset ) ^ [ebp + CookieOffset] == ​​_security_cookie`. Указатель на `scopetable` в стеке также подвергается операции XOR с `_security_cookie`. Кроме того, в SEH4 самый внешний уровень области действия равен -2, а не -1, как в SEH3
[Источник](https://www.openrce.org/articles/full_view/21)

# VC ++ Exceptions
![[Pasted image 20250305182726.png]]
У нас такая структура в vc++:
```cpp
struct vc_ExceptionRegistration {
	void* stack_pointer;
	ExceptionRegistration* registration;
	int state;    // use it to determine, which try block to search
};
```
Операторы `throw` преобразуются в вызовы `_CxxThrowException()`, которые фактически вызывают исключение Win32 (SEH) с кодом `0xE06D7363` `('msc'|0xE0000000)`. Пользовательские параметры исключения Win32 включают указатели на объект исключения и его структуру `ThrowInfo`, используя которые обработчик исключений может сопоставить тип выданного исключения с типами, ожидаемыми обработчиками catch.
## `FUNCINFO` структура
Где находится funcinfo?
```cpp
struct funcinfo{
	DWORD magic_signature;
	DWORD unwind_entries_num;
	void* unwind_table;       // array
	DWORD tryblock_num;
	void* tryblock_table;     // array
};

struct UnwindTableEntry {
	unsigned long to_state;     // target state
	void* action;               // action to perform (unwind funclet address)
};

struct tryblock_table {
	// states ranging from tryLow to tryHigh
	DWORD start_id;  // tryLow
	DWORD end_id;    // tryHigh
	DWORD catchblock_index;  // highes state inside catch handlers of this try
	DWORD catchblock_num;
	void* catchblock_table;   // array
};

struct catchblock {
	BOOL copy_exception_by_val_or_ref;
	void* type_info;
	DWORD offset;
	void* catchblock_addr;
};
```
`offset` в `catchblock` указывает на отступ от `frame_pointer` кадра catch-блока, куда мы должны скопировать исключение (по значению или ссылке).
Мы ищем нужное нам вхождение массива `tryblock_table`, используя значение `state`, переданное нам из `vc_ExceptionRegistration`. Если оно равно или находится в диапазоне `start_id` и `end_id`. (ох если бы...)
Компилятор в момент вхождения в блок try помещает в функцию инструкцию, которая записывает в кадр стека значение `start_id`. (как это всё-таки работает?!)
Я выяснил, что `state` в `EXCEPTINO_REGISTRATION` - соответствует текущему индексу блока, в котором исключение было вызвано. В подсчёт индекса входят предшествующие вхождения try-catch. Не понимаю, чем является `start_id` и `end_id` , почему-то они одинаковые и коррелируют с максимальной глубиной try-блоков.
## `ThrowInfo` и `type_info`
В `EXCEPTION_RECORD` хранится указатель на `ThrowInfo`, которую можно сравнить с `type_info` в `catchblock`.

# `CONTEXT` and `DISPATCH_CONTEXT`
> [!DeepSeek's answer]
> `CONTEXT` - хранит полное состояние процессора (регистры) на момент возникновения исключения или прерывания.
> `DISPATCH_CONTEXT` - содержит метаданные для раскрутки стека и поиска подходящего обработчика исключений в цепочке SEH. Используется системой для управления процессом диспетчеризации исключений. В частности, можно получить funcinfo через handler_data
## x86 CPU Context
```cpp
struct CONTEXT {
    //
    // The flags values within this flag control the contents of
    // a CONTEXT record.
    //
    // If the context record is used as an input parameter, then
    // for each portion of the context record controlled by a flag
    // whose value is set, it is assumed that that portion of the
    // context record contains valid context. If the context record
    // is being used to modify a threads context, then only that
    // portion of the threads context will be modified.
    //
    // If the context record is used as an IN OUT parameter to capture
    // the context of a thread, then only those portions of the thread's
    // context corresponding to set flags will be returned.
    //
    // The context record is never used as an OUT only parameter.
    //

    DWORD ContextFlags;

    //
    // This section is specified/returned if CONTEXT_DEBUG_REGISTERS is
    // set in ContextFlags.  Note that CONTEXT_DEBUG_REGISTERS is NOT
    // included in CONTEXT_FULL.
    //

    DWORD   Dr0;
    DWORD   Dr1;
    DWORD   Dr2;
    DWORD   Dr3;
    DWORD   Dr6;
    DWORD   Dr7;

    //
    // This section is specified/returned if the
    // ContextFlags word contians the flag CONTEXT_FLOATING_POINT.
    //

    FLOATING_SAVE_AREA FloatSave;

    //
    // This section is specified/returned if the
    // ContextFlags word contians the flag CONTEXT_SEGMENTS.
    //

    DWORD   SegGs;
    DWORD   SegFs;
    DWORD   SegEs;
    DWORD   SegDs;

    //
    // This section is specified/returned if the
    // ContextFlags word contians the flag CONTEXT_INTEGER.
    //

    DWORD   Edi;
    DWORD   Esi;
    DWORD   Ebx;
    DWORD   Edx;
    DWORD   Ecx;
    DWORD   Eax;

    //
    // This section is specified/returned if the
    // ContextFlags word contians the flag CONTEXT_CONTROL.
    //

    DWORD   Ebp;
    DWORD   Eip;
    DWORD   SegCs;              // MUST BE SANITIZED
    DWORD   EFlags;             // MUST BE SANITIZED
    DWORD   Esp;
    DWORD   SegSs;

    //
    // This section is specified/returned if the ContextFlags word
    // contains the flag CONTEXT_EXTENDED_REGISTERS.
    // The format and contexts are processor specific
    //

    BYTE    ExtendedRegisters[MAXIMUM_SUPPORTED_EXTENSION];
}
```

## x64 CPU Context
```cpp
struct CONTEXT {
    //
    // Register parameter home addresses.
    //
    // N.B. These fields are for convience - they could be used to extend the
    //      context record in the future.
    //

    DWORD64 P1Home;
    DWORD64 P2Home;
    DWORD64 P3Home;
    DWORD64 P4Home;
    DWORD64 P5Home;
    DWORD64 P6Home;

    //
    // Control flags.
    //

    DWORD ContextFlags;
    DWORD MxCsr;

    //
    // Segment Registers and processor flags.
    //

    WORD   SegCs;
    WORD   SegDs;
    WORD   SegEs;
    WORD   SegFs;
    WORD   SegGs;
    WORD   SegSs;
    DWORD EFlags;

    //
    // Debug registers
    //

    DWORD64 Dr0;
    DWORD64 Dr1;
    DWORD64 Dr2;
    DWORD64 Dr3;
    DWORD64 Dr6;
    DWORD64 Dr7;

    //
    // Integer registers.
    //

    DWORD64 Rax;
    DWORD64 Rcx;
    DWORD64 Rdx;
    DWORD64 Rbx;
    DWORD64 Rsp;
    DWORD64 Rbp;
    DWORD64 Rsi;
    DWORD64 Rdi;
    DWORD64 R8;
    DWORD64 R9;
    DWORD64 R10;
    DWORD64 R11;
    DWORD64 R12;
    DWORD64 R13;
    DWORD64 R14;
    DWORD64 R15;

    //
    // Program counter.
    //

    DWORD64 Rip;

    //
    // Floating point state.
    //

    union {
        XMM_SAVE_AREA32 FltSave;
        struct {
            M128A Header[2];
            M128A Legacy[8];
            M128A Xmm0;
            M128A Xmm1;
            M128A Xmm2;
            M128A Xmm3;
            M128A Xmm4;
            M128A Xmm5;
            M128A Xmm6;
            M128A Xmm7;
            M128A Xmm8;
            M128A Xmm9;
            M128A Xmm10;
            M128A Xmm11;
            M128A Xmm12;
            M128A Xmm13;
            M128A Xmm14;
            M128A Xmm15;
        } DUMMYSTRUCTNAME;
    } DUMMYUNIONNAME;

    //
    // Vector registers.
    //

    M128A VectorRegister[26];
    DWORD64 VectorControl;

    //
    // Special debug control registers.
    //

    DWORD64 DebugControl;
    DWORD64 LastBranchToRip;
    DWORD64 LastBranchFromRip;
    DWORD64 LastExceptionToRip;
    DWORD64 LastExceptionFromRip;
}
```

---
## x64 `DISPATCHER_CONTEXT`
> [!Attention!]
> Сама структура взята из `<winnt.h>`, но комментарии добавлены нейросетью DeepSeek
```cpp
struct DISPATCHER_CONTEXT {
    DWORD64 ControlPc;    // Адрес возникновения исключения
    DWORD64 ImageBase;    // Базовый адрес модуля (exe/dll)
    PRUNTIME_FUNCTION FunctionEntry;   // Данные о функции (PDB, unwind-информация)
    DWORD64 EstablisherFrame; // Указатель на фрейм обработчика 
                              // (EXCEPTION_REGISTRATION?)
    DWORD64 TargetIp;     // Целевой IP для продолжения выполнения
    PCONTEXT ContextRecord;   // Ссылка на структуру CONTEXT
    PEXCEPTION_ROUTINE LanguageHandler;
    PVOID HandlerData;     
    struct _UNWIND_HISTORY_TABLE *HistoryTable;
    DWORD ScopeIndex;
    DWORD Fill0;
};
```
# Источники
[Видео с CppCon про размотку стека в механизме исключений](https://youtu.be/COEv2kq_Ht8?si=PwXzUZCQI7o6-hPY)
[Презентация к видео CppCon](https://github.com/CppCon/CppCon2018/blob/master/Presentations/unwinding_the_stack_exploring_how_cpp_exceptions_work_on_windows/unwinding_the_stack_exploring_how_cpp_exceptions_work_on_windows__james_mcnellis__cppcon_2018.pdf)
[Красивые схемы кадров стека](https://www.openrce.org/articles/full_view/21)
[SEH и C++ Exceptions на английском](https://www.codeproject.com/KB/cpp/exceptionhandler.aspx)
[Win32 SEH изнутри](https://qxov.narod.ru/articles/seh/seh.html)
[Про SEH на MSDN](https://learn.microsoft.com/en-us/cpp/cpp/structured-exception-handling-c-cpp?view=msvc-170)
[Про SEH из Руководства по YASM (см. главы 15.2 и 16.2)](https://www.tortall.net/projects/yasm/manual/html/index.html)
[О передаче управления при вызове исключения](https://habr.com/ru/articles/267771/).
[(Немного про CRT) Анатомия C Run-Time, или Как сделать программу немного меньшего размера](https://rsdn.org/article/cpp/crt.xml)

---
Throw
Catch
Dispatch
destory
unwind
frame handler
catch frame handler


`throw` => `RaiseException`
`try`/`catch` => `__try`/`__except`
Local variable destruction => `__try`/`__finally`
`_CxxFrameHandler` => `_except_handler3`