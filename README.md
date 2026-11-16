# CLIProxyAPI 私有部署

保存丽萨服务器上 CLIProxyAPI 的部署配置与重建步骤。上游程序使用 [router-for-me/CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI) 的 **v8.0.73 Linux amd64** 发行版，本仓库不是上游源码 fork，也没有修改其程序。

## 当前结构

`AIMP → 模型路由 127.0.0.1:8318 → CLIProxyAPI 127.0.0.1:8317 → 模型提供商`

- GPT 通过 Codex OAuth 接入；换机器需要重新认证。
- DeepSeek 使用官方 API；配置保留 Flash、Pro 两个别名及模型映射。
- 仅监听本机，远程管理与控制面板关闭。
- 路由与代理之间使用独立密钥，与工作台访问路由的密钥分离。
- `config.example.json` 来自实际部署，密钥已替换为占位符。

## 在新服务器重建

需要 Linux x86_64、Python 3.10+、Git 和用户级 systemd。将本仓库克隆到 `~/.local/share/cliproxyapi-v8.0.73`，在该目录执行：

```sh
python3 install_binary.py
python3 setup_runtime.py
```

安装器校验固定 SHA256；初始化程序交互读取 DeepSeek Key，生成独立连接密钥。已有 `runtime` 时拒绝覆盖。GPT 使用设备码登录：

```sh
bin/cli-proxy-api -config runtime/config.json -codex-device-login
```

按照提示在浏览器完成认证，凭据保存在私有 `runtime/auths` 目录。

安装服务：

```sh
mkdir -p ~/.local/share/systemd/user/default.target.wants
cp systemd/cliproxyapi.service ~/.local/share/systemd/user/
ln -sfn ../cliproxyapi.service ~/.local/share/systemd/user/default.target.wants/cliproxyapi.service
systemctl --user daemon-reload
systemctl --user start cliproxyapi.service
systemctl --user status cliproxyapi.service
```

服务文件固定使用上述安装路径。需要退出登录后持续运行时，管理员可为该用户启用 systemd linger。部署模型路由时，通过本机文件 `runtime/link-key` 传递连接密钥，不要粘贴到仓库或命令参数。

## 日常维护

模型列表在 `runtime/config.json` 的 `api-keys.openai-compatibility[].models` 中修改，之后重启服务。增加模型后同步更新模型路由的 targets；上游模型可用性取决于账号权限。

本仓库排除运行配置、OAuth 凭据、密钥、日志、备份和二进制文件。它用于重建部署，不是凭据备份；现有服务器的运行目录没有被本次归档改动。
