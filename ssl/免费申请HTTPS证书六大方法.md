#### 1、阿里云

- 推荐指数：★★☆☆☆
- 免费证书类型：DV 域名型
- 免费证书品牌：DigiCert（原赛门铁克（Symantec））
- 免费通配符证书：不支持
- 易操作性：简单
- 证书有效期： 1年
- 自动更新：不支持
- 自动部署： 不支持
- 优点：有效期长

阿里云仅提供免费的单域名HTTPS证书，如果你仅只需要一个单域名的证书，可以使用阿里云的免费证书，毕竟DigiCert是大品牌，值得信赖。在证书即将到期前，需要再次手动申请证书，不支持自动化申请和部署。申请链接：https://common-buy.aliyun.com/?commodityCode=cas#/buy

#### 2、腾讯云

- 推荐指数：★★☆☆☆
- 免费证书类型：DV 域名型
- 免费证书品牌：TrustAsia（即亚洲诚信）
- 免费通配符证书：不支持
- 易操作性：简单
- 证书有效期： 1年
- 自动更新：不支持
- 自动部署： 不支持

腾讯云同阿里云一样，仅提供免费的单域名HTTPS证书，如果你仅只需要一个单域名的证书，同样可以使用腾讯云的免费证书，TrustAsia也是比较大的品牌。在证书即将到期前，需要再次手动申请证书，不支持自动化申请和部署。申请链接：https://buy.cloud.tencent.com/ssl?fromSource=ssl

#### 3、certbot

- 推荐指数：★★★☆☆
- 免费证书类型：DV 域名型
- 免费证书品牌：Let's Encrypt
- 免费通配符证书：支持
- 易操作性：困难
- 证书有效期： 90天
- 自动更新：支持
- 自动部署： 支持

这里先说下Let's Encrypt证书品牌：Let's Encrypt是免费、开放和自动化的世界知名证书颁发机构，由非盈利组织互联网安全研究小组（ISRG）运营。Let's Encrypt已为全世界1.8亿个网站提供HTTPS证书，可放心使用。
再说下这个[certbot](https://certbot.eff.org/)：certbot是一个脚本类型的Let's Encrypt证书申请客户端，需要一定的命令行使用经验方可操作，如需自动更新，还需要添加插件，使用起来比较困难。如有自动更新和自动部署需求，建议使用下面介绍的acme.sh和ohttps.com。

#### 4、acme.sh

- 推荐指数：★★★★★
- 免费证书类型：DV 域名型
- 免费证书品牌：Let's Encrypt
- 免费通配符证书：支持
- 易操作性：一般
- 证书有效期： 90天
- 自动更新：支持
- 自动部署： 支持

[acme.sh](https://github.com/acmesh-official/acme.sh)是一个知名的用于申请Let's Encrypt证书的开源项目，项目地址：https://github.com/acmesh-official/acme.sh，也是属于脚本类型，有比较详细的文档，支持自动化更新和自动化部署。唯一缺点，如果有更新后自动部署至多个节点的需求的话，acme.sh无法满足。如果你有一定的命令行使用经验，acme.sh使用起来还是非常方便，强烈推荐！关于更新后自动部署至多个节点的需求，建议使用下面介绍的ohttps.com。

#### 5、freessl.cn

- 推荐指数：★★★☆☆
- 免费证书类型：DV 域名型
- 免费证书品牌：Let's Encrypt、TrustAsia
- 免费通配符证书：支持Let's Encrypt的通配符类型证书
- 易操作性：简单
- 证书有效期： Let's Encrypt通配符证书有效期90天、TrustAsia双域名证书有效期1年
- 自动更新：不支持
- 自动部署： 不支持

[freessl.cn](https://freessl.cn/)提供免费的Let's Encrypt和TrustAsia证书申请，Let's Encrypt支持通配符类型，TrustAsia仅支持双域名类型证书。申请界面比较友好，根据提示在域名解析记录中添加指定的TXT类型域名解析即可，对于不会使用命令行的小白用户来说使用起来比较简单。申请链接：https://freessl.cn/

#### 6、ohttps.com

- 推荐指数：★★★★★
- 免费证书类型：DV 域名型
- 免费证书品牌：Let's Encrypt
- 免费通配符证书：支持
- 易操作性：简单
- 证书有效期： 90天
- 自动更新：支持
- 自动部署： 支持

[ohttps.com](https://ohttps.com/)提供了类似于acme.sh的功能，不过提供了友好的管理界面，可申请Let's Encrypt免费通配符类型证书，还提供了证书吊销、到期前提醒、自动更新、自动部署功能。另外比acme.sh增加了一些非常实用的功能，主要包括可自动部署至阿里云、腾讯云、七牛云的负载均衡、内容分发CDN、SSL证书列表等，并可自动部署至多个nginx容器中。如果你有在证书更新后自动部署至多个不同节点的需求，使用ohttps.com就对了，在这里强烈推荐大家使用ohttps.com申请和管理Let's Encrypt颁发的免费HTTPS证书。申请链接：[https://ohttps.com](https://ohttps.com/)