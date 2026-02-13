[[Start Page|На главную]]

---
lsof (List Open Files) — отображает информацию об открытых файлах, сетевых соединениях, сокетах и процессах, которые их используют.

```bash
lsof -p <PID>
```

Просмотр сетевых адаптеров Windows:
```run
ncpa.cpl
```

Быстро создать файл [StackOverflow](https://stackoverflow.com/questions/982659/quickly-create-large-file-on-a-windows-system):
```cmd
fsutil file createnew <filename> <length>
```

msvcmon parameters
```
msvcmon.exe /installed /nofirewallwarn /timeout 86400 /noauth /anyuser /nosecuritywarn
```

Проброс туннеля на примере отладки с `msvcmon`
```
ssh -nN -L localhost:4026:remotehost:4026 user@ssh-host
```

```
Get-ChildItem .\ProjectSourceFilesFolder -File -Recurse -Include "*.cpp","*.c","*.h","*.hpp" | Foreach {clang-format -i $_}
```

Путь до инструментов разработчика:
```
C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build
```

Путь до WDK (мб ещё и SDK)
```
C:\Program Files (x86)\Windows Kits\10\Include
```

Для автодеплоя использовать параметры для conan тулчейна:
```
conan install . --build=missing --profile msvc-194-w10x86_64-debug --options */*:dev_drive_deploy=True
```

Создать локальную ветку от удалённой и переключится на неё:
```git
git checkout -b branch1 origin/branch1
```

Использовать компилятор cl без создания проекта:
```cmd
cl /W4 /EHsc file1.cpp /link /out:program.exe
```

Конфиг для добавления профиля терминала в VS Code ${env:USERPROFILE}\AppData\Roaming\Code\User\setting.json
```json
"terminal.integrated.profiles.windows": {
	"Dev Command Prompt": {
		"path": [
			"${env:windir}\\Sysnative\\cmd.exe",
			"${env:windir}\\System32\\cmd.exe"
		],
		"args": [
			"/k",
			"C:\\Program Files\\Microsoft Visual Studio\\2022\\Community\\Common7\\Tools\\VsDevCmd.bat",
			"-arch=x64",
			"-host_arch=x64"
	],
		"icon": "terminal-cmd"
	},
}
```

Отобразить список переменных окружения в Powershell:
```powershell
gci env:* | Sort-Object name
```

Ещё один пример компиляции программы, только в batch-файле:
```bat
@echo off
del file.exe remove_entry.obj
cl /W4 /EHsc ../remove_entry.cpp /link /out:file.exe
echo:
echo:
.\file.exe
```


Для отображения общей информации о пакете можно использовать команду:
`apt-cache show <package_name>=<package_version>`

Если нужно отобразить только зависимости (например у метапакетов), то можно использовать:
`apt-cache depends <package_name>=<package_version>`

Список обратных зависимостей (то есть список пакетов, которые зависят от `package_name`):
`apt-cache rdepends --installed <package_name>`
`--installed` ограничивает вывод теми пакетами, которые в данный момент установлены.