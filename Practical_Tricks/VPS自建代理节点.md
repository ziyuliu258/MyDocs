## 方案
- `VLESS-TCP-XTLS-Vision-REALITY` from [Xray-examples](https://github.com/XTLS/Xray-examples)
- 香港优质BGP服务器 from [野草云](https://www.yecaoyun.com/)
### 方案解释
`VLESS-TCP-XTLS-Vision-REALITY` 不是一个单一的技术，而是一套**组合拳**。我们可以把它拆解成四个层级来理解：
#### 1. VLESS（核心代理协议：极简的货物装载规则）
- **它的角色：** 决定你的数据（比如你看的 YouTube 视频）在发出前如何被打包。
- **它的特点：** “Less is more”（少即是多）。老一代协议（如 VMess、Shadowsocks）会自带复杂的加密算法，导致数据包头部很臃肿。VLESS 放弃了自带加密，它假设下层通道已经是绝对安全的，只负责最纯粹的数据转发。
- **优势：** 性能极高，没有多余的代码开销，且绝对不会在数据包里留下自己的“特征指纹”。
#### 2. TCP（传输层：标准的高速公路）
- **它的角色：** 这是网络七层模型中的传输层，是你电脑和香港 VPS 之间运送数据的物理通道。
- **为什么选它：** TCP 是互联网上最普遍、最不引人注目的协议。相比于容易被运营商 QoS（限速）的 UDP 协议，TCP 在 163 骨干网上跑得更稳定。
#### 3. REALITY（安全与伪装层：终极的“人皮面具”）
这是这套架构的**灵魂**。以前我们要防封，得自己买个域名，假装建个博客，还要自己申请 TLS 证书。
- **它的角色：** TLS 握手劫持与目标消除。
- **它的原理：** 当防火墙（GFW）探测你的 VPS 时，REALITY 会直接截获这个探测包，然后去向真正的微软（或苹果）服务器请求一份真实的 TLS 证书发给防火墙。
- **通俗比喻：** 防火墙是个门卫，查你 VPS 的身份证。REALITY 并没有自己造假证，而是用极其精妙的手段，从真正的微软员工那里“偷”了一张真工牌给门卫看。门卫一扫，发现这真的是微软的服务器，就放行了。**你不需要买域名，不需要配 Nginx，却拥有了最高级别的伪装。**
#### 4. XTLS-Vision（流控机制：反机器学习的“隐形披风”）
即使你戴了微软的面具（REALITY），如果你的行为举止不像，依然会被识破。
- **它的角色：** 流量特征消除（TLS in TLS 消除）。
- **防什么：** 防火墙现在不仅看面具，还用 AI（机器学习）分析你发包的“节奏”。代理流量在建立连接时，数据包的大小和顺序（比如先发 500 字节，再发 100 字节）与普通人看网页是完全不同的。
- **怎么防：** `Vision` 会在你的数据包里动态塞入“随机沙子”（Padding），强制把你发包的大小、顺序改变，让它在 AI 眼里，和普通人正常访问微软网页的流量**在数学统计上完全一致**。
## 步骤
### 服务器配置
#### 测试回程路由
```bash
curl -L https://raw.githubusercontent.com/sjlleo/nexttrace/main/nt_install.sh | bash
nexttrace [你的本地公网IP]
```
#### BBR加速
```bash
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
lsmod | grep bbr #检查是否跑通
```
**BBR**（Bottleneck Bandwidth and Round-trip propagation time）是 Google 开发的一种 **TCP 拥塞控制算法**。
**BBR 算法逻辑是，** 它不以“丢包”作为减速的唯一标准。它会实时测量这条路的**实际宽度（带宽）和往返时间（延迟）**。
- 即使路面有点小坑（轻微丢包），只要路还没堵死，BBR 就会继续全速前进。
- 它能保证你的 VPS 发送数据时，始终填满那条线路的物理极限。
#### 设置防火墙规则
放行`TCP`的`8443`端口。用VPS官网的设置面板操作，详细设定如下（后面目的端口改成`8443`了）：
![](../attachments/Pasted%20image%2020260222135636.png)
#### 安装 Xray 核心程序
```bash
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
```
#### 生成核心凭证
- 生成专属账号 (UUID)
	```bash
	xray uuid
	```
- 生成 REALITY 伪装密钥对
	```bash
	xray x25519
	```
- 生成防探测短码 (ShortID)
	```bash
	openssl rand -hex 8
	```
#### 编写 JSON 配置文件
现在要把生成的各种凭证装进服务器里。Xray 的默认配置文件路径是 `/usr/local/etc/xray/config.json`。
打开配置文件，把内容替换为以下内容：
```json
{
  "inbounds": [
    {
      "port": 8443,
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "你的UUID",
            "flow": "xtls-rprx-vision"
          }
        ],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "tcp",
        "security": "reality",
        "realitySettings": {
          "show": false,
          "dest": "www.nvidia.com:443",
          "xver": 0,
          "serverNames": [
            "www.nvidia.com"
          ],
          "privateKey": "你的Private key",
          "shortIds": [
            "你的ShortId"
          ]
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}
```
#### 重启服务
```bash
# 重启 Xray 服务
systemctl restart xray

# 检查它是否正常运行
systemctl status xray
```
### 客户端配置
我的目标是制作成一个可以直接导入到`v2rayN`之类的代理软件的配置链接。
#### 拼接专属 VLESS 链接
```
vless://[你的UUID]@[你的VPS公网IP或ziyuliu258.cn]:8443?type=tcp&security=reality&pbk=[你的PublicKey]&fp=chrome&sni=www.nvidia.com&sid=[你的ShortID]&flow=xtls-rprx-vision#我的香港自建节点
```
#### 粘贴导入
如题。
### 缺陷
由于这是一个香港VPS，我也没有琢磨出来如何套上美国的WARP去正常访问Gemini、ChatGPT、Claude等服务，折腾了一天也没折腾好，所以它的用途十分有限，几乎只能刷刷油管。但是是一次很好的尝试。