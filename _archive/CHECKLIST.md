# 上架前自查清单

适用配置：**无自建账号 + Game Center 排行榜（仅 iOS）+ AdMob 广告 + 本地分数**

---

## 0. 先替换占位符

三个 HTML 里所有 `[...]` 都要改掉，带 `[ADJUST: ...]` 的按实际情况选一条、删另一条。

| 占位符 | 改成 |
|---|---|
| `[APP_NAME]` | 商店里的完整应用名（三处必须一致） |
| `[YEAR]` | 2026 |
| `[DATE]` | 如 `September 10, 2026` |
| 隐私政策 §5 | 没接 Firebase 就删掉那两行 |
| 隐私政策 §4.5 | 儿童条款二选一，删另一条 |
| 隐私政策 §7 | 填实际保留天数 |
| 隐私政策 §9 / §10 | "Delete My Data" 按钮没有就删；转收费后删掉 "currently free and ad-supported" |
| Support 页 FAQ | 付费版、离线支持两条 |

编辑器全站搜 `[` 即可找全。

---

## 1. Game Center 与排行榜（iOS 专有）

### 结论：不算"你的账号"，但算标识符收集

- ✅ **不适用 Apple 5.1.1(v)**（应用内删号）—— 那是针对你自己提供的账号系统。Game Center 走 Apple ID，你不接触凭据
- ⚠️ **必须披露**——Game Center player ID 是 Apple 官方点名的 **User ID** 类别标识符

### Apple App Privacy 问卷（iOS 版）

| 项目 | 怎么填 |
|---|---|
| 数据类型 | **Identifiers → User ID** |
| 用途 | **App Functionality**（排行榜、成就） |
| 是否与用户身份关联 | **是** |
| Data Used to Track You | **不勾** —— 仅用于自家排行榜，不发给第三方，不是追踪 |
| User Content → Gameplay Content | 通常不勾（分数是系统记录，非用户创作内容） |

### Google Play 数据安全表单（Android 版）

- Android **不上传积分** → **不要勾 User ID**
- 只填 AdMob 那几项（见 §3）

> 两边问卷内容不同是**正确的**——表单按各平台实际行为填写。政策文件是同一个 URL，所以政策里已用 "iOS only" 明确区分。

### 代码侧

- [ ] Game Center 登录保持**可选**，不要做成强制登录才能玩（否则可能被要求提供访客模式）
- [ ] 不要把 player ID 传给 AdMob、Firebase 或任何第三方 —— 一旦传了，就必须勾"追踪"并先过 ATT
- [ ] 本地分数用 UserDefaults / SharedPreferences 即可，卸载即清
- [ ] 确认没有把分数同步到自建服务器（当前是"没有"，若以后加了必须回来改政策 §2.1）

### 审核备注建议写

> The app uses Apple Game Center (iOS only) for leaderboards. Player IDs and scores are stored by Apple and cached locally; we operate no server and share this data with no third party. Game Center sign-in is optional. The Android build does not upload scores.

---

## 2. AdMob 专属必做项（最容易漏，漏了必拒）

### iOS
- [ ] **Info.plist → `NSUserTrackingUsageDescription`**：必须有，写明为什么请求追踪。缺这个 App 一启动就崩或被拒
- [ ] **Info.plist → `SKAdNetworkIdentifiers`**：加入 Google 的 SKAdNetwork ID 列表
- [ ] **ATT 弹窗**：在广告加载**前**调用 `ATTrackingManager.requestTrackingAuthorization`
- [ ] **App Privacy 问卷**：
  - Identifiers → **Device ID**
  - 用途勾选 **Third-Party Advertising**
  - "Data Used to Track You" → 勾 **Device ID**（用了 IDFA 就必须勾）

### Android
- [ ] **AndroidManifest**：AdMob App ID 写在 `<meta-data android:name="com.google.android.gms.ads.APPLICATION_ID">`
- [ ] **AD_ID 权限**（targetSdk 33+）：AdMob 17.x 起自动声明，不要手动删
- [ ] **Google Play 数据安全表单**：

| 数据类型 | 是否收集 | 用途 | 是否共享第三方 |
|---|---|---|---|
| 设备或其他 ID | 是 | 广告或营销、分析 | **是** |
| 应用活动（应用交互） | 是 | 广告或营销、应用功能 | **是** |
| 应用信息和性能（崩溃日志） | 是 | 应用功能 | 是 |
| 大致位置 | 是 | 广告或营销 | 是 |
| 用户 ID（Game Center） | **否** | — | 不填（Android 不上传积分） |

