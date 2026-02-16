# PAI 日常运行入口（RUNBOOK）

适用工作区：`/Users/jackhong/Documents/文档/Codex/PAI_OpenCode`

---

## 0. 运行前约定

- 本仓库是个人 PAI 基础设施，优先维护以下内容：
  - 身份与参考系：`.opencode/skills/PAI/USER/`（含 `TELOS/`）
  - 系统配置：`opencode.json`、`.opencode/settings.json`
  - 记忆与沉淀：`.opencode/MEMORY/`
- 敏感信息默认不外发；对外分享前先去标识化。

---

## 1. 每日流程（Daily）

### 1.1 启动工作区

```bash
cd /Users/jackhong/Documents/文档/Codex/PAI_OpenCode
git status --short --branch
opencode
```

### 1.2 会话前快速对齐（2-5 分钟）

```bash
sed -n '1,80p' .opencode/skills/PAI/USER/BASICINFO.md
sed -n '1,140p' .opencode/skills/PAI/USER/TELOS/TELOS.md
sed -n '1,120p' .opencode/skills/PAI/USER/DAIDENTITY.md
```

目标：
- 今天的任务是否对齐 M0/M1/M2（使命）
- 今天是否有明确可交付物（文档、清单、计算模板、决策记录）

### 1.3 执行中规则

- 每个任务都产出可复用工件（模板/清单/笔记）。
- 先本地结构化，再决定是否调用云模型。
- 遇到不确定项，记录到 `TELOS` 或 `MEMORY`，避免丢失上下文。

### 1.4 收尾（10 分钟）

```bash
git status --short
git add -A
git commit -m "chore: daily PAI update"
```

可选：
```bash
git push
```

---

## 2. 每周流程（Weekly）

### 2.1 周回顾（建议周日或周一）

```bash
cd /Users/jackhong/Documents/文档/Codex/PAI_OpenCode
git pull --rebase
```

重点检查：
- `TELOS` 是否需要更新（Goals/Challenges/Strategies/Predictions）
- 本周是否产生了可复用资产（报告、清单、流程模板）
- 下周 1-3 个最高优先级动作是否明确

### 2.2 TELOS 维护命令

```bash
sed -n '1,260p' .opencode/skills/PAI/USER/TELOS/TELOS.md
rg -n "To be added|\\[To be added\\]|TODO" .opencode/skills/PAI/USER/TELOS
```

### 2.3 备份（本地）

```bash
mkdir -p backups
tar -czf backups/pai-memory-$(date +%F).tgz .opencode/MEMORY .opencode/skills/PAI/USER
ls -lh backups | tail -n 5
```

---

## 3. 遇到问题时（Troubleshooting）

### 3.1 `opencode` 无法启动

```bash
which opencode
opencode --version
ls -lt ~/.local/share/opencode/log | head
```

查看最新日志：
```bash
tail -n 120 "$(ls -t ~/.local/share/opencode/log/*.log | head -n 1)"
```

### 3.2 模型/网络异常（如 models.dev 连接失败）

```bash
opencode --version
opencode models
```

如果网络受限，优先：
- 使用本地可用模型配置（或离线流程）
- 先完成本地文档与结构化产出，待网络恢复再补模型调用

### 3.3 配置改坏后的快速回退

```bash
git status --short
git log --oneline -n 5
git restore opencode.json .opencode/settings.json
```

### 3.4 提交前隐私检查

```bash
rg -n "(API[_-]?KEY|TOKEN|SECRET|PASSWORD|PRIVATE[_-]?KEY)" .opencode opencode.json
```

若命中敏感项：先脱敏，再提交。

---

## 4. 最小日常检查清单（可复制）

- [ ] 今天任务是否对齐 TELOS 使命与目标
- [ ] 今天是否有可复用工件输出
- [ ] 提交前是否完成敏感信息扫描
- [ ] 关键结论是否写入 TELOS/MEMORY

