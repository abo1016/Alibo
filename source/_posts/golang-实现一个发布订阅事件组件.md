---
title: golang 实现一个发布订阅事件组件
date: 2025-1-6 17:26:48
tags: goalng、发布订阅、event
---

## golang 实现一个发布订阅时间组件

### 🎈老规矩，实现思路

- 实现一个广播器，主要有订阅通道 `channel` 和 发布订阅事件 `publish()` 广播事件发送 `send()`组成
- 使用 `sync`包管理事件映射关系`map`
- 使用互斥锁 `sync.Mutex` 检查广播器状态和订阅者的优雅关闭退出
- 广播消息通过广播器 `channel` 分发到各个订阅者（`channel`） 

### 具体实现代码

#### 事件枚举

```go
package event

const (
	// ConfigChangeEvent 配置变更事件
	ConfigChangeEvent = "config-change"
)

// Events 事件
var Events = []string{
	ConfigChangeEvent,
}

```



#### 主要逻辑

```go
package event

import (
	"errors"
	"fmt"
	"sync"
	"time"

	"gitlab.com/bopop/tool"
	"gitlab.com/bopop/log"
)

// Broadcast 广播器接口
type Broadcast interface {
	publish()
	Subscribe(chan<- any)
	Send(any)
	Close()
}

// Broadcaster 广播器
type Broadcaster struct {
	subscribers []chan<- any
	broadcast   chan any
	mux         sync.Mutex
	eventName   string
	closeChan   chan struct{}
	closed      bool
}

// BroadcasterMap 广播器映射
var BroadcasterMap sync.Map

// NewBroadcaster 创建一个新的 Broadcaster
func NewBroadcaster(event string) *Broadcaster {
	if !tool.InSlice(event, Events) {
		log.ErrorP(fmt.Sprintf("invalid event: %s", event))
		return nil
	}

	if _, exist := BroadcasterMap.Load(event); exist {
		log.ErrorP(fmt.Sprintf("event %s already exists", event))
		return nil
	}

	b := &Broadcaster{
		subscribers: make([]chan<- any, 0),
		broadcast:   make(chan any),
		eventName:   event,
		closeChan:   make(chan struct{}),
		closed:      false,
	}
	BroadcasterMap.Store(event, b)
	b.publish() // 自动开始发布
	return b
}

// Close 停止广播并关闭所有订阅者通道
func (b *Broadcaster) Close() {
	if nilObjReturn(b) != nil {
		return
	}
	b.mux.Lock()
	defer b.mux.Unlock()
	if !b.closed {
		b.closed = true
		close(b.closeChan) // 向goroutine发出关闭信号
		for _, sub := range b.subscribers {
			close(sub) // 主动关闭所有订阅者通道
		}
		BroadcasterMap.Delete(b.eventName)                     // 从map中移除
		log.InfoP(fmt.Sprintf("closed %s event", b.eventName)) // nolint:gosimple
	}
}

// CloseAll 关闭所有事件广播和订阅者
func CloseAll() {
	BroadcasterMap.Range(func(key, value interface{}) bool {
		value.(*Broadcaster).Close()
		return true
	})

	log.InfoP("closed all events")
}

// GetBroadcaster 获取一个 Broadcaster
func GetBroadcaster(event string) *Broadcaster {
	if v, ok := BroadcasterMap.Load(event); ok {
		return v.(*Broadcaster)
	}
	return nil
}

// publish 发布订阅事件
func (b *Broadcaster) publish() {
	go func() {
		for {
			select {
			case msg, ok := <-b.broadcast:
				if !ok {
					return // 如果 broadcast 被关闭，结束 goroutine
				}
				// 发送消息到订阅者
				b.mux.Lock()
				for _, sub := range b.subscribers {
					sub <- msg
				}
				b.mux.Unlock()
			case <-b.closeChan:
				return // 收到关闭信号，结束 goroutine
			}
		}
	}()
	// 给一个 100ms 的延时，保证发布事件 goroutine 已经启动
	time.Sleep(100 * time.Millisecond)
	log.InfoP(fmt.Sprintf("publish event: %v", b.eventName)) // nolint:gosimple
}

// Subscribe 注册一个订阅者的通道
func (b *Broadcaster) Subscribe(sub chan<- any) {
	if err := nilObjReturn(b); err != nil {
		return
	}
	b.mux.Lock()
	defer b.mux.Unlock()
	if b.closed {
		errStr := fmt.Sprintf("attempt to subscribe to closed broadcaster: %v", b.eventName)
		log.InfoP(errStr)
		sub <- errors.New(errStr)
		close(sub)
		return
	}
	b.subscribers = append(b.subscribers, sub)
}

// Send 广播一个消息给所有订阅者
func (b *Broadcaster) Send(msg any) {
	if err := nilObjReturn(b); err != nil {
		return
	}
	b.mux.Lock()
	subCount := len(b.subscribers)
	closed := b.closed
	b.mux.Unlock()

	if closed {
		log.InfoP(fmt.Sprintf("broadcast to closed event: %v", b.eventName)) // nolint:gosimple
		return
	}

	if subCount == 0 {
		log.InfoP(fmt.Sprintf("no subscribers for event: %v", b.eventName)) // nolint:gosimple
		return
	}

	select {
	case b.broadcast <- msg:
		// 消息发送成功
	default:
		// 通道已满，记录警告
		log.InfoP(fmt.Sprintf("broadcast channel is full for event %v", b.eventName)) // nolint:gosimple
	}
}

// nilObjReturn 检查 Broadcaster 是否为 nil
func nilObjReturn(b *Broadcaster) (err error) {
	if b == nil {
		errStr := "called on nil broadcaster"
		log.InfoP(errStr)
		return errors.New(errStr)
	}
	return nil
}

```

