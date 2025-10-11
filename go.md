# Go语言学习笔记 - Java开发者视角

## 🚀 Go语言概述

### 核心特点
- **编译型语言**：编译为单个二进制文件，部署简单
- **垃圾回收**：自动内存管理（类似Java）
- **并发原生支持**：goroutine + channel
- **简洁语法**：没有类继承，接口隐式实现
- **强类型**：静态类型系统，类型安全

### 与Java对比
| 特性   | Java         | Go                  |
| ---- | ------------ | ------------------- |
| 继承   | 类继承          | 组合优于继承              |
| 接口   | 显式实现         | 隐式实现（duck typing）   |
| 并发   | 线程 + 线程池     | goroutine + channel |
| 异常处理 | try-catch    | 多返回值错误处理            |
| 包管理  | Maven/Gradle | go mod              |

## 📦 基础语法

### 包声明和导入
```go
package main

import (
    "fmt"
    "math/rand"
)

func main() {
    fmt.Println("Hello, Go!")
    fmt.Println(rand.Intn(100))
}
```

### 变量声明
```go
// 多种声明方式
var name string = "Go"
var age = 25           // 类型推断
count := 10            // 简短声明（函数内使用）
var (
    a int
    b string
)
```

### 常量
```go
const Pi = 3.14
const (
    StatusOK = 200
    StatusNotFound = 404
)

// iota 枚举
const (
    Monday = iota  // 0
    Tuesday        // 1
    Wednesday      // 2
)
```

## 🏗️ 控制结构

### 条件语句
```go
// if 语句（不需要括号）
if x > 10 {
    fmt.Println("x is greater than 10")
} else if x == 10 {
    fmt.Println("x is 10")
} else {
    fmt.Println("x is less than 10")
}

// 条件前可以执行简单语句
if err := doSomething(); err != nil {
    fmt.Println("Error:", err)
}
```

### 循环
```go
// 只有 for 循环
for i := 0; i < 10; i++ {
    fmt.Println(i)
}

// while 循环的替代
n := 0
for n < 5 {
    fmt.Println(n)
    n++
}

// 无限循环
for {
    // 需要 break 退出
    break
}

// 遍历数组、切片、map、字符串
arr := []string{"a", "b", "c"}
for index, value := range arr {
    fmt.Printf("Index: %d, Value: %s\n", index, value)
}
```

### Switch 语句
```go
switch day {
case "Monday":
    fmt.Println("Start of week")
case "Friday":
    fmt.Println("Almost weekend")
    fallthrough // 继续执行下一个case
default:
    fmt.Println("Midweek")
}

// 条件switch
switch {
case score >= 90:
    fmt.Println("A")
case score >= 80:
    fmt.Println("B")
default:
    fmt.Println("C")
}
```

## 📝 函数

### 基本函数
```go
// 函数声明
func add(a int, b int) int {
    return a + b
}

// 多返回值（常用于错误处理）
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("division by zero")
    }
    return a / b, nil
}

// 命名返回值
func calc(x, y int) (sum int, product int) {
    sum = x + y
    product = x * y
    return // 自动返回sum和product
}

// 可变参数
func sum(numbers ...int) int {
    total := 0
    for _, num := range numbers {
        total += num
    }
    return total
}
```

### 匿名函数和闭包
```go
// 匿名函数
func() {
    fmt.Println("Anonymous function")
}()

// 闭包
func counter() func() int {
    count := 0
    return func() int {
        count++
        return count
    }
}

c := counter()
fmt.Println(c()) // 1
fmt.Println(c()) // 2
```

## 🏛️ 数据类型

### 基本类型
- `int`, `int8`, `int16`, `int32`, `int64`
- `uint`, `uint8`, `uint16`, `uint32`, `uint64`, `uintptr`
- `float32`, `float64`
- `complex64`, `complex128`
- `bool`
- `string`
- `byte` (alias for uint8)
- `rune` (alias for int32, represents Unicode code point)

### 复合类型
```go
// 数组（固定长度）
var arr [3]int = [3]int{1, 2, 3}
arr2 := [...]int{1, 2, 3} // 编译器推断长度

// 切片（动态数组，更常用）
slice := []int{1, 2, 3}
slice = append(slice, 4) // 添加元素

// 映射（map）
ages := map[string]int{
    "Alice": 25,
    "Bob":   30,
}
ages["Charlie"] = 28

// 结构体
type Person struct {
    Name string
    Age  int
}

p := Person{"Alice", 25}
p.Age = 26
```

