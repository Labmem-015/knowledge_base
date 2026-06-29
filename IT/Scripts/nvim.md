```table-of-contents
```

---
# Windows
## tab
Ввести команду `:echo stdpath('config')`, которая покажет директорию для конфигураций. В данной директории в файле `init.lua` вводим следующие параметры:
```lua
vim.opt.tabstop = 4         -- A TAB character looks like 4 spaces
vim.opt.shiftwidth = 4      -- Number of spaces inserted when indenting
```
##  Как загрузить vim plug
В PowerShell:
```powershell
iwr -useb https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim |`
    ni $HOME/vimfiles/autoload/plug.vim -Force
```
Он установить его в `~\vimfiles\autoload\plug.vim`
## Как установить плагины
В `nvim.lua` прописать:
```lua
local Plug = vim.fn['plug#']
vim.call('plug#begin')
-- Shorthand notation for GitHub:
Plug('<user_name>/<repo_name>')
Plug('https://github.com/<user_name>/<repo_name>.git')
vim.call('plug#end')
```
Затем нужно перезайти в `nvim` и выполнить команду: `:PlugInstall`
## Как поменять бинды
Назначим кнопку `<leader>` которая позволит активировать последовательность из уже занятых стандартных клавиш. По умолчанию это обратный слэш `\`.
Меняем её:
```lua
vim.g.mapleader = " " -- это пробел
```
Далее можно менять бинды и задействовать клавишу `<leader>`:
```lua
local map = vim.keymap.set
local opts = { noremap = true, silent = true }
-- noremap: prohibit recursive execution
-- silent: hide command display at bottom

-- <mode>, <keys>, <action or keys>
map('n', '<leader>n', ':NERDTreeFocus<CR>', opts)
map('n', '<C-n>', ':NERDTree<CR>', opts)
map('n', '<C-t>', ':NERDTreeToggle<CR>', opts)
map('n', '<C-f>', ':NERDTreeFind<CR>', opts)
```