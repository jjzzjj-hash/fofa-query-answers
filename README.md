# FOFA 网络空间测绘查询题解（M001–M100）

> 题目来源：`题目.docx`　｜　整理日期：2026-11-19　｜　共 100 题

本文把 100 条自然语言需求转换为 FOFA 查询语句。查询语句按 FOFA 网页端常用语法书写；字段是否可用、返回量大小以及部分高级字段（证书校验、云标签、哈希、通配符等）会受账号套餐和 FOFA 数据版本影响。文中给出的是**可执行的查询构造**，不是对互联网的实时结果统计。

> ⚠️ 请仅对授权范围内的目标使用，并遵守 FOFA 服务条款及当地法律法规。本文用于教学与授权的资产梳理。

## 下载

- [Word 版（含表格排版）](./FOFA_answers_M001-M100.docx)
- [PDF 预览](./FOFA_answers_M001-M100.pdf)

**通用规则**：`&&` 与，`||` 或，`()` 分组，`!=` 不包含/不等于，`==` 完全匹配，`*=` 启用通配符模糊匹配（`*` 匹配 0+ 字符，`?` 恰好 1 个字符）。多个 OR 条件务必加括号。

## 目录

- [字段速查](#字段速查)
- [答案速查表](#答案速查表)
- [重难点与不可查询题解析](#重难点与不可查询题解析)
- [统计与参考](#统计与参考)

<a id="字段速查"></a>
## 字段速查

| 字段/语法 | 示例 | 说明 |
| --- | --- | --- |
| ip | `ip="1.1.1.1"、ip="1.0.0.0/8"` | 单 IP / CIDR 网段；不支持连字符区间 |
| port | `port="443"` | 端口；也可用 port*="????" 按位数模糊匹配 |
| domain / host | `domain="example.com"、host="admin"` | domain 为主域，host 为主机名/URL |
| title/body/header | `title="登录"、body="keyword"` | 标题/正文/响应头；banner 为协议响应原文 |
| cert / cert.* | `cert="TrustAsia"、cert.subject.cn="qq.com"` | 证书全文、持有者、颁发者、域名、序列号等 |
| protocol/base_protocol | `protocol="ssh"、base_protocol="tcp"` | 应用层协议 / 底层承载协议 |
| app/product/category | `app="Grafana"、product.version="10.0.0"` | 应用指纹 / 产品版本 / 产品门类 |
| geo | `country="JP" && region="Bavaria" && city="Munich"` | 国家 / 省州 / 城市，值用英文名 |
| org/asn/cloud_name | `asn="4134"、cloud_name="Amazon"` | 网络组织 / ASN / 云厂商 |
| tls.* / jarm | `tls.version="TLS 1.3"、tls.ja3s="..."、jarm="..."` | TLS 版本、JA3S、JARM 指纹 |
| icon_hash/fid/hashes | `icon_hash="-1493334710"` | 图标哈希、FID、响应头/正文哈希 |
| 标记/布尔字段 | `is_domain="false"、is_ipv6="false"` | 是否有域名/IPv6、是否云、是否蜜罐/欺诈 |
| 逻辑运算符 | `&&、\|\|、()、!、=、==、!=、*=、?、*` | OR 组务必加括号 |

<a id="答案速查表"></a>
## 答案速查表

| 题号 | 需求摘要 | FOFA 查询语句 | 要点 |
| :--- | :--- | :--- | :--- |
| **M001** | 查询 IP 地址为 20.247.40.92 的资产 | `ip="20.247.40.92"` | ip 字段按单个 IPv4 地址精确检索。 |
| **M002** | 查询 124.255.254.0/24 网段内的全部资产 | `ip="124.255.254.0/24"` | ip 字段支持 CIDR 网段写法。 |
| **M003** | 查询主域是 zendesk.com 的全部资产 | `domain="zendesk.com"` | domain 是根域名/主域字段，会覆盖其下子域资产。 |
| **M004** | 搜索网站名为 www.amazon.com 的资产 | `host="www.amazon.com"` | host 是按主机名/URL 检索；这里用具体站点名，而不是根域。 |
| **M005** | 找域名里带有 vpn 的所有网站 | `host="vpn"` | host 为包含匹配；若只限定根域含 vpn，可写 domain="vpn"。 |
| **M006** | 搜索英国的资产 | `country="GB"` | 英国的国家代码为 GB（大不列颠及北爱尔兰联合王国）。 |
| **M007** | 搜索韩国江原道的资产 | `country="KR" && region="Gangwon"` | 区域字段使用英文名；部分数据版本可能写作 Gangwon-do。 |
| **M008** | 搜索德国巴伐利亚州慕尼黑市的资产 | `country="DE" && region="Bavaria" && city="Munich"` | 地理字段三层叠加；München 的英文检索值为 Munich。 |
| **M009** | 搜索自治系统号为 4134 的资产 | `asn="4134"` | asn 字段按自治系统号检索。 |
| **M010** | 荷兰境内、使用亚马逊云计算/对象存储/应用托管服务的资产 | `country="NL" && cloud_name="Amazon"` | cloud_name 是云厂商标签，AWS 对应 Amazon；如标签不可用，可用 asn="16509" 兜底。 |
| **M011** | 查河北地区的华为云资产 | `country="CN" && region="Hebei" && cloud_name="HuaWeiCloud"` | 华为云的云标签值常见为 HuaWeiCloud（以当前指纹库为准）。 |
| **M012** | 查询开放 3000 端口的资产 | `port="3000"` | port 字段按端口号检索。 |
| **M013** | 标题包含 Jenkins 或正文包含 jenkins、国家不是中国、更新时间在 2026-08-01 之后 | `(title="Jenkins" \|\| body="jenkins") && country!="CN" && after="2026-08-01"` | OR 条件必须加括号；after 表示最后更新时间晚于指定时间。 |
| **M014** | 找中国的 DNS 服务 | `country="CN" && protocol="dns"` | protocol 字段识别应用层协议，dns 为小写协议名。 |
| **M015** | FOFA 有识别 SOCKS 的 v5 版本吗？ | `protocol="socks5"` | 有。SOCKS5 已被作为协议指纹识别；如需无认证 SOCKS5，可再叠加指纹文本 Version:5 Method:No Authentication(0x00)。 |
| **M016** | 搜索 SSH 协议且端口为 22022 的资产 | `protocol="ssh" && port="22022"` | 应用层协议与端口双条件。 |
| **M017** | 查看使用 TCP 协议的数据 | `base_protocol="tcp"` | base_protocol 表示底层承载协议（tcp/udp）。 |
| **M018** | 找互联网上提供 SNMP 网络管理服务的资产，不限定端口 | `protocol="snmp"` | 不写 port 即不限定端口。 |
| **M019** | 搜索 Memcached 缓存服务，地域限定美国 | `protocol="memcached" && country="US"` | protocol 值为小写协议名。 |
| **M020** | 查询真正跑在 TCP 上的 FTP 服务 | `protocol="ftp" && base_protocol="tcp"` | 同时限定应用层 ftp 与承载层 tcp。 |
| **M021** | 查询真正的 SSH 服务，且返回信息包含 OpenSSH_10.4 | `protocol="ssh" && banner="OpenSSH_10.4"` | banner 是服务响应原文，用于匹配 SSH 版本串。 |
| **M022** | 找日本的海康威视摄像头资产 | `country="JP" && app="HIKVISION-视频监控"` | app 是 FOFA 应用指纹；指纹库版本不同，名称也可能是 Hikvision。 |
| **M023** | 查 163.com 旗下恰好两级、每级一个字符的子域，例如 a.z.163.com | `domain="163.com" && host*="?.?.163.com"` | *= 启用通配符；? 精确匹配一个字符，两个 ? 即两级单字符子域。 |
| **M024** | 找产品版本号为 10.0.0 的资产，不限定具体产品 | `product.version="10.0.0"` | product.version 是产品版本字段，可不绑定具体产品名。 |
| **M025** | 按产品门类查找中间件（连接应用与底层基础服务的中间层软件） | `category="中间件"` | category 按 FOFA 产品门类检索；门类值以当前指纹库为准。 |
| **M026** | 搜索网页正文中出现 NoSuchUpload 错误标记的资产 | `body="NoSuchUpload"` | body 匹配 HTML 正文内容。 |
| **M027** | 找响应信息中出现 Content-Security-Policy-Report-Only 的资产 | `header="Content-Security-Policy-Report-Only"` | 这是 HTTP 响应头字段，用 header 检索。 |
| **M028** | 搜索状态码为 308 的网站类资产 | `status_code="308" && type="subdomain"` | status_code 是 HTTP 状态码；type=subdomain 限定网站类数据。 |
| **M029** | 查询响应信息有 SSH-2.0-OpenSSH_6.1 的资产 | `banner="SSH-2.0-OpenSSH_6.1"` | banner 保留完整协议返回串。 |
| **M030** | 搜索 header 包含 Set-Cookie:PHPSESSID 的资产 | `header="Set-Cookie" && header="PHPSESSID"` | 响应头实际常写成 "Set-Cookie: PHPSESSID=..."；拆成两个包含条件更稳妥。 |
| **M031** | 搜索引用 jquery.min.js 文件的网页 | `body="jquery.min.js"` | body 为包含匹配，可覆盖脚本路径。 |
| **M032** | 荷兰阿姆斯特丹、开放 554 端口、响应原文含 200 OK 的资产 | `country="NL" && city="Amsterdam" && port="554" && banner="200 OK"` | 题目明确 200 OK 是服务返回文本，因此使用 banner 而不是 status_code。 |
| **M033** | 查询由 GlobalSign 组织签发、证书还没有过期的资产 | `cert.issuer.org="GlobalSign" && cert.is_expired="false"` | cert.issuer.org 按颁发者组织检索；is_expired=false 表示未过期。 |
| **M034** | 证书持有者名称是 apple.com、但域名不是 apple.com 的资产 | `cert.subject.cn="apple.com" && domain!="apple.com"` | cert.subject.cn 是证书持有者 CN；domain 是资产主域。 |
| **M035** | 查询证书全文中包含 TrustAsia 的资产 | `cert="TrustAsia"` | cert 为整份证书内容的包含检索。 |
| **M036** | 搜索证书持有者组织名称包含 Alibaba 的资产 | `cert.subject.org="Alibaba"` | cert.subject.org 是证书持有者组织。 |
| **M037** | 查 sohu.com 下资产，要求 Banner 有内容但不包含 apache | `domain="sohu.com" && banner!="" && banner!="apache"` | !="" 表示非空；!="apache" 表示不包含该关键词。 |
| **M038** | 证书里面只要有 adobe.com 这个根域名的都找出来 | `cert.domain="adobe.com"` | cert.domain 匹配证书中的域名，包含裸域与子域。 |
| **M039** | 搜索证书持有者名称根域是 qq.com 的资产 | `cert.subject.cn="qq.com"` | 包含检索可覆盖 *.qq.com 的情况。 |
| **M040** | 查询证书包含 godaddy，但根域名不是 godaddy.com 的资产 | `cert="godaddy" && domain!="godaddy.com"` | 证书全文与资产主域同时限制。 |
| **M041** | 证书信息里包含 google.com，排除蜜罐和欺诈网站 | `cert="google.com" && is_honeypot="false" && is_fraud="false"` | 标记类字段用于排除蜜罐与仿冒/欺诈资产。 |
| **M042** | 证书持有者名称包含 example.com，或证书包含 example.com 域名，裸域和子域都要 | `cert.subject.cn="example.com" \|\| cert="example.com"` | cert 全文已覆盖 CN/SAN 等；前一项专门限定持有者名称。 |
| **M043** | 找仍在使用 TLS 1.0、且证书尚未过期的资产 | `tls.version="TLS 1.0" && cert.is_expired="false"` | tls.version 按 TLS 协议版本检索，值中含空格。 |
| **M044** | JARM 为 21d2...0ea、响应码 307、响应头含 Content-Length: 0 的资产 | `jarm="21d21d00021d21d00021d21d21d21dce319294781f0e12867fd06896a000ea" && status_code="307" && header="Content-Length: 0"` | jarm 是主动 TLS 服务端指纹字段。 |
| **M045** | 查操作系统被识别为 linux 的资产 | `os="Linux"` | os 字段按操作系统指纹检索。 |
| **M046** | 中国大陆、没有绑定域名的网站类资产 | `country="CN" && is_domain="false" && type="subdomain"` | CN 指中国大陆；is_domain=false 表示仅 IP 访问。 |
| **M047** | 找证书颁发者与持有者不匹配的资产 | `cert.is_match="false"` | FOFA 不能直接比较 issuer 与 subject 字段；可查询的近似条件是 cert.is_match（证书与资产域名不匹配）。如严格指 issuer≠subject，需导出后自行比对。 |
| **M048** | 找加拿大的资产，要求证书与域名不匹配 | `country="CA" && cert.is_match="false"` | is_match=false 即证书与访问域名不匹配。 |
| **M049** | 查找土耳其 TLS JA3S 指纹为 f4fe...da8 的资产 | `country="TR" && tls.ja3s="f4febc55ea12b31ae17cfb7e614afda8"` | JA3S 字段写作 tls.ja3s。 |
| **M050** | 找网络组织名称里包含 Inc 的资产 | `org="Inc"` | org 是网络组织/ISP 名称，包含匹配。 |
| **M051** | 美国 OVH SAS 网络、2087 端口、标题完整为 WHM 登录 | `country="US" && org="OVH SAS" && port="2087" && title=="WHM 登录"` | title== 表示完全匹配。 |
| **M052** | 查询 .io 域名下部署的 WordPress 资产 | `domain*="*.io" && app="WordPress"` | domain*= 用通配符限定 TLD；app 指纹识别 WordPress。 |
| **M053** | 搜索厦门市 80 端口、网页状态码 200 的资产 | `city="Xiamen" && port="80" && status_code="200"` | 城市英文名为 Xiamen。 |
| **M054** | 查询英国 DigitalOcean 上疑似提供 /api/show 接口的资产 | `country="GB" && cloud_name="DigitalOcean" && body="/api/show"` | cloud_name 锁定云厂商，body 匹配接口路径。 |
| **M055** | 中国大陆、响应信息含 Set-Cookie、正文有 /admin 的资产 | `country="CN" && header="Set-Cookie" && body="/admin"` | 响应头与正文双条件。 |
| **M056** | baidu.com 资产：正文含 Login/admin/Dashboard/管理，或响应头含 Set-Cookie | `domain="baidu.com" && (body="Login" \|\| body="admin" \|\| body="Dashboard" \|\| body="管理" \|\| header="Set-Cookie")` | 注意括号：所有 OR 条件都要被 domain 条件约束。 |
| **M057** | 13.0.0.0/8 内、响应码 200、正文含 Welcome/Sign in/Dashboard 的网页 | `ip="13.0.0.0/8" && status_code="200" && (body="Welcome" \|\| body="Sign in" \|\| body="Dashboard")` | CIDR 加括号分组。 |
| **M058** | 德国与 amazonaws.com 相关：访问地址、主域、CNAME 根域、证书全文、证书域名任一命中 | `country="DE" && (host="amazonaws.com" \|\| domain="amazonaws.com" \|\| cname_domain="amazonaws.com" \|\| cert="amazonaws.com" \|\| cert.domain="amazonaws.com")` | cname_domain 专门用于 CNAME 根域匹配。 |
| **M059** | 143 端口、证书序列号为 1，且证书含 TAK/ATAK/WinTAK/Squad/CoT 任一词 | `port="143" && cert.sn="1" && (cert="TAK" \|\| cert="ATAK" \|\| cert="WinTAK" \|\| cert="Squad" \|\| cert="CoT")` | cert.sn 是证书序列号字段；短序列号容易误报，建议结合其他指纹复核。 |
| **M060** | 113.247.228.0/24 网段内开放 80/443/8080/8443 任一端口 | `ip="113.247.228.0/24" && (port="80" \|\| port="443" \|\| port="8080" \|\| port="8443")` | 网段与端口 OR 组组合。 |
| **M061** | 标题含 Sign in 或 Login、端口 80、不是 IPv6 的资产 | `(title="Sign in" \|\| title="Login") && port="80" && is_ipv6="false"` | is_ipv6=false 排除 IPv6 资产。 |
| **M062** | 新西兰地区开放 80 或 443 端口、使用云服务商的资产 | `country="NZ" && (port="80" \|\| port="443") && is_cloud="true"` | is_cloud=true 表示云厂商资产。 |
| **M063** | Banner 同时包含 IIS 和 1.，或响应头包含 IIS 的资产 | `(banner="IIS" && banner="1.") \|\| header="IIS"` | AND 的优先级高于 OR，仍建议显式加括号。 |
| **M064** | 日本、已绑定域名、443、200、证书有效；标题含 大学/学部/研究 或正文含 入学案内/学生 | `country="JP" && is_domain="true" && port="443" && status_code="200" && cert.is_valid="true" && (title="大学" \|\| title="学部" \|\| title="研究" \|\| body="入学案内" \|\| body="学生")` | 多条件叠加，OR 组统一放在最后并加括号。 |
| **M065** | baidu.com 主域内：标题含系统/平台/登录/管理/API，或子域含 api/admin/dev/test/uat/pre | `domain="baidu.com" && (title="系统" \|\| title="平台" \|\| title="登录" \|\| title="管理" \|\| title="API" \|\| host="api" \|\| host="admin" \|\| host="dev" \|\| host="test" \|\| host="uat" \|\| host="pre")` | 子域用 host 包含匹配；短词可能产生误报。 |
| **M066** | 被归为邮件系统、标题包含 Mail、国家不是美国的资产 | `category="邮件系统" && title="Mail" && country!="US"` | category 按 FOFA 资产分类检索；分类值以指纹库为准。 |
| **M067** | 找中国大陆开放 8888 端口的资产，排除香港、澳门、台湾 | `country="CN" && port="8888"` | FOFA 的 CN 即中国大陆；香港/澳门/台湾分别为 HK/MO/TW，不会计入 CN。 |
| **M068** | 正文包含 Jenkins，但标题不包含同一个关键词的资产 | `body="Jenkins" && title!="Jenkins"` | 两个字段一正一反。 |
| **M069** | 台湾 AS20940、443 端口、TLS 1.3，排除 Let's Encrypt 和 ZeroSSL 签发的证书 | `country="TW" && asn="20940" && port="443" && tls.version="TLS 1.3" && cert.issuer.org!="Let's Encrypt" && cert.issuer.org!="ZeroSSL"` | tls.version 值含空格；排除项按颁发者组织字段。 |
| **M070** | 日本以外的 HFS 文件服务，状态码 200，标题完整等于 HFS /，排除亚马逊云 | `country!="JP" && app="HFS" && status_code="200" && title=="HFS /" && cloud_name!="Amazon"` | title== 为完全匹配；若 cloud_name 对非云资产为空，可补 \|\| cloud_name=""。 |
| **M071** | 查询不包含港澳台的中国 Grafana 资产 | `app="Grafana" && country="CN"` | country=CN 已是大陆范围，港澳台另有国家代码。 |
| **M072** | 以 .edu.ar 结尾的高校网站，只看 80/443，排除 Nginx 和 WAF | `domain*="*.edu.ar" && (port="80" \|\| port="443") && server!="nginx" && app!="WAF"` | server 与 app 分别排除 Web 服务器和 WAF 指纹。 |
| **M073** | IP 1.1.1.1 上已经绑定域名的资产记录 | `ip="1.1.1.1" && is_domain="true"` | is_domain=true 表示该资产有域名绑定。 |
| **M074** | 证书含根域 aliyun.com、证书当前有效、资产本身没有绑定域名 | `cert.domain="aliyun.com" && cert.is_valid="true" && is_domain="false"` | 证书域名、证书有效性、域名绑定状态三项组合。 |
| **M075** | 中国、已绑定域名、响应头含 x-qiniu-zone，主域不是 qiniudn.com | `country="CN" && is_domain="true" && header="x-qiniu-zone" && domain!="qiniudn.com"` | 七牛 CDN 响应头与主域排除。 |
| **M076** | 标题为空、200、fid 和图标哈希非空；访问域名含 admin，根域以 .cn 结尾或 .com 前恰好四个字符 | `title=="" && status_code="200" && fid!="" && icon_hash!="" && host="admin" && (domain*="*.cn" \|\| domain*="????.com")` | =="" 表示存在且为空；? 精确匹配一个字符，????.com 即 .com 前恰好四个字符。 |
| **M077** | 网页源码中包含原样文本 class="container" | `body="class=\"container\""` | FOFA 值用双引号包裹，内部双引号用反斜杠转义；若控制台不支持转义，可改用单引号包裹或拆分关键词。 |
| **M078** | 网页正文包含 JSON 异常片段 "exception": "Symfony\Component\ErrorHandler\Error\FatalError" | `body="\"exception\": \"Symfony\Component\ErrorHandler\Error\FatalError\""` | 保留双引号和反斜杠：双引号加反斜杠转义；若解析器要求再转义反斜杠，则每个反斜杠写成两个。 |
| **M079** | 搜索 40.88.14.254 的整个 B 段 | `ip="40.88.0.0/16"` | B 段即前两段不变，掩码使用 /16。 |
| **M080** | 47.96.0.1 到 47.96.0.5 闭区间，要求开放 80 或 443 端口 | `(ip="47.96.0.1" \|\| ip="47.96.0.2" \|\| ip="47.96.0.3" \|\| ip="47.96.0.4" \|\| ip="47.96.0.5") && (port="80" \|\| port="443")` | FOFA 只支持 CIDR，不支持连字符 IP 区间；5 个地址需逐个枚举。 |
| **M081** | Grafana 资产：响应头哈希、正文哈希、图标哈希、fid 四项中至少两项有值 | `app="Grafana" && ((header_hash!="" && body_hash!="") \|\| (header_hash!="" && icon_hash!="") \|\| (header_hash!="" && fid!="") \|\| (body_hash!="" && icon_hash!="") \|\| (body_hash!="" && fid!="") \|\| (icon_hash!="" && fid!=""))` | 四个字段两两组合共 6 种，穷举即可。 |
| **M082** | 6 个主域的资产或其证书含任一主域；证书须匹配且有效，到期时间在 2024-01-01 00:00:00 至 2026-08-30 18:17:49 之间 | `((domain="kuaihuoyun.com" \|\| domain="wlhyos.cn" \|\| domain="wlhyos.com" \|\| domain="56ctms.com" \|\| domain="xiaokuaikeji.com" \|\| domain="zhihuiwuliu.org.cn") \|\| (cert="kuaihuoyun.com" \|\| cert="wlhyos.cn" \|\| cert="wlhyos.com" \|\| cert="56ctms.com" \|\| cert="xiaokuaikeji.com" \|\| cert="zhihuiwuliu.org.cn")) && cert.is_match="true" && cert.is_valid="true" && cert.not_after.after="2024-01-01 00:00:00" && cert.not_after.before="2026-08-30 18:17:49"` | domain/cert 两个 OR 组并列；证书匹配、有效、到期时间区间为 AND 条件。若日期字段只接受 YYYY-MM-DD，可截断时间部分。 |
| **M083** | 把 Shodan 查询 "Server: nginx" http.title:"Welcome" 转换成 FOFA | `server="nginx" && title="Welcome"` | Server 头映射到 server 字段，http.title 映射到 title。 |
| **M084** | 把 Google Dork intitle:"Grafana" intext:"JavaScript" 转换为 FOFA | `title="Grafana" && body="JavaScript"` | intitle→title，intext→body。 |
| **M085** | urlscan：HTML 包含 login，并且域名包含 bet、casino 或 kasino | `body="login" && (domain="bet" \|\| domain="casino" \|\| domain="kasino")` | 若要覆盖子域，可把 domain 换成 host。 |
| **M086** | 把 Hunter 查询 domain="google" && domain.suffix!="google.com" && web.title="" 转成 FOFA | `domain="google" && domain!="google.com" && title==""` | domain 包含 google，同时排除 google.com 后缀；空标题用 title==""。 |
| **M087** | 把 Censys 查询 host.services.endpoints.http.headers: (key: "server" and value: "Tengine") or ... (key: "x-cache" and value: "MISS") 转换为 FOFA | `server="Tengine" \|\| (header="x-cache" && header="MISS")` | server 头有专用字段；x-cache 用 header 包含检索。 |
| **M088** | 修正：搜索香港地区、Server 精确为 Apache，并排除 ASN 4134；原查询使用中文弯引号 | `asn!="4134" && region="HK" && server=="Apache"` | 原查询问题是中文弯引号；改为 ASCII 直引号。若你的数据中香港用国家字段，可写 country="HK"。 |
| **M089** | 修正：HFS 的 80 端口都要；8080 端口仅状态码为 200 时要 | `(app="HFS" && port="80") \|\| (app="HFS" && port="8080" && status_code="200")` | 原查询有全角引号和非法分隔符，且 OR 优先级会让 80 条件绕过 app 约束；必须分组。 |
| **M090** | 依据 HTTP 响应片段生成查询：Etag、Content-Length 保留值，其余响应头只要求字段名存在，不限定状态码 | `header="Content-Length: 1171" && header="Etag: \"6aa522a3-493\"" && header="Accept-Ranges" && header="Cache-Control" && header="Content-Type" && header="Date" && header="Last-Modified" && header="Server" && header="Vary"` | Cache-Control 出现两次，只查一次字段名即可；若需 HTTP/2 可另加 protocol="http2"。 |
| **M091** | 搜索开放 1000000 端口的资产 | —— | 端口号范围是 0–65535，1000000 非法，FOFA 没有该端口数据；这不是语法能修正的问题。 |
| **M092** | 搜索地址或网段为 1.2.3.4.5 的资产 | —— | 1.2.3.4.5 既不是合法 IPv4，也不是合法 CIDR；FOFA 的 ip 字段不会命中。 |
| **M093** | 查同一条资产记录：国家代码等于 US，同时国家代码又不等于 US | `country="US" && country!="US"` | 逻辑矛盾，结果必然为空；可用于验证解析器对矛盾条件的处理。 |
| **M094** | 帮我查最安全的网站 | —— | FOFA 没有“安全评分”字段，无法直接回答。可给可量化近似：HTTPS、证书有效且匹配、排除蜜罐/欺诈，但“最安全”仍需额外风险评估。 |
| **M095** | 找出容器内存使用率超过 80% 的服务器（没有容器监控数据或运行时权限） | —— | FOFA 只索引网络暴露面（banner/证书/网页等），不采集主机内 cgroup/容器运行指标；需要 cAdvisor、Prometheus、Docker stats 等监控数据。 |
| **M096** | 按 Jenkins 文档 TCP Agent Listener Port 示例，提取 Jenkins-/Remoting- 开头的字段名；网页正文或服务响应原文出现任一字段名，不限端口 | `(body="Jenkins-Agent-Protocols" \|\| body="Jenkins-Version" \|\| body="Jenkins-Session" \|\| body="Remoting-Minimum-Version") \|\| (banner="Jenkins-Agent-Protocols" \|\| banner="Jenkins-Version" \|\| banner="Jenkins-Session" \|\| banner="Remoting-Minimum-Version")` | 示例中 Client/Server 不以 Jenkins 或 Remoting 开头，故不取；不保留版本号与会话值。 |
| **M097** | 找与 https://octodex.github.com/ 网站图标相同的其他资产 | `icon_hash="-1493334710"` | 用站点 /favicon.ico 按 FOFA/Shodan 的 mmh3+base64 算法计算；该图标是 GitHub Octocat。icon_hash 可能需会员权限。 |
| **M098** | 找和 msp.gov.cn.poliec.com 一样仿冒特征的网站，总结搜索语句 | `host*="*.gov.cn.*"` | 特征是把真实 gov.cn 域名嵌在仿冒域名前面：host 含 ".gov.cn." 且后面还有字符。针对同一目标可写 host*="*msp.gov.cn.*"；同拼写仿冒可加 domain*="*poliec*"；若只看已标记欺诈站可加 is_fraud="true"。 |
| **M099** | CVE-2021-41773 对应的产品在 FOFA 上怎么搜？ | `product="Apache HTTP Server" && product.version="2.4.49"` | CVE-2021-41773 影响 Apache HTTP Server 2.4.49；也可用 app="APACHE-Web-Server" 锁定指纹，再叠加 product.version="2.4.49"。 |
| **M100** | 证书颁发者 CN 为 ZeroSSL ECC DV SSL CA 2，序列号为 63:BD:83:71:62:14:83:6E:F8:00:75:35:61:61:EB:06 | `cert.issuer.cn="ZeroSSL ECC DV SSL CA 2" && cert.sn="132577581667764526475870473477991557894"` | 先去掉冒号得到十六进制，再转十进制；cert.sn 使用十进制序列号。若平台不识别 cert.sn，可改用 cert="132577581667764526475870473477991557894"（证书全文中的十进制序列号）。 |

<a id="重难点与不可查询题解析"></a>
## 重难点与不可查询题解析

### M015：FOFA 能识别 SOCKS5 吗？

能。FOFA 的协议识别中包含 SOCKS5，查询语句为 protocol="socks5"。若要进一步限定“无认证”的代理，可在 FOFA 的协议指纹文本中叠加 Version:5 Method:No Authentication(0x00)；不同账号/数据版本对协议数据的开放程度不同。

### M023 / M076 / M098：通配符 *= 与 ? 的用法

FOFA 的 * 匹配 0 个或多个字符，? 恰好匹配 1 个字符；*= 不是通配符本身，而是“启用模糊匹配”的运算符。host*="?.?.163.com" 表示 a.z.163.com 这类恰好两级、每级一个字符的子域；domain*="????.com" 表示 .com 前恰好四个字符。注意 *= 与 * 通配通常需要个人版及以上权限。

### M047：证书颁发者与持有者不匹配

严格意义上，FOFA 没有“issuer ≠ subject”这种跨字段比较条件。cert.is_match 判断的是“证书与资产域名是否匹配”，不是颁发者与持有者的比较；若题目指的是证书链/持有者异常，需要导出证书后离线比对。本文给出的是最接近的可查询条件。

### M059 / M100：证书序列号查询

FOFA 的证书序列号使用十进制。做法是：把十六进制序列号去掉冒号/空格 → 转十进制 → 用 cert.sn="十进制数"，或退化为 cert="十进制数"（在证书全文中匹配）。M100 的十六进制 63BD83716214836EF80075356161EB06 转为十进制为 132577581667764526475870473477991557894。短序列号（如 1）容易误报，应叠加端口、证书关键词等条件复核。

### M077 / M078 / M090：含有双引号和反斜杠的原文匹配

FOFA 的字符串值用双引号包裹，值内的双引号通常写成 \"；若查询解析器把反斜杠当作转义符，则原文里的每个反斜杠要写成两个。若控制台报语法错误，可改用单引号包裹值（如 body='class="container"'），或把原文拆成多个不含引号的关键词条件。不同 FOFA 版本/客户端的转义规则可能略有差异，建议先用小范围查询验证。

### M088 / M089：查询修正题

两题的核心错误都是全角/中文标点：M088 的 ” 应改成 ASCII 的 "，且 server== 用于完全匹配；M089 还有非法分隔符 |｜，并且 OR 的优先级会导致 80 端口条件脱离 app="HFS" 约束，必须加括号。M089 可写成 app="HFS" && (port="80" || (port="8080" && status_code="200"))，逻辑等价。

### M080：闭区间 IP 查询

FOFA 的 ip 字段支持单 IP 与 CIDR，不支持 47.96.0.1-47.96.0.5 这种连字符区间。这种只有 5 个地址的闭区间，最稳妥的写法是枚举 5 个 ip="..." 再用 || 连接。

### M091 / M092 / M093 / M094 / M095：不可查询或矛盾题

M091：端口 1000000 超出 0–65535，无有效查询。M092：1.2.3.4.5 不是合法 IPv4/CIDR，无有效查询。M093：country="US" && country!="US" 是逻辑矛盾，结果为空集，但语句可被解析。M094：“最安全的网站”不是 FOFA 字段，需先定义可量化标准。M095：FOFA 不采集容器内存/cgroup 指标，需要主机或容器监控系统（cAdvisor、Prometheus 等）。

### M097：octodex.github.com 的图标哈希

下载站点 /favicon.ico，按 FOFA/Shodan 常用算法：base64.encodebytes(图标字节) → mmh3.hash(字符串) → 有符号 32 位整数。本次算得 -1493334710，因此 icon_hash="-1493334710"。图标一旦被重新压缩/改尺寸，哈希会变化；icon_hash 也可能受会员权限限制。

<a id="统计与参考"></a>
## 统计与参考

- 共 100 题：**96 题**给出可执行查询；M093 可解析但结果必然为空。
- **4 题无法用 FOFA 直接查询**：M091（非法端口）、M092（非法 IP）、M094（无安全评分字段）、M095（无容器运行指标）。
- M047 的“颁发者与持有者不匹配”没有精确字段，文中用 `cert.is_match` 作为近似条件。
- 参考：FOFA 官方站点与 [Awesome-FOFA](https://github.com/FofaInfo/Awesome-FOFA)（通配符、证书、产品/分类字段）；[Jenkins Exposed Services and Ports](https://www.jenkins.io/doc/book/security/services/)；[octodex.github.com](https://octodex.github.com/)。
- 部分字段（证书校验、`icon_hash`、`*=`、`is_cloud` 等）需要对应账号权限。
