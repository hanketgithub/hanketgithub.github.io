---
title: "Embedded Question"
date: 2026-04-15
tags: [""]
draft: false
---


# Interrupt

## 為何無法在 interrupt handler 用 sleep, mutex, I/O 等會造成 context switch 的 system call?

因為 sleep, mutex 會造成 schedule() 被呼叫

以下的 code 是不管用的, 雖然 schedule() 可造成 save context -> switch to next task -> restore context, 但此招對於 interrupt handler 無效. 理由是 Linux 裡的 context switch 實作是針對 process, 而 interrupt handler 並非 process, 因此並無法產生 task 來輪排做剩下的工作.

In order to resume bottom half of ISR, a context must be placed in task queue. However, in Linux, interrupt does not have context. Therefore, once ISR is switched out, it can not be switched in and resumed.


```cpp
static irqreturn_t interrupt_handler()
{
  printk(KERN_INFO "This is top half");

  sleep() -> schedule(); // just for fun, try to produce context switch

  printk(KERN_INFO "This is bottom half");

  return IRQ_HANDLED;
}
```


因此, 真的要做以上的動作(在 interrupt handler 裡面做一些事 -> 讓其他 task 跑一下 -> 再回來 interrupt handler 剩下想做的), 應該使用 schedule_work() 來代替 scheudle(). 這樣就會將 interrupt handler 剩下的事情正確 create task, 並排進 work_queue.

```cpp
INIT_WORK(&my_work, bottom_half_work);  // associate my_work & bottom_half_work


static void bottom_half_work(struct work_struct *work)
{
  printk(KERN_INFO "This is bottom half\n");
}

// 中斷處理程序（頂半部）
static irqreturn_t interrupt_handler(int irq, void *dev_id)
{
  printk(KERN_INFO "This is top half\n");

  schedule_work(&my_work); // 安排底半部

  return IRQ_HANDLED;
}
```

## Follow up: 是否可以在 Linux 裡實作 interrupt handler 的 context, 這樣就能用 sleep?

答案是 可以.

問題是 是否應該設計這機制?

想像一下你是 kernel designer:

1. 為了讓 ISR 內可 sleep, 你必須為每個 ISR 實作 context. 這樣才能將 context 給放進 task queue.
2. 你故意不做 context for ISR, 因為已經有 schedule_work() or workqueue() 等等可以造成相同效果的工具.

若選擇 1, 感覺相當 expensive. ISR 的頻率可能相當高, 整個 task queue 可能會爆炸!
再來 無論你在 ISR 做不做 sleep, context blk 都已經實作了.

The point is, you can design context for ISR doesn't mean you should. This problem has already addressed by other elegant solutions.


## 如何處理 interrupt nesting. 

A. 當一個 ISR 還沒執行完, 另一個相同 interrupt 進入 CPU.

沒有完全正確的方法可以避免.

真正該有的 mindset 是, 理解這件事必然會發生, 然後仔細的設計 ISR 讓這種事情發生時, ISR 仍然能達到你要的效果

請看以下 code:

```cpp
int fpga_frame_write_complete_cnt;

static irqreturn_t interrupt_handler()
{
  fpga_frame_write_complete_cnt++;

  doSomething();

  return IRQ_HANDLED;
}
```

這段 code 最大的問題是, 假若 CPU busy, interrupt_handler() 在執行 doSomething() 時可能另一個新的 interrupt 又會過來. 這時新的 interrupt 可能會被 drop, 也可能舊的會被打斷.

冷靜的思考一下我們究竟要 fpga_frame_write_complete_cnt 做什麼? 事實上無非就是想知道是否有新的 frame 被寫入. 我們可以重新設計這段 code, 讓 ISR 只單純處理 event, 而真正的 frame counter 則紀錄在別處 如 register, memory.

```cpp
static irqreturn_t interrupt_handler()
{
  int x = chkFrameCounter(); // could be hw register, other memory address

  fpga_frame_write_complete_cnt = x;

  doSomething();

  return IRQ_HANDLED;
}
```


B. 一個高 priority interrupt 打斷低 priority interrupt?



# Lock

## Semaphore

假設此張卡片最高可同時執行 16 channels, 可能會設計成這樣的 service:

```cpp
int availableChs = 16;

mutex lock;

int worker() {
  mutex_lock(&lock);
  
  if (availableChs == 0) {
    mutex_unlock(&lock);
    return -1;
  }
  else {
    availableChs--;
    
    encodeTask();
    
    mutex_unlock(&lock);
    return 0;
  }
}
```