#### 自测

```go
package event_test

import (
	"testing"
	"time"

	"gitlab.com/bopop/event"
)

func TestBroadcaster(t *testing.T) {
	// 创建Broadcaster
	event.Events = append(event.Events, "testEvent")
	b := event.NewBroadcaster("testEvent")

	// 创建订阅者通道并订阅
	sub := make(chan any)
	b.Subscribe(sub)

	// 发送消息
	msg := "hello world"
	b.Send(msg)

	// 从订阅者通道中获取消息
	select {
	case receivedMsg := <-sub:
		if receivedMsg != msg {
			t.Errorf("expected message '%v', got '%v'", msg, receivedMsg)
		}
	case <-time.After(1 * time.Second):
		t.Fatal("did not receive message in time")
	}

	// 测试关闭
	b.Close()

	// 尝试监听关闭的通道，确保通道已关闭
	_, ok := <-sub
	if ok {
		t.Errorf("expected closed channel, but it's still open")
	}

	// 再次订阅并确认错误返回
	sub2 := make(chan any, 1)
	b.Subscribe(sub2)
	receivedError, ok := <-sub2
	if !ok {
		t.Errorf("expected error on subscribing to closed broadcaster, but channel is open")
	}
	if receivedError == nil {
		t.Errorf("expected error on subscribing to closed broadcaster, but got nil")
	}
}

// 测试关闭所有 Broadcaster
func TestCloseAll(t *testing.T) {
	// 创建两个Broadcaster
	event.Events = append(event.Events, "testEvent1", "testEvent2")
	b1 := event.NewBroadcaster("testEvent1")
	b2 := event.NewBroadcaster("testEvent2")

	// 创建订阅者并订阅
	sub1 := make(chan any, 1)
	sub2 := make(chan any, 1)

	b1.Subscribe(sub1)
	b2.Subscribe(sub2)

	// 使用CloseAll关闭所有Broadcaster
	event.CloseAll()

	// 确认所有订阅者通道已关闭
	_, ok := <-sub1
	if ok {
		t.Errorf("expected closed channel for Broadcaster 1, but it's still open")
	}

	_, ok = <-sub2
	if ok {
		t.Errorf("expected closed channel for Broadcaster 2, but it's still open")
	}
}

func TestBroadcastToMultipleSubscribers(t *testing.T) {
	// 创建一个新的Broadcaster
	event.Events = append(event.Events, "multiSubEvent")
	b := event.NewBroadcaster("multiSubEvent")
	// 创建多个订阅者并订阅
	subscriberCount := 3
	subscribers := make([]chan any, subscriberCount)
	for i := 0; i < subscriberCount; i++ {
		subscribers[i] = make(chan any, 1) // Buffer to prevent blocking
		b.Subscribe(subscribers[i])
	}

	// 发送消息
	expectedMsg := "broadcast message"
	b.Send(expectedMsg)

	// 确认所有订阅者都收到了消息
	for i, sub := range subscribers {
		select {
		case msg := <-sub:
			if msg != expectedMsg {
				t.Errorf("subscriber %d received wrong message: got %v, want %v", i, msg, expectedMsg)
			}
		case <-time.After(1 * time.Second):
			t.Fatalf("subscriber %d did not receive message in time", i)
		}
	}

	// 清理：关闭Broadcaster和所有订阅者通道
	b.Close()
	for _, sub := range subscribers {
		_, ok := <-sub
		if ok {
			t.Errorf("expected closed channel for subscriber, but it's still open")
		}
	}
}


```

**同样的，咱们的注释这块还是比较给力的，应该能看懂的**😜

