---
title: HUAWEI W626E超级用户密码破解
published: 2023-09-06
description : 无聊的大学生活中的一点尝试
tags : [技术]
category : 随笔
draft: false
---
# HUAWEI W626E超级用户密码破解

1. 拍摄光猫背板获取管理页面及普通用户（Epuser）密码
2. Google 查询是否已有解决方案，未找到
3. 猜测前端验证，翻阅源代码

   1. 管理页面无
   2. 进入后发现部分定义​![image](https://pub-d2c21cc922c14429b2c5c871ba58a50b.r2.dev/2024/08/8c6322a1ddacce112be2daff39e9ce8e.png)​

      初步确定非前端验证
4. 陷入困境，无思路（继续 Google）
5. 查找到资料[华为 P602E Telnet 用户名及密码（另附获取方法） (milkfish.site)](https://www.milkfish.site/2021/08/02/1050.loli)，获取基本思路

   1. 取得配置文件
   2. 获取密码（此时还不知道加密）
6. 以普通用户权限再次登录管理页面

   教程![image](https://pub-d2c21cc922c14429b2c5c871ba58a50b.r2.dev/2024/08/a1f6b0ab753db84b6f81a25e96ece361.png)​

   实际​![image](https://pub-d2c21cc922c14429b2c5c871ba58a50b.r2.dev/2024/08/c2505ab5e594d97d572d45b91380a4d2.png)​
7. 猜测前端隐藏

   F12（发现隐藏的 downloadConfig）![image](https://pub-d2c21cc922c14429b2c5c871ba58a50b.r2.dev/2024/08/f6eb5deec28086d8da0f7aadf2afd619.png)​
8. 控制台复现按钮操作（失败）
9. 前端更改（参考资料 [CSS 布局 - display 属性 (w3school.com.cn)](https://www.w3school.com.cn/css/css_display_visibility.asp)）

   ​![image](https://pub-d2c21cc922c14429b2c5c871ba58a50b.r2.dev/2024/08/e75e754a2ef8f48f78db0cf501f72f33.png)出现下载配置文件
10. 按钮有效（获取到配置文件 hw_ctree.xml）
11. 根据教程查找用户，此时发现加密，多方查找解密工具，最终查找到有效的[华为光猫破解 - 哔哩哔哩 (bilibili.com)](https://www.bilibili.com/read/cv14530172/)

    1. 根据教程查找到的密钥解密，仍为乱码，猜测多重加密（后有 salt 值）

       ​![image](https://pub-d2c21cc922c14429b2c5c871ba58a50b.r2.dev/2024/08/d7a1cb65847392913dbe54af7a9b6aa8.png)​
    2. 出去逛了一圈。。。。。
    3. 向下查找

       1508 行 `X_HW_AppRemoteManage`​​​​​（应该是留给 App 管理的）出现可解密的密码，解密结果 admin@Ep

       ​![image](https://pub-d2c21cc922c14429b2c5c871ba58a50b.r2.dev/2024/08/ae2025997a7ae99b25426693d76e468c.png)​
12. 成功登录（功能还挺多，可惜我用不到）

    ​![image](https://pub-d2c21cc922c14429b2c5c871ba58a50b.r2.dev/2024/08/e479845462885d76f6010a0a2d6d1674.png)​

    ‍

13. 结束

14. 后记

    发现 Eproot 用户，不知道什么用途

    ​![image](https://pub-d2c21cc922c14429b2c5c871ba58a50b.r2.dev/2024/08/0a413ceae7d8a403d2fdfe3c40ae531e.png)Web 管理页面无效

    ​![image](https://pub-d2c21cc922c14429b2c5c871ba58a50b.r2.dev/2024/08/8185b50c4595564882fa96f356e1ebff.png)根据前面的标签 `X_HW_CLIUserInfoInstance`​​​ 应该是用于命令管理，但 telnet 连接不上（之前试过），有时间再弄吧

    ‍
