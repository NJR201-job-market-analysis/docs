建立 python 虛擬環境

此種方式，可以在切換資料夾時，自動啟用/退出虛擬環境，並在 terminal 顯示當前虛擬服務的名稱

切換到對應的資料夾，執行
```bash
pipenv --python ~/.pyenv/versions/3.8.10/bin/python
```

安裝 direnv
```
sudo apt install direnv
```

建立 .envrc
```bash
export VIRTUAL_ENV=/home/iyauta/.local/share/virtualenvs/services-PW3YRg8f
export PATH="$VIRTUAL_ENV/bin:$PATH"
```

初始化 direnv
```bash
direnv allow
```

修改 .bashrc
```bash
code ~/.bashrc

# 加入以下幾行
# 不顯示 direnv 的提示訊息
export DIRENV_LOG_FORMAT=""

# 啟用 direnv 功能（請確保 direnv 已安裝）
eval "$(direnv hook bash)"

# 自動在 prompt 顯示虛擬環境名稱前綴
export PS1='$(if [[ -n "$VIRTUAL_ENV" ]]; then echo "\[\e[01;31m\]($(basename $VIRTUAL_ENV)) \[\e[00m\]"; fi)\[\e[01;32m\]\u@\h\[\e[00m\]:\[\e[01;34m\]\w\[\e[00m\]\$ '

source ~/.bashrc
```