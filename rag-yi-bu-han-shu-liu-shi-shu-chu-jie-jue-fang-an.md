# RAG异步函数流式输出解决方案

## 解决方案说明

使用**线程池 + 事件循环隔离**的方法解决RAG流水线中异步函数调用影响流式输出的问题。

## 核心思路

1. **保持主流程同步**: 将原来的异步`run_test_generator`函数改为同步版本
2. **隔离异步操作**: 将`query_search`异步调用封装在独立的线程池中执行
3. **独立事件循环**: 在线程池中创建新的事件循环来运行异步函数
4. **不影响流式输出**: 主流程保持同步，确保UI能正常接收流式数据

## 修改内容

### 1. pipeline.py 修改

* 添加了`asyncio`和`ThreadPoolExecutor`导入
* 创建了`run_test_generator_sync`同步版本函数
* 创建了`run_test_generator_ben_sync`带基准测试的同步版本
* 修改Pipeline初始化，使用同步版本的回调函数

### 2. chatqna.py 修改

* 移除了`run_pipeline`调用前的`await`关键字
* 保持API接口为async（用于其他异步操作）

## 关键技术要点

### 线程池执行异步函数

```python
def run_async_query_search():
    # 创建新的事件循环
    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)
    try:
        return loop.run_until_complete(
            query_search(query, search_config_path, search_dir)
        )
    finally:
        loop.close()

# 使用线程池执行异步查询
with ThreadPoolExecutor(max_workers=1) as executor:
    future = executor.submit(run_async_query_search)
    top1_issue, sub_questionss_result = future.result()
```

### 优势

1. ✅ **保持流式输出**: UI端能正常接收流式数据
2. ✅ **兼容性好**: 不破坏现有架构
3. ✅ **性能稳定**: 异步操作在独立线程中执行
4. ✅ **错误隔离**: 异步错误不会影响主流程

### 工作流程

1. 用户发送查询请求
2. 进入同步的pipeline处理流程
3. 如果需要query\_search，在独立线程池中执行
4. 继续正常的RAG流程（检索、后处理、生成）
5. 返回流式响应给UI

这个方案确保了异步函数的正常执行，同时保持了流式输出的流畅性。
