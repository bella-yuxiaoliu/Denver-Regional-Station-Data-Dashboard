# Denver 片区站点数据看板 · Denver Regional Station Data Dashboard

GOFO Denver 片区的站点网络看板：在地图上查看 13 个站点的邮编覆盖、线路、DSP、价格和日均单量，另附一个 DSP 定价分析页。纯静态网页，部署在 GitHub Pages 上，打开链接即可使用，不需要服务器。

This is a static site (no build step, no server). It is based on the NorCal ops console template.

## 文件夹结构 / Folder structure

```
index.html                     主页：网络地图（全部代码 + 内嵌数据都在这一个文件里）
analytics/dsp-pricing.html     子页：DSP 定价分析（从主页侧边栏进入）
data/price_adjustments.csv     价格表：日常调价只改这个文件
README.md                      本说明
```

文件夹结构必须保持不变，页面之间靠相对路径互相引用。

## 数据说明 / Data

| 项目 | 内容 |
|---|---|
| 数据来源 | `Denver片区各站点信息_整理版.xlsx`（2026-09-28 整理版） |
| 覆盖范围 | 13 个站点 · 11 个 DSP · 77 条线路 · 316 个邮编 · 日均约 97,680 件（CO、UT、NM、ID、WY、MT） |
| 站点代码 | 取自线路名前缀，例如 `DEN01-003` 属于 DEN01 |
| 均价口径 | 线路、站点、DSP 的均价全部按日均单量加权；单量为 0 时用简单平均 |
| 单量 | 日均单量，按邮编四舍五入为整数 |
| 站点定位 | 用每个站点地址所在邮编的中心点定位，不是精确门牌地址，可能偏差几公里 |
| 邮编边界 | US Census ZCTA5 2020（cb_2020_us_zcta520_500k），已简化 |
| 无边界的邮编 | 87131、87158（ABQ01）、83303（TWF01）：人口普查局没有这几个邮编的形状（邮政信箱 / 校园邮编），单量和价格照常计入汇总，只是地图上不画区块；三者日均单量合计约 1 件 |

## 第一次部署 / First-time deploy (GitHub Pages)

1. 在 github.com 新建一个仓库，例如 `den-ops-console`。
   Create a new repository.
2. 进入仓库，点 **Add file → Upload files**，把本文件夹里的**全部内容**拖进去（保留 `analytics/` 和 `data/` 两个子文件夹），然后点 **Commit changes**。
   Upload everything in this folder, keeping the subfolders, and commit.
3. 进入 **Settings → Pages**：Source 选 **Deploy from a branch**，Branch 选 **main**，文件夹选 **/ (root)**，点 **Save**。
   Settings → Pages → Deploy from a branch → main → / (root) → Save.
4. 约 1 分钟后链接生效：`https://<你的用户名或组织名>.github.io/den-ops-console/`
   Your link: `https://<username>.github.io/den-ops-console/`

> ⚠️ **数据可见性**：GitHub 免费账号只能从 **Public（公开）** 仓库部署 Pages。公开意味着任何拿到链接的人都能看到看板，而且仓库源码（每个邮编的价格、单量、DSP、负责人姓名）也对所有人公开。私有仓库需要 GitHub Pro / Team / Enterprise。部署前请先确认公司是否允许这些数据公开。
>
> 建议把仓库放在**公司的 GitHub 组织账号**下，而不是个人账号，避免人员变动后链接失效。

## 日常调价 / Updating prices

只需要改 `data/price_adjustments.csv`，不用动其他文件。

1. 在 GitHub 仓库里打开 `data/price_adjustments.csv`，点铅笔图标编辑（或在 Excel 里改完另存为 CSV，再用 Upload files 覆盖上传）。
2. 格式是两列：`zip,price`，例如 `80206,1.95`。只改需要调价的邮编即可。
3. 点 **Commit changes**，约 1 分钟后生效。所有打开链接的人都会看到新价格。

页面加载时会读取这个文件，自动重算邮编颜色、线路 / 站点 / DSP 均价和排名。主页侧边栏的 **Price sheet** 状态会显示读取了多少个邮编、有多少个和基础价格不同；定价分析页顶部的小标签也会显示。

注意：CSV 只能改**已有邮编**的价格。CSV 里出现看板上没有的邮编会被忽略。

## 更新单量、新增站点或邮编 / Updating volume, stations or ZIPs

单量、线路、DSP、站点和邮编归属都内嵌在 `index.html` 和 `analytics/dsp-pricing.html` 里，改 CSV 不会影响这些。需要更新时：

1. 按 `Denver片区各站点信息_整理版.xlsx` 的格式更新 Excel（Address 表 + 每个站点一个 sheet，列：站点、转运中心、线路、价格、邮编、日均单量）。
2. 把新 Excel、本文件夹，以及邮编边界文件 `cb_2020_us_zcta520_500k.zip`（下载地址：https://www2.census.gov/geo/tiger/GENZ2020/shp/cb_2020_us_zcta520_500k.zip ）一起交给 Claude，请它重新生成。
3. 用生成的新文件覆盖上传到仓库并提交。

新增站点时，Address 表里一定要填完整、准确的地址（地图用地址里的邮编定位）。

## 常见问题 / FAQ

**本地双击打开 `index.html`，价格表显示没生效？**
正常。浏览器出于安全限制，不允许本地文件（`file://`）读取 CSV，页面会退回使用内嵌的基础价格。部署到 GitHub Pages 后就能正常读取。

**链接打不开 / 显示 404？**
- 刚开启 Pages 需要等 1–2 分钟。
- 检查 Settings → Pages 里是否选了 main 分支和 / (root)。
- 检查 `index.html` 是否在仓库**根目录**，而不是在某个子文件夹里（上传整个文件夹时容易多套一层）。

**改了 CSV 但页面没变化？**
浏览器可能有缓存，强制刷新（Mac：Cmd+Shift+R；Windows：Ctrl+F5）。也可以在仓库的 **Actions** 标签页确认部署是否完成。

**站点位置不太准？**
站点是按地址邮编的中心定位的。如需精确位置，可以提供各站点经纬度，请 Claude 更新。
