# Начало работы
## Linux
### Pyenv для управления версиями
Для выбора версии можно использовать `pyenv`. Чтобы его скачать, нужно выполнить эту команду:
```bash
curl -fsSL https://pyenv.run | bash
```
Далее необходимо настроить shell для работы с `pyenv`. Нужно обратить внимание, что pyenv, может быть не добавлен в PATH, поэтому стоит поискать бинарник в папке `~/.pyenv/bin`. Ещё нюанс для Bash: не стоит использовать автоматическую установку, если `BASH_ENV` ссылается на `.bashrc` файл. Команда автоматической установки:
```
pyenv init --install [shell]
```
По идее он всё настроит сам, но если будут проблемы, то следует ссылаться на [официальную страницу проекта](https://github.com/pyenv/pyenv).
Далее можно скачать версию Python:
```bash
pyenv install <py_version>
```
И вывести список версий:
```bash
pyenv versions
```
Можно выбрать версию для данной директории:
```bash
pyenv shell <py_version> # Set version for the current shell session
pyenv local <py_version> # Set version for the current dir
pyenv global <py_version> # Set version globally
```
Вывести текущую версию можно:
```bash
pyenv version
```
### Создание виртуальной среды в Linux
После установки pyenv можно создать виртуальную среду следующей командой:
```bash
# First select version:
pyenv local <py_version>
# Then create venv
python -m venv <venv_name>
##################
# Via pyenv-virtualenv plugin
yay -S pyenv-virtualenv
# Configuring pyenv-virtualenv
echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bashrc
source ~/.bashrc
# Create venv
pyenv virtualenv <py_version> <venv_name>
# ??? pyenv exec <py_version> -m venv <venv_name>
```
Активируем среду:
```cmd
source ./<venv_name>/bin/activate
```
Профит.
## Создание виртуальной среды в Windows
Чтобы управлять версиями питона, можно использовать утилиту `py`.  Список установленных версий выводится командой `py --list`. Сами версии устанавливаются в систему через обычной инсталлятор с сайта проекта.
Создаём виртуальную среду с указанием конкретной версии интерпретатора:
```
py -<py_version> -m venv <venv_name>
```
Активируем среду:
```cmd
./<venv_name>/Scripts/activate
```
Профит.
## Настройка VS Code
Надо установить следующие расширения:
- Python
- PyLance
- PyLint
Рекомендуемые параметры в `.vscode/settings.json`:
```json
{
    "python.analysis.inlayHints.functionReturnTypes": true,
    "python.analysis.inlayHints.pytestParameters": true,
    "python.analysis.typeCheckingMode": "basic"
}
```

---
# Декораторы
Декораторы - программные вставки, которые дополняют или изменяют поведение функции.

---
`tasks.py` позволяет автоматизировать рутинные задачи сборки, тестирования и форматирования кода. Она является альтернативой Makefile для Python-проектов. Можно выполнять shell-команды.

`setup.py` - устаревающий скрипт, который используется для упаковки и распространения пакетов python в формате `.tar.gz` или `.whl`. через пакетный регистр `PyPI` или локальную установку. Формирует метаданные проекта

`pyproject.toml` - более современный стандарт по описанию проекта. Он заменяет `setup.py` в плане описания метаданных проекта и сборки

`py.typed` - обычно пустой файл в корне проекта, сигнализирующий о том, что проект содержит информацию о типах (Type Hints) и может быть использован инструментами статического анализа типов (например `mypy`).
