# 配置额外的服务账户权限

KubeVirt 的 virt-api 组件现在支持通过命令行参数配置额外的服务账户，这些服务账户将被允许更新 VMI 状态。

## 背景

默认情况下，只有以下 KubeVirt 系统服务账户可以更新 VMI 状态：
- `kubevirt-apiserver`
- `kubevirt-controller` 
- `kubevirt-handler`

## 新增功能

通过 `--additional-service-accounts` 参数，您可以添加其他服务账户到允许列表中。

## 使用方法

### 1. 命令行参数

```bash
# 启动 virt-api 时添加额外的服务账户
virt-api --additional-service-accounts=myapp:myuser --additional-service-accounts=otherns:otheruser

# 或者使用完整格式
virt-api --additional-service-accounts=system:serviceaccount:myapp:myuser
```

### 2. 参数格式

支持两种格式：
- **简短格式**: `namespace:serviceaccount` (自动添加 `system:serviceaccount:` 前缀)
- **完整格式**: `system:serviceaccount:namespace:serviceaccount`

### 3. 多个服务账户

可以多次使用该参数来添加多个服务账户：
```bash
virt-api \
  --additional-service-accounts=app1:user1 \
  --additional-service-accounts=app2:user2 \
  --additional-service-accounts=system:serviceaccount:app3:user3
```

## 示例场景

### 场景 1: 自定义应用需要更新 VMI 状态
```bash
virt-api --additional-service-accounts=myapp:vmi-manager
```

### 场景 2: 多个命名空间的应用
```bash
virt-api \
  --additional-service-accounts=monitoring:metrics-collector \
  --additional-service-accounts=logging:log-aggregator
```

### 场景 3: 使用完整格式
```bash
virt-api --additional-service-accounts=system:serviceaccount:custom:operator
```

## 验证配置

启动后，virt-api 会在日志中显示添加的额外服务账户：

```
I0123 10:00:00.000000       1 api.go:207] Added additional service account: system:serviceaccount:myapp:myuser
I0123 10:00:00.000001       1 api.go:207] Added additional service account: system:serviceaccount:otherns:otheruser
```

## 注意事项

1. **安全性**: 只有真正需要更新 VMI 状态的服务账户才应该被添加
2. **权限范围**: 这些服务账户只能更新 VMI 状态，不能修改其他字段
3. **重启要求**: 修改此参数后需要重启 virt-api 组件
4. **持久性**: 此配置不会持久化，重启后需要重新指定

## 故障排除

如果遇到权限问题，请检查：
1. 服务账户名称是否正确
2. 命名空间是否存在
3. 服务账户是否有相应的 RBAC 权限
4. virt-api 日志中是否显示了添加的服务账户

