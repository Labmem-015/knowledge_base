# Windows
## tab
Ввести команду `:echo stdpath('config')`, которая покажет директорию для конфигураций. В данной директории в файле `init.lua` вводим следующие параметры:
```lua
vim.opt.tabstop = 4         -- A TAB character looks like 4 spaces
vim.opt.shiftwidth = 4      -- Number of spaces inserted when indenting
```