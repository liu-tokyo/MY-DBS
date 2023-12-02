> https://blog.csdn.net/GYN_enyaer/article/details/132695244

## PostgreSQL 简介

PostgreSQL（通常简称为PG SQL）是一个强大、开源的关系型数据库管理系统（DBMS），它具有广泛的功能和可扩展性，被广泛用于企业和开发项目中,PostgreSQL 具有如下一些关键特点：

1. 开源和免费： PostgreSQL 是一个自由开源软件，任何人都可以免费使用、修改和分发它。这使得它成为了许多开发者和组织的首选数据库系统。
   
2. 高度可扩展： PostgreSQL 提供了丰富的扩展性选项，包括支持多种编程语言编写的插件，以及自定义数据类型和函数。这使得它能够满足各种不同规模和类型的应用需求。
   
3. ACID 兼容： PostgreSQL 遵循ACID（原子性、一致性、隔离性、持久性）事务属性，确保了数据的完整性和可靠性，特别适合需要高度数据一致性的应用程序。
   
4. 支持复杂数据类型： 除了传统的关系型数据，PostgreSQL还支持非常复杂的数据类型，如数组、JSON、XML、几何数据、全文搜索等。这使得它非常适合存储和查询各种类型的数据。
   
5. 强大的扩展性： PostgreSQL支持多种类型的索引，包括B树、哈希、GiST、SP-GiST、GIN和BRIN等，以便于不同类型的查询优化。此外，用户可以自定义索引类型。
   
6. 丰富的内置函数和扩展： PostgreSQL 包含大量内置函数，同时还支持用户定义的函数，这使得可以实现各种复杂的数据操作和计算。
   
7. 多版本并发控制： PostgreSQL 使用多版本并发控制（MVCC）来处理并发事务，这允许多个事务同时访问数据库，而不会相互干扰。
   
8. 安全性： PostgreSQL 提供了强大的安全功能，包括身份验证、授权和SSL支持，以保护数据免受未经授权的访问和窃听。
   
9. 活跃的社区和生态系统： PostgreSQL 拥有庞大的全球社区，提供了大量的文档、教程和支持资源。此外，有许多第三方工具和库可用于与 PostgreSQL 集成。

PostgreSQL 所具备的高度可定制和可扩展的数据库特性，使之适用于各种不同的应用场景，从小型项目到大规模企业级应用都能够发挥其优势。它的开源性质和活跃的社区支持使得它成为了数据库领域的一项重要选择。

PostgreSQL 作为一个跨平台的开源数据库管理系统，支持在多种操作系统上安装和运行：

- Linux：PostgreSQL 在几乎所有主流的 Linux 发行版上都有支持，包括但不限于：
  Ubuntu
  Debian
  CentOS
  Red Hat Enterprise Linux (RHEL)
  Fedora
  SUSE Linux Enterprise Server (SLES)
  Oracle Linux
  等等
  
- Windows：PostgreSQL 可以在不同版本的 Windows 操作系统上安装，包括：
  Windows 10
  Windows Server 2016/2019
  Windows 7/8/8.1
  等等
  
- macOS：PostgreSQL 也可以在 macOS 上安装，通常通过 Homebrew 或官方 PostgreSQL 安装程序进行安装。
  
- BSD 系统：PostgreSQL 在一些 BSD 系统上也有支持，例如：
  FreeBSD
  OpenBSD
  NetBSD
  
- Solaris：PostgreSQL 也可以在 Solaris 操作系统上安装。

需要注意的是，不同的操作系统版本和发行版可能会有不同的安装方法和配置步骤。你可以访问 PostgreSQL 官方网站 (https://www.postgresql.org/) 获取关于在特定操作系统上安装 PostgreSQL 的详细指南和下载链接。