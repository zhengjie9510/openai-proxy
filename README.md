## 1、准备工作

在开始之前，我们需要准备一些东西：
* Docker：这是一个用于容器化应用程序的开源平台。  

## 2、准备Nginx配置文件
完整的配置文件见**nginx.conf**，其核心部分如下：
```conf
server {
    listen 80;  # 监听80端口，用于HTTP请求
    location / {
        proxy_pass  https://api.openai.com/;  # 反向代理到https://api.openai.com/这个地址
        proxy_ssl_server_name on;  # 开启代理SSL服务器名称验证，确保SSL连接的安全性
        proxy_set_header Host api.openai.com;  # 设置代理请求头中的Host字段为api.openai.com
        chunked_transfer_encoding off;  # 禁用分块编码传输，避免可能的代理问题
        proxy_buffering off;  # 禁用代理缓存，避免数据传输延迟
        proxy_cache off;  # 禁用代理缓存，确保实时获取最新的数据
        #proxy_set_header X-Forwarded-For $remote_addr;  # 将客户端真实IP添加到代理请求头中的X-Forwarded-For字段中，用于记录客户端真实IP
    }
}
```

## 3、启动服务

### 3.1、docker run
```
docker run -itd -p 80:80 -v $PWD/nginx.conf:/etc/nginx/nginx.conf --name my-nginx nginx
```

### 3.2、docker compose up
```
docker compose up -d
```

## 4、应用
```python
# Note: you need to be using OpenAI Python v0.27.0 for the code below to work
import openai
openai.api_key = api_key
openai.api_base = "your_proxy_url" # 代理地址,入“http://www.test.com/v1”
openai.ChatCompletion.create(
  model="gpt-3.5-turbo",
  messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Who won the world series in 2020?"},
        {"role": "assistant", "content": "The Los Angeles Dodgers won the World Series in 2020."},
        {"role": "user", "content": "Where was it played?"}
    ]
)
```
