+++
title = "如何自動切換專案 NodeJS 版本"
description = "不同專案使用不同的 NodeJS 版本，如何讓切換版本自動化？"
date = 2023-09-27 20:00:00
author = "Rhan0"
tags=["nodeJS"]
weight= 3
+++

## 前言

前端開發人員經常需要在數個專案中切換，NodeJS 版本也會因專案不同而切換來切換去，雖然有方便的 nvm、n 等管理工具可以進行版本控管，但依舊是“手動”來執行切換指令，長期下來是一種困擾，更不用說如果同時在多個專案裡，更容易出現問題。

確保所有專案環境裡的 NodeJS 版本雖然不是什麼嚴重的大事，但可以讓我們從開發到 release 的程式碼運作能按預期的方式進行，至少比較不會發生我在開發時看到的結果和 release 出去後運行的結果不一樣的困擾。

近期團隊正好遇到這個痛點，為了讓溝通成本降低，也減少不必要的人為疏失，萌生了讓 NodeJS 版本隨著不同專案自動切換的想法，也找到了解決方式記錄在這邊。

## 實作
我是使用 zsh，如果要用其他方式可以參考[shell config section](https://github.com/nvm-sh/nvm#deeper-shell-integration).

### 第一步：新增自動切換 NodeJS 的功能
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

在檔案最後的地方加入以下這段，重新啟動 vscode，只要專案有 .nvmrc，你會發現它會自動幫你變更 NodeJS 版本！

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


### 第二步：嚴格把關 NodeJS 版本
1. 在專案中的 package.json 中，加入 NodeJS, pnpm 版本資訊

```json
{
  "name": "example",
  "engines": {
    "node": ">= 18.18",
    "pnpm": "= 8.7.6"
  }
}
```

2. 在與 package.json 同一層的位置，新增 .npmrc 文件，內容只需要一行：

```
engine-strict=true
```

3. `engine-strict` 的設定，會告诉 npm 在不支援的版本上直接拋出錯誤，並且停止往下執行

```
Your Node version is incompatible with "/Users/XXX/XXX".

Expected version: >= 18.18.0
Got: v18.17.0

This is happening because the package's manifest has an engines.node field specified.
To fix this issue, install the required Node version.
```

如果没有設定 engine-strict，npm只會 log 出警告，但是會繼續往下執行