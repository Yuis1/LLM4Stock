# 基于LLM的股票投资决策系统  
## 背景
2023年LLM领域的突破性进展，进一步强化了量化投资中对新闻资讯进行分析解读的深度。  
[唐岳华博士的论文](https://mp.weixin.qq.com/s/sfhK0hjCKXUU-d74-6KrQQ) 对此进行了证明：使用2021年10月至2022年12月美国证券价格研究中心公开的真实股市数据和新闻进行测试，基于ChatGPT提供的“多空策略”交易建议，预测股市走势，在此期间最理想状态下的投资组合回报率达到了惊人的500%。  
当然了，这个超额收益随着LLM在量化中的普及，很快就会消失。  
在2023年下半年，明显感觉A股市场对新闻的深度解读速度更快、更高效了，LLM实时解读新闻+融券T+0逃顶，成了最高效的收割机器，再次站在了舆论的风口浪尖。虽然监管有所强化，但这个趋势不会逆转，人工解读新闻正在被LLM替代。  
## 目标
本项目目标是搭建LLM在股票投资中的工程框架，逐步实现LLM在股票投资中的平民化，让散户们在面对LLM的收割时也有工具可用。  
规划的功能模块包括几部分：  
1. 行情数据  ——构建全量的股票列表、基础信息、行情数据，并根据行情数据加工出一些常用因子。目前已完成A股基础信息的构建。
2. 新闻资讯  ——实时连接到各大权威的新闻资讯源，以进行实盘投资决策；获取历史全量新闻资讯数据，为策略回测提供支持。目前已实现的是对接 财联社-电报。
3. LLM决策  ——对接各大主流的LLM，实现对新闻资讯的解读。依次实现LLM的Prompt、嵌入、微调三种方案，目标是让用户可以通过书写自己的投资经验，实现个性化的投资LLM。目前已实现的是通过Prompt语句对接ChatGPT。
4. 策略回测  ——[暂未实现] 提供一个回测框架，可以对LLM的决策结果继续回测，并定位决策失败案例，以便进行决策规则调整。
5. 实盘对接  ——[暂未实现] 计划先支持通过微信、Email进行决策信息推送，再支持 迅投MiniQMT 等方式对接实盘。

## 代码   
请按顺序执行以下代码：  
· [0_初始化股票列表.ipynb](0_初始化股票列表.ipynb) 使用AKShare，爬取全量股票列表，对每只股票从巨潮资讯网获取公司概况、历史简称等信息，并进行合并、清洗。  
· [1_获取历史简称_bak.ipynb](1_获取历史简称_bak.ipynb) 使用AKShare，从新浪财经，获取股票历史简称。由于 巨潮资讯 已提供历史简称变更信息，所以这个文件仅供参考，不再需要执行了。  
· [2_新闻匹配股票.ipynb](2_新闻匹配股票.ipynb) 使用AKShare获取财联社电报；从电报内容中解析出股票名称，匹配股票代码。  
· [3_LLM对新闻进行解读.ipynb](3_LLM对新闻进行解读.ipynb) 使用LLM对财联社电报进行解读；LLM当前使用的是ChatGPT 3.5，由于ChatGPT在中国无法直接访问，故使用 [API2D的转发接口](https://api2d-doc.apifox.cn/api-84787447)。考虑到复杂的Prompt，或可能需要在一次提问中批量查询，故还加入了计算token数量的函数。  

提示词：  
· [system_prompt.md](system_prompt.md) 输入给LLM的提示词。由于格式较为复杂，故独立成文件，程序运行时将读取并输入给LLM。
· [system_prompt_啰嗦版.md](system_prompt_啰嗦版.md) 在system_prompt.md的基础上多，针对不同的新闻场景加了几个示例，后来发现ChatGPT一个示例就够了，其它LLM可能需要多个示例。保留以供参考。

## 数据存储  
目前采用csv格式存储，位于 data 目录下。  
最终结果：  
· data/stock_info.csv   最终整合后的本地股票列表及历史名称。目标：构建全量的A股代码列表，标准化股票代码，获取全量的名称信息（含 股票简称、公司名称、历史简称、历史全称等）以便用来关联新闻资讯中提到的公司名称，并附加企业基础信息。  
· data/stock_cls_telegram.csv   财联社-电报，每条电报匹配的股票代码，以及LLM对电报对应股票的涨跌预测。  

中间结果：  
· data/stock_profile_cninfo.csv 从 巨潮资讯网 获取的股票信息，用于整合。 
· data/stock_info_sh.csv    股票列表-上证
· data/stock_info_sz.csv    股票列表-深证
· data/stock_info_bj.csv    股票列表-北证
· data/change_name_sina.csv 新浪财经 提供的股票历史曾用名，目前已被巨潮资讯提供的内容代替。

## 运行步骤
### 安装Python包  
目前依赖的包有 akshare, openai, pandas, requests, difflib, python-dotenv  
~~~shell
pip install akshare, openai, pandas, requests, difflib, python-dotenv  
~~~

### 创建 .env 文件
本项目采用 .env 文件实现代码和配置分离  
参考阅读 [python中如何优雅的实现代码与敏感信息分离？](https://juejin.cn/post/7099283807953977358)  

在项目根目录下创建 .env 文件，内容格式如下：  
~~~shell
# 快代理
kdl_username = "快代理用户名"
kdl_password = "快代理密码"
kdl_tunnel = "快代理隧道"

# OpenAI
openai_key = 'OpenAI的key'

# api2d
api2d_Authorization = 'api2d的验证，格式是 Bearer fk····'
~~~
如果代码要传git，记得将 .env 添加到 .gitignore 中。  
### 注册快代理
akshare本质上是爬虫，对一些数据源（比如巨潮资讯、新浪财经）调用次数过于频繁会被封。  
可以考虑使用快代理的隧道代理进行爬取，也可以尝试用免费的代理。  
隧道代理购买地址： https://www.kuaidaili.com/tps  
免费的代理： https://www.kuaidaili.com/free/inha/   
### 注册OpenAI
注册并开通OpenAI的API： https://openai.com/product  

### 注册API2D
由于OpenAI在国内无法直接调用，需要使用代理转发，我用的是API2D。  
注册地址： https://www.api2d.com/r/210668  
接口地址： https://api2d-doc.apifox.cn/api-84787447  

## 参考资料  
· [提示工程指南](https://www.promptingguide.ai/zh)  
· [OpenAI嵌入](https://platform.openai.com/docs/guides/embeddings)  
· [OpenAI微调模型](https://platform.openai.com/docs/guides/fine-tuning)  
· [专访唐岳华博士：支招ChatGPT炒股（内附独家视频）](https://mp.weixin.qq.com/s/sfhK0hjCKXUU-d74-6KrQQ)  
· [Can ChatGPT Forecast Stock Price Movements? Return Predictability and Large Language Models](https://arxiv.org/pdf/2304.07619.pdf)  