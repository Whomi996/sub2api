# 型号定价数据

此目录包含镜像模型定价数据的本地副本作为后备机制。

## Source
原始文件由 LiteLLM 项目维护，并通过 GitHub Actions 镜像到此存储库的 `price-mirror` 分支：
- 镜像分支（可通过 `PRICE_MIRROR_REPO` 配置）：https://raw.githubusercontent.com/<your-repo>/price-mirror/model_prices_and_context_window.json
- 上游来源：https://raw.githubusercontent.com/BerriAI/litellm/main/model_prices_and_context_window.json

## Purpose
当由于以下原因无法下载远程文件时，此本地副本可作为后备：
- 网络限制
- 防火墙规则
- DNS解析问题
- GitHub 在某些地区被屏蔽
- Docker容器网络限制

## 更新过程
定价服务将：
1.首先尝试从GitHub下载最新版本
2. 如果下载失败，请使用此本地副本作为后备
3. 使用后备文件时记录警告

## 手动更新
要使用最新的定价数据手动更新此文件（如果自动化不可用）：
```bash
curl -s https://raw.githubusercontent.com/BerriAI/litellm/main/model_prices_and_context_window.json -o model_prices_and_context_window.json
```

## 文件格式
该文件包含带有模型定价信息的 JSON 数据，包括：
- 型号名称和标识符
- 输入/输出代币成本
- 上下文窗口大小
- 模型能力

最后更新: 2025-08-10
