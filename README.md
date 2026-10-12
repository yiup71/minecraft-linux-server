# MC服务器Linux开服：从选机器到开机自启，新手也能跑起一台不崩的我的世界服务器

用自己电脑开服，最常见的结果是这样的：你在线上，世界就在；你一关机，三个人集体掉线，存档停在半空。想解决它没有别的办法，只能把服务端从卧室搬到一台 24 小时不断电、有公网 IP、带宽还行的机器上。

而 Linux 恰好是最适合干这活的系统——占用低、稳定、JVM 的表现比 Windows 好一截，而且几乎所有 MC 服务端的教程和脚本都是以 Linux 为准写的。下面这套流程，从"我该买哪台"一直讲到"服务器开机自动起来"。

## 一、先决定路线：一键面板服，还是自己装 Linux

这两条路不是谁更好，是给不同的人准备的。买之前花两分钟想清楚，能省掉后面很多折腾。

**面板服**是把 Linux 藏起来了。你拿到的是一个网页控制台，选服务端类型（官方原版、Paper、Purpur、Fabric、Forge、NeoForge、Mohist、Folia、BDS 等等，基本覆盖了市面上主流核心）、选版本、点开启，剩下的事平台包了。雨云的面板还额外提供了 SFTP、在线文件管理、备份还原、计划任务和 MySQL 数据库——不少插件要连库才能跑。这类面板基于 K8s 集群，配置能随时升降，CPU 机型甚至可以在 i9 14900KS 和 Platinum 8259CL 之间切换，卡了直接换，不用重新开服。

**游戏云 VPS** 给你的是一台裸的 Linux（也有 Windows 可选），带独立公网 IP 和 SSH。你要自己装 Java、自己下核心、自己写启动脚本。听起来麻烦，但换来的是完全的控制权：想跑什么加载器跑什么，想开几台实例开几台，Dynmap 的网页、反向代理、定时重启脚本都能一起塞进去。

判断标准其实很简单：

|  | 面板服 | 游戏云 VPS（Linux） |
| --- | --- | --- |
| 上手门槛 | 网页操作，基本不用命令行 | 需要会用 SSH 和基础 Linux 命令 |
| 服务端自由度 | 平台内置的核心列表内 | 任意 JAR，包括自编译核心 |
| 多实例 / 附加服务 | 单端为主 | 想开多少开多少 |
| 网络 | NAT 共享 IP + 端口映射 | 可加独立公网 IP |
| 适合谁 | 第一次开服、和朋友 3–10 人联机 | 要跑重型整合包、长期运营、老手 |

如果你看完这篇教程里的 Linux 命令觉得发怵，直接选面板服，不丢人。如果你打算把这台机器长期用下去，Linux 这条路值得走一次。

## 二、Linux 开服前必须知道的三个硬件真相

买错配置比不会装更贵。

**第一，MC 服务端吃的是单核，不是核数。** Java 版服务端的多线程优化一直很差，主线程卡住全服就卡。这就是为什么高主频 CPU（i9 14900K、Ryzen 9950X / 7950X / 5900X）比十六核低频服务器更适合开 MC。虚拟机里 4 个 4.9GHz 的核，跑 MC 会比 8 个 2.2GHz 的核流畅得多。

**第二，内存要按整合包算，不是按人数算。** 一个 100+ 模组的整合包，光加载就吃掉好几个 G。参考量级：原版纯净服 2 核 6G 够 2–6 人；50 模组以内的轻量整合包 4 核 12G；ATM9、ATM10 这种中型整合包 6 核 24G；海上余生、愚者这类超重整合包就得上 8 核 32G。

**第三，10M 带宽对十几个人是够的。** MC 的带宽消耗远小于你的想象，真正的瓶颈是 TPS。选线路时优先看延迟（离你和队友所在地近的机房），其次看高防——公开服被 DDoS 打是家常便饭。

## 三、选机器：雨云游戏云的机型与价位

