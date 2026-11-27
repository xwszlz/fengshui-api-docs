# 人文风水核心本体构建工程 (Ontology Builder)

> **版本**: v0.1.0 | **启动日期**: 2025-11-27 | **负责**: 知识工程组 | **Git**: `ontology-builder/` 子目录 (将来可拆分为独立仓库)

---

## 🎯 工程目标

构建 **胡焱照人文风水核心本体 (Huyanzhao Feng Shui Core Ontology, HFSCO)**，覆盖四大核心理论域：

| 领域本体 | 核心类估算 | 核心关系估算 | 关键公理 | 优先级 |
|----------|------------|--------------|----------|--------|
| **易学时空本体** (Yijing Spatiotemporal) | 60+ | 80+ | 纳甲/河图洛书/九宫/元运/卦气/飞星/二十四山 推理规则 | P0 |
| **地理环境本体** (Geographic Environment) | 80+ | 100+ | 龙脉/穴场/砂水/朝案/水口/风水林/气场/微气候 空间拓扑/因果规则 | P0 |
| **人文生态本体** (Human Ecology) | 50+ | 70+ | 聚落/族群/记忆/产业/心理/社群/传承/治理 社会-空间耦合规则 | P1 |
| **精算生存本体** (Actuarial Survival) | 40+ | 60+ | 表观年龄/表型年龄/衰老速度/生存函数/竞争风险/多状态/精算假设/监管合规 | P0 |

**总计预估**：核心类 **230+**、对象属性/数据属性 **500+**、关系 **310+**、SWRL/SPARQL 规则 **500+**。

---

## 📁 目录结构

```
ontology-builder/
├── README.md                    # 本文件
├── CLAUDE.md                    # 开发规范 (给 AI/人类)
├── Makefile                     # 常用命令
├── requirements.txt             # Python 依赖
├── config/
│   ├── namespaces.yaml          # 命名空间前缀定义
│   ├── prefixes.sparql          # SPARQL 前缀声明
│   └── shacl_shapes/            # SHACL 约束形状 (按域分目录)
├── src/
│   ├── ontology/
│   │   ├── core/                # 核心上层本体 (BFO/DUL/SSN/SOSA/GEO 对齐)
│   │   ├── yijing/              # 易学时空本体
│   │   ├── geography/           # 地理环境本体
│   │   ├── ecology/             # 人文生态本体
│   │   └── actuarial/           # 精算生存本体
│   ├── pipeline/
│   │   ├── extract/             # 文本/案例/标准/数据 → 三元组抽取
│   │   ├── transform/           # 清洗/对齐/推理/实体链接
│   │   ├── load/                # 加载到 Fuseki/Neo4j/GraphDB
│   │   └── validate/            # SHACL/SPARQL/一致性/完整性校验
│   └── tools/
│       ├── sparql_templates/    # 常用 SPARQL 查询模板
│       ├── shacl_validation/    # SHACL 验证脚本
│       └── ontology_metrics/    # 本体度量 (类/属性/公理/覆盖率/一致性)
├── data/
│   ├── raw/                     # 原始输入 (文本/CSV/JSON/标准/案例)
│   ├── interim/                 # 中间产出
│   └── processed/               # 最终本体文件 (OWL/Turtle/JSON-LD)
├── tests/
│   ├── unit/                    # 单元测试 (类/属性/公理/约束)
│   ├── integration/             # 集成测试 (推理/查询/推理链)
│   └── regression/              # 回归测试 (版本间不兼容检测)
├── docs/
│   ├── design_decisions/        # 设计决策记录 (ADR)
│   ├── modeling_guidelines.md   # 建模规范
│   ├── namespace_policy.md      # 命名空间策略
│   ├── versioning_policy.md     # 版本策略
│   └── release_notes/           # 版本发布说明
└── deployment/
    ├── docker/                  # Dockerfile / docker-compose
    ├── k8s/                     # K8s manifests (Fuseki/Neo4j/GraphDB)
    └── helm/                    # Helm Charts
```

---

