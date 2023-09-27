+++
title = "如何自動切換專案 NodeJS 版本"
description = "不同專案使用不同的 NodeJS 版本，如何讓切換版本自動化？"
date = 2023-09-27
author = "Rhan0"
+++

## 前言

前端開發人員經常需要在數個專案中切換，NodeJS 版本也會因專案不同而切換來切換去，雖然有方便的 nvm、n 等管理工具可以進行版本控管，但依舊是“手動”來執行切換指令，長期下來是一種困擾，更不用說如果同時在多個專案裡，更容易出現問題。

近期團隊正好遇到這個痛點，為了讓溝通成本降低，也減少不必要的人為疏失，萌生了讓 NodeJS 版本隨著不同專案自動切換的想法，也找到了解決方式記錄在這邊。

## 實作
我是使用 zsh，如果要用其他方式可以參考[shell config section](https://github.com/nvm-sh/nvm#deeper-shell-integration).


1. 在專案中先切換成正確的 node 版本

```
nvm use 18.18.0
```

2. 輸入以下指令產生一個 .nvmrc 檔案

```
node -v > .nvmrc
```

3. 再來就是要去terminal設定一下，打開 `$HOME/.zshrc` 

```
sudo vim .zshrc
```

在檔案最後的地方加入以下這段，重新啟動 vscode，只要專案有 .nvmrc，你會發現它會自動幫你變更 nodeJS 版本！

```
# place this after nvm initialization!
autoload -U add-zsh-hook
load-nvmrc() {
  local node_version="$(nvm version)"
  local nvmrc_path="$(nvm_find_nvmrc)"

  if [ -n "$nvmrc_path" ]; then
    local nvmrc_node_version=$(nvm version "$(cat "${nvmrc_path}")")

    if [ "$nvmrc_node_version" = "N/A" ]; then
      nvm install
    elif [ "$nvmrc_node_version" != "$node_version" ]; then
      nvm use
    fi
  elif [ "$node_version" != "$(nvm version default)" ]; then
    echo "Reverting to nvm default version"
    nvm use default
  fi
}
add-zsh-hook chpwd load-nvmrc
load-nvmrc
```
> vim模式下編輯記得先按一下 i，完成編輯在案 esc + :wq + enter