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

```
static irqreturn_t interrupt_handler()
{
  printk(KERN_INFO "This is top half");

  sleep() -> schedule(); // just for fun, try to produce context switch

  printk(KERN_INFO "This is bottom half");

  return IRQ_HANDLED;
}
```


因此, 真的要做以上的動作(在 interrupt handler 裡面做一些事 -> 讓其他 task 跑一下 -> 再回來 interrupt handler 剩下想做的), 應該使用 schedule_work() 來代替 scheudle(). 這樣就會將 interrupt handler 剩下的事情正確 create task, 並排進 work_queue.
```
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


## 如何處理 interrupt nesting. 

A. 當一個 ISR 還沒執行完, 另一個相同 interrupt 進入 CPU.

沒有完全正確的方法可以避免.

真正該有的 mindset 是, 理解這件事必然會發生, 然後仔細的設計 ISR 讓這種事情發生時, ISR 仍然能達到你要的效果

請看以下 code:

```
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

```
static irqreturn_t interrupt_handler()
{
  int x = chkFrameCounter(); // could be hw register, other memory address

  fpga_frame_write_complete_cnt = x;

  doSomething();

  return IRQ_HANDLED;
}
```


B. 一個高 priority interrupt 打斷低 priority interrupt?
