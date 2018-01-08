## 超级账本Fabric CA应用与配置
* 超级账本Fabric CA项目为超级账本Fabric网络提供了基于PKI的身份证书管理服务。基于它提供的ECert和TCert，Fabric网络中可以根据业务需求实现十分灵活的权限控制和审计功能。毫不夸张地说，Fabric CA为Fabric提供了构建面向企业场景联盟链的信任基石。
* Fabric CA项目原来是超级账本Fabric内的MemberService组件，负责对网络内各个实体的身份证书进行管理。鉴于其功能十分重要，2017年2月正式成立Fabric CA独立子项目，负责相关代码的维护。Fabric CA项目主要实现了如下几个功能：
        ·负责Fabric网络内所有实体（Identity）的身份管理，包括身份的注册、注销等；
        ·负责证书管理，包括ECerts（身份证书）、TCerts（交易证书）等的发放和注销；
        ·服务端支持基于客户端命令行和RESTful API的交互方式。
##### 基本组件
* Fabric CA采用了典型的CS（Client-Server）架构，目前包含两个基本组件，分别实现服务端功能和客户端功能。
        * 服务端（Server）：fabric-ca-server实现核心的PKI服务功能，支持多种数据库后台（包括MySql、PostgreSQL等），并支持集成LDAP作为用户注册管理功能；
        * 客户端（Client）：fabric-ca-client封装了服务端的RESTful API，提供访问服务端的命令，供用户与服务端进行交互。
        * 通常情况下，用户通过客户端从Fabric CA服务端获取合法的证书文件后，即可参与到Fabric网络中发起交易。处理交易过程中，无需实时依赖Fabric CA的在线。同时，服务端本身是个可扩展的Web服务，可以很容易地采用集群部署的方式进行负载均衡，同时提高可靠性。

##### 启动CA

