---
title: "Linked List"
date: 2026-01-01
--- 

## Define

```cpp
class Node {
public:
  int val;
  Node *next;

  Node(int v) {
    this->val = v;
    this->next = NULL;
  }
};
```

## Declare

```cpp
Node n(3); // in stack
Node *np = new Node(5); // in heap
```


## 第一階段：單向 linked list 手感

先把：

* next pointer
* dummy node
* traversal
* insertion
* deletion

練熟。

推薦：

1. LC 206 Reverse Linked List
2. LC 21 Merge Two Sorted Lists
3. LC 83 Remove Duplicates from Sorted List
4. LC 203 Remove Linked List Elements
5. LC 876 Middle of the Linked List


## 第二階段：pointer choreography

推薦：

1. LC 19 Remove Nth Node From End
2. LC 24 Swap Nodes in Pairs
3. LC 92 Reverse Linked List II
4. LC 143 Reorder List




## Common Technique:
1. slow fast pointer
2. interweaving
3. dummay node (for node deletion)


2.1 Write code to remove duplicates from an unsorted linked list.

void removeDup(struct ListNode *head) {
  bool exists[] = { false; }
  struct ListNode *p = head;
  struct ListNode *prev = NULL;

  while (p) {
    if (exist[p->val]) {
      prev->next = p->next;
      struct ListNode *t = p;
      p = p->next;
      free(t);
    }
    else { 
      exist[p->val] = true;
      prev = p;
      p = p->next;
    }
  }
}

FOLLOW UP

Two pointer solutions in O(N^2)

void removeDup(struct ListNode *head) {
  struct ListNode *fast = head;
  struct ListNode *slow = NULL;
  struct ListNode *prev = NULL;

  if (head == NULL) { return; }
  slow = head;
  fast = fast->next;
  while (fast) {
    while (slow) {
      if (slow == fast) { slow = head; break; }
      else if (slow->val == fast->val) {
        // remove slow
        prev->next = slow->next;
        slow = slow->next;
      }
    else {
        prev = slow;
        slow = slow->next;
      }
    }
    fast = fast->next;
  }
}

2.2 Find kth to last element in linked list.

struct ListNode *kthToLast(struct ListNode *head, int k) {
  struct ListNode *fast = head;
  struct ListNode *slow = head;

  for (int i = 0; i < k; i++) {
    if (!fast) { return NULL; }
    fast = fast->next;
  }

  while (fast) {
    fast = fast->next;
    slow = slow->next;
  }

  return slow;
}

2.6 Implement a function to check if a linked list is a palindrome

bool isPal(struct ListNode *head) {
  // 1. Find mid
  struct ListNode *mid = head;
  struct ListNode *fast = head;

  while (fast && fast->next) {
    mid = mid->next;
    fast = fast->next->next;
  }

  struct ListNode *rev = reverse(mid, NULL);

  struct ListNode *p = head;
  struct ListNode *q = rev;

  while (p && q) {
    if (p->val != q->val) { return false; }
      p = p->next; q = q->next;
    }
  return true;
}

struct ListNode *reverse(struct ListNode *np, struct ListNode *prev) {
  if (np == NULL) { return NULL; }
  if (np->next == NULL) { np->next = prev; return np; }
  else {
    struct ListNode *next = np->next;
    np->next = prev;
    return reverse(next, np);
  }
}