- [ ] 勾选"数据在传输过程中加密"
- [ ] **数据删除**：勾"提供请求删除数据的途径"，URL 填 `https://dean41.github.io/delete-account.html`
- [ ] **账号删除**：无账号 → Play Console 声明"应用不支持创建账号"，可免填

### 两端共同
- [ ] **UMP 同意弹窗**（EEA/UK/瑞士强制）：在 AdMob 初始化**之前**请求同意
- [ ] **隐私设置入口**：Settings → Privacy，可重新打开同意表单
- [ ] **AdMob 后台**：CCPA 设置打开；隐私政策 URL 填 `https://dean41.github.io/privacy.html`
- [ ] **儿童向判断**：若面向儿童，设 `tagForChildDirectedTreatment` 并禁用个性化广告，政策选 Option B

---

## 3. Apple App Store

- [ ] App Information → **Privacy Policy URL**：`https://dean41.github.io/privacy.html`
- [ ] **Support URL**：`https://dean41.github.io/`
- [ ] App Privacy 问卷与政策 §1 摘要表逐行对齐（见 §1、§2）
- [ ] 无自建账号 → **不适用 5.1.1(v)**

**常见拒审点**
| 条款 | 触发原因 |
|---|---|
| 5.1.1 / 5.1.2 | 政策与实际数据收集不符 |
| 5.1.4 | 用了 IDFA 但没有 ATT 弹窗或缺 `NSUserTrackingUsageDescription` |
| 3.1.1 | 引导用户绕开 IAP 付费 |
| 2.3.x | 政策页面截图与实际不符 |

---

## 4. Google Play

- [ ] 应用内容 → **隐私政策 URL**：`https://dean41.github.io/privacy.html`
- [ ] **数据安全表单**：按 §2 表格填
- [ ] **广告声明**：标注"包含广告"
- [ ] **目标受众**：如实填写是否面向儿童
- [ ] 应用内可访问隐私政策

**常见拒审点**
| 问题 | 触发原因 |
|---|---|
| 数据安全表单与政策不一致 | 最高频 |
| 广告 SDK 未披露 | 与 SDK 清单比对后拒 |
| 缺 GDPR 同意机制 | EEA 地区必查 |
| 政策链接 404 / 非 HTTPS | 直接拒 |

---

## 5. 一致性铁律

```
隐私政策 §1 摘要表
      ↕
Apple App Privacy 问卷 / Google Play 数据安全表单
      ↕
App 实际行为（代码里的 SDK 和权限）
```

任一处对不上 → 拒审。

注意：本 App 的 iOS 与 Android 实际行为不同（iOS 有 Game Center，Android 没有），所以两边问卷本就不该一致 —— 政策里已用 "iOS only" 区分，提交时不要为了"对齐"而误填。

---

## 6. 部署

```bash
cd D:\github\dean41.github.com
git add .
git commit -m "Update support & privacy pages for store compliance"
git push origin master
```

推送后 1-3 分钟生效，验证三个地址都返回 200：

```
https://dean41.github.io/
https://dean41.github.io/privacy.html
https://dean41.github.io/delete-account.html
```

**手机上也要能打开**，审核员用手机看。

---

## 7. 后续改动的连带更新

| 改动 | 需要同步改什么 |
|---|---|
| 转付费 / 加内购 | 政策 §10 删 "currently free and ad-supported"；两边问卷加 Purchases |
| 加"去广告"内购 | 政策 §10 补一句"购买后可关闭广告" |
| 分数改存自建服务器 | 政策 §2.1 必须改写（现在写的是"无自建服务器"）；需提供删除途径 |
| Android 也上传积分 | 改用 Play Games Services；Play 数据安全表单要补 User ID |
| 加账号系统 | Apple 5.1.1(v) 生效 → 补应用内删号；`delete-account.html` 改成删账号页 |

---

## 8. 其他

- 邮箱 `YeehawStudio@gmail.com` 必须真实可收，Apple 有时真的发测试邮件
- 政策里的"30 天内删除""1-2 个工作日回复"是承诺值，做得到再留
- 政策更新后同步改 `Last updated` 日期
- 想面向国内用户可另存 `privacy-zh.html`，英文页顶部加切换链接