## 🛠 快速开始

### 环境要求
- Python 3.11+
- Java 17+ (Fuseki/GraphDB)
- Docker / Podman
- Git LFS

### 安装依赖
```bash
# Python 依赖
pip install -r requirements.txt

# 或使用 conda
conda env create -f environment.yml
conda activate ontology-builder
```

### 启动开发环境 (Docker Compose)
```bash
cd deployment/docker
docker-compose up -d
# 启动: Fuseki (3030), Neo4j (7474/7687), GraphDB (7200), MinIO (9000/9001)
```

### 常用命令 (Makefile)
```bash
make help                    # 显示所有命令
make dev-up                  # 启动开发环境
make dev-down                # 关闭开发环境
make extract                 # 运行抽取管线
make transform               # 运行转换管线
make load                    # 加载到三元组存储
make validate                # SHACL/SPARQL 验证
make metrics                 # 本体度量报告
make test                    # 运行所有测试
make release                 # 生成发布版本 (OWL/Turtle/JSON-LD)
make doc                     # 生成文档
```

---

## 📋 开发规范 (核心)

### 1. 命名空间策略
| 前缀 | URI | 说明 |
|------|-----|------|
| `hfsco` | `http://huyanzhao.org/ontology/hfsco#` | 核心本体主命名空间 |
| `hfsco-yj` | `http://huyanzhao.org/ontology/hfsco/yijing#` | 易学时空 |
| `hfsco-geo` | `http://huyanzhao.org/ontology/hfsco/geography#` | 地理环境 |
| `hfsco-eco` | `http://huyanzhao.org/ontology/hfsco/ecology#` | 人文生态 |
| `hfsco-act` | `http://huyanzhao.org/ontology/hfsco/actuarial#` | 精算生存 |
| `hfsco-ext` | `http://huyanzhao.org/ontology/hfsco/extension#` | 扩展/应用层 |

**规则**：所有新增类/属性必须使用上述前缀，禁止自造命名空间。

### 2. 建模规范 (核心)
- **上层对齐**：核心类必须对齐 BFO 2.0 / DUL / SSN-SOSA / GeoSPARQL / PROV-O
- **命名规范**：
  - 类：PascalCase (`DragonVein`, `Acupoint`, `ChineseGrimAge`)
  - 对象属性：camelCase 以动词开头 (`hasPart`, `flowsTo`, `inheritsFrom`)
  - 数据属性：camelCase 以名词/形容词开头 (`hasLength`, `isAuspicious`, `confidenceScore`)
  - 实例：kebab-case (`dragon-vein-001`, `case-xiong-an-001`)
- **标签/注释**：每个类/属性**必须**有 `rdfs:label` (中英双语) 和 `rdfs:comment` (定义来源/定义/示例/约束)
- **版本控制**：每次提交对应本体微版本，重大变更走 ADR 流程

### 3. 质量门禁 (CI 必过)
| 检查项 | 工具 | 通过标准 |
|--------|------|----------|
| **语法合法** | `rdf-toolkit` / `jena` | 解析无报错 |
| **SHACL 约束** | `shacl` (Jena/TopBraid) | 0 违规 |
| **SPARQL 一致性** | 自定义 SPARQL | 核心一致性查询 0 反例 |
| **推理一致性** | HermiT / ELK / Pellet | 无不满足类 |
| **度量基线** | `ontology-metrics` | 类/属性/公理/覆盖率/耦合度 在基线内 |
| **命名规范** | 自定义脚本 | 100% 符合命名规范 |

### 4. 发布流程
1. 开发分支 `feature/*` → PR → CI 全过 → 合并 `develop`
2. `develop` 积累到里程碑 → 切 `release/vX.Y.Z` → 全量测试
3. 打 Tag `vX.Y.Z` → 生成发布包 (OWL/Turtle/JSON-LD/JSON-LD framed/SHACL/文档/度量报告)
4. 发布到 GitHub Releases / Ontology Library / 生产环境 Fuseki/Neo4j

---

## 📊 当前进度 (v0.1.0 启动版)

