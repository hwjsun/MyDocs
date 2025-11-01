---

<h3 id="VIaA5">前言：跟ai沟通时，使用这段话让ai更好的定位和解决问题</h3>
我们是一套 ABP 9.3.5 分层 + OpenIddict 授权中心 的纯前后端分离后台：

1. 授权站点独立部署，只负责登录/发 token

 URL：[https://openid.f2b2c.shop](https://openid.f2b2c.shop)

 发现文档：[https://openid.f2b2c.shop/.well-known/openid-configuration](https://openid.f2b2c.shop/.well-known/openid-configuration)

2. API 站点完全无 Cookie，只认  Authorization: Bearer 

3. 前端是 Arco Vue3 SPA，走 授权码 + PKCE，无 client_secret

 客户端 ID（client_id）： XShop_Arco 

 回调地址（redirect_uri）： [https://arco.f2b2c.shop/signin-oidc](https://arco.f2b2c.shop/signin-oidc) 

 退出地址（post_logout_redirect_uri）： [https://arco.f2b2c.shop/signout-callback-oidc](https://arco.f2b2c.shop/signout-callback-oidc) 

4. 数据库表  OpenIddictApplications  里给 Arco 的权限集合

["ept:end_session","gt:authorization_code","rst:code",

 "ept:authorization","ept:token","ept:revocation","ept:introspection",

 "scp:openid","scp:profile","scp:roles","scp:email","scp:phone","scp:XShop"]

grant_type 只用 authorization_code；password / client_credentials 与我们无关

5. 本地开发若 401，99 % 是 本机时间偏差 或 DNS 指向旧容器；token 本身没问题

6. 令牌过期不会自动 302；需在 axios 响应拦截器里手动拼接

/connect/authorize?

  response_type=code&

  client_id=XShop_Arco&

  redirect_uri=[https://arco.f2b2c.shop/signin-oidc&](https://arco.f2b2c.shop/signin-oidc&)

  scope=openid profile roles email phone XShop&

  code_challenge=xxx&

  code_challenge_method=S256&

  state=随机字符串

然后  location.href = 上面地址  重新走登录

7. 静默刷新：先用本地 refresh_token 换 AT，失败再执行第 6 步

<h3 id="IKjeU">一、总体目标  </h3>
<h4 id="D6MMc">OIDC-PKCE 登录 / 刷新令牌 / 401 自动刷新  </h4>
登录/刷新/注销走 OIDC-PKCE（授权码流），access_token 自动续期

登录 → 拿到 access_token + refresh_token + expires_in 

退出登录 → 清除 token → 跳转登录页

<h4 id="Oxzld">租户下拉切换（自动带 `__tenant` 头）  </h4>
租户切换实时生效（tenant 头 + application-configuration 重载）

切换租户 → 请求头 tenant 变化 → 数据隔离

<h4 id="V16oK">动态左侧菜单（ABP `/api/abp/application-configuration`）  </h4>
<h4 id="vQy8Q">按钮级权限（`v-auth` 指令）  </h4>
按钮/路由级权限由后端 auth.policies 驱动，前端仅 `v-permission="'AbpIdentity.Users.Create'"`

权限按钮 → 无权限自动隐藏 → 403 不弹窗

<h4 id="Gwqxa">多语言（ABP `/api/language-management/languages`）  </h4>
多语言与 ABP 后端 100% 同源（`/api/abp/application-localization` → Arco locale）

语言切换 → Arco 组件即时换语言 → 后端本地化文本正确

<h4 id="bRvCU">使用JavaScript客户端代理</h4>
零手写 URL：全部调用通过 ABP generate-proxy 生成的 TS 客户端（`abp.services.identity.user.getList(...)`）

<h4 id="Jv9M0">主题/布局/锁屏/404/500 等走查</h4>
<h3 id="qW4zN">二、开发资料</h3>
<h4 id="LhAVQ"><font style="color:rgb(31, 35, 40);">1、arco提交仓库</font></h4>
<font style="color:rgb(31, 35, 40);"> </font>[git@gitee.com:tita2025/xshop-pc-arco.git](git@gitee.com:tita2025/xshop-pc-arco.git)

<h4 id="ApkuW">2、arco对应的swagger api</h4>
 [https://admin.f2b2c.shop/swagger/index.html](https://admin.f2b2c.shop/swagger/index.html)

<h4 id="iwor1">3、arco的API文档 </h4>
[https://www.yuque.com/u253383/hlft5f/vtun8reruh965v0q?singleDoc#Bze5](https://www.yuque.com/u253383/hlft5f/vtun8reruh965v0q?singleDoc#Bze5) 《API》 密码：rdpd

<h4 id="udMH1">4、oidc授权站点</h4>
 [https://openid.f2b2c.shop](https://openid.f2b2c.shop)

<h4 id="Wb1JJ">5、参考angular代码</h4>
 [https://gitee.com/tita2025/abp-angular.git](https://gitee.com/tita2025/abp-angular.git)

angular代码的<font style="color:#000000;">environment.prod.ts 和 protractor.conf.js ，这里会配置api站点和oidc授权站点</font>

<font style="color:#000000;">npm install -g @angular/cli   安装cli</font>

<font style="color:#000000;">执行 ng serve 启动站点 </font>[http://localhost:4200/](http://localhost:4200/)    

angular 线上站点  [https://ng.f2b2c.shop/](https://ng.f2b2c.shop/)       账号 admin 密码 1q2w3E*

<h4 id="A3fGe">6、使用ftp工具发布arco站点</h4>
安装ftp客户端工具：[https://download.filezilla.cn/client/windows/FileZilla_3.68.1_win64-setup.exe](https://download.filezilla.cn/client/windows/FileZilla_3.68.1_win64-setup.exe)

安装后启动程序，文件=》站点管理器=》新站点=》arco

主机：43.139.163.237

端口：21

登录类型：正常

账号：arco 

密码：Ftp@67890

点击连接，保存密码，连接成功后，可以看到远程的目录

![](https://cdn.nlark.com/yuque/0/2025/png/419761/1761881279640-1b03483e-5ad1-47de-83f8-67f4ff165bf3.png)

<h4 id="pPcA7">7、关于微信验证/安全类  .txt  文件（如  MP_verify_*.txt ）</h4>
必须放在「站点根目录」，即：

[https://arco.f2b2c.shop/MP_verify_xxx.txt](https://arco.f2b2c.shop/MP_verify_xxx.txt)

不能放在  assets  里，否则微信会 404，验证失败。

FTP 上传时的路径

假设你打包后的  index.html  直接放在：

/wwwroot/index.html

那么  .txt  文件应该放在：/wwwroot/MP_verify_xxx.txt

与  index.html  同级，在  assets  外面。

如果你是把整个  dist  文件夹内容（含  assets ）直接拖进  /wwwroot ，那就把  .txt  文件并排放在  assets  同级即可：

<h4 id="YSfrv">8、快速接入步骤</h4>
<h5 id="NmXYY">（1）装包</h5>
pnpm add oidc-client-ts

<h5 id="BWT6J">（2）授权</h5>
src/utils/oidc.ts 

```typescript
import { UserManager, UserManagerSettings } from 'oidc-client-ts';

const settings: UserManagerSettings = {
  authority: 'https://openid.f2b2c.shop',
  client_id: 'Arco_Vue_App',
  redirect_uri: window.location.origin + '/signin-oidc',
  post_logout_redirect_uri: window.location.origin + '/signout-callback-oidc',
  response_type: 'code',
  scope: 'openid profile roles email phone XShop',
  loadUserInfo: true,
  automaticSilentRenew: true,
  monitorSession: true,
  userStore: new WebStorageStateStore({ store: window.localStorage }),
};

export const userManager = new UserManager(settings);

```

<h5 id="XcI8y">（3）登录按钮</h5>
```typescript
import { userManager } from '@/utils/oidc';
async function login() {
  await userManager.signinRedirect(); // 跳到授权站
}
```

<h5 id="JKSUu">（4）回调路由  /signin-oidc </h5>
加  /signin-oidc  路由 → 调用  signinRedirectCallback()

```typescript
import { userManager } from '@/utils/oidc';
onMounted(async () => {
  const user = await userManager.signinRedirectCallback(); // 换 token
  localStorage.setItem('access_token', user.access_token);
  localStorage.setItem('refresh_token', user.refresh_token);
  router.push('/'); // 回首页
});
```

<h5 id="iYqql">（5）令牌过期处理（axios 拦截器）</h5>
axios 拦截器 → 401 自动跳登录

```typescript
import axios from 'axios';
import { userManager } from '@/utils/oidc';

axios.interceptors.response.use(
  res => res,
  async err => {
    if (err.response?.status === 401) {
      const user = await userManager.getUser();
      if (user && user.refresh_token) {
        try {
          const newUser = await userManager.signinSilent(); // 换 AT
          axios.defaults.headers['Authorization'] = 'Bearer ' + newUser.access_token;
          return axios(err.config); // 重放原请求
        } catch {
          // 刷新失败 → 跳登录
          userManager.signinRedirect();
        }
      } else {
        // 没 RT → 直接跳
        userManager.signinRedirect();
      }
    }
    return Promise.reject(err);
  });
```

<h5 id="lQsqh">（6）获取令牌</h5>
打包部署 → 浏览器测试 → 登录后  pwd  返回  /  且能拿到  access_token  即成功

---

<h3 id="FQhdM">三、上线前验证</h3>
验证步骤（上线前 checklist）

<h4 id="XIlaQ">1. 登录是否跳转</h4>
访问 [https://arco.f2b2c.shop](https://arco.f2b2c.shop) → 点击登录 → 应跳转到 [https://openid.f2b2c.shop/connect/authorize](https://openid.f2b2c.shop/connect/authorize)

<h4 id="sAIJc">2. 登录成功后是否拿到code</h4>
登录成功后 → 浏览器地址栏回到 [https://arco.f2b2c.shop/signin-oidc ](https://arco.f2b2c.shop/signin-oidc )并拿到 code

<h4 id="gWAFk">3. 前端是否使用code换取到令牌</h4>
前端立即用 code 换 token → 可看到 access_token、refresh_token、id_token

<h4 id="hZiBv">4. 前端请求api是否成功</h4>
访问 [https://admin.f2b2c.shop/api/abp/application-configuration](https://admin.f2b2c.shop/api/abp/application-configuration) → 请求头带 Bearer access_token → 返回 200

<h4 id="qHVrb">5. 前端注销是否跳转</h4>
点击注销 → 跳回 [https://arco.f2b2c.shop/signout-oidc](https://arco.f2b2c.shop/signout-oidc) → 本地 token 被清空

<h4 id="MOCXE">6. 编译是否正常</h4>
```bash
# 编译
npm build
# 预览
npm preview
```

✅ Console 0 错误 0 警告

✅ Network 0 失败请求

✅ 登录 / 切换租户 / 刷新令牌 / 403 / 404 / 500 页面正常

✅ 按钮权限生效（无权限按钮消失）

✅ 语言切换即时生效  

---

<h3 id="J6QIg">四、跟授权中心握手</h3>
拦截器配完后，Arco 即可实现：

首次登录 → 授权码 → 换令牌 → 令牌过期 → 自动重定向到授权中心 → 重新登录 → 回到 Arco。

<h4 id="kVgnm">1、令牌失效后的 302 跳转 不会自动发生</h4>
需要你在 Arco 里加 拦截器 手动触发  location.href = .../connect/authorize?... ；

```typescript
// src/utils/request.ts
import axios from 'axios';

const AUTH_SERVER = 'https://openid.f2b2c.shop';
const CLIENT_ID   = 'XShop_Arco';
const REDIRECT_URI= encodeURIComponent('https://arco.f2b2c.shop/signin-oidc');

axios.interceptors.response.use(
  res => res,
  err => {
    const rsp = err.response;
    // ABP 默认返回 401 表示 AT 失效
    if (rsp && rsp.status === 401) {
      // 清理本地缓存
      localStorage.removeItem('access_token');
      localStorage.removeItem('refresh_token');
      // 构造 PKCE 跳转地址（code 流程）
      const codeVerifier  = generateCodeVerifier();   // 随机字符串 43~128
      const codeChallenge = generateCodeChallenge(codeVerifier);
      localStorage.setItem('code_verifier', codeVerifier);

      const url =
        `${AUTH_SERVER}/connect/authorize?` +
        `client_id=${CLIENT_ID}` +
        `&redirect_uri=${REDIRECT_URI}` +
        `&response_type=code` +
        `&scope=openid profile roles email phone XShop` +
        `&state=${Math.random().toString(36).slice(2)}` +
        `&code_challenge=${codeChallenge}` +
        `&code_challenge_method=S256` +
        `&prompt=login`;   // 强制重新登录，可按需删掉

      location.href = url;          // 302 跳转
    }
    return Promise.reject(err);
  });

```

<h4 id="l7iWE">2、令牌在你电脑 401，别人电脑 200</h4>
为什么同一接口、同一 grant_type 拿到的 token，在你电脑 401，别人电脑 200

本机 401 → 看返回头里的  WWW-Authenticate  会告诉你  error="invalid_token"  原因。

别人 200 → 直接证明 token 本身没问题，100% 环境差异

先对时，再比对 issuer/audience → 解决 90% 的「本地 401」。

扩展 Grant 绕过了默认策略，所以 sms 能跑 ≠ password 没问题。

JWT 只要解出来一致，就一定是请求环境/中间件配置差异，别再跟 token 较劲。

| <font style="color:#000000;">现象</font> | <font style="color:#000000;">本地电脑</font> | <font style="color:#000000;">别人电脑</font> | <font style="color:#000000;">可能根因</font> | <font style="color:#000000;">快速验证办法</font> |
| --- | --- | --- | --- | --- |
| <font style="color:#000000;">✅</font><font style="color:#000000;"> password 令牌 401</font> | <font style="color:#000000;">❌</font> | <font style="color:#000000;">✅</font> | <font style="color:#000000;">本地时间偏差 > 5 min → JWT `nbf` / `exp` 校验失败</font> | <font style="color:#000000;">对时：`time.is` 或 `w32tm /resync`</font> |
| <font style="color:#000000;">✅</font><font style="color:#000000;"> password 令牌 401</font> | <font style="color:#000000;">❌</font> | <font style="color:#000000;">✅</font> | <font style="color:#000000;">本机 DNS 缓存指向了旧容器 / 旧集群 → 拿到别人的 issuer</font> | <font style="color:#000000;">`ping openid.f2b2c.shop` 对比 IP</font> |
| <font style="color:#000000;">✅</font><font style="color:#000000;"> password 令牌 401</font> | <font style="color:#000000;">❌</font> | <font style="color:#000000;">✅</font> | <font style="color:#000000;">浏览器/Postman 自动带上了过期 Cookie → 冲突</font> | <font style="color:#000000;">无痕窗口或 `curl -H "Cookie:"` 复现</font> |
| <font style="color:#000000;">✅</font><font style="color:#000000;"> SmsTokenExtensionGrant 令牌 200</font> | <font style="color:#000000;">✅</font> | <font style="color:#000000;">✅</font> | <font style="color:#000000;">扩展 Grant 里手动写了 issuer/audience，不受默认中间件校验策略影响</font> | <font style="color:#000000;">把两边 token 丢到 https://jwt.io 比对 `iss` / `aud`</font> |
| <font style="color:#000000;">✅</font><font style="color:#000000;"> password 令牌 401</font> | <font style="color:#000000;">❌</font> | <font style="color:#000000;">✅</font> | <font style="color:#000000;">本机代理 / VPN 把 Authorization 头吃掉</font> | <font style="color:#000000;">关闭代理再试</font> |


<h5 id="QXzfz">（1）把两条 token 都复制出来</h5>
电脑 A（401） →  eyJ0eXAiOiJKV1QiLCJhbGc... 

电脑 B（200） →  eyJ0eXAiOiJKV1QiLCJhbGc... 

在线解一下（jwt.io 或  jq -R 'split(".") | .[1] | @base64d | fromjson' ）

 重点看：

"iss": "[https://openid.f2b2c.shop"](https://openid.f2b2c.shop")   // 必须和 API 配置的 Authority 完全一致

"aud": "XShop_Api"                   // 必须和 API 的 JwtBearer:Audience 一致

"nbf": 1720000000                    // 检查本地 Unix 时间

"exp": 1720003600

"iat": 1720000000

如果 iss 多了端口或少了 https，说明本机请求走到了另一套授权中心。

<h5 id="ohYS7">（2）解决</h5>
把本机  C:\Windows\System32\drivers\etc\hosts  里硬绑的 IP 注释掉；

如果时间差 > 60 s

Windows：

net stop w32time

w32tm /config /syncfromflags:manual /manualpeerlist:ntp.aliyun.com

net start w32time

w32tm /resync

如果 token 里  amr  不一样

 password 返回  amr: ["pwd"] 

 SmsTokenExtensionGrant 返回  amr: ["sms"] 

 而 API 的  AddJwtBearer()  里又写了

options.TokenValidationParameters.AuthenticationType = "pwd";

会把 sms的token拒绝掉 — 你这里是反过来的，说明 API 没限制 amr，问题仍在时间/issuer/audience三点。

<h4 id="IX82x">3、回调页  /signin-oidc </h4>
 纯前端路由即可，拿到  ?code=xxx  后立刻用  code + code_verifier  换 token：

```typescript
const params = new URLSearchParams(location.search);
const code = params.get('code');
const verifier = localStorage.getItem('code_verifier');

const body = new URLSearchParams({
  grant_type: 'authorization_code',
  client_id: 'XShop_Arco',
  code: code!,
  redirect_uri: 'https://arco.f2b2c.shop/signin-oidc',
  code_verifier: verifier!
});

fetch('https://openid.f2b2c.shop/connect/token', {
  method: 'POST',
  headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
  body
})
.then(r => r.json())
.then(tok => {
  localStorage.setItem('access_token', tok.access_token);
  localStorage.setItem('refresh_token', tok.refresh_token);
  history.replace('/');   // 回到首页
});

```

<h4 id="fiXoc">4、静默刷新（可选）</h4>
如果希望「AT 过期但 RT 还在」时不跳授权中心，把上面 401 分支改成先用 RT 换 AT，失败再跳授权中心即可。

<h4 id="WD62T">5、常见坑速查</h4>
| **<font style="color:#000000;">现象</font>** | **<font style="color:#000000;">原因</font>** |
| --- | --- |
| <font style="color:#000000;">授权中心报 `invalid_client`</font> | <font style="color:#000000;">SQL 里 ClientType 给成 `confidential` 却没带 secret；SPA 必须是 `public`。</font> |
| <font style="color:#000000;">报 `missing code_challenge`</font> | <font style="color:#000000;">前端跳转地址没加 `code_challenge` 参数；SPA 强制 PKCE。</font> |
| <font style="color:#000000;">报 `unauthorized_client`</font> | <font style="color:#000000;">Permissions 里没给 `gt:authorization_code` 或 `rst:code`。</font> |
| <font style="color:#000000;">跳转后无限循环</font> | <font style="color:#000000;">回调页又触发 401 → 再跳授权中心；回调页里一定要先换完 token 再 `replace('/')`。</font> |


<h3 id="hO5MK">**<font style="color:rgb(38, 38, 38);">五、前端与授权站点和API站点的交互</font>**</h3>
<h4 id="pKNRE"><font style="color:rgb(38, 38, 38);">1、用户认证流程</font></h4>
![](https://cdn.nlark.com/yuque/__mermaid_v3/6661c86af4f2e1ead5f8501152c01160.svg)

<h4 id="mXtNP"><font style="color:rgb(38, 38, 38);">2、前端交互流程</font></h4>
![](https://cdn.nlark.com/yuque/__mermaid_v3/3d818c8cfd3f20b531c7853f13d32a06.svg)

<h4 id="CZKzv"><font style="color:rgb(38, 38, 38);">3、前端交互细节</font></h4>
<font style="color:rgb(223, 42, 63);">注意：Arco站点需要传递clientId给授权站点（clientId：</font><font style="color:#000000;">XShop_App</font><font style="color:rgb(223, 42, 63);">）</font>

```typescript
站点在向授权中心请求时，传递的clientId，设置为 XShop_App
const oAuthConfig = {
  issuer: 'https://openid.f2b2c.shop/',
  redirectUri: baseUrl,
  clientId: 'XShop_App',
  responseType: 'code',
  scope: 'offline_access XShop',
  requireHttps: true,
  impersonation: {
    tenantImpersonation: true,
    userImpersonation: true,
  }
};
```

<h4 id="M7J81"><font style="color:rgb(38, 38, 38);">4、自有登录页与oidc授权码流的对比</font></h4>
<font style="color:rgb(38, 38, 38);">简单起见，先使用跳转授权站点的方式来完成登录认证</font>

| **<font style="color:rgb(0, 0, 0);">对比项</font>** | **<font style="color:rgb(0, 0, 0);">跳转授权站点（PKCE）</font>** | **<font style="color:rgb(0, 0, 0);">Arco 自有页面（直接登录）</font>** |
| --- | --- | --- |
| <font style="color:rgb(0, 0, 0);">用户体验</font> | <font style="color:rgb(0, 0, 0);">跳走 → 登录中心 → 跳回；地址栏可见变化</font> | <font style="color:rgb(0, 0, 0);">全程留在 Arco 页面，无跳转</font> |
| <font style="color:rgb(0, 0, 0);">协议流</font> | <font style="color:rgb(0, 0, 0);">授权码流（Authorization Code + PKCE）</font> | <font style="color:rgb(0, 0, 0);">资源所有者密码流（ROPC）</font> |
| <font style="color:rgb(0, 0, 0);">前端实现</font> | <font style="color:rgb(0, 0, 0);">用 `oidc-client-ts` 做重定向、回调、静默刷新</font> | <font style="color:rgb(0, 0, 0);">用 axios 直接调 `/api/account/login` 即可</font> |
| <font style="color:rgb(0, 0, 0);">安全等级</font> | <font style="color:rgb(0, 0, 0);">高（token 只在后端交互，前端无密码）</font> | <font style="color:rgb(0, 0, 0);">中等（密码经过前端，需 HTTPS+防 XSS）</font> |
| <font style="color:rgb(0, 0, 0);">刷新/退出</font> | <font style="color:rgb(0, 0, 0);">标准 `/connect/token` & `/connect/logout`</font> | <font style="color:rgb(0, 0, 0);">自定义 `/api/account/refresh` & `/logout`</font> |
| <font style="color:rgb(0, 0, 0);">多租户/多语言</font> | <font style="color:rgb(0, 0, 0);">通过 authorize URL 参数传递</font> | <font style="color:rgb(0, 0, 0);">通过 login/refresh 请求体 Header 传递</font> |


<h4 id="vvqxT"><font style="color:rgb(38, 38, 38);">5、自有登录页面的认证流程</font></h4>
<font style="color:rgb(38, 38, 38);">点击全屏按钮查看</font>

![](https://cdn.nlark.com/yuque/__mermaid_v3/76678bbd4e5e598962f0e216bc619d02.svg)

<h4 id="wYG1v">6、angular站点与授权站点的交互流程（跟Arco站点略有不同）</h4>
code 换 token 的动作在 API 层完成（ AccountController.ExternalLoginCallback ）。

浏览器不会保存 access_token，也不会发  Authorization: Bearer ；它只认同域 Cookie。

因为后端有 client_secret，所以  Requirements  列可以空着；PKCE 对 confidential client 是可选。

与 Arco 纯 SPA 的差异对照

| **<font style="color:#000000;">维度</font>** | **<font style="color:#000000;">ABP 默认 Angular</font>** | **<font style="color:#000000;">Arco 纯 SPA</font>** |
| --- | --- | --- |
| <font style="color:#000000;">client_type</font> | <font style="color:#000000;">confidential（带 secret）</font> | <font style="color:#000000;">public（无 secret）</font> |
| <font style="color:#000000;">换 code 地点</font> | <font style="color:#000000;">后端 API (`/signin-oidc`)</font> | <font style="color:#000000;">浏览器 (`fetch('/connect/token')`)</font> |
| <font style="color:#000000;">需要 PKCE</font> | <font style="color:#000000;">❌</font><font style="color:#000000;"> 可空</font> | <font style="color:#000000;">✅</font><font style="color:#000000;"> 必须 `["PKCE"</font> |
| <font style="color:#000000;">需要保存令牌</font> | <font style="color:#000000;">❌</font><font style="color:#000000;"> 用 Cookie</font> | <font style="color:#000000;">✅</font><font style="color:#000000;"> localStorage / memory</font> |
| <font style="color:#000000;">请求 API 时</font> | <font style="color:#000000;">自动带 Cookie</font> | <font style="color:#000000;">手动加 `Authorization: Bearer`</font> |
| <font style="color:#000000;">令牌 401 处理</font> | <font style="color:#000000;">后端 302 → /account/login</font> | <font style="color:#000000;">前端拦截器 → 拼 authorize URL</font> |


什么情况下 Angular 也要改成 PKCE？

把 Angular 部署到非 ABP 同源域名（例如  [https://admin.f2b2c.shop](https://admin.f2b2c.shop)  vs  [https://api.f2b2c.shop](https://api.f2b2c.shop) ）

想彻底去掉 Cookie，用 Bearer 做真正的无状态

需要第三方登录（微信、GitHub）且前端要拿到 access_token 去调用外部 API

此时只要把

```typescript
Requirements: ["PKCE","HTTPS"]
ClientType: "public"
ClientSecret: 删空
```

再把  AccountController  的代理逻辑砍掉

让 Angular 自己用 oidc-client-js 走标准授权码即可，流程就和 Arco 完全一致了。

![](https://cdn.nlark.com/yuque/__mermaid_v3/ad6115c66c0226dea4ea4f205cb4f38c.svg)

<h3 id="Ax5HC">六、Arco 端工程结构</h3>
```plain
portal/
├─ .env.development
├─ src/
│  ├─ abp/                    # generate-proxy 输出（只读）
│  │  ├─ services/            # abp.services.xxx
│  │  └─ models/              # 后端 DTO 类型
│  ├─ plugins/                # 自写 5 大插件
│  │  ├─ oidc.ts
│  │  ├─ tenant.ts
│  │  ├─ permission.ts
│  │  ├─ locale.ts
│  │  └─ api.ts               # axios 实例
│  ├─ stores/                 # Pinia
│  │  ├─ auth.store.ts
│  │  ├─ tenant.store.ts
│  │  └─ locale.store.ts
│  ├─ views/                  # 预置 5 套页面
│  │  ├─ account/             # 登录、个人中心
│  │  ├─ identity/            # 用户/角色/组织
│  │  ├─ language/            # 多语言文本
│  │  ├─ tenant/              # 租户管理
│  │  └─ saas/                # 版本/功能
│  ├─ components/
│  │  └─ Authority/
│  │     ├─ index.vue         # <a-authority>
│  │     └─ directive.ts      # v-permission
│  └─ utils/
│     └─ abp-error-handler.ts # 统一 __abp 包装解析
├─ package.json
└─ vite.config.ts
```

---

<h3 id="kOLoG">七、5 大自写插件（核心，直接复制）  </h3>
<h4 id="BedfZ">OIDC 插件 `src/plugins/oidc.ts`</h4>
```typescript
import { UserManager, User } from 'oidc-client-ts'
import { useAuthStore } from '@/stores/auth.store'

const mgr = new UserManager({
  authority: import.meta.env.VITE_OIDC_AUTHORITY,
  client_id: 'acme_portal',
  redirect_uri: `${window.location.origin}/signin-callback`,
  silent_redirect_uri: `${window.location.origin}/silent-callback`,
  post_logout_redirect_uri: `${window.location.origin}/logout-callback`,
  response_type: 'code',
  scope: 'openid profile email phone roles offline_access Acme.Product Acme.Order',
  automaticSilentRenew: true,
  monitorSession: true,
  loadUserInfo: true
})

mgr.events.addAccessTokenExpiring(async () => {
  await mgr.signinSilent()
})

export default mgr
```

<h4 id="f8l3C">Axios 实例 + 拦截器 `src/plugins/api.ts`</h4>
```typescript
import axios, { AxiosInstance } from 'axios'
import { useAuthStore } from '@/stores/auth.store'
import { useTenantStore } from '@/stores/tenant.store'
import { abpErrorHandler } from '@/utils/abp-error-handler'

const api: AxiosInstance = axios.create({
  baseURL: import.meta.env.VITE_API_BASE,
  timeout: 10 * 1000
})

api.interceptors.request.use(cfg => {
  const authStore = useAuthStore()
  const tenantStore = useTenantStore()
  if (authStore.accessToken) cfg.headers.Authorization = `Bearer ${authStore.accessToken}`
  if (tenantStore.current?.id) cfg.headers['__tenant'] = tenantStore.current.id
  return cfg
})

api.interceptors.response.use(
  res => {
    // 统一拆 __abp
    const abp = res.data?.__abp
    if (abp === true) return res.data.result ?? res.data
    return res.data
  },
  async err => {
    await abpErrorHandler(err) // 401→自动刷新，403→弹 tenant 冲突
    return Promise.reject(err)
  }
)

export default api
```

<h4 id="gxMKC">多租户插件 `src/plugins/tenant.ts`</h4>
```typescript
import mgr from './oidc'
import { useTenantStore } from '@/stores/tenant.store'

export async function switchTenant(tenantId: string | null) {
  const store = useTenantStore()
  store.setCurrent(tenantId)
  // 重新获取权限 & 菜单
  await store.loadConfiguration()
  // 静默刷新令牌，保证新租户立即生效
  await mgr.signinSilent()
  window.location.reload() // 简单暴力，亦可动态重载路由
}
```

<h4 id="F52cL">权限指令 & 组件</h4>
```typescript
// src/components/Authority/directive.ts
import { useAuthStore } from '@/stores/auth.store'
export const vPermission = {
  mounted(el, binding) {
    const authStore = useAuthStore()
    const granted = authStore.grantedPolicies[binding.value]
    if (!granted) el.parentNode?.removeChild(el)
  }
}
```

```vue
<!-- src/components/Authority/index.vue -->
<template>
  <slot v-if="granted" />
</template>
<script setup>
import { computed } from 'vue'
import { useAuthStore } from '@/stores/auth.store'
const props = defineProps<{ policy: string }>()
const authStore = useAuthStore()
const granted = computed(() => authStore.grantedPolicies[props.policy])
</script>

```

<h4 id="lYw84">本地化插件 `src/plugins/locale.ts`</h4>
```typescript
import { createI18n } from 'vue-i18n'
import axios from 'axios'
import { Locale } from '@arco-design/web-vue/es/locale/interface'

export async function loadAbpLocale(culture: string): Promise<Locale> {
  const { data } = await axios.get(`/api/abp/application-localization?culture=${culture}`)
  const messages = flattenTexts(data.resources) // 把嵌套 key 展平
  return { locale: culture, messages } as Locale
}

function flattenTexts(obj: any, prefix = '', res: Record<string, string> = {}) {
  for (const [k, v] of Object.entries(obj)) {
    const key = prefix ? `${prefix}.${k}` : k
    if (typeof v === 'object') flattenTexts(v, key, res)
    else res[key] = v as string
  }
  return res
}
```

---

<h3 id="dZut4">八、生成 TS 客户端（零手写 URL）  </h3>
```bash
npm i -D @abp/generator-proxy
npx abp-generate-proxy -t vue -u https://api.x.com -m Identity
npx abp-generate-proxy -t vue -u https://api.x.com -m Saas
# 输出到 src/abp/，git 忽略，CI 自动生成
```

使用示例：  

```typescript
import { IdentityUserService } from '@/abp/services/IdentityUserService'
const { items } = await IdentityUserService.getList({ maxResultCount: 10 })
```

---

<h3 id="NF9JH">九、预置 5 套页面（直接复用 Arco Pro 模板）  </h3>
页面	Arco Pro 模板	需要字段（后端 DTO 已带）	权限码	  
登录	`<a-form>`	用户名/密码/验证码	无	  
租户列表	`<a-table>`	Id, Name, EditionName, IsActive	`Saas.Tenants`	  
用户列表	`<a-table>`	Id, UserName, Email, IsActive, Roles[]	`AbpIdentity.Users`	  
角色列表	`<a-table>`	Id, Name, IsDefault, IsStatic	`AbpIdentity.Roles`	  
语言文本	`<a-table>`	Key, BaseValue, TargetValue	`LanguageManagement.LanguageTexts`	

---

<h3 id="ZFD4Y">十、常见坑 & 一键诊断  </h3>
现象	根因	解决	  
登录后 401	未带 `__tenant` 或 `Authorization`	检查 axios 拦截器顺序	  
403 tenant mismatch	切换租户未重载权限	`switchTenant()` 后 `reload()`	  
接口返回 `__abp: true` 但前端拿不到 data	axios 拦截器未拆 `result`	在 `api.ts` 统一拆包	  
生成代理 404	模块未启用或 URL 错	确认 `-m` 大小写与后端一致	  
CORS blocked	AuthServer 未配前端域	`appsettings.json::CorsOrigins` 加 `https://portal.x.com`	

---

---



