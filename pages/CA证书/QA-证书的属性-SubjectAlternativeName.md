---
title: "QA-证书的属性-SubjectAlternativeName.md"
---
### SubjectAlternativeName的作用？
其实就是对证书Owner/Subject的一个扩展，因为Owner字段装不下这么多信息。
SubjectAlternativeName（SAN）是CA证书中的一个扩展字段，用于指定证书可以用于验证的额外主体名称。SAN的作用包括：

    支持多个域名：SAN允许在同一个证书中指定多个域名，这在服务器证书中特别有用，可以让同一个证书同时适用于多个域名。
    
    支持通配符：SAN允许使用通配符来指定一类域名，例如*.example.com可以匹配所有以example.com结尾的域名。
    
    IP地址支持：SAN也可以用于指定IP地址，这在一些特定的网络环境下很有用。
    
    增强安全性：SAN可以增强证书的安全性和灵活性，使得证书可以适用于更多的场景和需求。

总之，SAN的作用是在CA证书中指定额外的主体名称，提供了更灵活的证书使用方式，特别是在多域名、通配符域名和IP地址验证方面。