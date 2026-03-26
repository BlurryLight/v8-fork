# V8 Profiler 能力速记（Embedder 视角）

最后更新：2026-03-25

## 1. `v8-profiler.h` 里有什么能力

核心在 `include/v8-profiler.h`，对 embedder 常用能力分 5 类。

### 1) CPU Profiling（看时间花在哪）
- 入口：`v8::CpuProfiler`
- 常用接口：
  - `New()` / `Dispose()`
  - `SetSamplingInterval(int us)`
  - `Start(...)` / `Stop(...)`
  - `StartProfiling(...)` / `StopProfiling(...)`
  - `UseDetailedSourcePositionsForProfiling(...)`
  - `CollectSample(...)`
- 结果对象：
  - `CpuProfile`
  - `CpuProfileNode`
- 能拿到调用树、函数命中数、脚本/行列号、deopt 信息，并可 `Serialize(...)` 导出。

### 2) Heap Snapshot（看堆里有什么、引用关系）
- 入口：`v8::HeapProfiler`
- 常用接口：
  - `TakeHeapSnapshot(...)`
  - `GetSnapshotCount()` / `GetHeapSnapshot(...)`
  - `DeleteAllHeapSnapshots()`
  - `GetObjectId(...)` / `FindObjectById(...)` / `ClearObjectIds()`
- 结果对象：
  - `HeapSnapshot`
  - `HeapGraphNode`
  - `HeapGraphEdge`
- 用于分析对象图和保留路径，适合泄漏定位。

### 3) Heap Objects Tracking（连续时间序列统计）
- 入口：`v8::HeapProfiler`
- 常用接口：
  - `StartTrackingHeapObjects(bool track_allocations = false)`
  - `GetHeapStats(OutputStream*, int64_t* timestamp_us = nullptr)`
  - `StopTrackingHeapObjects()`
- 用于做随时间变化的对象数量/大小统计。

### 4) Sampling Heap Profiler（分配采样）
- 入口：`v8::HeapProfiler`
- 常用接口：
  - `StartSamplingHeapProfiler(sample_interval, stack_depth, flags)`
  - `GetAllocationProfile()`
  - `StopSamplingHeapProfiler()`
- 结果对象：`AllocationProfile`
- 结论：
  - 可以按调用栈/函数聚合“分配对象 count/size”。
  - 这是采样统计，不是逐个对象精确计数。

### 5) Code Event（代码创建/搬移事件）
- 类型：
  - `CodeEvent`
  - `CodeEventHandler`
- 用途：
  - 监听 code creation / relocation 事件。
  - 更像“JIT 产物事件流”，不是编译生命周期 start/stop 本身。

---

## 2. Sampling Heap Profiler 在 Chrome 里的对应工具

在 Chrome DevTools 的 **Memory** 面板里有对应封装：

- **Allocation sampling**
  - 最接近 V8 sampling heap profiler，低开销采样分配，按函数/调用栈看分配统计。
- **Allocation instrumentation on timeline (Allocations on timeline)**
  - 看一段时间内分配/存活变化。
- **Heap snapshot**
  - 看对象图和引用关系，不是采样分配统计。

官方文档：
- https://developer.chrome.com/docs/devtools/memory/
- https://developer.chrome.com/docs/devtools/memory-problems/allocation-profiler/

程序化方式（不手动点 DevTools）：
- 使用 Chrome DevTools Protocol 的 `HeapProfiler` domain：
  - `startSampling`
  - `getSamplingProfile`
  - `stopSampling`

---

## 3. 快速结论（回答“某函数分配了多少对象”）

- 能做：用 `StartSamplingHeapProfiler + GetAllocationProfile` 可得到“函数级分配统计表（采样）”。
- 不能直接保证精确：若要严格精确计数，需要改 V8 内部分配路径做额外插桩。
