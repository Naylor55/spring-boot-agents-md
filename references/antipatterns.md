# 典型反例对照

给 Agent 看抽象规则，不如给它一个 ❌/✅ 对照。下面 7 类场景是 Spring Boot 项目里最常见的「能跑但不对」，可以直接贴进 code review 评论，或作为追加约束塞进项目的 `AGENTS.md`。

示例基于 JDK 17 / Spring Boot 3 / Lombok，Java 11 项目只需把语法特性换掉，判定标准一致。

---

## 1. 有 Lombok 还在手写样板代码，字段注入代替构造器注入

❌

```java
public class UserVO {
    private String name;
    private Integer age;

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public Integer getAge() { return age; }
    public void setAge(Integer age) { this.age = age; }
}

@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserMapper userMapper;
}
```

✅

```java
@Data
public class UserVO {
    private String name;
    private Integer age;
}

@Service
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {

    private final UserMapper userMapper;
}
```

判定：`pom.xml` 里有 `lombok` 依赖，就不接受手写 getter/setter/log 字段；新增代码不接受 `@Autowired` 字段注入。

---

## 2. 业务逻辑写进 Controller

❌

```java
@RestController
@RequiredArgsConstructor
public class OrderController {

    private final OrderMapper orderMapper;
    private final UserMapper userMapper;

    @GetMapping("/orders/{id}")
    public OrderVO detail(@PathVariable Long id) {
        Order order = orderMapper.selectById(id);
        if (order == null) {
            throw new RuntimeException("订单不存在");
        }
        User user = userMapper.selectById(order.getUserId());
        OrderVO vo = new OrderVO();
        vo.setOrderNo(order.getOrderNo());
        vo.setUserName(user.getName());
        vo.setPayable(order.getAmount().subtract(order.getDiscount()));
        return vo;
    }
}
```

✅

```java
@RestController
@RequiredArgsConstructor
public class OrderController {

    private final OrderService orderService;

    @GetMapping("/orders/{id}")
    public Result<OrderDetailVO> detail(@Valid @PathVariable Long id) {
        return Result.ok(orderService.detail(id));
    }
}
```

判定：Controller 里出现 Mapper / Repository、出现跨表组装、出现金额计算，一律打回。Controller 只做参数接收、校验、调用 Service、包统一响应。

---

## 3. 一条查询链路过五层抽象

❌

```text
OrderListController
  → OrderListHandler
    → OrderListManager
      → OrderListService
        → OrderListServiceImpl
          → OrderListConverter
            → OrderListAssembler
```

✅

```text
OrderController
  → OrderService#page
```

判定：新引入的 Handler / Manager / Factory / Assembler 必须能回答「谁和谁的实现需要在这层被隔离」。答不上来，就是转发代码，删掉。

---

## 4. 无视项目已有能力，重新造轮子

❌

```java
// 项目已有 common/api/Result
public static <T> Map<String, Object> success(T data) {
    Map<String, Object> map = new HashMap<>();
    map.put("code", 0);
    map.put("data", data);
    return map;
}

// 项目已有 XxxUtils.isBlank
public static boolean empty(String s) {
    return s == null || s.trim().length() == 0;
}
```

✅

```java
return Result.ok(data);
```

判定：新增任何工具类、返回体、分页对象、日期格式化方法之前，先在项目里搜一遍同类实现（`git grep`、IDE 的 Find Usages 或 Agent 的语义搜索）。找到就复用；确实需要新写，说明为什么现有实现不适用。

---

## 5. 空目录、无用依赖、调试输出一起提交

❌

```text
新增 mq/、oss/、job/ 三个空目录（项目根本没用到）
pom.xml 同时引入 hutool + guava + commons-lang3 三套工具库
log.info 之外还有 System.out.println("userId = " + userId)
```

✅

```java
log.info("order created, orderNo={}", orderNo);
```

```java
// 只添加本次需求实际用到的依赖，并在 PR 说明里写清用途
```

判定：提交 diff 里出现空目录、（按需）分类下的新目录、新依赖、`System.out`、被注释掉的旧代码，逐条说明来源。

---

## 6. 修一个 bug，顺手重构十二个文件

❌

```text
修复分页参数越界 → 顺便改了 12 个文件的命名
                  → 抽了 3 个接口
                  → 重排了 import 顺序
```

✅

```text
修复分页参数越界 → 只改分页参数校验 + 对应测试
                  → PR 描述里列出「发现但未修改」的问题
```

判定：改动文件清单超出任务边界的部分，要么移出 PR，要么在描述里单独说明并经确认。发现的问题写进总结，不夹带。

---

## 7. 用通用异常代替业务异常，对外泄露内部细节

❌

```java
throw new RuntimeException("select count(*) from t_order failed: " + e.getMessage());
```

```java
@ExceptionHandler(Exception.class)
public Map<String, Object> handle(Exception e) {
    return Map.of("msg", e.getMessage(), "trace", e.getStackTrace());
}
```

✅

```java
throw new BizException(ErrorCode.ORDER_NOT_FOUND);
```

```java
@ExceptionHandler(BizException.class)
public Result<Void> handle(BizException e) {
    log.warn("biz error, code={}, msg={}", e.getCode(), e.getMessage());
    return Result.fail(e.getCode(), e.getMessage());
}
```

判定：业务失败走项目统一异常 + 错误码；对外响应里出现 SQL、堆栈、类名、密钥，一律视为缺陷。

---

## 附：这些反例怎么用

1. **贴进 PR**：不要只说「不符合规范」，直接贴对应的 ❌/✅ 片段。
2. **塞进项目 AGENTS.md**：如果项目里某一类问题反复出现，把对应段落追加到项目自己的 `AGENTS.md` 末尾，比反复口头说明有效。
3. **交给工具**：反例里能自动判定的部分（import 顺序、`System.out`、空目录、`select *`），优先用 Checkstyle / PMD / ArchUnit / CI 规则拦截，不要指望 Agent 每次都记得。