雨云（RainYun）2018 年成立，是做国产自研财务和产品系统的那一类云厂商，产品线里有专门的游戏云。它的游戏云用高主频 CPU + DDR4 内存 + NVMe 硬盘，分了面板服和 VPS 两种形态，区域覆盖宁波、宿迁、镇江、温州、绍兴、重庆以及香港、首尔、法兰克福、洛杉矶等。

面板服的档位大致是这样的（价格随区域、带宽、库存浮动，下单页显示多少就是多少）：

| 机型档位 | 典型配置 | 参考月付 | 适合场景 | 购买 |
| --- | --- | --- | --- | --- |
| Platinum 8259CL 入门 | 2核 4G / 50M / 10G | 约 25 元/月 | 原版纯净服，5 人以内 | [ 领 5 折券开一台入门面板服试试](https://www.rainyun.com/MzA2MTc2_) |
| Platinum 8259CL 进阶 | 4核 8G / 50M / 10G | 约 65 元/月 | 轻量整合包，10 人以内 | [ 查看当前实时价格并下单](https://www.rainyun.com/MzA2MTc2_) |
| Platinum 8259CL 高端 | 6核 12G / 50M / 10G | 约 120 元/月 | 中型整合包，15 人档位 | [ 按这个配置购买](https://www.rainyun.com/MzA2MTc2_) |
| Epyc 7A23 | 2核 4G 起，可到 6核 12G | 约 45–160 元/月 | Zen3，性能接近 5900X，性价比档 | [ 查看 7A23 机型与库存](https://www.rainyun.com/MzA2MTc2_) |
| Ryzen 9950X | 2核 4G 起，可到 6核 12G | 约 90–270 元/月 | Zen5，高版本 / 重型整合包 / 基岩版 | [ 查看 9950X 机型](https://www.rainyun.com/MzA2MTc2_) |
| i9 14900KS | 2核 4G 起 | 约 90 元/月起 | 主频 6.2GHz，1.20+ 高消耗场景 | [ 查看 14900KS 机型](https://www.rainyun.com/MzA2MTc2_) |
| E5 2666v3 | 2核 4G / 10M | 约 65 元/月 | 最低成本起步，人少够用 | [ 用最低价开一台试水](https://www.rainyun.com/MzA2MTc2_) |

如果你走 Linux VPS 路线，同一批 CPU 会有独立公网 IP 和更灵活的带宽/盘配置，价格随区域差异明显。几个有公开评测数据的区域参考：

| 区域与机型 | 参考配置 | 参考月付 | 备注 |
| --- | --- | --- | --- |
| 宁波 · Epyc 7A23 | 4核 8G / 15M | 约 78 元/月（年付 + 券后） | 100G 高防，可加独立高防 IP |
| 镇江 · i9 14900K BGP | 4核 8G / 15M | 约 123 元/月 | 200G 高防，BGP 线路三网通吃 |
| 温州双线 · i9 14900K | 4核 8G / 15M | 约 112 元/月（年付 + 券后） | 电信 + 移动双线，100G 高防 |
| 宿迁 · Ryzen 7950X | 4核 8G / 15M | 约 153 元/月（年付七折约 107） | 150G 防御，5.7GHz 主频 |
| 宿迁 · Ryzen 9950X | 4核 16G / 15M | 约 315 元/月（券后） | 150G 防御，重型整合包首选 |

> 上面所有数字都受区域、带宽、硬盘、IP 类型和库存影响，属于参考区间。真实价格以下单页实时显示为准，别拿任何第三方文章里的数字当合同。

### 计费方式：固定还是动态，这是省钱的关键

雨云有两种计费模式。**固定计费**就是传统包月，24 小时开机成本固定，年付有 7 折。

**动态计费**的基础月租明显更低，但会按 CPU 超额使用量日结一笔"CPU 电量"。规则是：CPU 使用率基准值 20%，低于 20% 不耗电，超出部分才计费；内存、硬盘、带宽不产生电量费用。不同 CPU 的电量单价不一样——E5 2666v3 是 0.0005 元/电量，Gold 6146 和 Platinum 8275CL 是 0.0006 元，Ryzen 5900X 是 0.0008 元，9950X / 7950X / 13900K / 14900K 这类高主频型号是 0.001 元。

怎么选？一句话：**服是"常开"还是"偶尔开"。**

- 每天有人在线超过 2 小时、长期运营 → 固定计费。生存服的 CPU 占用基本压不到 20% 以下，动态计费天天烧电量，一个月下来反而不便宜。
- 只有周末几个人上来玩 → 动态计费。闲置时段只付很低的基础月租。

雨云还提供**1 元试用一天**（部分较贵产品试用价会高一档）和**7 天无理由退款**（每个账号限 3 次）。正确的操作顺序是：先 1 元试用测延迟 → 满意了月付 → 确定长期才去年付。反过来先买一年再发现延迟 200ms，退起来就折腾了。

新用户从推广入口注册并绑定微信，通常能领到首月 5 折券，搭配年付 7 折会更低。👉 [👉 用这个入口注册，先领券再试用一天](https://www.rainyun.com/MzA2MTc2_)

## 四、手把手：在 Linux VPS 上开一台 Paper 服务器

下面以 Ubuntu 22.04 / 24.04 为例。选系统时别选 CentOS 7，它太老，装 MCSManager 之类的现代面板会直接提示系统版本过低。

**第 1 步：登录并更新系统**

bash
ssh root@你的服务器IP
apt update && apt upgrade -y


**第 2 步：装 Java**

版本对应关系很关键：MC 1.18 到 1.20.4 用 Java 17；1.20.5 之后的版本需要 Java 21。

bash
apt install -y openjdk-21-jre-headless wget curl jq ufw
java -version   # 确认输出是 21.x


**第 3 步：建一个专用账号，别用 root 跑 MC**

服务端一旦被利用漏洞，root 权限的后果是整台机器。正确做法是建一个无登录 shell 的系统用户：

bash
useradd -r -m -U -d /opt/minecraft -s /usr/sbin/nologin minecraft
install -d -o minecraft -g minecraft /opt/minecraft/paper


**第 4 步：下载 Paper 核心**

注意一个变化：Paper 旧的下载接口 `api.papermc.io/v2` 已经下线并返回 410，现在要走 `fill.papermc.io`。每个响应都会带 SHA-256 校验值，下完顺手对一遍：

bash
PAPER_URL=$(curl -s "https://fill.papermc.io/v3/projects/paper/versions/1.21.4/builds/latest" \
  -H "accept: application/json" | jq -r '.downloads["server:default"].url')
curl -fsSL -o /opt/minecraft/paper/paper.jar "$PAPER_URL"
chown minecraft:minecraft /opt/minecraft/paper/paper.jar


想要别的核心（Fabric、Forge、Purpur）就去各自的官方下载页拿 JAR，流程完全一样。

**第 5 步：接受协议、改配置**

第一次启动会生成 `eula.txt` 和 `server.properties`，然后退出：

bash
cd /opt/minecraft/paper
sudo -u minecraft java -Xms2G -Xmx2G -jar paper.jar nogui


把 `eula.txt` 里的 `eula=false` 改成 `true`。然后打开 `server.properties`，几个值值得动：`online-mode=true`（正版验证，关掉意味着任何人拿任意 ID 都能进，风险自己衡量）、`max-players`、`view-distance` 和 `simulation-distance`（调小是提升 TPS 最快的手段）、`motd`。

**第 6 步：用 systemd 让它常驻并开机自启**

用 `screen` 或 `nohup` 开服的问题是：SSH 一断、机器一重启，服就没了。这是 Linux 开服和面板服最大的体验差，而 systemd 能补上：

ini
# /etc/systemd/system/minecraft.service
[Unit]
Description=Minecraft Paper Server
After=network.target

[Service]
User=minecraft
WorkingDirectory=/opt/minecraft/paper
ExecStart=/usr/bin/java -Xms4G -Xmx4G -XX:+UseG1GC -XX:+ParallelRefProcEnabled -jar paper.jar nogui
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target


bash
systemctl daemon-reload
systemctl enable --now minecraft
systemctl status minecraft      # 看是否 active
journalctl -efu minecraft       # 实时看日志，Ctrl+C 退出


`-Xms` 和 `-Xmx` 建议设成同一个值（这里按你的内存调整，别超过物理内存的 70%），避免运行中反复扩堆造成卡顿。`Restart=on-failure` 保证服崩了会自动拉起来。

**第 7 步：放行端口**

MC Java 版默认端口是 25565。如果只开基岩版，端口是 19132。

bash
ufw allow 25565/tcp
ufw reload


雨云的 VPS 如果有独立的云平台安全组，也要在控制台里放行同一端口。很多人卡在"服务器明明 Done 了却连不上"，九成是这两道防火墙只开了一道。

**第 8 步：进游戏**

客户端 → 多人游戏 → 添加服务器，地址填 `IP:25565`。进去之后在控制台给自己 OP：


op 你的游戏ID
whitelist add 好友的游戏ID


开着白名单跑公开服，比事后补救省事得多。

## 五、跑得久才是真开服：三个运维习惯

**定时备份。** 存档丢了是不可逆事故。一条 cron 搞定，保留 7 天：

bash
0 4 * * * tar -czf /opt/minecraft/backup/world_$(date +\%F).tar.gz \
  -C /opt/minecraft/paper world world_nether world_the_end


顺手把备份目录设成服务器根目录之外的路径，雨云的面板服在做"重装/切换游戏"时会保留根目录下的 backup 目录，这个细节值得留意。

**别让虚拟机睡过去。** 长时间无人进入的世界会被卸载区块，看起来是"性能变好了"，其实是玩家一上线就开始疯狂加载。如果服是公开的，保持视距参数别开太大，稳定比好看重要。

**升级要停服。** 无论是换核心版本还是升 Java，都是停服 → 备份 → 替换 JAR → 起服 → 看日志四步。热替换 JAR 的后果通常是一个说不清原因的 `NullPointerException`。

## 六、几个真会踩的坑

**"面板服能一键开服，我为什么还要学 Linux？"** 这不是必选项。如果你只是和几个朋友玩，面板服的性价比和上手速度都更好，而且它同样基于 K8s 支持升降配、切 CPU 机型、MySQL 和计划任务。Linux 的价值在于自由度和附加服务，不在于"更专业"。

**"动态计费一定更便宜吗？"** 不一定。生存服 CPU 长期高于 20%，动态计费的电量费会把基础月租的差价吃掉还倒贴。判断依据是你的服日均在线时长是否超过 2 小时。

**"基岩版要注意什么？"** Bedrock 专属服务端在 Linux 上官方只支持 Ubuntu（22.04 LTS 或更高），其他发行版不在支持范围内，别在这上面省事。

**"CentOS 7 还能用吗？"** 纯命令行操作可以，但装 MCSManager 这类现代面板会提示系统版本过低。新机器直接用 Ubuntu 或 Debian。

**"低配机器能开整合包服吗？"** 一个 200+ 模组的整合包，2 核 4G 大概率在加载阶段就 OOM。与其省这点钱，不如从 4 核 8G 起步，或者先花 1 元试用一天看看到底吃多少内存。

**"要不要买独立公网 IP？"** NAT 机型也能开服，靠端口映射对外，但端口号和地址会不固定，朋友每次都得改配置。长期运营、要做域名解析的服，独立 IP 值得加钱。

## 七、怎么选，一句话

- 第一次开服、3–10 人、不想碰命令行 → 面板服，Platinum 8259CL 或 Epyc 7A23 档位起步，先试用一天测延迟。
- 高版本（1.17+）、几十个模组 → 上 i9 14900K 或 Ryzen 9950X 这类高主频机型，4 核 8G 起步。
- 要跑重型整合包、要开多台实例、要挂 Dynmap 和数据库 → 游戏云 VPS 装 Ubuntu，按上面的 systemd 流程走一遍，以后换机器也就是复制一遍配置文件的事。

先把机器开出来，剩下的都是照着命令敲。👉 [👉 从这里进雨云开一台，1 元试用一天，满意再决定买多久](https://www.rainyun.com/MzA2MTc2_)
