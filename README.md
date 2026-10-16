# n8n vps：年付 10 美元出头的自托管方案，RackNerd 套餐选择与 Docker 部署要点

搜 "n8n vps" 的人，目标基本一致：找一台便宜够用的 VPS，把 n8n 跑在自己手里，告别按执行次数计费。结论先给：RackNerd 的年付特价是这类需求里最便宜的一档方案，主流的 2.5GB 内存档折算下来每月只要两三美元，续费还按原价走；代价是纯自管理——没有预装、没有托管，机房只有美国和欧洲。愿意自己敲几条命令的读者可以放心选；想开箱即用或者在意机房位置的，往下看第四节的对比再说。

**选购速览**

- 大多数人：2.5GB 年付档（2 vCPU / 50GB SSD / 5TB 月流量，约 $27/年），跑 Postgres 加几十条工作流没有压力
- 只想先试试：1GB 入门档，约 $11/年，建议只用来测试
- 高频 webhook 或 AI 类节点：4GB 档起步
- 👉 [查看 RackNerd 当前年付特价套餐](https://bit.ly/my-rn)

顺便给个参照：OliveVPS、LightNode 这类预装 n8n 的月付 VPS，入门配置要 $7.7–8/月，一年下来 $90 往上；配置相近时，RackNerd 的年付价大约是它们的四分之一。差在哪、值不值，第四节展开。

## 自托管 n8n 到底能省多少钱

先说答案：月执行量超过一两千次，自托管就开始明显划算；超过一万次，差距大到不用犹豫。

几个可以直接对数的参照价：Zapier Starter 约 $20–30/月，额度只有 750 个任务；Make Basic $9/月，10,000 ops；n8n Cloud Starter $20/月，2,500 次执行。自托管的 n8n 执行次数不设限，唯一的固定成本就是那台 VPS。

换个角度更能看出按次计费的问题：一条每分钟触发一次的定时工作流，一个月就是 43,200 次执行。放 Zapier 上早就爆额度了，放在一台年付 20 多美元的 RackNerd 上，只是让它多干点活。

| 方案 | 月费 | 执行额度 |
| --- | --- | --- |
| Zapier Starter | 约 $20–30 | 750 个任务 |
| Make Basic | $9 | 10,000 ops |
| n8n Cloud Starter | $20 | 2,500 次执行 |
| RackNerd 2.5GB 年付档 | 折合约 $2.3 | 不限 |

成交点大概在每月 1,000–2,000 次任务：低于这个量，Zapier 的省心值得付费；高于这个量，自托管一个月就把差价省回来了。

👉 [用年付特价开一台自己的 n8n 服务器](https://bit.ly/my-rn)

## RackNerd 套餐怎么选

RackNerd 走的是低价年付路线，官网上长年挂着几档特价套餐，价格重心全在年付上。跑 n8n 的话，多数人选 2.5GB 档就够了；具体价格会随促销活动浮动，下表是长期常见档位的大致参考。

| 套餐 | 内存 / CPU / 硬盘 | 年付参考价 | 适合谁 | 购买 |
| --- | --- | --- | --- | --- |
| 入门 1GB | 1GB / 1 vCPU / 20GB SSD | 约 $11 | 试水、轻量测试 | [ 查看实时价](https://bit.ly/my-rn) |
| 均衡 2.5GB | 2.5GB / 2 vCPU / 50GB SSD | 约 $27 | 多数自托管场景的甜点档 | [ 查看实时价](https://bit.ly/my-rn) |
| 进阶 4GB | 4GB / 3 vCPU / 80GB SSD | 约 $43 | AI 节点、webhook 密集 | [ 查看实时价](https://bit.ly/my-rn) |
| 高配 8GB | 8GB / 4 vCPU / 160GB SSD | 约 $85 | queue mode、团队使用 | [ 查看实时价](https://bit.ly/my-rn) |

以上是年付促销的大致价位，黑五、新年这类节点会临时放出更低或新增的档位，1.5GB、3GB、6GB 这些中间档也会轮换出现，下单前以下单页实时价格为准。

几个影响选择的实情：

- **续费不涨价**。RackNerd 的特价套餐续费按同一价格走，没有首年低价、次年翻倍的套路。按 7 出头的汇率粗算，2.5GB 档一年不到 200 块人民币，摊到每月十几块钱，第三年还是这个价。
- **机房在美国和欧洲**：洛杉矶、圣何塞、西雅图、达拉斯、芝加哥、纽约、亚特兰大，外加荷兰阿姆斯特丹，1Gbps 端口。如果你的工作流主要对接 Stripe、GitHub、OpenAI 这类海外 API，问题不大；在意的是国内打开 n8n 编辑器的速度，美西机房延迟通常在 150ms 上下，能用，但谈不上快。
- **流量按月配额**，入门档 1TB 起。n8n 的 webhook 和 API 调用流量极小，这个配额对自动化场景基本是名义参数，很难触顶。
- KVM 虚拟化，完整 root，Ubuntu、Debian 等系统模板随便选，装 Docker 没有任何障碍。

分档建议说直白点：

- **1GB**：能装能跑，但叠上 Postgres 后内存比较紧，只适合验证流程。
- **2.5GB**：2GB 是自托管 n8n 的实用下限，2.5GB 多出的一点余量刚好够 Postgres 常驻，个人几十条工作流稳稳当当。
- **4GB**：要跑 OpenAI、本地脚本这类吃内存的节点，或者 webhook 触发很密集，再上这一档。
- **8GB**：执行量爬到每天几千次、需要切 queue mode（主进程 + worker + Redis）时才用得上，个人用户暂时不用考虑。

👉 [打开 RackNerd 促销页核对实时价格和档位](https://bit.ly/my-rn)

## 和预装型 n8n VPS 比，RackNerd 差在哪

一句话：RackNerd 赢在价格和续费政策，输在省心程度和机房覆盖。

| 方案 | 配置 | 价格 | 流量 | 备注 |
| --- | --- | --- | --- | --- |
| RackNerd 2.5GB 年付档 | 2 vCPU / 2.5GB / 50GB SSD | 约 $27/年 | 5TB/月 | 纯自管理，需自己装环境 |
| OliveVPS Pro | 2 vCPU / 4GB / 60GB NVMe | $15.99/月 | 2TB/月 | 预装 n8n + Postgres，20+ 机房 |
| LightNode Agency | 2 vCPU / 4GB / 50GB SSD | $14.7/月 | 2TB/月 | 40+ 机房可选，支持按小时计费 |
| BuyVPS S-4 | 2 vCPU / 4GB / 60GB NVMe | $13.99/月（首月 $1） | 不限 | 机房仅阿姆斯特丹和纽约，文档给到 compose 级别 |

差距是实打实的：同样 2 vCPU 配置，RackNerd 一年的钱约等于别人半个多月。换来的省心也有区别——预装型商家开机就有现成的 n8n 加 Postgres，RackNerd 给你的是一台干净的系统，环境全靠自己的命令行。

所以判断标准很简单：

- 预算敏感、会基本 Linux 操作、工作流对接的多是海外服务 → RackNerd 是性价比最高的选择。
- 完全不想碰命令行、看重开机即用 → 预装型商家更合适，多花的钱买的就是省事。
- 需要亚洲机房或按小时计费试水 → RackNerd 这两项都提供不了，LightNode 那类机房覆盖更广的商家更对口。

## 在 RackNerd 上把 n8n 跑起来：部署清单

全程大约半小时，核心是 Docker Compose 加一层反向代理。按顺序走：

1. 开通后装 Ubuntu 22.04 或 Debian 12，SSH 登录，用 ufw 只放行 22、80、443 端口。
2. 安装 Docker 和 compose 插件（官方 apt 源或 get.docker.com 脚本都行）。
3. 生成加密密钥：`openssl rand -hex 32`，存到 VPS 之外的地方。这个 key 用来加密数据库里的所有凭据，**丢了就等于全部已保存的账号密码作废**，务必离线备份。
4. 写 compose 文件，n8n 加 Postgres 16 两个容器。别用默认的 SQLite——断电容易损坏、高负载锁文件、备份麻烦，Postgres 从第一天就用上。
5. 域名加一条 A 记录指向 VPS 的 IP，然后在环境变量里设置 `N8N_HOST` 和 `WEBHOOK_URL`。漏了这步，n8n 生成的 webhook 地址里会带着 localhost，外部服务根本回调不到。
6. Nginx 反代到 `127.0.0.1:5678`，certbot 签一张免费证书。webhook 型工作流记得加 `client_max_body_size 16M`，`proxy_read_timeout` 拉长到 600s，避免大 payload 和长任务被掐断。
7. crontab 加一条每日凌晨的备份任务，把数据卷打包压缩，至少保留 7 天。便宜 VPS 不含自动备份，这步不能省。
8. 以后更新就是 `docker compose pull && docker compose up -d`，半分钟的事。

compose 的关键部分长这样（密码和域名记得替换）：

yaml
services:
  postgres:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: n8n
      POSTGRES_PASSWORD: 换成强密码
      POSTGRES_DB: n8n
    volumes:
      - ./pgdata:/var/lib/postgresql/data

  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    restart: unless-stopped
    depends_on: [postgres]
    ports:
      - "127.0.0.1:5678:5678"
    environment:
      DB_TYPE: postgresdb
      DB_POSTGRESDB_HOST: postgres
      DB_POSTGRESDB_DATABASE: n8n
      DB_POSTGRESDB_USER: n8n
      DB_POSTGRESDB_PASSWORD: 换成强密码
      N8N_ENCRYPTION_KEY: 粘贴第3步生成的key
      N8N_HOST: n8n.example.com
      WEBHOOK_URL: https://n8n.example.com/
    volumes:
      - ./n8n-data:/home/node/.n8n


补一句授权的事：n8n 是 fair-code 授权，公司内部使用、帮客户跑工作流都没问题，只有把它包装成托管服务转售才受限，个人和团队用户不用担心。

## 常见问题

**n8n 至少要多少内存？**
1GB 只够装上看看界面；生产环境 2GB 起步；4GB 更稳。跑 GPT 类节点时内存消耗明显上升，这类工作流建议直接 4GB。

**RackNerd 能装 Docker 吗？**
能。KVM 全虚拟化加完整 root，Ubuntu、Debian 系统模板直接选，装 Docker 和普通服务器没有任何区别。

**不买域名行不行？**
测试阶段可以用 IP 加端口访问编辑器，但生产环境的 webhook 集成需要 HTTPS，而证书需要域名。Stripe、GitHub 这类回调型服务接进来之前，先把域名备好。

**数据安全怎么保障？**
凭据用你自己的加密密钥存在 VPS 的数据库里，不经过第三方；配合每日备份任务，风险主要来自你自己忘备份和丢密钥，这两件事做好了就没什么可担心的。

## 下单前确认两件事

按情况对号入座：预算优先、愿意动手，RackNerd 的 2.5GB 年付档就是最合适的起点；只跑三五条低频工作流，1GB 档先练手也行，后面随时升级；看重预装和托管服务的话，前面提到的预装型商家更对你的胃口。

下单前值得确认的就两件事：一看实时价格——RackNerd 的促销档位和价位随活动轮换，别拿旧截图下单；二选机房——工作流对接的 API 和触发 webhook 的用户在哪儿，机房就往哪儿靠，美西还是美东还是阿姆斯特丹，这一步选对了，后面的延迟体验完全不同。

👉 [去 RackNerd 挑一台年付 VPS，把工作流搬回自己手里](https://bit.ly/my-rn)
