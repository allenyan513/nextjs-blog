---
title: '如何在10分钟内在Amazon EC2带有GPU的实例上部署Ollama Server'
date: '2024/03/02'
lastmod: '2024/03/02'
tags: ['Ollama', 'Amazon EC2', 'GPU']
draft: false
summary: '本指南非常适合个人或者小团队在云中搭建自己的开源本地大语言模型 (LLM) AI 功能，确保数据隐私、可定制性、法规遵从性和经济高效的 AI 解决方案。'
images:
  [
    'https://opengraph.githubassets.com/62354a5ca230d774e2b2d3e9ce07f5a0612861511fb8416d1fcf42f31bf704f5/ollama/ollama',
  ]
authors: ['default']
layout: PostLayout
---

欢迎阅读有关如何 10 分钟内在 [Amazon EC2](https://aws.amazon.com/cn/pm/ec2/?gclid=CjwKCAiA3JCvBhA8EiwA4kujZqdhMCkcDk1QTHnaB4Ikue7seBVHJtedE84crA-Kh0lAMGGIzCD3HxoClbgQAvD_BwE&trk=8c0f4d22-7932-45ae-9a50-7ec3d0775c47&sc_channel=ps&ef_id=CjwKCAiA3JCvBhA8EiwA4kujZqdhMCkcDk1QTHnaB4Ikue7seBVHJtedE84crA-Kh0lAMGGIzCD3HxoClbgQAvD_BwE:G:s&s_kwcid=AL!4422!3!472464674288!e!!g!!amazon%20ec2!11346198414!112250790958) 带有 GPU 的实例上部署 Ollama Server 的综合指南。
本指南非常适合个人或者小团队在云中搭建自己的开源本地大语言模型 (LLM) AI 功能，确保数据隐私、可定制性、法规遵从性和经济高效的 AI 解决方案。
在本指南中，我们将逐步完成该过程，提供详细的说明和参考以确保顺利设置。

## LLM 和 Ollama

要部署一个大语言模型并且对外提供服务，不仅需要有高端 GPU，而且还需要有广泛的技术专业知识，这使得很多人只能依赖于大公司提供的 API 服务，如 OpenAI 的 GPT-3 或者 Google 的 BERT。

然而[llama.cpp](https://github.com/ggerganov/llama.cpp)这个开源项目通过优化 CPU 和简化的部署方式实现了 AI 的'民主化',也就是说个人开发者或者小团队也能够在云端部署自己的 AI 服务。

[Ollama](https://ollama.com/) 是基于 llama.cpp 的一个项目，它提供了一个更加友好的 API 接口，可以直接通过 HTTP 请求来调用 AI 服务。
它支持在 macOS, Linux, Windows 上部署，也支持在云端部署，而且支持 GPU 加速。

## 在 EC2 上部署 Ollama

### Step 1: 创建 EC2 实例

以下的配置列表是带有 GPU 加速的 EC2 实例的配置，虽然 Ollama 也支持只有 CPU 来运行，但是推理速度会比较慢，所以我们选择带有 GPU 的实例。

注意：当你申请创建 G 类实例后，可能会收到 AWS 的创建失败通知，要求你先申请额度，通知里会附带链接，申请过程大概需要半天时间。审批通过后，再次创建实例即可。

Configure an Amazon Linux 2 EC2 instance:

- Amazon Machine Image (AMI): Deep Learning OSS Nvidia Driver AMI GPU PyTorch 2.0.1 (Amazon Linux 2) 20240227
- Instance Type: g4dn.xlarge （大约$380/month [预计费用](https://calculator.aws/#/))
- vCPU: 4
- RAM: 16GB
- EBS: 50GB (gp3)

![img.png](/static/images/img.png)

![img_1.png](/static/images/img_1.png)

### Step 2: 安装部署 Ollama

我们使用 ssh 的方式登录到 EC2 实例上，然后执行以下命令来安装 Ollama。

```bash
ssh -i "Key1.pem" ec2-user@ec2-3-228-24-35.compute-1.amazonaws.com
```

![img_2.png](/static/images/img_2.png)

查看 Nvidia 驱动是否安装成功

```bash
nvidia-smi
```

![img_4.png](/static/images/img_4.png)

安装 Ollama 并查看服务状态. 如图所示，如果日志中提醒`Nvidia GPU Detected`，这意味着 Ollama 已经成功检测到了 GPU。

```bash
curl -fsSL https://ollama.com/install.sh | sh
sudo systemctl status ollama
```

![img_3.png](/static/images/img_3.png)

### Step 3: 运行 llama2 模型

Ollama 安装完成后，我们可以尝试运行一个 llama2 模型来测试 Ollama 服务是否正常，是否已经启动了 GPU 加速

```bash
ollama run llama2
```

![img_5.png](/static/images/img_5.png)

### Step 4: 安装配置 Nginx

接下来我们计划通过安装 Nginx 来做反向代理，将 80 端口的请求转发到 11434 端口, 对外提供服务。

```bash
sudo amazon-linux-extras install nginx1
```

```bashe
sudo vim /etc/nginx/conf.d/myapp.conf
```

```bash
server {
    listen 80;

    location / {
        proxy_pass http://127.0.0.1:11434;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

重启 Nginx，使配置生效

```bash
sudo systemctl restart nginx
```

### Step 5: 部署 open-webui

[open-webui](https://github.com/open-webui/open-webui) 是一个开源的前端项目，它提供了一个友好的界面来调用 Ollama 服务。

我们现在用在本地使用 docker 来快速部署一个 open-webui 服务，指定 Ollama 的 url，来测试一下效果.
确保你的本地已经安装了 docker，然后执行以下命令

将其中的`https://x.x.x.x/api` 的 x.x.x.x 替换成你的 EC2 实例的公网 IP 地址

```bash
docker run -d -p 3000:8080 -e OLLAMA_API_BASE_URL=https://x.x.x.x/api -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```

### 总结

在 AWS EC2 上能够快速部署一个带有 GPU 的 Ollama 服务，非常重要的一个点就是 AWS 提供了预先配置好的 AMI
`(Deep Learning OSS Nvidia Driver AMI GPU PyTorch 2.0.1 (Amazon Linux 2) 20240227)`
这个 AMI 预先安装了 Nvidia 的驱动，省去了我们很多安装配置的时间, 几乎和 Ollama 完美切换，开箱即用。

另外一个重点就是 Nginx 的配置，通过 Nginx 的反向代理，我们可以将 Ollama 服务对外提供服务，而且可以通过 Nginx 的配置来做负载均衡，限流等，后续还可以增加身份认证，日志记录等功能。

### 广告时间

[mistreeai.com](https://mistreeai.com) 是一个 AI 人物聊天网站，无审查无过滤的，在这里你可以和数百个真实或者虚拟人物展开有创造力和有趣的对话

![img.png](/static/images/homepage.png)
