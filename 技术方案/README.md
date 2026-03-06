# 技术方案

> 针对 **Mac Mini M4 (24G 内存)** 的硬件限制，采用**"单引擎、多灵魂 (Single Engine, Multi-Soul)"**的技术方案是最科学的。
> 

> 这种方案的核心逻辑是：**物理层只跑一个最强的本地模型（Qwen2.5-14B），逻辑层通过不同的系统宪法（System Prompts）将一个模型幻化为三个截然不同的角色。**
> 

---

### 一、 算力底座配置 (Inference Engine)

- **本地模型**：`Qwen2.5-14B-Instruct-Q4_K_M`
    - **理由**：M4 芯片处理 14B 模型非常丝滑。Q4 压缩版占用约 **9.5GB** 内存，留给系统和其它应用（尤其是 Chrome 浏览器）约 **14GB** 空间，完美避免内存交换（Swap）。
- **部署命令**：
    
    ```bash
    ollama run qwen2.5:14b
    ```
    

---

### 二、 “单引擎·多角色” 逻辑架构

所有本地 Agent（发财、执远、如一）都指向同一个 API 端点（`localhost:11434`），但通过 OpenClaw 注入不同的 **宪法 (Constitutions)**。

### 1. 流量分配图

- **[用户指令]** -> **阿管助手 (Feishu Bot)** -> **发财 (宿主机进程/本地模型)**
- **[发财决策]** -> 分发任务给：
    - **执远 (本地模型)** -> 调用 Playwright 操控浏览器。
    - **如一 (本地模型)** -> 查询 PostgreSQL 进行审计。
    - **云端 Agent (Claude/GPT)** -> 处理创意与高阶代码。

---

### 三、 核心技术实现路径

### 1. 数据库：公司的大脑记忆 (PostgreSQL)

在 Docker 中启动，作为所有 Agent 共享的"状态机"。

```sql
-- 核心事务表：支持断点续传
CREATE TABLE agent_transactions (
    tx_id UUID PRIMARY KEY,
    project_id VARCHAR(50),
    actor_name VARCHAR(20), -- 发财/执远/如一等
    action_type VARCHAR(50),
    payload JSONB,          -- 原始参数
    status VARCHAR(20),     -- PENDING, SUCCESS, FAIL
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 2. 【执远】的本地操作实现 (Playwright + 本地模型)

由于【执远】需要操作 Mac 本地浏览器，他不能关在 Docker 里。

- **技术栈**：Python + Playwright + Ollama API。
- **逻辑**：
    1. 接收【聚宝】或【发财】的 JSON 指令。
    2. 调用本地 `qwen2.5:14b` 解析指令为具体的网页操作步骤。
    3. 驱动 Chrome 浏览器执行（发货、下架等）。

### 3. 跨岗位通信协议 (Feishu Gateway)

开发一个轻量级的 Python 转发器（Gateway），将飞书 Webhook 转化为 OpenClaw 指令。

---

### 四、 针对 24G 内存的资源优化方案

| 资源项 | 占用预算 | 说明 |
| --- | --- | --- |
| **macOS 系统** | 4.5 GB | M4 核心运行环境 |
| **Ollama (Qwen-14B)** | 9.5 GB | 常驻内存，多角色共用 |
| **Chrome (淘宝后台)** | 3.0 GB | 【执远】操作时的浏览器开销 |
| **Postgres/Redis/Qdrant** | 2.0 GB | Docker 容器基础服务 |
| **OpenClaw 实例 x 6** | 1.5 GB | Node.js 进程开销 |
| **剩余缓冲 (Buffer)** | **3.5 GB** | **关键冗余，防止系统掉线** |

---

### 五、 落地执行步骤

### 第一步：初始化宿主机 (Mac Mini)

1. 安装 **Ollama** 并拉取模型。
2. 安装 **Node.js 22** (运行 OpenClaw)。
3. 安装 **Python 3.11** (运行执远的自动化脚本)。

### 第二步：启动数据中枢 (Docker Compose)

运行我们之前定义的 `docker-compose.yml`，启动 PostgreSQL 和 Redis。

### 第三步：配置“单引擎”接入点

在 `~/FedAgentCompany/config/agents.json` 中配置：

```json
{
  "local_engine": "<http://localhost:11434>",
  "roles": {
    "发财": { "model": "qwen2.5:14b", "prompt": "CEO_宪法.md" },
    "执远": { "model": "qwen2.5:14b", "prompt": "执行_宪法.md" },
    "如一": { "model": "qwen2.5:14b", "prompt": "分析_宪法.md" }
  }
}
```

### 第四步：实现【发财】的“复盘定时任务”

编写一个 Cron Job，在每日 **23:00** 唤醒【发财】实例，执行 `EOD_Review` 脚本：

1. 查询 PostgreSQL 中当日所有 `SUCCESS` 的 `tx_id`。
2. 调用飞书文档 API，检查对应的归档文件。
3. 生成《健康度报表》发至飞书群。

---

### 方案评价：

- **稳定性**：极高。因为只跑一个模型，不会发生内存挤占。
- **成本**：极低。本地三个核心角色完全免费，且 M4 的能效比极高。
- **安全性**：最高。发货单号、经营利润、CEO 决策过程全在你的 Mac Mini 内部完成。

**这就是最适合您这台 M4 Mac Mini 的“高智商 AI 公司”实现方案。**

---

## 延伸阅读

- [协作协议](协作协议.md)
- [每日复盘](每日复盘.md)
