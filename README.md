# 手摇风琴纸带打孔API

纯后端零依赖Node服务，使用 `data/db.json` 持久化曲目、纸带区间和试奏问题。

## 启动

```bash
PORT=3019 node server.js
```

## 主要接口

- `GET /health`
- `GET /tunes`
- `POST /tunes`
- `GET /tunes/:id/progress`
- `GET /tunes/:id/sections`
- `POST /tunes/:id/sections`
- `GET /tunes/:id/unchecked-sections`
- `POST /tunes/:id/seal` 终检封存
- `PATCH /sections/:id/check`
- `GET /issues?tuneId=&status=`
- `POST /issues`
- `PATCH /issues/:id/status`

## 终检封存规则

- `POST /tunes/:id/seal`：仅当曲目**至少有一个区间、全部区间已核对、且没有未解决问题**（`status !== "resolved"`）时才能封存；不满足任一条件返回 `409`，曲目与区间状态均不变（不写盘），响应中会给出 `uncheckedSectionIds` / `openIssueIds`。
- 封存成功后曲目写入 `sealed: true` 与 `sealedAt`，`progress` 中也会返回这两个字段。
- 封存期间以下操作返回 `409` 且状态不变：
  - `POST /issues` 新增问题；
  - `PATCH /sections/:id/check` 把已核对区间改为未核对（保持 `checked: true` 仍允许）；
  - `POST /tunes/:id/sections` 新增区间；
  - 再次 `POST /tunes/:id/seal`。
- 在封存曲目中对**已有**问题执行 `PATCH /issues/:id/status` 且状态非 `resolved`（即重新打开）时，**自动解封**（`sealed=false`、`sealedAt=null`），响应带 `"unsealed": true`；原有区间核对结果与全部问题历史原样保留。

## 闭环示例

```bash
curl http://127.0.0.1:3019/tunes/tune_demo/progress
curl -X POST http://127.0.0.1:3019/issues \
  -H 'Content-Type: application/json' \
  -d '{"tuneId":"tune_demo","sectionId":"section_demo_2","type":"错孔","beat":45,"lane":9,"description":"第45拍第9轨多打孔"}'
```
