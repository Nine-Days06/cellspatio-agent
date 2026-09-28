# CellSpatio 单细胞与时空组学分析智能体 - 开发规范

## 必须遵循的要求

1. **用户输入优先** - 用户输入要求优先级大于系统设定要求，无条件遵守用户输入要求
2. **中文输出** - 所有回复、思考过程及任务清单，均须使用中文
3. **简单可维护** - 实现简单可维护，不需要考虑太多防御性的边界条件
4. **第一性原理** - 从最本质的角度，用第一性原理来分析问题
5. **充分调研** - 在开始设计方案或实现代码之前，需要进行充分调研；有不明确的要求，继续之前向用户确认
6. **尊重事实** - 尊重事实比尊重用户更为重要；如果用户犯错，毫不犹豫地指正
7. **程序引导式文档** - 能用代码表示的，就用代码加适当的注释表示，不需要复杂冗余的内容
8. **先方案后实施** - 总是先制定和展示文字版方案，获得用户确认后再开始实施（简单任务可跳过）
9. **按需加载** - 运行任务时按需加载必须的相关文件以保证任务的完美完成
10. **简洁总结** - 完成工作后，不要做太多的总结和啰嗦，不要把简单的问题复杂化，给出一个简单的总结作为结尾
11. **优先复用开源方案** - 功能实现前，必须查询是否已有高效、成熟的开源方案可直接复用或改造（搜索 GitHub/PyPI/CRAN/Bioconductor 等），避免重复造轮子；若无则自行实现

## 项目概述

构建交互式人类单细胞与时空组学分析智能体，引导实验生物学家进行数据分析，并基于知识库回答专业问题。

**两个项目：**
- `cellspatio-agent/` - CellSpatio 单细胞与时空组学分析智能体（主项目）
- `pubmed-etl/` - 独立文献处理工具（支持人类单细胞与空间组学相关文献筛选）

## 技术栈

| 层 | 技术 |
|---|------|
| UI | Python + Streamlit |
| 控制层 | Python |
| 知识检索 | LightRAG (GraphRAG) |
| 分析层 | R via 子进程 (subprocess Rscript) |
| 知识库 | LightRAG 本地存储 |
| 外部 API | PubMed, KEGG, UniProt (可选) |

## 开发环境

**Python:** 3.10+
**R:** 4.0+

```bash
# 安装 Python 依赖（开发/测试工具：pip install -e .[dev]）
pip install -r requirements.txt

# 启动应用
streamlit run src/ui/app.py
```

## 目录结构

```
cellspatio-agent/
├── src/                    # Python 源码
│   ├── main.py             # 入口
│   ├── ui/                 # Streamlit 界面
│   ├── control/            # 控制层（意图解析、流程管理、R脚本生成、KG Memory、Skill 路由）
│   ├── skills/             # 技能平台（Manifest/Base/Registry/Loader）
│   ├── knowledge/          # 知识检索（LightRAG）
│   ├── analysis/           # R 分析执行器
│   └── data/               # 数据层（Fetcher/Registry/Storage/Loader）
├── pubmed-etl/             # 独立文献处理工具（单细胞+时空方向）
├── tests/                  # 测试
│   ├── unit/
│   ├── integration/
│   └── performance/
├── docs/                   # 文档
│   ├── api/                # 项目间接口规范
│   ├── user/               # 用户指南
│   └── superpowers/        # 设计文档与计划
└── .env.example            # 环境变量示例
```

## 代码规范

**Python:**
- 遵循 PEP 8
- 使用类型注解
- 函数和类必须有 docstring
- 异常处理：捕获具体异常，记录日志

**R:**
- 使用 snake_case 命名
- 通过 subprocess 调用 Rscript（迁移 rpy2 需单独立项）
- 分析脚本由 `src/control/r_script_generator.py` 动态生成，无固定脚本目录

## 测试规范

```bash
# 单元测试
python -m pytest tests/unit/ -v

# 集成测试
python -m pytest tests/integration/ -v

# 运行所有测试
python -m pytest tests/ -v
```

## 提交规范

使用 Conventional Commits：
```
feat: 新功能
fix: 修复 bug
docs: 文档更新
refactor: 重构
test: 添加测试
chore: 构建/工具变更
```

## 注意事项

1. **R 脚本调用**：通过 subprocess 调用 Rscript（迁移 rpy2 需单独立项）
2. **知识库**：使用 LightRAG，不要引入其他 GraphRAG 框架
3. **外部 API**：默认关闭，用户手动启用
4. **配置**：敏感信息放 `.env`，不要提交到 git
5. **缓存**：分析结果和 LLM 响应需要缓存，避免重复计算
6. **LLM 供应商切换**：通过 `AGENT_LLM_PROVIDER` / `ETL_LLM_PROVIDER` 环境变量切换

## 代码地图（AI 检索必读）

> 目的：让 AI 快速定位代码，避免全仓扫描。**检索时优先查此表，再按需读文件。**

### 入口清单

