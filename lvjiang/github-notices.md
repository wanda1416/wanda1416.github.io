# 律匠 GitHub Pages 公告建立说明

本文说明如何为律匠客户端建立远程公告文件。公告与软件 Release 相互独立，适合在修复版本尚未完成时立即通知用户。

客户端固定读取：

```text
https://wanda1416.github.io/lvjiang/notices.json
```

该地址对应 GitHub 仓库 `wanda1416/lvjiang` 的 Pages 站点。

## 一、首次建立 `gh-pages` 分支

建议使用独立的孤立分支，只存放公告文件，避免把项目源码作为网站公开目录。以下操作在一个临时克隆目录中进行，不会切换或修改正在开发的项目目录。

Windows PowerShell：

```powershell
cd $env:USERPROFILE
git clone --no-checkout git@github.com:wanda1416/lvjiang.git lvjiang-pages
cd lvjiang-pages
git switch --orphan gh-pages
```

新建 `notices.json`，初始内容：

```json
{
  "schema_version": 1,
  "notice_version": 1,
  "updated_at": "2026-08-24T10:00:00Z",
  "notices": []
}
```

提交并推送：

```powershell
git add notices.json
git commit -m "chore: initialize announcement page"
git push -u origin gh-pages
```

## 二、在 GitHub 启用 Pages

1. 打开 `https://github.com/wanda1416/lvjiang/settings/pages`。
2. 在 **Build and deployment** 中将 Source 选择为 **Deploy from a branch**。
3. Branch 选择 `gh-pages`。
4. Folder 选择 `/(root)`。
5. 点击 **Save**。
6. 等待 GitHub 显示站点已发布。
7. 浏览器访问：

```text
https://wanda1416.github.io/lvjiang/notices.json
```

浏览器应直接显示 JSON。首次启用通常需要等待片刻；部署完成前出现 404 属于正常现象。

## 三、发布紧急公告

编辑 `lvjiang-pages/notices.json`：

```json
{
  "schema_version": 1,
  "notice_version": 2,
  "updated_at": "2026-08-24T12:30:00Z",
  "notices": [
    {
      "id": "critical-20260824-input-trace",
      "level": "critical",
      "title": "高精度录制功能风险提示",
      "published_at": "2026-08-24T12:30:00Z",
      "min_app_version": "0.5.1",
      "max_app_version_exclusive": "0.5.2",
      "body": "## 请立即停止使用高精度录制\n\n0.5.1 的高精度录制存在严重问题。在修复版本发布前，请不要继续使用该功能。",
      "url": "https://github.com/wanda1416/lvjiang/issues/123",
      "active": true
    }
  ]
}
```

然后提交：

```powershell
git pull --ff-only
git add notices.json
git commit -m "notice: warn about input trace issue"
git push
```

部署完成后，访问公告地址确认内容已经更新。

## 四、字段规则

| 字段 | 规则 |
|---|---|
| `schema_version` | 当前固定为整数 `1`，不要自行修改 |
| `notice_version` | 非负整数；每次需要客户端重新处理公告时必须递增 |
| `updated_at` | 推荐使用 UTC ISO 8601，例如 `2026-08-24T12:30:00Z` |
| `id` | 单条公告的唯一稳定标识，同一文件内不能重复 |
| `level` | `info`、`warning` 或 `critical` |
| `title` | 非空标题，最长200字符 |
| `body` | 非空 Markdown 正文，最长32000字符 |
| `published_at` | 公告发布时间，可留空 |
| `min_app_version` | 最低受影响版本，包含该版本；可留空 |
| `max_app_version_exclusive` | 受影响版本上界，不包含该版本；可留空 |
| `url` | 可留空；填写时必须是 HTTPS 地址 |
| `active` | `true` 表示展示，`false` 表示撤回 |

客户端最多接受20条公告，整个文件不能超过256 KB。

## 五、`notice_version` 操作原则

客户端使用 `notice_version` 防止每次启动重复弹窗。

- 修改正文且需要再次提醒：递增版本。
- 新增公告：递增版本。
- 撤回公告：将 `active` 改为 `false`，并递增版本。
- 只修正不需要重新提醒的文字：可以不递增，但客户端缓存可能不会立即更新，不推荐这样做。
- 不要降低或重复使用已经发布的版本号。

推荐每次修改文件都递增 `notice_version`，保持操作简单、可审计。

## 六、版本范围示例

只通知 `0.5.1` 系列，不通知 `0.5.2`：

```json
"min_app_version": "0.5.1",
"max_app_version_exclusive": "0.5.2"
```

通知 `0.5.1` 及以后所有版本：

```json
"min_app_version": "0.5.1",
"max_app_version_exclusive": ""
```

通知所有版本：

```json
"min_app_version": "",
"max_app_version_exclusive": ""
```

## 七、撤回公告

不要直接删除仍可能需要留档的公告。将其改为：

```json
"active": false
```

同时递增顶层 `notice_version`、更新 `updated_at` 并推送。客户端下一次同步后，“帮助 → 公告”中也不会再展示该公告。

## 八、发布前检查

1. `notices.json` 是合法 JSON，不能包含注释或末尾多余逗号。
2. `notice_version` 已递增。
3. 时间和受影响版本范围准确。
4. `body` 中的换行使用 `\n`。
5. `url` 使用 HTTPS。
6. 推送后直接访问线上地址确认，不要只检查本地文件。
7. 用一个尚未记录该公告版本的客户端验证自动弹窗。

## 九、故障排查

### 地址返回 404

- 检查 Pages 是否选择 `gh-pages` 和 `/(root)`。
- 检查文件是否位于分支根目录，名字必须是 `notices.json`。
- 查看仓库 Actions/Pages 部署状态。

### 客户端不弹窗

- 确认线上 `notice_version` 已递增。
- 确认 `active` 是布尔值 `true`，不是字符串。
- 检查当前客户端版本是否落在公告范围内。
- 从客户端“帮助 → 公告”点击“重新获取”，查看明确错误。
- 客户端已经处理过同一 `notice_version` 时不会再次自动弹窗。

### 需要再次强提醒同一问题

修改公告正文并递增顶层 `notice_version`。不要复用旧版本号。

## 十、安全要求

- 公告文件中不要放 Token、密码或私人信息。
- 公告只提供文本与 HTTPS 链接，不能远程执行客户端命令。
- `gh-pages` 分支的写权限等同于向所有客户端发布公告，应保护 GitHub 账号并启用双因素认证。