### 指针
```go
var x int = 10
var ptr *int = &x
fmt.Println(*ptr) // 10

// 与Java不同，Go有指针但不需要手动内存管理
func modifyValue(ptr *int) {
    *ptr = 20
}

modifyValue(&x)
fmt.Println(x) // 20
```

## 🎯 面向对象编程

### 方法
```go
type Rectangle struct {
    Width, Height float64
}

// 方法（接收者）
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

// 指针接收者（可以修改结构体）
func (r *Rectangle) Scale(factor float64) {
    r.Width *= factor
    r.Height *= factor
}

rect := Rectangle{10, 5}
fmt.Println(rect.Area()) // 50
rect.Scale(2)
fmt.Println(rect.Area()) // 200
```

### 接口
```go
type Shape interface {
    Area() float64
    Perimeter() float64
}

// 隐式实现：只要实现了接口所有方法，就自动实现接口
func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}

// 多态
func printArea(s Shape) {
    fmt.Println("Area:", s.Area())
}

printArea(rect)
```

## ⚡ 并发编程

### Goroutine
```go
func sayHello() {
    fmt.Println("Hello from goroutine")
}

// 启动goroutine（类似线程但更轻量）
go sayHello()

// 匿名函数
go func() {
    fmt.Println("Anonymous goroutine")
}()

time.Sleep(time.Second) // 等待goroutine完成
```

### Channel
```go
// 创建channel
ch := make(chan int)

// 发送和接收
go func() {
    ch <- 42 // 发送数据
}()

value := <-ch // 接收数据
fmt.Println(value) // 42

// 带缓冲的channel
bufferedCh := make(chan int, 2)
bufferedCh <- 1
bufferedCh <- 2
// bufferedCh <- 3 // 这里会阻塞，因为缓冲区已满

// 遍历channel
for v := range ch {
    fmt.Println(v)
}

// 关闭channel
close(ch)
```

### Select 语句
```go
select {
case msg1 := <-ch1:
    fmt.Println("Received", msg1)
case msg2 := <-ch2:
    fmt.Println("Received", msg2)
case ch3 <- data:
    fmt.Println("Sent data")
default:
    fmt.Println("No communication")
}
```

## 🛠️ 错误处理

### 多返回值错误模式
```go
func readFile(filename string) ([]byte, error) {
    data, err := os.ReadFile(filename)
    if err != nil {
        return nil, fmt.Errorf("read file %s: %w", filename, err)
    }
    return data, nil
}

// 使用
content, err := readFile("test.txt")
if err != nil {
    log.Fatal(err)
}
fmt.Println(string(content))
```

### 自定义错误类型
```go
type MyError struct {
    Code    int
    Message string
}

func (e *MyError) Error() string {
    return fmt.Sprintf("Error %d: %s", e.Code, e.Message)
}

func someFunction() error {
    return &MyError{Code: 404, Message: "Not found"}
}
```

## 📚 常用标准库

### net/http (Web开发)
```go
func helloHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, %s!", r.URL.Path[1:])
}

func main() {
    http.HandleFunc("/", helloHandler)
    fmt.Println("Server starting on :8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### encoding/json (JSON处理)
```go
type User struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
}

// Marshal
user := User{"Alice", 25}
data, _ := json.Marshal(user)
fmt.Println(string(data)) // {"name":"Alice","age":25}

// Unmarshal
var newUser User
json.Unmarshal(data, &newUser)
fmt.Println(newUser.Name) // Alice
```

### 其他重要包
- `os` - 操作系统交互
- `io/ioutil` - 文件操作
- `time` - 时间处理
- `sync` - 同步原语（Mutex, WaitGroup等）
- `testing` - 单元测试

## 🔧 工具链

### Go Modules (依赖管理)
```bash
# 初始化模块
go mod init github.com/yourname/project

# 添加依赖
go get github.com/gin-gonic/gin

# 整理依赖
go mod tidy

# 查看依赖图
go mod graph
```

### 测试
```go
// 测试文件：xxx_test.go
func TestAdd(t *testing.T) {
    result := add(2, 3)
    if result != 5 {
        t.Errorf("Expected 5, got %d", result)
    }
}

