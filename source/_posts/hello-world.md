---
title: 斗鱼app个人中心整体优化桥接文档（RN）
---

### 斗鱼app个人中心整体优化桥接文档（RN）

<img src="https://cdn.jsdelivr.net/gh/daxiahu/ImageBed@master/img/20201130211818.jpg" width="400"/>

&ensp;


#### 跳转实名认证逻辑
```mermaid
graph TB
    start[点击实名认证按钮] --> hasPhone{判断是否绑定了手机号}
    hasPhone -- 是 --> A(判断是否需要实名)
    hasPhone -- 否 --> noPhone[toast提示需要先绑定手机号]
    A --> B{状态为未通过或者未实名}
    B -- 是 --> F[判断是否需要跳转H5认证页面]
    F --> G{DYBundleConfig.switchInfo.h5Ident}
    B -- 否 -->C[状态为正在获取或正在审核,toast提示]
    G -- 是 --> D[jumpH5RealNameVerifyPage跳转]
    G -- 否 --> E[跳转RN]
    noPhone --> stop
    C --> stop
    D --> stop[结束]
    E --> stop
```

#### 跳转绑定邮箱逻辑
```mermaid
graph TB
    start[点击绑定邮箱按钮] --> A{判断是否已绑定邮箱}
    A -- 是 --> B[不作任何操作]
    A -- 否 --> C[跳转到绑定邮箱RN页面]
    B --> stop[结束]
    C --> stop
```

**原生Android和iOS通过‘模块名’+‘组件名’方式跳转对应RN页面**

|页面名称 | 个人资料-实名认证 | 个人资料-绑定邮箱|
|:---:|:---:|:---:|
|模块名（module） | DYRNPersonalCenter | DYRNPersonalCenter|
|组件名（component）| RealNameVerify | BindEmail|
|参数| 无 | 无|
