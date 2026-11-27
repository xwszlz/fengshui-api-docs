# 本体构建工程 - 开发规范 (给 AI 与人类协作参考)

> **核心原则**: 所有代码/本体/文档必须**可验证、可复现、可追溯**。严禁臆造不存在的类/属性/实例/验证结果。

---

## 🛑 绝对红线 (违反即停止)

| 红线 | 说明 |
|------|------|
| **严禁臆造本体元素** | 所有类/属性/实例/公理必须有据可查 (文本抽取/专家确认/标准引用/数据映射)，禁止凭空创建 |
| **严禁臆造验证结果** | SHACL/SPARQL/推理/度量结果必须来自真实工具运行输出，禁止手写「通过」 |
| **严禁绕过质量门禁** | `make validate` `make test` 必须真实通过，禁止跳过/注释掉断言 |
| **严禁破坏命名空间** | 所有新增 URI 必须使用 `config/namespaces.yaml` 定义的前缀，禁止自造命名空间 |
| **严禁破坏命名规范** | 类 PascalCase、对象属性 camelCase 动词、数据属性 camelCase 名词、实例 kebab-case |
| **严禁缺失标签注释** | 每个类/属性/实例必须有 `rdfs:label` (zh/en) 和 `rdfs:comment` (定义/来源/约束/示例) |

---

## 🎯 开发工作流 (强制)

### 1. 任务启动前
- [ ] 阅读 `README.md` `config/namespaces.yaml` `docs/modeling_guidelines.md`
- [ ] 确认任务属于哪个领域本体 (易学/地理/生态/精算/核心/扩展)
- [ ] 创建 `feature/issue-xxx` 分支

### 2. 本体建模 (核心)
- [ ] **先查**：现有本体中是否已有相关类/属性/模式可复用 (`src/ontology/` 目录)
- [ ] **再查**：上层本体 (BFO/DUL/SSN/GeoSPARQL/PROV-O) 对齐要求
- [ ] **再查**：外部标准本体 (ChEBI/GO/OM/QUDT/Schema.org) 可复用项
- [ ] **建模**：在对应 `src/ontology/<domain>/` 目录下创建/修改 `.ttl` 文件
- [ ] **标注**：每个新增元素**必须**包含：
  - `rdfs:label` (中文 + 英文，用 `@zh` `@en` 标注语言)
  - `rdfs:comment` (定义来源：文献/标准/专家/数据/推理；定义文本；约束条件；典型示例)
  - `dcterms:source` (来源引用：文献 DOI/标准号/专家姓名/数据集 ID)
  - `prov:wasGeneratedBy` (生成活动：抽取脚本版本/专家会议记录/会议纪要链接)

### 3. 约束定义 (SHACL)
- [ ] 在 `config/shacl_shapes/<domain>/` 创建对应 `.ttl` 形状文件
- [ ] 覆盖：基数约束、值类型约束、值范围约束、唯一性约束、逻辑约束 (SPARQL-based SHACL)
- [ ] 每个形状必须有 `sh:message` (中英双语错误提示)

### 4. 代码实现 (管线/工具)
- [ ] 遵循 `src/pipeline/` 现有模式
- [ ] 类型注解完整 (Python 3.11+)
- [ ] 日志使用 `loguru`，级别规范 (DEBUG/INFO/WARNING/ERROR)
- [ ] 配置通过 `config/namespaces.yaml` 统一管理，禁止硬编码 URI
- [ ] 异常处理完整，错误信息可定位

### 5. 测试 (强制)
- [ ] 单元测试 (`tests/unit/`)：每个新增类/属性/规则/函数对应测试
- [ ] 集成测试 (`tests/integration/`)：管线端到端 / 推理链 / 查询模板
- [ ] 回归测试 (`tests/regression/`)：版本间不兼容检测
- [ ] 测试数据使用 `tests/fixtures/` 管理，版本控制

### 5. 验证 (提交前必须通过)
```bash
make validate   # SHACL + SPARQL + 推理 + 命名 + 度量
make test       # 单元 + 集成 + 回归测试
make lint       # ruff + mypy
```
**任何一项失败 = 不得提交 PR**

