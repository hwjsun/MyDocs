### 一、总体目标
Vue-Vben-Admin v5（域名：https://vben.neibuguanli.cn），完整对接ABP后端，跑通以下基础功能：

#### OIDC-PKCE 登录 / 刷新令牌 / 401 自动刷新  
#### 租户下拉切换（自动带 `__tenant` 头）  
#### 动态左侧菜单（ABP `/api/abp/application-configuration`）  
#### 按钮级权限（`v-auth` 指令）  
#### 多语言（ABP `/api/language-management/languages`）  
#### 主题/布局/锁屏/404/500 等走查
### 二、前端与授权站点和API站点的交互
#### 1、用户认证流程
![](https://cdn.nlark.com/yuque/__mermaid_v3/77b0e57da734129217a8a8ee2981c347.svg)

#### 2、前端交互流程
![](https://cdn.nlark.com/yuque/__mermaid_v3/42dbf5dbdd194b0e8f2300eb9c868264.svg)



#### 3、前端交互细节
<font style="color:#DF2A3F;">注意：vben站点需要传递clientId给授权站点（clientId：ProjectAdmin_Vben_App）</font>

```typescript
vben站点在向授权中心请求时，传递的clientId，设置为 ProjectAdmin_Vben_App
const oAuthConfig = {
  issuer: 'https://oidc.neibuguanli.cn/',
  redirectUri: baseUrl,
  clientId: 'ProjectAdmin_Vben_App',
  responseType: 'code',
  scope: 'offline_access ProjectAdmin',
  requireHttps: true,
  impersonation: {
    tenantImpersonation: true,
    userImpersonation: true,
  }
};
```

![](https://cdn.nlark.com/yuque/__mermaid_v3/e8cf4d2f4099d8f12233ac831ea86adc.svg)

#### 4、自有登录页与oidc授权码流的对比
简单起见，先使用跳转授权站点的方式来完成登录认证

| **<font style="color:#000000;">对比项</font>** | **<font style="color:#000000;">跳转授权站点（PKCE）</font>** | **<font style="color:#000000;">Vben 自有页面（直接登录）</font>** |
| --- | --- | --- |
| <font style="color:#000000;">用户体验</font> | <font style="color:#000000;">跳走 → 登录中心 → 跳回；地址栏可见变化</font> | <font style="color:#000000;">全程留在 Vben 页面，无跳转</font> |
| <font style="color:#000000;">协议流</font> | <font style="color:#000000;">授权码流（Authorization Code + PKCE）</font> | <font style="color:#000000;">资源所有者密码流（ROPC）</font> |
| <font style="color:#000000;">前端实现</font> | <font style="color:#000000;">用 `oidc-client-ts` 做重定向、回调、静默刷新</font> | <font style="color:#000000;">用 axios 直接调 `/api/account/login` 即可</font> |
| <font style="color:#000000;">安全等级</font> | <font style="color:#000000;">高（token 只在后端交互，前端无密码）</font> | <font style="color:#000000;">中等（密码经过前端，需 HTTPS+防 XSS）</font> |
| <font style="color:#000000;">刷新/退出</font> | <font style="color:#000000;">标准 `/connect/token` & `/connect/logout`</font> | <font style="color:#000000;">自定义 `/api/account/refresh` & `/logout`</font> |
| <font style="color:#000000;">多租户/多语言</font> | <font style="color:#000000;">通过 authorize URL 参数传递</font> | <font style="color:#000000;">通过 login/refresh 请求体 Header 传递</font> |


#### 5、自有登录页面的认证流程
点击全屏按钮查看

![](https://cdn.nlark.com/yuque/__mermaid_v3/3ce010609afd3cf8b62531aa11e8e0e2.svg)

完成上述的登录授权之后，再接入以下菜单的功能

#### 
```typescript
apps/vben5/src
├─views/admin
│  ├─identity
│  │  ├─user
│  │  ├─role
│  │  ├─ou
│  │  ├─claim
│  │  └─security-log
│  ├─openiddict
│  │  ├─application
│  │  └─scope
│  ├─language
│  ├─text-template
│  ├─audit
│  ├─setting
│  └─saas
│      ├─tenant
│      └─edition
├─api/        # 按模块拆 service
│  ├─identity.ts
│  ├─openiddict.ts
│  ├─language.ts
│  ├─audit.ts
│  ├─setting.ts
│  └─saas.ts
├─hooks/abp/  # 通用 CRUD Hook
│  └─useAbpCrud.ts

```



### 三、常见坑速查表
| **<font style="color:#000000;">现象</font>** | **<font style="color:#000000;">原因</font>** | **<font style="color:#000000;">解决</font>** |
| --- | --- | --- |
| <font style="color:#000000;">列表无数据</font> | <font style="color:#000000;">只取 `res.data` 没取 `res.data.result`</font> | <font style="color:#000000;">统一走 `abp.ts` 拦截器</font> |
| <font style="color:#000000;">树表无 children</font> | <font style="color:#000000;">扁平权限</font> | <font style="color:#000000;">后端加 `children` 或前端递归</font> |
| <font style="color:#000000;">403 租户不匹配</font> | <font style="color:#000000;">缺 `__tenant` 头</font> | <font style="color:#000000;">请求拦截统一加</font> |
| <font style="color:#000000;">OIDC 死循环</font> | <font style="color:#000000;">刷新失败仍重试</font> | <font style="color:#000000;">401 刷新失败立即 `signoutRedirect`</font> |
| <font style="color:#000000;">编辑 400</font> | <font style="color:#000000;">ABP 大小写敏感</font> | <font style="color:#000000;">`name` → `Name`，用后端 DTO 生成 TS 类型</font> |


### 
