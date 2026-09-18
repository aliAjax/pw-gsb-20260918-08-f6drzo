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
- `PATCH /sections/:id/check`
- `POST /tunes/:id/seal`
- `GET /issues?tuneId=&status=`
- `POST /issues`
- `PATCH /issues/:id/status`

## 终检封存

- `POST /tunes/:id/seal`：仅当该曲目全部区间已核对且没有未解决问题时封存成功；否则返回 409，曲目与区间状态不变。已封存的曲目重复封存为幂等成功。
- 封存后新增问题（`POST /issues`）或取消任一区间核对（`PATCH /sections/:id/check` 传 `checked:false`）均返回 409。
- 封存后将该曲目已有问题重新打开（`PATCH /issues/:id/status` 传非 `resolved` 状态）会自动解封，原有区间核对与问题历史全部保留。

## 闭环示例

```bash
curl http://127.0.0.1:3019/tunes/tune_demo/progress
curl -X POST http://127.0.0.1:3019/issues \
  -H 'Content-Type: application/json' \
  -d '{"tuneId":"tune_demo","sectionId":"section_demo_2","type":"错孔","beat":45,"lane":9,"description":"第45拍第9轨多打孔"}'
```