Caller:

```cpp
while (worker() != 0) {
  sleep(1);
}
```

or conditional variable 來避免固定時間等待.

使用 conditional variable:
```cpp
int resourceCnt = 16;
cond_t cond;
mutex_t lock;

mutex_lock(&lock); // lock 1
while (1) {
  if (resourceCnt > 0) {
    break;
  }
  else {
    cond_wait(&cond, &lock);
  }
}

resourceCnt--;
mutex_unlock(&lock); // unlock 1

encodeTask();

mutex_lock(&lock); // lock 2
resourceCnt++;
cond_signal(&cond);
mutex_unlock(&lock); // unlock 2
```

```
Thread A                                  Thread B
-----------------------------------------------------------------------
mutex_lock(&lock);                        mutex_lock(&lock); // blocked
// own lock

resourceCnt--;
mutex_unlock(&lock);                      mutex_lock() returns
                                          // B now owns lock

encodeTask();                             if (resourceCnt == 0) {
                                            cond_wait(&cond, &lock);
                                            // atomically:
                                            // 1. release lock
                                            // 2. sleep
                                          }

mutex_lock(&lock);
// A owns lock again

resourceCnt++;
cond_signal(&cond);                       // B is awakened
                                          // but cond_wait does not return yet
                                          // because B must re-acquire lock first

mutex_unlock(&lock);                      // lock released by A
                                          // B re-acquires lock
                                          // cond_wait returns
```


然而 Semaphore comes to rescure:

```cpp
Semaphore availableChs(16);

int worker() {
  availableChs.wait();
  
  encodeTask();
  
  availableChs.signal();
}
```

Semaphore does: lock, conditional variable, and counter.


## Lock Free
核心概念: CAS (Compare and Swap)

```
CAS(ptr_to_var, old_val, new_val):

ptr_to_var: var is the target you want to update, such as tail of queue
old_val: the var expected value.
new_val: the new value you want var to be

old_val = tail; // copy the address of last element
CAS(&tail, old_val, node) // if success, tail == node
old_val->next = node;
```

### Difference of lock version.

```
mutex_t lock;
mutex_lock(&lock);

Node *old_tail = tail;
tail = node;
old_tail->next = node;

mutex_unlock(&lock);
```

vs


```
Node *old_tail = tail;

while ( !CAS(&tail, old_tail, node) ) {
  // retry until we can put node to tail
}
// If CAS return true, tail is already == node

old_tail->next = node;

```

The difference is, one is using mutex lock to lock whole queue and operate; the lock-free version is only spin CAS on 'tail' of the queue, because when you enqueue, tail is the only thing you care.

So, you can fetch front data in lock-free version because it have nothing to do with tail. But if you lock the whole queue, then you can't fetch front.


# Memory

## text

## bss, data

## stack, heap


# DMA

```
CPU -> DMA -> Memory Controller -> DDR
```

DMA Engine is a logic nearby CPU. It helps CPU to move data.

Every Logic can attach a DMA engine to it. VCU can have one, A53 has one, even FPGA can 'make one'.

1. CPU sends DMA a descriptor to move data, DMA engine takes the task and talks to Memory controller to finish the job.
2. Meanwhile, CPU can do other jobs.
3. When task is done, DMA sends interrupts to inform CPU.

DMA engine can implement clever way to move data. For example:
1. The task is to move 1KB from dst A to dst B
2. The cache line of DMA engine is 256B, it can split 1KB -> 256B x 4, pipeline 4 requests.
3. Obviously, the order doesn't matter.

```
--- 256B: A[0]   -> B[0]
--- 256B: A[256] -> B[256]
--- 256B: A[512] -> B[512]
--- 256B: A[768] -> B[768]
```

4. Because cache line fixed. Pay attention to memory address alignment.


# RTOS

## Task
RTOS 裡的 task 與 Linux 最大的不同點, 在於 memory. 由於 MPU 沒有 MMU, 因此也沒有所謂 virtual memory.

所有的 task 共享一個 memory space. 因此可能寫壞別的 task 的 stack, heap, bss, data.

RTOS program 更接近一個 multi-threaded program. 每個 task 就是一個 thread.


# Ring Buffer

## Interview 成功的心法: 冷靜. 有步驟

## 第一步: 正確的定義 strcture and function

```cpp
typedef struct {
  // TBA
} ring_buf_t;

int ring_read(ring_buf_t &buf, uint8_t &rb_data) {
}

int ring_write(ring_buf_t &buf, uint8_t &wr_data) {
}
```

