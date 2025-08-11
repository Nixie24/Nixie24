# 📘 廿肆のAPI开发规范

> 设计清晰、一致且安全的接口，确保长期可维护性与可扩展性。

---

## 一、协议：HTTPS 是必需条件

* 所有接口必须通过 **HTTPS** 提供服务。
* 禁止明文 HTTP 通信。

> 未加密传输会导致数据泄露和安全风险，必须使用加密协议。

---

## 二、域名结构：统一与规范

* 所有 API 应统一托管在主域名下，路径以 `/api/` 开头。

```url
https://foo.bar/api/
```

* 禁止使用混乱路径或子域名嵌套，例如：

```url
https://api.foo.bar/v1/data/backend/...
```

> `/api/` 是统一入口，应保持简洁明确。

---

## 三、版本控制：响应头声明

* **不在路径中使用版本号**（如 `/v1/`）。
* 版本信息通过响应头返回：

```http
X-API-Version: 2025.07.1  # <年>.<批次>.<修订号>
```

> 版本管理在后台完成，避免影响客户端路径结构。

---

## 四、路径命名：RESTful 规范

* 路径仅包含资源名，**使用复数形式**，例如：

```
/users
/orders/123/items
```

* 嵌套路由用于表示资源从属关系：

```http
GET /zoos/123/animals  # 查询动物园的所有动物
```

---

## 五、HTTP 方法：语义明确

| 方法     | 含义     | 幂等性 | 示例               |
| ------ | ------ | --- | ---------------- |
| GET    | 获取资源   | ✅   | GET /users       |
| POST   | 创建新资源  | ❌   | POST /orders     |
| PUT    | 替换整个资源 | ✅   | PUT /users/123   |
| PATCH  | 部分更新资源 | ❌   | PATCH /users/123 |
| DELETE | 删除资源   | ✅   | DELETE /cats/456 |

> 根据操作选择正确方法，确保语义与行为一致。

---

## 六、查询参数：保持简洁

* 遵循 KISS 原则（Keep It Simple, Stupid）。

```http
GET /users?filter[name]=Alice&sort=-created_at&page[size]=20&page[number]=1
```

**推荐结构：**

* `filter`：过滤字段，如 `filter[status]=active`
* `sort`：排序字段，前加 `-` 表示降序
* `page[size]` 和 `page[number]`：分页控制

> 避免参数名重复前缀，如 `user_name` → `name`。

---

## 七、状态码规范：准确表达结果

| 状态码 | 含义     | 场景          |
| --- | ------ | ----------- |
| 200 | 成功     | 普通 GET 请求   |
| 201 | 创建成功   | POST 新建资源   |
| 204 | 无内容成功  | DELETE 操作   |
| 400 | 请求参数错误 | 缺失或格式不正确    |
| 401 | 未授权    | Token 缺失或错误 |
| 403 | 禁止访问   | Token 无权限   |
| 404 | 未找到资源  | 查询不存在的 ID   |
| 429 | 请求过多   | 触发限流策略      |
| 500 | 服务器错误  | 未捕获异常       |

> 按实际情况返回合适的状态码，避免所有响应都使用 200。

---

## 八、错误响应格式：结构化输出

```json
{
  "error": {
    "code": "MISSING_HEADER",
    "message": "缺少必需的请求头",
    "details": {
      "header_field": "X-Auth-Token"
    }
  }
}
```

**字段说明：**

* `code`：错误编号（机器可读）
* `message`：错误提示（用户可读）
* `details`：可选，提供问题定位信息

> 错误信息应明确到具体字段与问题，便于快速排查。

---

## 九、返回结构规范

| 操作                   | 返回结构         | 示例                             |
| -------------------- | ------------ | ------------------------------ |
| `GET /collection`    | 列表 + 分页 meta | `{ data: [...], meta: {...} }` |
| `GET /collection/ID` | 单个对象         | `{ data: {...} }`              |
| `POST`               | 创建对象 + 状态码   | HTTP 201 + Location 头          |
| `PUT / PATCH`        | 更新后的完整对象     | `{ data: {...} }`              |
| `DELETE`             | 无响应体         | HTTP 204                       |

---

## 🔐 安全与认证

### 1. OAuth 2.0 授权

* 推荐使用 OAuth 2.0 Bearer Token。
* 请求需带 Authorization 头：

```http
Authorization: Bearer <token>
```

### 2. 请求格式

* 所有接口应采用 JSON 格式：

```http
Content-Type: application/json
Accept: application/json
```

### 3. 安全建议

* 校验所有外部输入，防止注入攻击。
* 限速（Rate Limiting），返回 429 状态码。
* 精确设置 CORS，避免使用 `*`。

---

## 📘 附录

### A. 错误码对照表

| 错误码                   | 含义          | 建议处理         |
| --------------------- | ----------- | ------------ |
| `INVALID_INPUT`       | 参数校验失败      | 检查字段格式与必填项   |
| `UNAUTHORIZED`        | Token 错误或缺失 | 提示用户重新登录或授权  |
| `FORBIDDEN`           | 权限不足        | 提示无访问权限      |
| `NOT_FOUND`           | 未找到资源       | 提示资源不存在或已被删除 |
| `RATE_LIMIT_EXCEEDED` | 请求频率过高      | 告知用户稍后重试     |

### B. 返回格式示例

```json
{
  "data": {},
  "meta": {
    "page": 1,
    "page_size": 20,
    "total": 123
  },
  "error": null
}
```

---

## 🎯 总结

* 路径使用复数名词，操作通过 HTTP 方法表达。
* 错误提示应明确、可定位。
* 状态码应准确匹配实际结果。
* 规范是保障质量和效率的工具，而非限制。

---

> 本规范由 Nix24 主笔，AI 辅助整理与优化 ✍️