| 领域 | 类 | 对象属性 | 数据属性 | 实例(种子) | SHACL 形状 | 状态 |
|------|----|----------|----------|------------|------------|------|
| 核心上层 | 15 | 20 | 10 | 5 | 10 | 🟡 构建中 |
| 易学时空 | 12 | 18 | 8 | 20 | 15 | 🟡 构建中 |
| 地理环境 | 18 | 25 | 12 | 30 | 20 | 🟡 构建中 |
| 人文生态 | 8 | 12 | 6 | 15 | 10 | ⏳ 待开始 |
| 精算生存 | 10 | 15 | 8 | 10 | 12 | 🟡 构建中 |
| **合计** | **55** | **90** | **44** | **80** | **67** | **🟡 启动阶段** |

---

## 🎯 里程碑计划

| 里程碑 | 目标日期 | 核心交付物 | 质量门槛 |
|--------|----------|------------|----------|
| **M1: 核心上层 + 易学时空 + 地理环境 v0.2** | 2025-12-15 | 核心上层对齐/易学/地理本体 v0.2 + SHACL + 测试 | 类≥120、SHACL 0 违规、推理一致 |
| **M2: 精算生存 + 人文生态 v0.3** | 2026-01-15 | 精算/生态本体 v0.3 + 跨域对齐 + 种子实例 | 类≥200、跨域关系≥50 |
| **M3: 全域本体 v1.0 候选版** | 2026-02-15 | 完整 HFSCO v1.0-RC + 完整 SHACL/测试/文档/度量 | 类≥230、覆盖率≥90%、国际对齐 |
| **M4: v1.0 正式发布** | 2026-03-01 | HFSCO v1.0 正式版 + 文档/工具/部署/国际标准提案 | 国际标准提案就绪 |

---

## 🤝 贡献指南

1. Fork 仓库 → 创建 `feature/issue-xxx` 分支
2. 遵循 `CLAUDE.md` 和 `docs/modeling_guidelines.md`
3. 编写单元测试 (`tests/unit/`) + 集成测试 (`tests/integration/`)
4. 本地跑通 `make validate` `make test`
4. 提交 PR → 填写 ADR (如涉及设计决策) → CI 全过 → 合并

---

## 📚 关键参考资料

| 资料 | 链接/位置 | 说明 |
|------|-----------|------|
| BFO 2.0 规范 | `http://purl.obolibrary.org/obo/bfo/2.0/bfo.owl` | 上层本体基石 |
| DUL (DOLCE+DnS Ultralite) | `http://www.ontologydesignpatterns.org/ont/dul/DUL.owl` | 认知/社会实体建模 |
| SSN/SOSA | `https://www.w3.org/TR/vocab-ssn/` | 传感器/观测本体 (环境监测对齐) |
| GeoSPARQL 1.1 | `https://opengeospatial.github.io/ogc-geosparql/` | 地理空间本体标准 |
| PROV-O | `https://www.w3.org/TR/prov-o/` | 来源/过程本体 (血缘/溯源) |
| SHACL | `https://www.w3.org/TR/shacl/` | 约束验证标准 |
| SKOS | `https://www.w3.org/TR/skos-reference/` | 知识组织/词表对齐 |

---

## 📞 联系人

| 角色 | 姓名 | 职责 | 联系方式 |
|------|------|------|----------|
| **本体架构师** | [首席科学家] | 总体架构/跨域对齐/质量把关 | 内部联系 |
| **易学时空领域专家** | [知识工程师] | 易学本体/推理规则/古籍对齐 | 内部联系 |
| **地理环境领域专家** | [知识工程师] | 地理本体/空间拓扑/气场建模 | 内部联系 |
| **精算生存领域专家** | [精算师/知识工程师] | 精算本体/监管合规/因子治理 | 内部联系 |
| **知识图谱工程师** | [知识工程师] | 管线/抽取/推理/部署/运维 | 内部联系 |

---

> **本文件随工程演进持续更新，版本号遵循语义化版本。**