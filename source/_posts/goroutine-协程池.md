---
title: goroutine 协程池
date: 2025-03-27 16:26:30
tags: golang、channel、goroutine、context
---

## Go 语言实现协程池

### 🎈核心思路

- 生产者消费者模型
- 通过 `channel` 作为任务投递渠道
- 异步执行闭包 `handler` 
- 通过 `context`  做任务执行实体的超时控制
- 协程池优雅退出机制，避免异步函数执行被暴力终止

### 📄Talk is cheap, Show me code 

#### 实现代码

```go
package pool

import (
	"context"
	"errors"
	"fmt"
	"sync"
	"sync/atomic"
	"time"

	"gitlab.com/bopop/log"
)

// Worker 任务工作接口
type Worker interface {
	// Task 任务
	Task() error
}

// TaskHandler 任务实体,避免造成 goroutine 泄露请勿作为一个长期（常驻）的工作任务
type TaskHandler func(ctx context.Context, params any) error

// Process 任务实体
type Process struct {
	// Ctx 任务上下文
	ctx context.Context
	// Cancel 任务上下文取消函数
	cancel context.CancelFunc
	// Params 任务处理数据
	Params any
	// TaskFunc 任务实体
	TaskFunc TaskHandler
	// Timeout 任务超时时间
	Timeout time.Duration
}

// DefaultTimeout 默认超时时间
const DefaultTimeout = 5 * time.Second

// WithTimeout 设置任务超时时间 (秒)
func WithTimeout(timeout time.Duration) func(*Process) {
	return func(w *Process) {
		w.Timeout = timeout * time.Second
	}
}

// WithParams 设置任务处理数据
func WithParams(params any) func(*Process) {
	return func(w *Process) {
		w.Params = params
	}
}

// Task 执行任务
func (p *Process) Task() error {
	return p.TaskFunc(p.ctx, p.Params)
}

// NewProcess 创建任务
func NewProcess(taskFunc TaskHandler, opts ...func(*Process)) *Process {
	if taskFunc == nil {
		return nil
	}

	p := &Process{
		TaskFunc: taskFunc,
		Timeout:  DefaultTimeout,
	}
	for _, opt := range opts {
		opt(p)
	}

	return p
}

// WorkerPool 工作池
type WorkerPool struct {
	work chan Worker
	wg   sync.WaitGroup
	sync.Once
	runNum uint32
}

var pools []*WorkerPool

// NewPool 创建工作池
func NewPool(cap int) *WorkerPool {
	p := &WorkerPool{
		work: make(chan Worker),
	}

	for i := 0; i < cap; i++ {
		p.wg.Add(1)
		go func() {
			defer func() {
				p.wg.Done()
				atomic.AddUint32(&p.runNum, ^uint32(0))
			}()
			for w := range p.work {
				process := w.(*Process) //nolint:errcheck
				process.ctx, process.cancel = context.WithTimeout(context.Background(), process.Timeout)
				execTask(process)
			}
		}()
		atomic.AddUint32(&p.runNum, 1)
	}
	add(p)
	return p
}

// execTask 执行任务
func execTask(w *Process) {
	defer w.cancel()
	done := make(chan struct{})
	var err error
	go func() {
		err = w.Task()
		close(done)
	}()

	select {
	case <-w.ctx.Done():
		log.Error(fmt.Sprintf("任务执行超时: %v", w.ctx.Err()))
		return
	case <-done:
		if err != nil {
			log.Error(fmt.Sprintf("任务执行失败: %v", err))
		}
		return
	}
}

// Delivery 投递任务
func (p *WorkerPool) Delivery(w Worker) error {
	if w == nil {
		return errors.New("delivery task is nil")
	}
	p.work <- w
	return nil
}

// Close 关闭工作池
func (p *WorkerPool) Close() {
	p.Once.Do(func() {
		close(p.work)
		p.wg.Wait()
	})
}

// Add 记录新增的工作池
func add(p *WorkerPool) {
	pools = append(pools, p)
}

// PoolsShutdown 关闭所有的工作池
func PoolsShutdown() {
	if len(pools) > 0 {
		for _, p := range pools {
			p.Close()
		}
	}
}


```

