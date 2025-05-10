# 代码库分析：整合SSE服务

## 现有代码库结构

### 主服务器组件
1. **src/main.rs**：
   - 应用程序入口点
   - 初始化环境（wei_windows::init, wei_env::bin_init）
   - 创建单例实例确保只有一个服务器运行
   - 调用wei_server::start()启动HTTP服务器

2. **src/lib.rs**：
   - 定义start()函数，用于启动HTTP服务器
   - 查找可用端口
   - 将选定的端口写入server.dat文件
   - 创建路由表（routes::routes()）
   - 绑定地址并启动服务器

3. **src/routes/mod.rs**：
   - 定义所有HTTP路由
   - 导入和注册各种模块的路由处理函数
   - 包含api_proxy功能，用于转发请求
   - 设置CORS和其他中间件

### SSE服务组件（测试目录）
1. **test/main.rs**：
   - 定义并启动SSE服务器
   - 使用mcp_core库创建Server实例
   - 注册各种工具
   - 配置ServerSseTransport在端口3001上运行
   - 初始化RIG agent与SSE服务器交互

2. **test/tools.rs**：
   - 定义各种工具函数（add_tool, sub_tool等）
   - 使用mcp_core_macros的#[tool]宏标注工具函数
   - 实现工具的功能逻辑

## 依赖项分析
1. **主项目（Cargo.toml）**：
   - 需要添加：mcp-core, mcp-core-macros, rig-core等依赖

2. **测试项目（test/Cargo.toml）**：
   - 关键依赖：anyhow, mcp-core, mcp-core-macros, rig-core, tokio等

## 集成策略

### 1. 依赖项更新
- 将test/Cargo.toml中的关键依赖添加到主项目的Cargo.toml中

### 2. 模块结构
- 创建src/sse/mod.rs模块来包含SSE服务功能
- 创建src/tools.rs模块来包含工具定义和实现

### 3. 路由集成
- 在src/routes/mod.rs中添加新的'/sse'路由
- 将路由连接到SSE服务处理函数

### 4. 服务启动
- 修改src/main.rs以同时启动HTTP和SSE服务
- 使用tokio的多线程功能确保两个服务可以并行运行

### 5. 工具注册
- 实现工具注册系统，将tools.rs中的工具注册到SSE服务

## 关键挑战
1. 确保两个服务能够并行运行而不冲突
2. 正确处理依赖关系，避免版本冲突
3. 维护现有HTTP服务功能的同时添加SSE功能
4. 确保路由系统正确地将请求转发到适当的处理函数

## 集成步骤
1. 更新Cargo.toml添加必要依赖
2. 创建工具模块并导入工具定义
3. 创建SSE服务模块
4. 更新路由系统添加SSE路由
5. 修改main.rs启动两个服务
6. 实现工具注册系统
7. 添加错误处理和日志记录
8. 进行集成测试和优化 