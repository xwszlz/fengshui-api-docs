# Makefile - 本体构建工程常用命令
# 使用: make <target>

.PHONY: help dev-up dev-down install test lint format validate extract transform load metrics clean release doc

# 默认目标
default: help

# 显示帮助
help:
	@echo "=== 本体构建工程命令 ==="
	@echo "开发环境:"
	@echo "  make dev-up        - 启动开发环境"
	@echo "  make dev-down      - 关闭开发环境"
	@echo "  make install       - 安装 Python 依赖"
	@echo ""
	@echo "管线执行:"
	@echo "  make extract       - 运行抽取管线"
	@echo "  make transform     - 运行转换管线"
	@echo "  make load          - 加载到三元组存储"
	@echo "  make validate      - SHACL/SPARQL 验证"
	@echo "  make metrics       - 本体度量报告"
	@echo ""
	@echo "质量保障:"
	@echo "  make test          - 运行所有测试"
	@echo "  make lint          - 代码风格检查"
	@echo "  make format        - 代码格式化"
	@echo ""
	@echo "发布/文档:"
	@echo "  make release       - 生成发布版本"
	@echo "  make doc           - 生成文档"
	@echo ""
	@echo "维护:"
	@echo "  make clean         - 清理临时文件"
	@echo "  make help          - 显示此帮助"

# ============================================================
# 开发环境
# ============================================================
dev-up:
	@echo "🚀 启动开发环境..."
	cd deployment/docker && docker-compose up -d
	@echo "✅ 服务已启动:"
	@echo "  Fuseki:      http://localhost:3030  (admin/admin)"
	@echo "  Neo4j:       http://localhost:7474  (neo4j/password)"
	@echo "  GraphDB:     http://localhost:7200  (admin/admin)"
	@echo "  MinIO:       http://localhost:9001  (minioadmin/minioadmin)"

dev-down:
	@echo "🛑 关闭开发环境..."
	cd deployment/docker && docker-compose down

# ============================================================
# 依赖安装
# ============================================================
install:
	@echo "📦 安装 Python 依赖..."
	pip install --upgrade pip
	pip install -r requirements.txt
	@echo "💡 提示: 中文 spaCy 模型需单独安装: python -m spacy download zh_core_web_lg"

# ============================================================
# 管线执行
# ============================================================
extract:
	@echo "🔍 运行抽取管线..."
	python -m src.pipeline.extract.main --config config/namespaces.yaml --input data/raw --output data/interim

transform:
	@echo "🔄 运行转换管线..."
	python -m src.pipeline.transform.main --config config/namespaces.yaml --input data/interim --output data/processed

load:
	@echo "📥 加载到三元组存储..."
	python -m src.pipeline.load.main --config config/namespaces.yaml --input data/processed --target fuseki

validate:
	@echo "✅ 运行 SHACL/SPARQL 验证..."
	python -m src.pipeline.validate.main --config config/namespaces.yaml --shapes config/shacl_shapes --data data/processed

metrics:
	@echo "📊 生成本体度量报告..."
	python -m src.tools.ontology_metrics.main --input data/processed --output docs/metrics_report.md

# ============================================================
# 质量保障
# ============================================================
test:
	@echo "🧪 运行所有测试..."
	pytest tests/ -v --cov=src --cov-report=html --cov-report=term

lint:
	@echo "🔍 代码风格检查..."
	ruff check src/ tests/
	mypy src/

format:
	@echo "🎨 代码格式化..."
	black src/ tests/
	ruff check --fix src/ tests/

# ============================================================
# 发布/文档
# ============================================================
release:
	@echo "📦 生成发布版本..."
	@echo "💡 请手动执行: python scripts/release.py --version X.Y.Z"

doc:
	@echo "📚 生成文档..."
	mkdocs build --site-dir site
	@echo "✅ 文档已生成到 site/ 目录"

# ============================================================
# 维护
# ============================================================
clean:
	@echo "🧹 清理临时文件..."
	rm -rf data/interim/* data/processed/* __pycache__ .pytest_cache .mypy_cache .ruff_cache site/ dist/ build/ *.egg-info
	find . -name "*.pyc" -delete
	find . -name "*.pyo" -delete
	find . -name "*.pyd" -delete

# ============================================================
# 安装 pre-commit 钩子
# ============================================================
pre-commit-install:
	pre-commit install
	@echo "✅ pre-commit 钩子已安装"