接下來, 我們需要兩個 pointer 分別記錄 read & write. 然後一個 array 紀錄真正寫入的 data. 因此, 正確的 ring_buf_t 為:

```cpp
const int SZ = 4;

typedef struct {
  int head;
  int tail
  uint8_t data[SZ];
} ring_buf_t;
```

至此, 第一步就大功告成了. 當然此時還可以做一些準備工作來降低摩擦, 如 return 0.

```cpp
int ring_read(ring_buf_t &buf, uint8_t &rb_data) {
  return 0;
}

int ring_write(ring_buf_t &buf, uint8_t &wr_data) {
  return 0;
}
```

## 第二步: 快速寫出 ring_read & ring_write

這一步就是快速寫完此兩 function 的主體. 不要多想特殊狀況.

```
int ring_read(ring_buf_t &buf, uint8_t &rb_data) {
  rb_data = buf.data[ buf.tail ];
  buf.tail = (buf.tail + 1) % SZ;

  return 0;
}

int ring_write(ring_buf_t &buf, uint8_t wr_data) {
  buf.data[ buf.head ] = wr_data;
  buf.head = (buf.head + 1) % SZ;

  return 0;
}
```

## 第三步: 解決考點 underflow & overflow

先寫好 framework:

```
int ring_read(ring_buf_t &buf, uint8_t &rb_data) {
  // chk for underflow
  if (...) { return -1; }

  rb_data = buf.data[ buf.tail ];
  buf.tail = (buf.tail + 1) % SZ;

  return 0;
}

int ring_write(ring_buf_t &buf, uint8_t wr_data) {
  // chk for overflow
  if (...) { return -1; }

  buf.data[ buf.head ] = wr_data;
  buf.head = (buf.head + 1) % SZ;

  return 0;
}
```

至此. 就要進入真正的主題: 處理 underflow & overflow.

先畫出圖

```
initial case:

0 1 2 3
- - - -
h
t
```

首先處理 ring_read. 由於剛開始 head and tail 都是 0. 他倆重疊, 並且因為 buffer 是空的, ring_read 為不合理, 因此 if 的條件可以簡單地寫成
if (head == tail) { return }

再來處理 ring_write, 先讓 write 寫入 3 次, 分別為 'A', 'B', 'C':

```
0 1 2 3
A B C -
      h
t
```

這是最難的地方, 就是我們是否可允許寫入 'D'. 理論上因為 buffer size = 4, 所以我們勇敢的寫入 'D'.

```
0 1 2 3
A B C D
h
t
```

這時 buffer 已滿. 不能再寫. 因此 if 的條件簡單地寫下 if (head == tail) { return }. 但是 這會跟 initial case 衝突. 如果 head == tail 就要 return, 那麼一開始就完全不能動.

這時我們就要想到另一種解法, 那就是無一法. 最多 head 只能多寫 (size-1) 次. 也就是說, 寫到 'C' 就得停止.

if condition 就變成:

```cpp
if ( ((buf.head + 1) % SZ) == buf.tail) { return -1; }
```


### Counter 法: 自然, 直覺

```cpp
#define SZ 4
typedef struct {
  int head;
  int tail;
  uint8_t data[SZ];
  
  int counter;
} ring_buffer_t;

int read_data(ring_buffer_t &buf, char &ret) {
  if (buf.counter == 0) { return -1; }

  ret = buf.queue[ buf.r_idx ];
  buf.r_idx = (buf.r_idx + 1) % SZ;
  buf.counter--;

  return 0;
}

int write_data(ring_buffer_t &buf, char c) {
  if (buf.counter == SZ) {
    return -1;
  }

  buf.queue[ buf.w_idx ] = c;
  buf.w_idx = (buf.w_idx + 1) % SZ;
  buf.counter++;

  return 0;
}
```

### 無一法: 無需 counter. 所以無需保護 counter

```cpp
#define SZ 4
typedef struct {
  int head;
  int tail;
  uint8_t data[SZ];
} ring_buffer_t;

int ring_read(ring_buffer_t &buf, uint8_t &rb_data) {
  if ( buf.tail == buf.head ) { return -1; }
  
  rb_data = buf.data[ buf.tail ];
  buf.tail = (buf.tail + 1) % SZ;

  return 0;
}

int ring_write(ring_buffer_t &buf, uint8_t data) {
  if ( (buf.head + 1) % SZ == buf.tail ) { return -1; }
  
  buf.data[ buf.head ] = data;
  buf.head = (buf.head + 1) % SZ;

  return 0;
}
```