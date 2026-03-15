Clothing-RAG-KnowledgeBase-System
基于RAG技术的服装领域轻量化智能客服知识库系统，解决大模型问答幻觉问题，实现精准贴合业务场景的智能问答服务。

📌 项目简介
本项目是面向服装领域的垂类RAG智能问答系统，聚焦服装洗涤养护、尺码推荐、颜色搭配三大核心业务场景，完整实现了**知识库动态更新、智能语义检索、多轮对话记忆、问答精准可控**的全链路能力，可快速适配企业自有知识库的轻量化部署与定制化需求。

🎯 项目背景
通用大模型的开放式问答存在明显的「幻觉」问题，无法精准贴合企业自有业务知识库内容，难以满足服装行业垂直场景的标准化、精准化客户咨询需求。本项目基于检索增强生成（RAG）技术，构建了一套可落地、可扩展的轻量化知识库问答系统，约束大模型仅基于企业知识库内容作答，大幅提升问答准确率与业务适配性。

✨ 核心功能
- 结构化知识库管理：支持服装领域业务知识的批量入库、文本分块、去重过滤、向量持久化存储
- 精准语义检索：基于向量相似度匹配，实现用户问题与知识库内容的精准召回，过滤无关信息
- 可控式RAG问答：严格约束大模型仅基于检索到的知识库内容作答，从根源上解决幻觉问题
- 多轮对话记忆：支持对话历史持久化存储，基于session_id实现上下文连贯问答，页面刷新/重启后可恢复历史对话
- 知识库动态更新：支持增量文本入库，自动去重，无需重启服务即可完成知识库更新

🛠️ 技术栈
| 技术类别 | 核心选型 |
|----------|----------|
| 开发语言 | Python 3.10+ |
| 大模型框架 | LangChain |
| 向量数据库 | Chroma |
| 嵌入模型 | 阿里云DashScope Embeddings |
| 对话大模型 | 通义千问Qwen ChatModel |
| 前端交互 | Streamlit |

📁 项目结构
Clothing-RAG-KnowledgeBase-System/
├── config/ # 项目配置文件（API Key、向量库路径等）
├── data/ # 原始知识库文本文件
├── vector_db/ # Chroma 本地向量库持久化存储
├── chat_history/ # 对话历史 JSON 文件持久化目录
├── services/
│ ├── vector_store_service.py # 向量库操作封装类
│ ├── rag_service.py # RAG 问答链核心逻辑
│ └── chat_history_service.py # 对话历史管理模块
├── utils/ # 工具类（文本分块、MD5 去重、日志等）
├── app.py # Streamlit 前端主程序入口
├── requirements.txt # 项目依赖清单
└── README.md
plaintext

🚀 快速开始
1. 环境准备
- Python 3.10 及以上版本
- 开通阿里云DashScope API Key、通义千问API访问权限

2. 安装依赖
```bash
git clone https://github.com/crepus-huiyi/Clothing-RAG-KnowledgeBase-System.git
cd Clothing-RAG-KnowledgeBase-System
pip install -r requirements.txt

3. 配置环境变量
在项目根目录创建.env文件，填写如下配置：
env
DASHSCOPE_API_KEY=你的阿里云DashScope API Key
QWEN_API_KEY=你的通义千问API Key
VECTOR_DB_PATH=./vector_db
CHAT_HISTORY_PATH=./chat_history

4. 启动项目
bash
运行
streamlit run app.py
启动后访问浏览器输出的本地地址，即可进入智能客服交互页面，上传知识库文本、开启问答。

🔧 核心模块实现

1. 知识库管理模块
文本预处理：通过RecursiveCharacterTextSplitter按段落 / 句子自然分割文本，设置chunk_size=1000、chunk_overlap=100，保证语义完整性
文本去重：基于 MD5 哈希值实现文本内容去重，上传文件前先校验哈希值，已入库内容直接跳过，避免向量库冗余
向量入库：调用阿里云 DashScopeEmbeddings 将文本向量化，存入 Chroma 本地向量库并绑定元数据，完成知识库与嵌入模型的交互

2. 向量检索与匹配模块
封装VectorStoreService类统一管理向量库的增删改查操作，兼容 LangChain Retriever 接口标准
优化检索策略：设置检索参数k=1、similarity_threshold=1，精准过滤无关内容，为 RAG 问答链提供高匹配度的参考资料
实现用户问题实时向量化转换与相似度匹配，毫秒级完成检索响应

3. RAG 问答链与多轮对话模块
串联「检索 - 提示词 - 大模型」核心流程，构建 RAG 全链路逻辑闭环
定制提示词模板，拼接「系统指令 + 参考资料 + 对话历史 + 用户问题」，强约束大模型仅基于知识库内容回答，无匹配内容时明确告知用户，拒绝编造信息
通过RunnableWithMessageHistory关联对话历史存储，实现多轮上下文连贯问答，完整保留用户咨询轨迹

4. 对话历史与状态管理模块
封装FileChatMessageHistory类，将 LangChain BaseMessage 对象转为 JSON 格式，按 session_id 分文件持久化存储到本地
提供add_messages、clear等方法，灵活管理对话生命周期
基于 Streamlit st.session_state存储核心服务实例与对话状态，避免页面刷新后重新初始化，保证交互连续性
支持通过 session_id 恢复历史对话，即使关闭页面后再次打开，也能保证上下文统一

📌 项目亮点
轻量化部署：无需复杂的分布式环境，本地即可一键启动，适配中小企业轻量化知识库需求
幻觉问题根治：通过强提示词约束 + 精准检索匹配，实现大模型回答 100% 基于知识库内容，无业务无关编造信息
高可用交互：完整的对话历史持久化与状态管理，保证多轮对话的连贯性与用户体验
可扩展架构：模块化设计，可快速替换嵌入模型、对话大模型、向量数据库，适配其他垂类行业场景
🐛 问题与解决方案
问题：多次上传同一 TXT 文件时，向量库中会生成重复的文本向量，导致检索结果出现重复内容，影响问答体验。
解决方案：
对每一段待入库的文本内容计算 MD5 哈希值，建立已入库哈希值的本地记录文件
每次上传文件并完成文本分块后，先逐段校验 MD5 哈希值
若哈希值已存在于记录中，直接跳过该文本块的入库流程；若为新内容，完成入库后同步更新哈希值记录
从根源上避免了向量库的内容冗余，保证检索结果的唯一性与准确性
📄 许可证
本项目仅用于学习与交流，非商业用途。
