# GitTools SSH 权限与跨电脑使用说明

## 结论

GitTools 本身没有绑定到某一台电脑的固定用户目录。它使用系统 OpenSSH，并由 OpenSSH 按当前 Windows 用户解析 SSH 配置和私钥。

因此：

- GitTools 程序可以跨电脑使用。
- 当前电脑修复过的 ACL 不会自动迁移到另一台电脑。
- 另一台电脑只要当前账户能够正常读取自己的 SSH 文件，GitHub remote 和 GitTools 通常可以继续使用。
- SSH 私钥、SSH 配置和代理环境仍需要在每台电脑分别准备。

## 本次问题的实际边界

用户在实际 CMD 中运行 `gs` 时，账户是：

```text
JUSDASCM\G1704069
```

Codex 工具执行诊断命令时可能使用另一个沙盒账户，例如：

```text
LH-NB-G1704069\CodexSandboxOffline
```

这两个账户不能混为一谈。诊断命令运行账户不是用户手动 CMD 的运行账户。

本次 SSH 报错的直接原因是 `.ssh` 文件的 ACL 中存在失效的域组或不匹配的账户权限。OpenSSH 对 `config` 和私钥的权限要求较严格，只要发现其他组或账户具有不安全权限，就可能拒绝使用文件。

这不是 C 盘天然敏感造成的，也不是 GitHub 仓库授权本身造成的。

## 本次 GS 失败的两个阶段

需要把下面两个问题分开判断：

1. SSH 权限或认证失败：Git 无法执行 `fetch`。
2. Git stash 恢复冲突：`fetch` 和 `rebase` 已成功，但恢复本地修改时发生冲突。

本次后续日志已经证明：

- `git fetch --prune --tags github` 成功。
- `git rebase github/main` 成功。
- 失败发生在 `git stash pop --index`。
- 冲突文件是 `DongGe/dexercise/app/app.js`、`DongGe/dexercise/sw.js` 和 `DongGe/dexercise/version.json`。

## 当前修复的可移植性

本次 ACL 修复使用了当前电脑上的 Windows SID，因此属于本机环境修复，不能直接复制到另一台电脑。

但是 GitTools 代码没有写死以下路径：

```text
C:\Users\G1704069\.ssh
```

正常情况下，OpenSSH 会根据另一台电脑当前账户的用户目录读取：

```text
%USERPROFILE%\.ssh\config
%USERPROFILE%\.ssh\id_ed25519
%USERPROFILE%\.ssh\known_hosts
```

只要另一台电脑的这些文件存在、当前用户可读、私钥权限符合 OpenSSH 要求，就不需要复用本机 SID，也不需要把凭据放进仓库。

## 另一台电脑的使用前检查

在实际运行 `gs` 的同一个 CMD 窗口中执行：

```cmd
whoami
echo %USERPROFILE%
icacls "%USERPROFILE%\.ssh\config"
icacls "%USERPROFILE%\.ssh\id_ed25519"
ssh -G -F "%USERPROFILE%\.ssh\config" git@ssh.github.com
```

如果需要收集完整信息，可以运行：

```cmd
D:\DongBase\scripts\infra\shortcuts\diagnose_ssh_identity.cmd
```

输出文件为：

```text
D:\DongBase\scripts\infra\shortcuts\ssh-diagnostic.txt
```

该诊断脚本只读取账户、环境变量、SSH 路径和 ACL，不会修改权限，也不会输出私钥内容。

## SSH 配置中的本机依赖

SSH 配置本身可能包含另一台电脑没有的代理设置，例如：

```text
127.0.0.1:18080
D:/Program Files/Git/mingw64/bin/connect.exe
```

如果另一台电脑没有相同的代理程序、端口或 Git 安装路径，即使私钥权限正确，也可能无法连接 GitHub。跨电脑使用前应检查这些配置，不要只复制私钥。

## 不建议的做法

不建议把私钥放进仓库的忽略目录。被 Git 忽略不等于安全，仍然可能被备份、压缩、同步或误提交。更重要的是，这只能绕开某一次用户目录 ACL 问题，不能修复域信任、账户 SID 或代理配置问题。

推荐继续使用每台电脑自己的用户 SSH 目录，并让程序提供明确的启动前检查。

## 后续程序增强方向

GitTools 可以增加轻量级 SSH 预检，但预检不应自动修改 Windows ACL：

- 动态解析当前 `%USERPROFILE%`。
- 检查 `config`、私钥和 `known_hosts` 是否存在且可读。
- 检查 SSH 配置是否能被 `ssh -G` 解析。
- 将权限、网络、认证和 stash 冲突分别分类提示。
- 在执行完整多仓库同步前，先给出明确的失败原因。
- 不把私钥、Cookie、token 或机器专属路径写入仓库。

这种方案提升的是跨电脑诊断和可维护性。真正的域信任或 Windows ACL 损坏，仍应在操作系统或账户环境层面修复。
