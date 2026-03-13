#!/bin/bash

# ==============================================================================
# 目标: #ZeroHomeDot 极致规矩、XDG 全兼容、mise 托管环境
# ==============================================================================

set -e

echo "🚀 开始构建规矩的系统环境..."

# 1. 创建 XDG 标准目录
echo "📂 初始化 XDG 目录结构..."
mkdir -p ~/.config/{zsh/conf.d,mise,starship,npm,python,git}
mkdir -p ~/.local/{share,state/python}
mkdir -p ~/.cache/{zsh,npm,pip,go-build,m2}

# 2. 写入核心引信 (~/.zshenv)
echo "⚡ 写入根目录唯一引信: .zshenv"
cat << 'EOF' > ~/.zshenv
export ZDOTDIR="$HOME/.config/zsh"
EOF

# 3. 基础环境变量与拦截逻辑 (~/.config/zsh/conf.d/env.zsh)
echo "🛡️ 注入拦截逻辑与 XDG 环境变量..."
cat << 'EOF' > ~/.config/zsh/conf.d/env.zsh
# XDG Base Directory
export XDG_CONFIG_HOME="$HOME/.config"
export XDG_DATA_HOME="$HOME/.local/share"
export XDG_CACHE_HOME="$HOME/.cache"
export XDG_STATE_HOME="$HOME/.local/state"

# Python 拦截
export PYTHONSTARTUP="$XDG_CONFIG_HOME/python/pythonrc"

# Maven 拦截
export MAVEN_OPTS="-Dmaven.repo.local=$XDG_DATA_HOME/m2"

# Go 拦截
export GOPATH="$XDG_DATA_HOME/go"
export GOCACHE="$XDG_CACHE_HOME/go-build"

# Starship 路径
export STARSHIP_CONFIG="$XDG_CONFIG_HOME/starship/starship.toml"

# Path 增强
path=("$XDG_DATA_HOME/go/bin" "$HOME/.local/bin" $path)
EOF

# 4. Python 历史记录拦截脚本
cat << 'EOF' > ~/.config/python/pythonrc
import atexit, os, readline
history = os.path.join(os.environ.get("XDG_STATE_HOME"), "python/history")
try: readline.read_history_file(history)
except OSError: pass
atexit.register(readline.write_history_file, history)
EOF

# 5. 安装基础工具 (Homebrew)
if ! command -v brew &> /dev/null; then
    echo "🍺 安装 Homebrew..."
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    eval "$(/opt/homebrew/bin/brew shellenv)"
fi

echo "📦 安装核心工具集..."
brew install mise starship eza bat fzf jq

# 6. 配置 mise 与 starship 激活
echo "🔌 激活工具链..."
cat << 'EOF' > ~/.config/zsh/conf.d/mise.zsh
export MISE_JAVA_HOME=1
if command -v mise &> /dev/null; then
  eval "$(mise activate zsh)"
fi
EOF

echo 'eval "$(starship init zsh)"' > ~/.config/zsh/conf.d/starship.zsh

# 7. 写入全局 mise 配置 (Java 21, Node 22, Python 3.12)
echo "⚙️ 预设 mise 全局工具..."
cat << 'EOF' > ~/.config/mise/config.toml
[tools]
java = "openjdk-21"
node = "22"
python = "3.12"
go = "1.24"

[settings]
autoinstall = true
experimental = true
EOF

# 8. 修正 Git 规矩
echo "📜 搬迁 Git 配置..."
git config --global core.hooksPath "$XDG_CONFIG_HOME/git/hooks"
# 提示用户手动搬迁 .gitconfig 如果已存在

echo "✅ 架构部署完成！"
echo "👉 请执行: source ~/.zshenv && source ~/.config/zsh/.zshrc"
echo "👉 你的根目录现在非常干净，尽情享受吧。"