// 基准测试
func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        add(1, 2)
    }
}
```

## 💡 面试常见问题

### 1. Goroutine vs Thread
- **Goroutine**: 轻量级（2KB栈），由Go运行时管理，M:N调度
- **Thread**: 重量级（1MB栈），由操作系统调度

### 2. Channel vs Mutex
- **Channel**: 用于goroutine间通信，更高级的抽象
- **Mutex**: 用于共享内存的同步，更底层

### 3. Go的垃圾回收
- 并发标记清除算法
- 低延迟（STW时间短）
- 自动内存管理

### 4. 接口实现
- 隐式实现：不需要显式声明
- 只要实现了接口的所有方法，就自动实现该接口

### 5. 错误处理哲学
- 多返回值而不是异常
- 明确处理错误，不忽略
- 错误也是值

## 🚀 学习资源

### 官方资源
- [Go官方文档](https://golang.org/doc/)
- [Go Tour](https://tour.golang.org/)
- [Effective Go](https://golang.org/doc/effective_go.html)

### 实践项目
1. 命令行工具
2. RESTful API服务
3. 并发数据处理
4. 微服务组件

### 面试准备
- 理解goroutine和channel
- 掌握错误处理模式
- 熟悉常用标准库
- 了解性能优化技巧

## 🏆 Go语言在后端开发中的显著优势

### 1. 性能优势
- **编译型语言**：直接编译为机器码，执行速度接近C/C++
- **内存管理**：自动垃圾回收，避免手动内存管理错误
- **高效调度**：Goroutine调度器优化，百万级并发性能优异

### 2. 并发优势
- **轻量级Goroutine**：仅2KB栈空间，可轻松创建数百万个
- **CSP并发模型**：通过Channel通信，避免共享内存的竞态条件
- **内置并发原语**：select、channel、sync包等

### 3. 部署优势
- **单二进制文件**：编译后生成独立可执行文件，无依赖
- **跨平台编译**：支持Windows、Linux、macOS等多平台
- **容器友好**：镜像体积小，启动速度快，适合Docker部署

### 4. 开发效率
- **简洁语法**：关键字少，代码可读性高
- **丰富标准库**：网络、加密、编码等常用功能内置
- **快速编译**：编译时间短，开发迭代效率高

### 5. 云原生生态
- **微服务框架**：Gin、Echo、Fiber等高性能Web框架
- **服务网格**：Istio、Linkerd等服务网格技术支持
- **云原生工具**：Kubernetes、Docker、Prometheus等采用Go开发

## 🎯 后端开发场景示例：高并发订单处理系统

### 场景描述
电商平台秒杀活动，需要处理大量并发订单请求，要求：
- 支持10万+并发用户
- 订单处理延迟<100ms
- 数据一致性保证
- 水平扩展能力强

### Go实现方案

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "sync"
    "time"

    "github.com/gin-gonic/gin"
    "github.com/go-redis/redis/v8"
)

// 订单结构
type Order struct {
    ID        string    `json:"id"`
    UserID    string    `json:"user_id"`
    ProductID string    `json:"product_id"`
    Quantity  int       `json:"quantity"`
    Status    string    `json:"status"`
    CreatedAt time.Time `json:"created_at"`
}

// 订单处理器
type OrderProcessor struct {
    redisClient *redis.Client
    orderChan   chan *Order
    workerPool  int
    wg          sync.WaitGroup
}

// 创建订单处理器
func NewOrderProcessor(redisAddr string, workerPool int) *OrderProcessor {
    rdb := redis.NewClient(&redis.Options{
        Addr: redisAddr,
    })

    return &OrderProcessor{
        redisClient: rdb,
        orderChan:   make(chan *Order, 10000), // 缓冲channel
        workerPool:  workerPool,
    }
}

// 启动工作池
func (op *OrderProcessor) Start() {
    for i := 0; i < op.workerPool; i++ {
        op.wg.Add(1)
        go op.worker(i)
    }
}

// 工作goroutine
func (op *OrderProcessor) worker(id int) {
    defer op.wg.Done()
    
    for order := range op.orderChan {
        // 处理订单
        if err := op.processOrder(order); err != nil {
            log.Printf("Worker %d: Failed to process order %s: %v", id, order.ID, err)
            order.Status = "failed"
        } else {
            order.Status = "success"
            log.Printf("Worker %d: Successfully processed order %s", id, order.ID)
        }
    }
}

// 订单处理逻辑
func (op *OrderProcessor) processOrder(order *Order) error {
    ctx := context.Background()
    
    // 1. 检查库存（使用Redis分布式锁）
    lockKey := fmt.Sprintf("lock:product:%s", order.ProductID)
    lockValue := fmt.Sprintf("order:%s", order.ID)
    
    // 获取分布式锁
    lock := op.redisClient.SetNX(ctx, lockKey, lockValue, 5*time.Second)
    if !lock.Val() {
        return fmt.Errorf("failed to acquire lock for product %s", order.ProductID)
    }
    defer op.redisClient.Del(ctx, lockKey)
    
    // 2. 检查并扣减库存
    stockKey := fmt.Sprintf("stock:%s", order.ProductID)
    currentStock := op.redisClient.Get(ctx, stockKey)
    stock, _ := currentStock.Int()
    
    if stock < order.Quantity {
        return fmt.Errorf("insufficient stock: available %d, requested %d", stock, order.Quantity)
    }
    
    // 扣减库存
    op.redisClient.DecrBy(ctx, stockKey, int64(order.Quantity))
    
    // 3. 保存订单到Redis
    orderKey := fmt.Sprintf("order:%s", order.ID)
    orderData, _ := json.Marshal(order)
    op.redisClient.Set(ctx, orderKey, orderData, 24*time.Hour)
    
    // 4. 发送订单到消息队列（这里简化处理）
    queueKey := fmt.Sprintf("order_queue:%s", order.ProductID)
    op.redisClient.LPush(ctx, queueKey, orderData)
    
    return nil
}

// 提交订单（HTTP处理器）
func (op *OrderProcessor) SubmitOrder(c *gin.Context) {
    var order Order
    if err := c.ShouldBindJSON(&order); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }
    
    // 设置订单基本信息
    order.ID = fmt.Sprintf("order_%d", time.Now().UnixNano())
    order.CreatedAt = time.Now()
    order.Status = "pending"
    
    // 非阻塞发送订单到channel
    select {
    case op.orderChan <- &order:
        c.JSON(http.StatusAccepted, gin.H{
            "order_id": order.ID,
            "status":   "accepted",
            "message":  "Order is being processed",
        })
    default:
        c.JSON(http.StatusServiceUnavailable, gin.H{
            "error": "Server is busy, please try again later",
        })
    }
}

// 获取订单状态
func (op *OrderProcessor) GetOrderStatus(c *gin.Context) {
    orderID := c.Param("id")
    orderKey := fmt.Sprintf("order:%s", orderID)
    
    ctx := context.Background()
    orderData := op.redisClient.Get(ctx, orderKey)
    if orderData.Err() != nil {
        c.JSON(http.StatusNotFound, gin.H{"error": "Order not found"})
        return
    }
    
    var order Order
    json.Unmarshal([]byte(orderData.Val()), &order)
    c.JSON(http.StatusOK, order)
}

// 优雅关闭
func (op *OrderProcessor) Shutdown() {
    close(op.orderChan)
    op.wg.Wait()
    op.redisClient.Close()
}

func main() {
    // 创建订单处理器（100个工作goroutine）
    processor := NewOrderProcessor("localhost:6379", 100)
    processor.Start()
    defer processor.Shutdown()
    
    // 设置HTTP服务器
    router := gin.Default()
    
    // API路由
    router.POST("/api/orders", processor.SubmitOrder)
    router.GET("/api/orders/:id/status", processor.GetOrderStatus)
    
    // 健康检查
    router.GET("/health", func(c *gin.Context) {
        c.JSON(http.StatusOK, gin.H{"status": "healthy"})
    })
    
    // 启动服务器
    fmt.Println("Order processing server starting on :8080")
    if err := router.Run(":8080"); err != nil {
        log.Fatal("Failed to start server:", err)
    }
}
```

### 性能优势分析

1. **高并发处理能力**
   - 100个工作goroutine可轻松处理数万并发请求
   - Goroutine切换开销小，内存占用低
   - Channel实现高效的goroutine间通信

2. **低延迟响应**
   - 非阻塞订单提交，立即返回响应
   - 异步订单处理，用户体验好
   - 分布式锁保证数据一致性

3. **易于水平扩展**
   - 无状态服务设计，支持多实例部署
   - Redis作为共享存储，支持服务发现
   - 容器化部署简单（单二进制文件）

4. **开发效率高**
   - 标准库支持HTTP、JSON、并发等常用功能
   - 代码简洁，业务逻辑清晰
   - 编译速度快，开发迭代效率高

### 与传统Java方案对比

| 特性 | Go方案 | Java方案 |
|------|--------|----------|
| 内存占用 | 100MB（单进程） | 500MB+（JVM开销） |
| 启动时间 | <1秒 | 10-30秒 |
| 并发模型 | Goroutine（轻量） | 线程池（重量） |
| 部署方式 | 单二进制文件 | JAR+JVM依赖 |
| 容器支持 | 原生支持 | 需要JVM层 |

---
*最后更新: 2024年*  
*为Java开发者准备的Go语言快速入门指南*