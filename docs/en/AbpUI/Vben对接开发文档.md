### 一、总体目标
Vue-Vben-Admin v5（域名：https://vben.neibuguanli.cn），完整对接ABP后端，跑通以下基础功能：

#### OIDC-PKCE 登录 / 刷新令牌 / 401 自动刷新  
#### 租户下拉切换（自动带 `__tenant` 头）  
#### 动态左侧菜单（ABP `/api/abp/application-configuration`）  
#### 按钮级权限（`v-auth` 指令）  
#### 多语言（ABP `/api/language-management/languages`）  
#### 主题/布局/锁屏/404/500 等走查
### 二、上线前验证
验证步骤（上线前 checklist）

#### 1. 登录是否跳转
访问 [https://vben.neibuguanli.cn](https://vben.neibuguanli.cn) → 点击登录 → 应跳转到 [https://oidc.neibuguanli.cn/connect/authorize](https://oidc.neibuguanli.cn/connect/authorize)

#### 2. 登录成功后是否拿到code
登录成功后 → 浏览器地址栏回到 [https://vben.neibuguanli.cn/signin-oidc](https://vben.neibuguanli.cn/signin-oidc) 并拿到 code

#### 3. 前端是否使用code换取到令牌
前端立即用 code 换 token → 可看到 access_token、refresh_token、id_token

#### 4. 前端请求api是否成功
访问 [https://api.neibuguanli.cn/api/abp/application-configuration](https://api.neibuguanli.cn/api/abp/application-configuration) → 请求头带 Bearer access_token → 返回 200

#### 5. 前端注销是否跳转
点击注销 → 跳回 [https://vben.neibuguanli.cn/signout-oidc](https://vben.neibuguanli.cn/signout-oidc) → 本地 token 被清空

#### 6. 编译是否正常
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

### 三、开发资料
基于vben开发 [https://github.com/vbenjs/vue-vben-admin](https://github.com/vbenjs/vue-vben-admin) 

vben文档 [https://doc.vben.pro/](https://doc.vben.pro/)	

vben演示 [https://www.vben.pro/](https://www.vben.pro/)  <font style="color:rgb(31, 35, 40);">账号：vben     密码：123456</font>

<font style="color:rgb(31, 35, 40);">参考vben5集成案例 </font>[https://gitee.com/project-admin_2026/vben5.git](https://gitee.com/project-admin_2026/vben5.git)

<font style="color:rgb(31, 35, 40);">提交仓库 </font>[https://gitee.com/gdmzfs0753/vben-admin.git](https://gitee.com/gdmzfs0753/vben-admin.git)

swagger [https://api.neibuguanli.cn/swagger/index.html](https://api.neibuguanli.cn/swagger/index.html)

API文档 [https://www.yuque.com/u253383/hlft5f/vtun8reruh965v0q?singleDoc#Bze5](https://www.yuque.com/u253383/hlft5f/vtun8reruh965v0q?singleDoc#Bze5) 《API》 密码：rdpd

oidc授权 [https://oidc.neibuguanli.cn/](https://oidc.neibuguanli.cn/)

参考angular代码 [https://gitee.com/project-admin_2026/project-admin.-angular.git](https://gitee.com/project-admin_2026/project-admin.-angular.git)

angular代码的<font style="color:#000000;">environment.prod.ts 和 protractor.conf.js ，这里会配置api站点和oidc授权站点</font>

<font style="color:#000000;">npm install -g @angular/cli   安装cli</font>

<font style="color:#000000;">执行 ng serve 启动站点 </font>[http://localhost:4200/](http://localhost:4200/)    

angular 线上站点  [https://vue.neibuguanli.cn/](https://vue.neibuguanli.cn/)       账号 admin 密码 1q2w3E*

#### 使用ftp工具发布vben站点
安装ftp客户端工具：[https://download.filezilla.cn/client/windows/FileZilla_3.68.1_win64-setup.exe](https://download.filezilla.cn/client/windows/FileZilla_3.68.1_win64-setup.exe)

安装后启动程序，文件=》站点管理器=》新站点=》vben

主机：43.139.163.237

端口：21

登录类型：正常

账号：vben

密码：Vben@67890

点击连接，保存密码，连接成功后，可以看到远程的目录

### 四、跟授权中心握手
拦截器配完后，Vben 即可实现：

首次登录 → 授权码 → 换令牌 → 令牌过期 → 自动重定向到授权中心 → 重新登录 → 回到 Vben。

#### 1、令牌失效后的 302 跳转 不会自动发生
需要你在 Vben 里加 拦截器 手动触发  location.href = .../connect/authorize?... ；

```typescript
// src/utils/request.ts
import axios from 'axios';

const AUTH_SERVER = 'https://openid.f2b2c.shop';
const CLIENT_ID   = 'ProjectAdmin_Vben_App';
const REDIRECT_URI= encodeURIComponent('https://vben.f2b2c.shop/signin-oidc');

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
        `&scope=openid profile roles email phone ProjectAdmin` +
        `&state=${Math.random().toString(36).slice(2)}` +
        `&code_challenge=${codeChallenge}` +
        `&code_challenge_method=S256` +
        `&prompt=login`;   // 强制重新登录，可按需删掉

      location.href = url;          // 302 跳转
    }
    return Promise.reject(err);
  });

```

#### 2、回调页  /signin-oidc 
 纯前端路由即可，拿到  ?code=xxx  后立刻用  code + code_verifier  换 token：

```typescript
const params = new URLSearchParams(location.search);
const code = params.get('code');
const verifier = localStorage.getItem('code_verifier');

const body = new URLSearchParams({
  grant_type: 'authorization_code',
  client_id: 'ProjectAdmin_Vben_App',
  code: code!,
  redirect_uri: 'https://vben.f2b2c.shop/signin-oidc',
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

#### 3、静默刷新（可选）
如果希望「AT 过期但 RT 还在」时不跳授权中心，把上面 401 分支改成先用 RT 换 AT，失败再跳授权中心即可。

#### 4、常见坑速查
| **<font style="color:#000000;">现象</font>** | **<font style="color:#000000;">原因</font>** |
| --- | --- |
| <font style="color:#000000;">授权中心报 `invalid_client`</font> | <font style="color:#000000;">SQL 里 ClientType 给成 `confidential` 却没带 secret；SPA 必须是 `public`。</font> |
| <font style="color:#000000;">报 `missing code_challenge`</font> | <font style="color:#000000;">前端跳转地址没加 `code_challenge` 参数；SPA 强制 PKCE。</font> |
| <font style="color:#000000;">报 `unauthorized_client`</font> | <font style="color:#000000;">Permissions 里没给 `gt:authorization_code` 或 `rst:code`。</font> |
| <font style="color:#000000;">跳转后无限循环</font> | <font style="color:#000000;">回调页又触发 401 → 再跳授权中心；回调页里一定要先换完 token 再 `replace('/')`。</font> |


### 五、前端与授权站点和API站点的交互
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

### 六、架构改动总览
模块	主要改动	文件	

#### OIDC 登录
用 `oidc-client-ts` 替换原有 mock	`/src/utils/auth/oidc.ts`	

#### 请求封装
统一处理 `__abp` 包装、tenant 头、401 刷新	`/src/utils/http/abp.ts`	

#### 菜单/权限
解析 `application-configuration`	`/src/store/modules/abp.ts`	

#### 多语言
动态加载 ABP JSON	`/src/locale/lang/abp.ts`	

#### 租户切换
下拉组件 + 请求头拦截	`/src/components/TenantSelect/index.vue`	

#### 指令	`v-auth` 
绑定 ABP policy	`/src/directives/auth.ts`

---

### 七、详细功能拆解与代码
#### 1、关键配置
| <font style="color:#000000;">#</font> | **<font style="color:#000000;">文件/位置</font>** | **<font style="color:#000000;">关键字段</font>** | **<font style="color:#000000;">值</font>** | **<font style="color:#000000;">备注</font>** |
| --- | --- | --- | --- | --- |
| <font style="color:#000000;">1</font> | **<font style="color:#000000;">.env / environment.ts</font>** | <font style="color:#000000;">issuer</font> | <font style="color:#000000;">https://oidc.neibuguanli.cn</font> | <font style="color:#000000;">授权中心地址</font> |
| <font style="color:#000000;">2</font> | **<font style="color:#000000;">.env / environment.ts</font>** | <font style="color:#000000;">clientId</font> | <font style="color:#000000;">ProjectAdmin_Vben_App</font> | <font style="color:#000000;">与数据库保持一致</font> |
| <font style="color:#000000;">3</font> | **<font style="color:#000000;">.env / environment.ts</font>** | <font style="color:#000000;">redirectUri</font> | <font style="color:#000000;">https://vben.neibuguanli.cn/signin-oidc</font> | <font style="color:#000000;">必须与数据库第 4 行一致</font> |
| <font style="color:#000000;">4</font> | **<font style="color:#000000;">.env / environment.ts</font>** | <font style="color:#000000;">postLogoutRedirectUri</font> | <font style="color:#000000;">https://vben.neibuguanli.cn/signout-oidc</font> | <font style="color:#000000;">必须与数据库第 5 行一致</font> |
| <font style="color:#000000;">5</font> | **<font style="color:#000000;">.env / environment.ts</font>** | <font style="color:#000000;">responseType</font> | <font style="color:#000000;">code</font> | <font style="color:#000000;">授权码模式</font> |
| <font style="color:#000000;">6</font> | **<font style="color:#000000;">.env / environment.ts</font>** | <font style="color:#000000;">scope</font> | <font style="color:#000000;">openid profile offline_access ProjectAdmin</font> | <font style="color:#000000;">与数据库 Permissions 中的 scp 对应</font> |
| <font style="color:#000000;">7</font> | **<font style="color:#000000;">.env / environment.ts</font>** | <font style="color:#000000;">requireHttps</font> | <font style="color:#000000;">TRUE</font> | <font style="color:#000000;">生产环境强制 HTTPS</font> |
| <font style="color:#000000;">8</font> | **<font style="color:#000000;">oidc-client-ts / angular-oauth2-oidc</font>** | <font style="color:#000000;">useSilentRefresh</font> | <font style="color:#000000;">TRUE</font> | <font style="color:#000000;">可选，开启静默刷新</font> |
| <font style="color:#000000;">9</font> | **<font style="color:#000000;">oidc-client-ts / angular-oauth2-oidc</font>** | <font style="color:#000000;">silentRequestTimeout</font> | <font style="color:#000000;">10000</font> | <font style="color:#000000;">静默刷新超时</font> |
| <font style="color:#000000;">10</font> | **<font style="color:#000000;">nginx / CDN</font>** | <font style="color:#000000;">CSP / CORS</font> | <font style="color:#000000;">允许 https://oidc.neibuguanli.cn 的 iframe 与重定向</font> | <font style="color:#000000;">避免静默刷新被浏览器拦截</font> |


#### 2、OIDC-PKCE 登录
ABP 9.3 默认使用 OpenIddict，前端必须走 PKCE 模式；

登录后把 `access_token` + `refresh_token` 存 `localStorage`，

后续请求自动带 `Authorization: Bearer`。

安装依赖

```bash
pnpm add oidc-client-ts
```

代码：/src/utils/auth/oidc.ts

```typescript
import { UserManager, UserManagerSettings, User } from 'oidc-client-ts';

const settings: UserManagerSettings = {
  authority: import.meta.env.VITE_GLOB_OIDC_AUTHORITY,
  client_id: import.meta.env.VITE_GLOB_OIDC_CLIENT_ID,
  redirect_uri: `${window.location.origin}/oidc-callback`,
  silent_redirect_uri: `${window.location.origin}/oidc-silent`,
  response_type: 'code',
  scope: 'openid profile role email offline_access',
  automaticSilentRenew: true,
  monitorSession: true,
  loadUserInfo: true,
};

export const userManager = new UserManager(settings);
export const login = () => userManager.signinRedirect();
export const signinCallback = () => userManager.signinCallback();
export const renewToken = () => userManager.signinSilent();
export const logout = () => userManager.signoutRedirect();
```

路由守卫：/src/router/guard.ts

```typescript
import { userManager } from '/@/utils/auth/oidc';
router.beforeEach(async (to) => {
  const user = await userManager.getUser();
  if (!user && to.path !== '/oidc-callback') {
    return { path: '/oidc-callback', query: { redirect: to.fullPath } };
  }
});
```

组件：/src/views/sys/oidc-callback.vue

```vue
<script setup lang="ts">
import { onMounted } from 'vue';
import { useRouter } from 'vue-router';
import { userManager } from '/@/utils/auth/oidc';

onMounted(async () => {
  await userManager.signinCallback();
  router.push('/');
});
</script>

```

---

#### 3、请求封装（解决 `__abp` 包装 / 401 / 403 / tenant）
文件：/src/utils/http/abp.ts

```typescript
import axios, { AxiosInstance } from 'axios';
import { useMessage } from '/@/hooks/web/useMessage';
import { useI18n } from '/@/hooks/web/useI18n';
import { getToken, refreshToken } from '/@/utils/auth';
import { useTenantStore } from '/@/store/modules/tenant';

const http: AxiosInstance = axios.create({
  baseURL: import.meta.env.VITE_GLOB_API_URL,
  timeout: 10 * 1000,
});

// 请求拦截
http.interceptors.request.use((config) => {
  // token
  const token = getToken();
  if (token) config.headers.Authorization = `Bearer ${token}`;
  // tenant
  const tenantStore = useTenantStore();
  if (tenantStore.current?.id) {
    config.headers['__tenant'] = tenantStore.current.id;
  }
  return config;
});

// 响应拦截
http.interceptors.response.use(
  (res) => {
    // ABP 包装
    const abp = res.data?.__abp;
    if (abp === true) return res.data.result ?? res.data;
    return res.data;
  },
  async (err) => {
    const { response } = err;
    if (response?.status === 401) {
      try {
        await refreshToken();
        return http(err.config); // 重试
      } catch {
        await userManager.signoutRedirect();
      }
    }
    if (response?.status === 403 && response.data?.error?.message?.includes('tenant')) {
      useMessage().error(t('sys.tenantMismatch'));
    }
    return Promise.reject(err);
  },
);

export default http;
```

#### 4、动态菜单 & 权限
ABP 返回格式

`GET /api/abp/application-configuration`

其中 `auth.policies` 为权限列表，`menu` 由后端写死或自定义表，建议后端新增：

```plain
/api/menu/user-menus → List<MenuDto>
```

前端：/src/store/modules/abp.ts

```typescript
export const useAbpStore = defineStore('abp', {
  state: () => ({
    configuration: null as any,
    menus: [] as Menu[],
  }),
  actions: {
    async load() {
      const [config, menus] = await Promise.all([
        http.get('/api/abp/application-configuration'),
        http.get('/api/menu/user-menus'),
      ]);
      this.configuration = config;
      this.menus = menus;
    },
  },
});
```

动态路由：/src/router/routes.ts

```typescript
export async function generateDynamicRoutes() {
  const abpStore = useAbpStore();
  await abpStore.load();
  const routeList = transformAbpMenuToRoute(abpStore.menus);
  routeList.forEach((r) => router.addRoute(r));
}
```

#### 5、按钮权限 v-auth
指令：/src/directives/auth.ts

```typescript
import { useAbpStore } from '/@/store/modules/abp';

function check(el: HTMLElement, binding: DirectiveBinding) {
  const policy = binding.value;
  const abpStore = useAbpStore();
  const granted = abpStore.configuration.auth.grantedPolicies[policy];
  if (!granted) el.remove();
}

export const auth: Directive = {
  mounted: check,
  updated: check,
};
```

全局注册：/src/directives/index.ts

```typescript
app.directive('auth', auth);
```

使用

```vue
<a-button v-auth="'AbpIdentity.Users.Create'">新增用户</a-button>

```

---

#### 6、租户下拉组件
组件：/src/components/TenantSelect/index.vue

```vue
<template>
  <a-select :value="current?.id" @change="switchTenant">
    <a-option v-for="t in list" :key="t.id" :label="t.name" :value="t.id" />
  </a-select>
</template>
<script setup lang="ts">
import { useTenantStore } from '/@/store/modules/tenant';
const tenantStore = useTenantStore();
const list = computed(() => tenantStore.list);
const current = computed(() => tenantStore.current);
function switchTenant(id) {
  tenantStore.setCurrent(id);
  location.reload(); // 简单暴力
}
onMounted(() => tenantStore.load());
</script>

```

#### 7、多语言
ABP 接口

`/api/language-management/languages?onlyActive=true`

`/api/language-management/labels?culture=zh-Hans`

代码：/src/locale/lang/abp.ts

```typescript
export async function loadAbpMessages(culture: string) {
  const { data } = await http.get(`/api/language-management/labels`, { params: { culture } });
  return data; // { key: value }
}
```

在 useI18n 中合并

```typescript
const messages = await loadAbpMessages(locale);
i18n.setLocaleMessage(locale, messages);
```

#### 8、环境变量清单
`src\environments\environment.prod.ts`

```plain
VITE_GLOB_API_URL=https://api.neibuguanli.cn
VITE_GLOB_OIDC_AUTHORITY=https://oidc.neibuguanli.cn
VITE_GLOB_OIDC_CLIENT_ID=ProjectAdmin_Vben_App
VITE_GLOB_OIDC_SCOPE=openid profile offline_access ProjectAdmin 
```

### 八、基础功能开发
完成上述的登录授权之后，再接入以下菜单的功能

#### 1、菜单功能
| **<font style="color:#000000;">一级菜单</font>** | **<font style="color:#000000;">二级菜单</font>** | **<font style="color:#000000;">后端主接口（Swagger）</font>** | **<font style="color:#000000;">Vben 页面路径</font>** | **<font style="color:#000000;">备注</font>** |
| --- | --- | --- | --- | --- |
| <font style="color:#000000;">管理</font> | <font style="color:#000000;">身份管理</font> | | | |
| | <font style="color:#000000;">用户</font> | <font style="color:#000000;">`/api/identity/users`</font> | <font style="color:#000000;">`/src/views/admin/identity/user/index.vue`</font> | <font style="color:#000000;">含组织、角色、声明</font> |
| | <font style="color:#000000;">角色</font> | <font style="color:#000000;">`/api/identity/roles`</font> | <font style="color:#000000;">`/views/admin/identity/role/index.vue`</font> | <font style="color:#000000;">权限树</font> |
| | <font style="color:#000000;">组织机构</font> | <font style="color:#000000;">`/api/identity/organization-units`</font> | <font style="color:#000000;">`/views/admin/identity/ou/index.vue`</font> | <font style="color:#000000;">树表</font> |
| | <font style="color:#000000;">声明类型</font> | <font style="color:#000000;">`/api/identity/claim-types`</font> | <font style="color:#000000;">`/views/admin/identity/claim/index.vue`</font> | <font style="color:#000000;">动态声明</font> |
| | <font style="color:#000000;">安全日志</font> | <font style="color:#000000;">`/api/identity/security-logs`</font> | <font style="color:#000000;">`/views/admin/identity/security-log/index.vue`</font> | <font style="color:#000000;">登录日志</font> |
| <font style="color:#000000;">OpenId</font> | <font style="color:#000000;">应用</font> | <font style="color:#000000;">`/api/openiddict/applications`</font> | <font style="color:#000000;">`/views/admin/openiddict/application/index.vue`</font> | <font style="color:#000000;">PKCE 客户端</font> |
| | <font style="color:#000000;">范围</font> | <font style="color:#000000;">`/api/openiddict/scopes`</font> | <font style="color:#000000;">`/views/admin/openiddict/scope/index.vue`</font> | <font style="color:#000000;">API 范围</font> |
| <font style="color:#000000;">语言管理</font> | <font style="color:#000000;">语言</font> | <font style="color:#000000;">`/api/language-management/languages`</font> | <font style="color:#000000;">`/views/admin/language/index.vue`</font> | <font style="color:#000000;">多语言列表</font> |
| | <font style="color:#000000;">语言文本</font> | <font style="color:#000000;">`/api/language-management/labels`</font> | <font style="color:#000000;">`/views/admin/language/text.vue`</font> | <font style="color:#000000;">Key-Value 表格</font> |
| <font style="color:#000000;">文本模板</font> | <font style="color:#000000;">文本模板</font> | <font style="color:#000000;">`/api/text-management/templates`</font> | <font style="color:#000000;">`/views/admin/text-template/index.vue`</font> | <font style="color:#000000;">模板引擎</font> |
| <font style="color:#000000;">审计日志</font> | <font style="color:#000000;">审计日志</font> | <font style="color:#000000;">`/api/audit-logging/audit-logs`</font> | <font style="color:#000000;">`/views/admin/audit/index.vue`</font> | <font style="color:#000000;">审计日志</font> |
| <font style="color:#000000;">设置</font> | <font style="color:#000000;">设置</font> | <font style="color:#000000;">`/api/setting-management/settings/by-provider`</font> | <font style="color:#000000;">`/views/admin/setting/index.vue`</font> | <font style="color:#000000;">Tab 分组</font> |
| | | | | |
| <font style="color:#000000;">SaaS</font> | <font style="color:#000000;">租户</font> | <font style="color:#000000;">`/api/saas/tenants`</font> | <font style="color:#000000;">`/views/admin/saas/tenant/index.vue`</font> | <font style="color:#000000;">版本+版税</font> |
| | <font style="color:#000000;">版本</font> | <font style="color:#000000;">`/api/saas/editions`</font> | <font style="color:#000000;">`/views/admin/saas/edition/index.vue`</font> | <font style="color:#000000;">版本定价</font> |


#### 2. 目录约定（Monorepo）
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

#### 3. 通用脚手架（减少 60 % 代码）
 所有模块页面只写「列定义 + 表单 schema」即可。

```typescript
export function useAbpCrud<T = any>(api: {
  getList: (params) => Promise<PagedResult<T>>;
  create: (dto) => Promise<T>;
  update: (id, dto) => Promise<T>;
  delete: (id) => Promise<void>;
}) {
  const loading = ref(false);
  const dataSource = ref<T[]>([]);
  const pagination = reactive({ pageSize: 10, current: 1, total: 0 });
  const [register, { getFieldsValue, resetFields }] = useForm();

  async function fetchList() {
    loading.value = true;
    const req = { ...getFieldsValue(), skipCount: (pagination.current - 1) * pagination.pageSize, maxResultCount: pagination.pageSize };
    const res = await api.getList(req);
    dataSource.value = res.items;
    pagination.total = res.totalCount;
    loading.value = false;
  }

  async function handleAdd(dto: T) {
    await api.create(dto);
    createMessage.success(t('sys.success'));
    fetchList();
  }

  async function handleEdit(id: string, dto: T) {
    await api.update(id, dto);
    createMessage.success(t('sys.success'));
    fetchList();
  }

  async function handleDelete(id: string) {
    await api.delete(id);
    createMessage.success(t('sys.success'));
    fetchList();
  }

  onMounted(fetchList);

  return { loading, dataSource, pagination, fetchList, handleAdd, handleEdit, handleDelete, register, resetFields };
}

```

#### 4. 菜单权限汇总
##### （1）已按 ABP菜单层级、权限名、多语言 key 排版，直接复制即可用
```vue
// src/router/routes/modules/admin.ts
import type { RouteRecordRaw } from 'vue-router';
import { t } from '/@/hooks/web/useI18n';

const admin: RouteRecordRaw = {
  path: '/admin',
  name: 'Admin',
  component: () => import('/@/layouts/default/index.vue'),
  redirect: '/admin/identity/user',
  meta: {
    orderNo: 10,
    icon: 'ion:settings-outline',
    title: t('routes.admin.admin'),
    policy: 'AbpUi.Navigation.Administration', // 能看到一级菜单的最低权限
  },
  children: [
    /* ====== Identity ====== */
    {
      path: 'identity',
      name: 'Identity',
      redirect: '/admin/identity/user',
      meta: { title: t('routes.admin.identity'), icon: 'ion:people-outline' },
      children: [
        {
          path: 'user',
          name: 'IdentityUser',
          component: () => import('/@/views/admin/identity/user/index.vue'),
          meta: {
            title: t('routes.admin.users'),
            policy: 'AbpIdentity.Users',
            keepAlive: true,
          },
        },
        {
          path: 'role',
          name: 'IdentityRole',
          component: () => import('/@/views/admin/identity/role/index.vue'),
          meta: {
            title: t('routes.admin.roles'),
            policy: 'AbpIdentity.Roles',
            keepAlive: true,
          },
        },
        {
          path: 'organization-unit',
          name: 'OrganizationUnit',
          component: () => import('/@/views/admin/identity/ou/index.vue'),
          meta: {
            title: t('routes.admin.organizationUnits'),
            policy: 'AbpIdentity.OrganizationUnits',
            keepAlive: true,
          },
        },
        {
          path: 'claim-type',
          name: 'ClaimType',
          component: () => import('/@/views/admin/identity/claim/index.vue'),
          meta: {
            title: t('routes.admin.claimTypes'),
            policy: 'AbpIdentity.ClaimTypes',
            keepAlive: true,
          },
        },
        {
          path: 'security-log',
          name: 'SecurityLog',
          component: () => import('/@/views/admin/identity/security-log/index.vue'),
          meta: {
            title: t('routes.admin.securityLogs'),
            policy: 'AbpIdentity.SecurityLogs',
            keepAlive: true,
          },
        },
      ],
    },

    /* ====== OpenIddict ====== */
    {
      path: 'openiddict',
      name: 'OpenIddict',
      redirect: '/admin/openiddict/application',
      meta: { title: t('routes.admin.openIddict'), icon: 'ion:lock-closed-outline' },
      children: [
        {
          path: 'application',
          name: 'OpenIddictApplication',
          component: () => import('/@/views/admin/openiddict/application/index.vue'),
          meta: {
            title: t('routes.admin.applications'),
            policy: 'OpenIddictApplications.Manage',
            keepAlive: true,
          },
        },
        {
          path: 'scope',
          name: 'OpenIddictScope',
          component: () => import('/@/views/admin/openiddict/scope/index.vue'),
          meta: {
            title: t('routes.admin.scopes'),
            policy: 'OpenIddictScopes.Manage',
            keepAlive: true,
          },
        },
      ],
    },

    /* ====== Language Management ====== */
    {
      path: 'language',
      name: 'Language',
      redirect: '/admin/language/language',
      meta: { title: t('routes.admin.languages'), icon: 'ion:language-outline' },
      children: [
        {
          path: 'language',
          name: 'LanguageList',
          component: () => import('/@/views/admin/language/index.vue'),
          meta: {
            title: t('routes.admin.languages'),
            policy: 'LanguageManagement.Languages',
            keepAlive: true,
          },
        },
        {
          path: 'text',
          name: 'LanguageText',
          component: () => import('/@/views/admin/language/text.vue'),
          meta: {
            title: t('routes.admin.languageTexts'),
            policy: 'LanguageManagement.LanguageTexts',
            keepAlive: true,
          },
        },
      ],
    },

    /* ====== Text Template ====== */
    {
      path: 'text-template',
      name: 'TextTemplate',
      component: () => import('/@/views/admin/text-template/index.vue'),
      meta: {
        title: t('routes.admin.textTemplates'),
        icon: 'ion:document-text-outline',
        policy: 'TextTemplateManagement.Templates',
        keepAlive: true,
      },
    },

    /* ====== Audit Logs ====== */
    {
      path: 'audit',
      name: 'AuditLog',
      component: () => import('/@/views/admin/audit/index.vue'),
      meta: {
        title: t('routes.admin.auditLogs'),
        icon: 'ion:shield-checkmark-outline',
        policy: 'AuditLogging.AuditLogs',
        keepAlive: true,
      },
    },

    /* ====== Settings ====== */
    {
      path: 'setting',
      name: 'SettingManagement',
      component: () => import('/@/views/admin/setting/index.vue'),
      meta: {
        title: t('routes.admin.settings'),
        icon: 'ion:options-outline',
        policy: 'SettingManagement.ManageSettings',
        keepAlive: true,
      },
    },

    /* ====== SaaS ====== */
    {
      path: 'saas',
      name: 'Saas',
      redirect: '/admin/saas/tenant',
      meta: { title: t('routes.admin.saas'), icon: 'ion:business-outline' },
      children: [
        {
          path: 'tenant',
          name: 'Tenant',
          component: () => import('/@/views/admin/saas/tenant/index.vue'),
          meta: {
            title: t('routes.admin.tenants'),
            policy: 'Saas.Tenants',
            keepAlive: true,
          },
        },
        {
          path: 'edition',
          name: 'Edition',
          component: () => import('/@/views/admin/saas/edition/index.vue'),
          meta: {
            title: t('routes.admin.editions'),
            policy: 'Saas.Editions',
            keepAlive: true,
          },
        },
      ],
    },
  ],
};

export default admin;

```

##### （2）使用方式（已在  src/router/routes/index.ts  自动合并）
所有  policy  与 ABP 后端  PermissionDefinition  完全对齐；

若用户无对应权限，框架会自动在「菜单渲染」&「路由守卫」两级隐藏；

按钮级再补  v-auth="'xxx'"  即可。

```vue
import admin from './modules/admin';
export const constantRoutes: RouteRecordRaw[] = [
  // 登录、404 等
];
export const asyncRoutes = [admin];   // 动态挂载

```

#### 5.各模块实现要点（按菜单顺序）
##### （1）Identity > Users
| **<font style="color:#000000;">点</font>** | **<font style="color:#000000;">实现</font>** |
| --- | --- |
| <font style="color:#000000;">列</font> | <font style="color:#000000;">头像 / 用户名 / Email / 电话 / 组织 / 锁定 / 创建时间</font> |
| <font style="color:#000000;">表单</font> | <font style="color:#000000;">Tabs：基本｜角色｜组织｜声明（动态）</font> |
| <font style="color:#000000;">上传头像</font> | <font style="color:#000000;">`/api/identity/users/{id}/avatar` POST `multipart/form-data`</font> |
| <font style="color:#000000;">权限</font> | <font style="color:#000000;">`AbpIdentity.Users.Create/Update/Delete`</font> |
| <font style="color:#000000;">特殊列</font> | <font style="color:#000000;">`lockoutEnabled` 用 Switch，需调用 `/api/identity/users/{id}/lock`</font> |


关键代码片段

表单抽屉： useDrawer  嵌套  RoleTransfer  /  OuTree  组件。

```typescript
// api/identity.ts
export const getUserList = (params) => http.get<PagedResult<UserListItem>>('/api/identity/users', { params });
export const createUser = (dto: CreateOrUpdateUserDto) => http.post('/api/identity/users', dto);
export const updateUser = (id: string, dto: CreateOrUpdateUserDto) => http.put(`/api/identity/users/${id}`, dto);
export const deleteUser = (id: string) => http.delete(`/api/identity/users/${id}`);
export const getRolesByUser = (id: string) => http.get<string[]>(`/api/identity/users/${id}/roles`);
export const setRoles = (id: string, roleNames: string[]) => http.put(`/api/identity/users/${id}/roles`, { roleNames });
export const getOuByUser = (id: string) => http.get<string[]>(`/api/identity/users/${id}/organization-units`);
export const setOu = (id: string, ouIds: string[]) => http.put(`/api/identity/users/${id}/organization-units`, { organizationUnitIds: ouIds });
```

##### （2）Identity > Roles
| **<font style="color:#000000;">点</font>** | **<font style="color:#000000;">实现</font>** |
| --- | --- |
| <font style="color:#000000;">权限树</font> | <font style="color:#000000;">调用 `/api/identity/roles/{id}/permissions` 扁平 → 转 Tree</font> |
| <font style="color:#000000;">默认角色</font> | <font style="color:#000000;">`isDefault` 开关</font> |
| <font style="color:#000000;">公共角色</font> | <font style="color:#000000;">`isPublic` 开关</font> |


权限 Tree 组件：/src/components/PermissionTree/index.vue

```vue
<script setup lang="ts">
  const props = defineProps<{ providerKey: string; providerName: string }>();
    const { data } = await http.get(`/api/permission-management/permissions`, { params: { providerName: props.providerName, providerKey: props.providerKey } });
    const treeData = buildPermissionTree(data.groups); // 递归转 children
  </script>

```



##### （3）Organization Units
左树右表：点击节点加载成员

拖拽改父级（AntD Tree  draggable ）

接口： /api/identity/organization-units/{id}/move   { parentId } 

##### （4）Claim Types
动态声明：支持  Regex  /  Required  /  Static or User 

表单字段： name  /  regex  /  regexDescription  /  valueType ( Int / String / Boolean )

##### （5）Security Logs
仅列表 & 详情抽屉，无编辑

列：IP / 浏览器 / 时间 / 结果 / 用户名

支持按用户、IP、日期筛选

##### （6）OpenIddict > Applications
Scope 多选来源于  /api/openiddict/scopes  下拉。

| **<font style="color:#000000;">字段</font>** | **<font style="color:#000000;">控件</font>** |
| --- | --- |
| <font style="color:#000000;">ClientId</font> | <font style="color:#000000;">Input</font> |
| <font style="color:#000000;">ClientType</font> | <font style="color:#000000;">Select(`confidential`/`public`)</font> |
| <font style="color:#000000;">ClientSecrets</font> | <font style="color:#000000;">TagInput（仅 confidential）</font> |
| <font style="color:#000000;">RedirectUris</font> | <font style="color:#000000;">TextArea 每行一个</font> |
| <font style="color:#000000;">Permissions</font> | <font style="color:#000000;">CheckboxGroup（`openid` / `profile` / `email` / `roles` / `offline_access` / `Scp:{name}`）</font> |
| <font style="color:#000000;">PostLogoutRedirectUris</font> | <font style="color:#000000;">TextArea</font> |


##### （7）OpenIddict > Scopes
资源作用域：Name / DisplayName / Description / Resources（API 名称）

支持动态添加到 JWT  scp  claim。

##### （8）Language Management
表格：Culture / UiCulture / DisplayName / Flag 图标

启用/禁用：调用  /api/language-management/languages/{id}/toggle 

Texts 子页：左侧 Key 列表，右侧表格多语言行内编辑（类似 Excel）

批量保存：一次性  PUT /api/language-management/labels   [{ culture, key, value }] 

##### （9） Text Template
模板列表：Name / DisplayName / IsInline`

内容编辑器：Monaco Editor（带  {{model.xxx}}  高亮）

局部 Test 按钮：POST  /api/text-management/templates/{name}/render  弹出预览。

##### （10） Audit Logs
列表：URL / 方法 / 用户 / IP / 执行时间 / 状态码

详情抽屉：Parameters / Response / Exceptions / EntityChanges（Tab）

支持按日期 / 用户 / 状态码 / 实体类型筛选

EntityChanges 子表：实体字段旧值→新值 Diff 高亮

##### （11） Settings
分组 Tab： General  /  Email  /  Identity  /  Tenant  /  OpenIddict  ...

统一接口： GET /api/setting-management/settings/by-provider?providerName=Global&providerKey= 

表单动态生成：根据  SettingDefinition  的  Type  ( Bool / String / Int / Email / Json )

保存：按分组批量  PUT /api/setting-management/settings 

##### （12） SaaS > Tenant
列：Name / Edition / ActivationState / CreationTime

弹窗：连接字符串 / 启用「独立数据库」Switch

功能切换：下拉选择 Edition 后，自动带出功能树（ /api/saas/tenants/{id}/features ）

权限： Saas.Tenants.Create/Update/Delete/ManageFeatures/ManageConnectionStrings 

##### （13） SaaS > Edition
左右布局：左列表，右「功能 & 价格」卡片

功能树同 Tenant，但保存到  /api/saas/editions/{id}/features 

价格支持多币种（ Currency  /  Amount  /  Period ： Monthly / Annual 

### 九、常见坑速查表
| **<font style="color:#000000;">现象</font>** | **<font style="color:#000000;">原因</font>** | **<font style="color:#000000;">解决</font>** |
| --- | --- | --- |
| <font style="color:#000000;">列表无数据</font> | <font style="color:#000000;">只取 `res.data` 没取 `res.data.result`</font> | <font style="color:#000000;">统一走 `abp.ts` 拦截器</font> |
| <font style="color:#000000;">树表无 children</font> | <font style="color:#000000;">扁平权限</font> | <font style="color:#000000;">后端加 `children` 或前端递归</font> |
| <font style="color:#000000;">403 租户不匹配</font> | <font style="color:#000000;">缺 `__tenant` 头</font> | <font style="color:#000000;">请求拦截统一加</font> |
| <font style="color:#000000;">OIDC 死循环</font> | <font style="color:#000000;">刷新失败仍重试</font> | <font style="color:#000000;">401 刷新失败立即 `signoutRedirect`</font> |
| <font style="color:#000000;">编辑 400</font> | <font style="color:#000000;">ABP 大小写敏感</font> | <font style="color:#000000;">`name` → `Name`，用后端 DTO 生成 TS 类型</font> |


### 
