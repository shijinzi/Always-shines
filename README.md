# GMP 知识库问答助手（RAG）

> 基于检索增强生成（RAG）的医疗器械 GMP 知识库问答系统。支持上传 SOP、ISO 13485、设计开发验证文件，基于文档内容回答并标注引用来源，辅助研发与质量人员快速检索体系文件。

## 项目背景（业务痛点）

医疗器械企业的 GMP / ISO 13485 体系文件量大、版本多、条款分散，研发与质量人员查找特定条款往往需要翻阅数份 PDF，耗时且容易遗漏。本项目将体系文件本地向量化，通过自然语言问答快速定位条款原文，降低检索成本，同时严格控制输出范围，仅作为内部辅助工具。

## 技术栈

| 模块 | 选型 | 说明 |
| --- | --- | --- |
| 文档解析 | pdfplumber / python-docx | PDF、Word 文本提取 |
| 文本切分 | LangChain RecursiveCharacterTextSplitter | 章节感知切分，重叠窗口 |
| 向量化 | 待定（如 bge-small-zh） | 生成文本向量 |
| 向量库 | Chroma | 本地轻量部署 |
| 检索 | 关键词 + 向量混合检索 | 解决条款号/标准编号精确匹配 |
| 重排 | 待定（Rerank 模型） | 提升召回准确率 |
| LLM | 待定（DeepSeek / 通义千问） | 生成回答 |
| 前端 | Streamlit | 网页交互 Demo |

## 系统架构

```mermaid
flowchart LR
    A[上传文档<br/>SOP / ISO 13485 / DHF] --> B[文档解析<br/>PDF / Word]
    B --> C[章节感知切分<br/>重叠窗口]
    C --> D[Embedding 向量化]
    D --> E[(Chroma 向量库)]
    F[用户提问] --> G[混合检索<br/>关键词 + 向量]
    G --> H[Rerank 重排]
    E --> G
    H --> I[拼接 Prompt<br/>附原文引用片段]
    I --> J[LLM 生成回答]
    J --> K[输出<br/>回答 + 引用来源 + 合规声明]

## 核心特性

- 支持批量上传 PDF / Word 体系文件
- 问答结果强制标注原文引用来源
- 混合检索 + Rerank，提升条款号匹配准确率
- 输出附合规声明：仅供参考，需质量人员复核

## 快速开始

### 环境要求

- Python 3.10+
- LLM API Key（DeepSeek / 通义千问）

安装
git clone [仓库地址]
cd [项目目录]
pip install -r requirements.txt

配置
LLM_API_KEY=你的密钥
LLM_BASE_URL=https://api.deepseek.com
EMBEDDING_MODEL=bge-small-zh-v1.5

运行
# 1. 文档入库
python ingest.py --data ./docs

# 2. 启动问答
streamlit run app.py

项目结构
.
├── app.py              # Streamlit 问答界面
├── ingest.py           # 文档解析、切分、入库
├── retriever.py        # 混合检索 + Rerank
├── prompt.py           # 提示词模板（含合规声明）
├── docs/               # 测试语料（公开资料）
├── tests/              # 评估测试集
└── README.md

评估报告
用真实 SOP / ISO 条款构造测试集，记录召回准确率、幻觉案例与改进手段。这是本项目区别于普通 RAG Demo 的加分项。
指标	结果	说明
测试集规模	待补充	覆盖条款查询、流程问答、边界问题
Top5 召回准确率	待补充	优化手段（如加入 Rerank 后提升）
幻觉案例	待补充	详见评估文档，已记录缓解方案

合规声明（重要）
本项目定位为**企业内部辅助检索工具，非上市医疗器械软件（SaMD）**：
- AI 输出内容仅供参考，**不能直接作为正式体系文件、验证方案或注册申报资料使用**
- 所有 AI 生成内容必须由具备资质的研发 / 质量人员复核后，方可进入正式流程
- 系统不对外提供诊疗、诊断类建议，不涉及患者数据
- 部署于企业内部网络，文档权限遵循公司文档管理制度

更新日志
每周迭代记录见 [Releases](https://github.com/shijinzi/Always-shines/releases)，按周倒序更新。

后续规划
- 增加多版本文件对比（DHF 版本追溯）
- 接入 Agent，自动生成 CAPA 草稿
- 补充 SaMD 法规要点文档（NMPA 指导原则摘要）

License
MIT