**注意：`gitlab.com/bopop/log` 为私有组件，若要测试本代码请注释或使用自用组件**



#### Testing

```go

package pool

import (
	"context"
	"fmt"
	"math/rand"
	"runtime"
	"sync"
	"testing"
	"time"
)

func random() int {
	// 以当前时间生成种子
	rand.Seed(time.Now().UnixNano())

	// 设定范围
	min := 1
	max := 10

	// 生成 min 到 max 范围内的随机数
	return rand.Intn(max-min+1) + min
}

// Task 定义要执行的任务
func Task(ctx context.Context, extra any) error {
	randNum := extra.(int) //nolint:errcheck
	time.Sleep(time.Duration(randNum) * time.Second)

	if randNum%2 == 0 {
		return fmt.Errorf("randNum = %d, worker failed1", randNum)
	}

	// 模拟 context deadline 终止
	if ctx.Err() != nil {
		return fmt.Errorf("randNum = %d, worker failed2", randNum)
	}

	fmt.Printf("randNum = %d, worker finished\n", randNum)
	return nil
}

// TestNewPool 测试 NewPool 函数的创建工作池行为是否正常
func TestNewPool(t *testing.T) {
	wg := sync.WaitGroup{}
	wg.Add(1)
	var p *WorkerPool
	go func() {
		defer wg.Done()
		p = NewPool(5)
	}()
	wg.Wait()
	defer p.Close()

	if p.runNum != 5 {
		t.Errorf("Expected pool size of %d, but got %d", 5, cap(p.work))
	}
}

// TestWorkerPool_Run 测试工作池能否正常运行任务
func TestWorkerPool_Run(t *testing.T) {

	p := NewPool(runtime.NumCPU())
	// 保证 pool 初始化
	time.Sleep(300 * time.Millisecond)
	defer p.Close()

	var wg sync.WaitGroup
	taskCount := 5
	wg.Add(taskCount)

	for i := 0; i < taskCount; i++ {
		go func(n int) {
			defer wg.Done()
			randNum := random()
			fmt.Println("randNum = ", randNum)
			process := NewProcess(Task, WithParams(randNum), WithTimeout(5))
			p.Delivery(process)
		}(i)
		time.Sleep(300 * time.Millisecond)
	}

	wg.Wait()
}

// TestWorkerPool_Close 测试 Close 函数能否正确关闭并完成所有任务
func TestWorkerPool_Close(t *testing.T) {
	p := NewPool(2)
	time.Sleep(300 * time.Millisecond)
	defer p.Close() // 退出时关闭

	var counter int
	var wg sync.WaitGroup
	wg.Add(1)

	go func() {
		defer wg.Done()
		for i := 0; i < 5; i++ {
			process := NewProcess(Task, WithParams(counter))
			p.Delivery(process)
		}
	}()

	wg.Wait()

	p.Close() // 第一次显示关闭，测试多次关闭

	finalCount := counter
	if finalCount != 0 {
		t.Errorf("expected counter to be 0, got %d", finalCount)
	}
}

// TestPoolsShutdown 测试 PoolsShutdown 函数能否关闭所有工作池
func TestPoolsShutdown(t *testing.T) {
	// 创建并添加几个工作池
	var wg sync.WaitGroup
	wg.Add(3)
	for i := 0; i < 3; i++ {
		go func() {
			defer wg.Done()
			NewPool(3)
		}()
	}
	wg.Wait()

	PoolsShutdown()

	// 验证工作池是否全部关闭
	for _, p := range pools {
		select {
		case _, ok := <-p.work:
			if ok {
				t.Errorf("work channel should be closed")
			}
		default:
			t.Errorf("work channel should be closed and empty")
		}
	}
}


```

*注释比较全了应该很容易看懂了哈😁*