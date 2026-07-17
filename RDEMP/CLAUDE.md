# BMC-RDM 开发工作流

## 代码提交

```bash
# 1. 提交前先拉最新
git pull --no-edit

# 2. 提交到 Gerrit
git push origin HEAD:refs/for/master
```

## 编译

```bash
cd /home/bmc/sd1/CODE/Kits-RDEMP
go build -o BMC-RDM .
pkill -9 -f "BMC-RDM" 2>/dev/null; sleep 1  # 清理旧进程, 释放8077端口
```

## 本地验证

每次代码修改后执行:

```bash
# 1. 停止旧进程, 启动新服务
pkill -f "./BMC-RDM" 2>/dev/null
setsid ./BMC-RDM </dev/null >/dev/null 2>&1 &
disown
sleep 4
ss -tlnp | grep 8077 || echo "启动失败, 查看日志"

# 2. 登录获取 Token
TOKEN=$(curl -s -X POST http://127.0.0.1:8077/bmc/rdm/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"eric_zhou","password":"123456"}' \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['accessToken'])")

# 3. 调用本次变更涉及的接口, 验证 200 返回
curl -s -X POST http://127.0.0.1:8077/bmc/rdm/<新增/修改的路由> \
   -H "Authorization: Bearer $TOKEN" \
   -H "Content-Type: application/json" \
   -d '{}'
```

根据本次变更的接口替换第 3 步中的 URL 和请求体.

## 数据库

| 项目 | 值 |
|------|-----|
| Host | 10.17.48.160 |
| Port | 5432 |
| 账号 | postgres |
| 密码 | 123456 |
| 库名 | testdatabase |

## Artifactory

| 项目 | 值 |
|------|-----|
| Host | 10.32.129.210:8081 |
| 账号 | autotest |
| 密码 | At@241106 |

## 验证信息

| 项目 | 值 |
|------|-----|
| 端口 | 8077 |
| 用户名 | eric_zhou |
| 密码 | 123456 |
| 登录接口 | POST /bmc/rdm/auth/login |
| 鉴权方式 | Bearer Token (Header: Authorization) |
| 登录字段 | `username` + `password`|

## 调试环境配置 (⚠️ 临时修改, 提交前必须还原)

本地调试依赖远程数据库和 Artifactory 时, `configs/config.yaml` 需临时修改以下配置, **提交代码时必须还原**:

```bash
# 还原命令
git checkout configs/config.yaml
```

## 光圈消息测试 (⚠️ 临时修改, 测试完成后必须还原)

涉及 Guangquan 消息推送的测试时, 需临时将 `updatemachine` 设为 `false`:

```bash
sed -i 's/updatemachine: true/updatemachine: false/' configs/config.yaml
# 测试完成后还原
sed -i 's/updatemachine: false/updatemachine: true/' configs/config.yaml
```

> 光圈推送仅在 `updatemachine: false` 时真实发送, `true` 时仅打日志.

## 生产环境

| 项目 | 值 |
|------|-----|
| 前端 | `http://10.32.129.210:3200/` |
| API | `http://10.32.129.210:3200/api/bmc/rdm/` |
| 登录接口 | `POST /api/bmc/rdm/auth/login` |
| 用户名 | eric_zhou |
| 密码 | 123456 |

> 生产环境数据完整, 可用于查看页面结构、调用真实 API 获取数据辅助开发.