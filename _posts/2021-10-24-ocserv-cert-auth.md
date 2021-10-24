---
layout: post
title: Ocserv 配置证书认证
---

# 安装证书工具 `certtool`

**Ubuntu**: `apt install gnutls-bin`

**CentOS/RHEL**: `dnf install gnutls-utils`

# 创建 CA 证书

```bash
# 创建 CA 私钥
certtool --generate-privkey --outfile ca-key.pem

# Create CA certificate template
cat <<EOF > ca.tmpl
# X.509 Certificate options
cn = "VPN CA"               # Common name
organization = "VPN CA"     # Organization
serial = 001                # 证书序列号
expiration_days = -1        # 过期时间，-1 表示永不过期
ca                  # 表示是个 CA 证书
signing_key         # 可用于签署
cert_signing_key    # 可用于签署其他证书
crl_signing_key     # 可用于签署 CRL
EOF

# 生成自签名 CA 证书
certtool --generate-self-signed \
--load-privkey ca-key.pem
--template ca.tmpl
--outfile ca-cert.pem
```

# 生成并签署服务器证书

```bash
# 创建私钥
certtool --generate-privkey --outfile server-key.pem

cat <<EOF > server.tmpl
cn = "<server_domain_name>" 
organization = "<server_domain_name>"
expiration_days = -1    # 永不过期
signing_key
encryption_key          # 用于加密数据
tls_www_server
EOF

# 生成服务器证书
certtool --generate-certificate \
--load-privkey server-key.pem \
--load-ca-certificate ca.pem \
--load-ca-privkey ca-key.pem \
--template server.tmpl \
--outfile server-cert.pem
```

# 生成客户端证书

用于客户端认证

```bash
# 生成私钥（一般从客户机生成）
certtool --generate-privkey --outfile client-key.pem

cat <<EOF > client.tmpl
organization = "VPN"    # 组织名称
cn = "<username>"       # Common Name
uid = "<username>"      # 这里的 uid 要和 ocpasswd 中的一致？
expiration_days = -1    # 永不过期
tls_www_client          # 可用于 TLS Client
signing_key
encryption_key
EOF

############################################

# - 客户端证书生成方法 1: 
# 使用 CA 直接签署证书
certtool --generate-certificate \
--load-privkey client-key.pem \
--load-ca-certificate ca.pem \
--load-ca-privkey ca-key.pem \
--template client.tmpl \
--outfile client-cert.pem

# - 客户端证书生成方法 2: 
# 从客户机生成 CSR (Certificate Signing Request)
certtool --generate-request \
--load-privkey client-key.pem \
--template client.tmpl \
--outfile request.pem
# 管理员生成证书
certtool --generate-certificate \
--load-ca-certificate ca.pem \
--load-ca-privkey ca-key.pem \
--load-request request.pem \
--template client.tmpl \
--outfile client-cert.pem

############################################

# 生成 PKCS #12 文件（合并私钥和证书）
certtool --to-p12 \
--load-privkey client-key.pem \
--load-certificate client-cert.pem \
--pkcs-cipher aes-256 \
--outfile client.p12 --outder

# iOS AnyConnect 不支持 AES-256，可以更换为 3des-pkcs12
certtool --to-p12 \
--load-privkey client-key.pem \
--load-certificate client-cert.pem \
--pkcs-cipher 3des-pkcs12 \
--outfile client.p12 --outder
```

# ocserv 配置

仅需要修改配置文件 `ocserv.conf` 的相关部分

```conf
# enable password authentication
auth = "plain[passwd=/etc/ocserv/ocpasswd]"

# 开启证书认证，与原有 auth 并行
enable-auth = "certificate"

# CA 证书位置
ca-cert = "<ca certificate file>"

# 服务器证书位置
server-cert = "<server certificate file>"
# 服务器私钥位置
server-key = "<server private key file>"
```

# 客户端配置

`CA证书` 和 `服务器证书` 主要用于验证服务器和 TLS 加密，不涉及登陆认证。

对于一般 PC OpenConnect 客户端认证，需要提供 `client-key.pem` `client-cert.pem` 两个文件，或者仅 `client.p12` 文件。

对于 iOS，将 `client.p12` 文件发送到手机，分享到 AnyConnect 将其导入软件。成功导入后，在 AnyConnect 中通过 `Advanced -> Certificate` 选择指定的客户端证书。

# 常见问题

* iOS AnyConnect 通过证书认证时无法连接，并且服务端报错 `GnuTLS error (at worker-vpn.c:795): A TLS fatal alert has been received.` <br>
**解决方法：** 向 `ocserv.conf` 的 `tls-priorities` 字段末尾添加 `:-VERS-TLS1.3` 以禁用 TLS 1.3。
* 链接建立后，每 4 分钟便会断掉链接 <br>
**解决方法：** 修改 `ocserv.conf`，使 `isolate-workers = false`

# 参考资料

* https://www.linuxbabe.com/ubuntu/certificate-authentication-openconnect-vpn-server-ocserv
* https://github.com/MarkusMcNugen/docker-openconnect/blob/master/docker-entrypoint.sh