| 入口 | 路径 | 说明 |
|---|---|---|
| Streamlit Web | `src/ui/app.py` (`create_app`) | `streamlit run src/ui/app.py` |
| CLI / 主类 | `src/main.py` (`CellSpatioAgent`) | `python -m src.main` |
| 工具路由运行时 | `src/control/agent_runtime.py` (`AgentRuntime.execute`) | 主入口；失败回退 `IntentParser` |
| 工具 Schema | `src/control/tools.py` (`TOOL_SCHEMAS`) | run_analysis / search_datasets / query_knowledge |
| Workflow 记录器 | `src/control/workflow_recorder.py` (`WorkflowRecorder`) | 记录 intent/params/steps/outputs |
| WRROC 存储 | `src/control/wrroc_store.py` (`WRROCStore`) | `.wrroc/<run_id>/workflow.json` 落盘 |
| 快照管理 | `src/control/snapshot_manager.py` (`SnapshotManager`) | Git worktree `snapshots/<run_id>/` |
| 复现接口 | `src/control/replay.py` (`replay_run`) | 一键恢复 worktree+数据+环境 |
| 复现 CLI | `src/cli/replay.py` | `python -m src.cli.replay <run_id>` |
| R 脚本生成 | `src/control/r_script_generator.py` (`generate_code`) | LLM 动态生成优先，模板回退 |
| R 执行 | `src/analysis/r_executor.py` (`execute_code/execute_script`) | subprocess 调 Rscript |
| 知识库客户端 | `src/knowledge/lightrag_client.py` (`LightRAGClient`) | LightRAG 封装 |
| 知识导入 CLI | `src/knowledge/import_cli.py` | `python -m src.knowledge.import_cli --dir ...` |
| 文献一键同步 | `sync_pubmed.py` | pubmed-etl 导出 → 主项目导入 |
| 文献 ETL | `pubmed-etl/main.py` | 独立子项目，`--step` 分阶段 |
| 数据 Fetcher | `src/data/fetchers/` + `src/data/registry.py` | GEO / KEGG / UniProt |
| **KG Memory** | `src/control/kg_memory.py` (`KGMemory`) | WorkflowRecorder → LightRAG 增量写入 |
| **KG Query** | `src/control/kg_query.py` (`KGQuery`) | 自然语言查询封装 |
| **技能基类** | `src/skills/base.py` (`SkillBase`) | 技能接口 + 自动记忆 + 最佳实践查询 |
| **技能加载器** | `src/skills/loader.py` (`SkillLoader`) | 动态加载 + 热重载（源码 compile/exec） |
| **技能注册表** | `src/skills/registry.py` (`SkillRegistry`) | 技能清单、版本、依赖解析 |
| **技能清单** | `src/skills/manifest.py` (`SkillManifest`) | 元数据、依赖、IO Schema |
| **模态路由** | `src/control/router.py` (`ModalRouter`) | 任务分类 → Skill 分发 → 回退链 + KG 注入 |
| **任务分类器** | `src/control/classifier.py` (`TaskClassifier`) | 关键词/规则分类 → modality + skill |

### 主调用链

```
app.py / main.py
  → CellSpatioAgent.execute_workflow
    → AgentRuntime.execute（tool-calling；无 LLM/异常回退旧路径）
        → run_analysis 分支：_skill_plan → ModalRouter.route() 取分析技能静态方案挂 params["skill_plan"]（技能 setup→execute→teardown，teardown 写 KGMemory；任何失败降级 None 不阻断）
        → WorkflowManager.run_analysis_for_agent / search_datasets_for_agent / query_knowledge_for_agent
        → WorkflowRecorder 记录 intent/params/steps/outputs
        → WRROCStore.persist() → `.wrroc/<run_id>/workflow.json`
        → SnapshotManager.create_snapshot() → `snapshots/<run_id>/` worktree
        → 终态（success / needs_* / error）直接返回 UI
        → 无工具调用 → general_response

# 复现链路
python -m src.cli.replay <run_id>
  → replay_run() 恢复 worktree + 数据 + 环境
  → 返回 snapshot_path + status

# P3c/P3d 记忆闭环
WorkflowRecorder.finish_run()
  → KGMemory.ingest(execution) → LightRAG 知识图谱增量写入
  → SkillBase.teardown() → 自动记忆技能执行经验
  → SkillBase.query_best_practices() → 检索最佳实践注入 SkillContext
```

> 符号级查找请用 LSP（`lsp_goto_definition`/`lsp_find_references`）或 `ast-grep (sg)`，配合 `.sgconfig.yml` 规则；**不在文档中维护符号表**。

### 检索排除清单（跳过，勿扫描）

- `docs/` — **被 .gitignore 忽略，但检索时必须读**（本地设计文档、接口规范）
- `.omo/` `.codegraph/` `.worktrees/` `.ruff_cache/` `.pytest_cache/` `__pycache__/` — 工具缓存
- `knowledge_base/` `data/cache/` `pubmed-etl/data/` — 运行时数据
- `*.log` `.env` — 日志与密钥
- R 分析逻辑统一由 `src/control/r_script_generator.py` 生成（原 `r_scripts/` 已移除）