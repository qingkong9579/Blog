---
title: Ollama部署遇到的一些坑
published: 2024-07-19
description : 一些新的技术尝试
tags : [技术,新尝试]
category : 随笔
draft: false
---

* updated time: 2024-08-17

​![image](https://pub-d2c21cc922c14429b2c5c871ba58a50b.r2.dev/2024/08/b44bb9aa225a0cd72c20dd3f89484027.png)​

* ollama安装

  * 网络环境良好

    ```bash
    curl -fsSL https://ollama.com/install.sh | sh
    ```
  * 网络环境较差

    [Ollama安装指南:解决国内下载慢和安装卡住问题 | AGI方法 (defagi.com)](https://defagi.com/ai-case/ollama-installation-guide-china/)

于是开始跑超大参数的大模型，目前用着最顺手的ollama部署，类似docker的命令，以及其专为推理所作的优化，太爱了

1. 相关ollama模型下载问题

    走的cloudflare，可以直连，速度很快
2. ollama外网调用的设置问题

    1. ```bash
        sudo vim /etc/systemd/system/ollama.service
        ```
    2. 开始只设置了，Page Assist插件正常使用，以及openai-translator等原生支持ollama的正常使用

        ```bash
        Environment="OLLAMA_HOST=0.0.0.0:11434"
        ```
    3. 后续提出需求，使用浏览器插件进行翻译，但openai-translator中只提供了openai的调用

        * 查询资料了解到，ollama具有openai兼容性，详见（[OpenAI 兼容性 - Ollama中文网](https://ollama.fan/reference/openai/)）
        * 于是乎，使用openai-translator的openai进行调用，无反应，无错误日志，换插件
        * 查到该文章[给沉浸式翻译插件配置本地大模型ollama服务 - 前端黑科技 - SegmentFault 思否](https://segmentfault.com/a/1190000044798180)，配置后出现错误代码403: fetchError，陷入盲区
        * [在 LobeChat 中使用 Ollama · LobeChat Docs · LobeHub](https://lobehub.com/zh/docs/usage/providers/ollama)，拨雾见云，该文章中还配置了OLLAMA_ORIGINS，配置后使用openai调用方法的均正常工作，猜测该选项控制原生调用和openai调用（暂不确定，官方文档对此解释并不准确）

          ```bash
          [Service]
          Environment="OLLAMA_HOST=0.0.0.0:11434"
          Environment="OLLAMA_ORIGINS=*"
          ```
    4. 配置修改后执行

        ```bash
        systemctl daemon-reload #重载daemon文件
        systemctl start ollama #启动ollama服务
        ```
    5. 若仍未生效，则执行

        ```bash
        ps aux | grep ollama #查看当前运行的ollama进程
        kill PID
        ```
3. ollama的使用问题

    使用浏览器插件

    1. [Page Assist - 本地 AI 模型的 Web UI - Chrome 应用商店 (google.com)](https://chromewebstore.google.com/detail/page-assist-%E6%9C%AC%E5%9C%B0-ai-%E6%A8%A1%E5%9E%8B%E7%9A%84-web/jfgfiigpkhlkbnfnbobbkinehhfdhndo)
    2. [openai-translator/openai-translator: 基于 ChatGPT API 的划词翻译浏览器插件和跨平台桌面端应用 - Browser extension and cross-platform desktop application for translation based on ChatGPT API. (github.com)](https://github.com/openai-translator/openai-translator)
    3. [沉浸式翻译 - 双语对照网页翻译插件 | PDF翻译 | 视频字幕翻译 (immersivetranslate.com)](https://immersivetranslate.com/zh-Hans/)（强力推荐，双语对照，非常好用）

4. ollama模型文件目录

    ​`/usr/share/ollama/.ollama/models/blobs`​

    把别的地方下载好的ollama文件丢过去就行。

    可以用python开个http服务器

    ​`python -m http.server 8088 --bind 0.0.0.0`​

    然后wget直接拉整个文件夹

    ​`wget -r -np -N -nH --cut-dirs=1 http://10.0.8.222:8088/`​