### 6. 文档同步
- [ ] 更新 `docs/design_decisions/ADR-xxx.md` (如涉及设计决策)
- [ ] 更新 `docs/modeling_guidelines.md` (如新增建模模式)
- [ ] 更新 `docs/release_notes/vX.Y.Z.md` (发布说明)

### 7. 提交 PR
- [ ] PR 标题规范：`feat/domain: 简述` / `fix/domain: 简述` / `refactor/domain: 简述`
- [ ] PR 描述包含：变更摘要、影响范围、测试情况、ADR 链接 (如有)
- [ ] CI 全绿通过
- [ ] 至少 1 名领域专家 + 1 名知识工程师 Review 通过

---

## 📐 建模规范速查 (核心)

| 元素 | 命名 | 示例 | 必需标注 |
|------|------|------|----------|
| **类** | PascalCase | `DragonVein`, `ChineseGrimAge`, `AcupointDetection` | `rdfs:label`(zh/en), `rdfs:comment`, `dcterms:source`, `prov:wasGeneratedBy` |
| **对象属性** | camelCase 动词开头 | `hasPart`, `flowsTo`, `inheritsFrom`, `detectedBy` | 同上 + `rdfs:domain`, `rdfs:range` |
| **数据属性** | camelCase 名词/形容词 | `hasConfidenceScore`, `hasLength`, `isAuspicious` | 同上 + `rdfs:range` (xsd 类型) |
| **实例** | kebab-case | `dragon-vein-001`, `case-xiong-an-001`, `model-chinese-grimage-v2` | `rdf:type`, `rdfs:label`, 关键属性赋值 |
| **命名空间** | 统一前缀 | `hfsco-yj:DragonVein` | 禁止自造前缀 |

---

## 🧪 测试规范

| 测试层级 | 目标 | 位置 | 命令 |
|----------|------|------|------|
| **单元测试** | 每个类/属性/规则/函数/脚本 | `tests/unit/` | `pytest tests/unit -v` |
| **集成测试** | 管线端到端/推理链/查询模板 | `tests/integration/` | `pytest tests/integration -v` |
| **回归测试** | 版本间不兼容/破坏性变更 | `tests/regression/` | `pytest tests/regression -v` |
| **验证测试** | SHACL/SPARQL/推理/度量基线 | `make validate` | `make validate` |

**测试数据** 统一放在 `tests/fixtures/`，使用 Git LFS 管理大文件。

---

## 📦 发布规范

| 版本类型 | 触发条件 | 流程 |
|----------|----------|------|
| **Patch (vX.Y.Z+1)** | Bug 修复/文档更新/小幅度量提升 | `develop` → `release/vX.Y.Z+1` → Tag → 发布 |
| **Minor (vX.Y+1.0)** | 新增领域本体/新增推理能力/新增查询能力/非破坏性扩展 | `develop` → `release/vX.Y+1.0` → 全量测试 → Tag → 发布 |
| **Major (vX+1.0.0)** | 破坏性变更 (命名空间/核心类层级/上层对齐/推理语义变更) | `develop` → `release/vX+1.0.0` → 完整迁移指南 → 全量测试 → Tag → 发布 |

**发布产物**：OWL/Turtle/JSON-LD/JSON-LD Framed/SHACL/文档/度量报告/迁移指南 (Major)

---

## 📞 协作联系人

| 角色 | 职责 | 协作方式 |
|------|------|----------|
| **本体架构师** | 总体架构/跨域对齐/质量把关/ADR 裁决 | PR Review / ADR 评审 / 周例会 |
| **领域专家 (易学/地理/精算/生态)** | 概念定义/公理确认/案例校验/标准对齐 | 专家评审会 / 标注任务 / 术语表维护 |
| **知识工程师** | 抽取/转换/加载/验证/工具/文档 | 日常协作 / Pair Programming / 代码 Review |
| **精算师/合规官** | 精算本体合规/监管对齐/因子治理/审计 | 合规评审 / 监管预沟通同步 |

---

> **记住**：本体是知识的基石，**准确性 > 完整性 > 速度**。宁可慢一点，不可错一点。