---
title: "Bits"
date: 2026-01-01
--- 

## Basic operations

- Get bit k

```cpp
(x >> k) & 0x1
```

- Set bit k

```cpp
x |= (1 << k);
```
- Clear bit k

```cpp
x &= ~(1 << k);
```

- Toggle bit k

```cpp
x ^= ~(1 << k);
```

判斷 power of two

## Level 1 — Interview 超常見

- 判斷 power of two

(x > 0) && (x & (x-1) == 0)

- Clear lowest set bit

```cpp
x & (x-1)
```

- 4-byte alignment:

```cpp
(x + 3) & ~3
```


Easy

- [x] LC 190 Reverse Bits
- [x] LC 191 Number of 1 Bits
- [x] LC 231 Power of Two
- [x] LC 338 Counting Bits

⸻

Medium

- [ ] LC 78 Subsets
- [ ] LC 90 Subsets II
- [ ] LC 137 Single Number II
- [ ] LC 260 Single Number III
- [ ] LC 201 Bitwise AND of Numbers Range
- [ ] LC 421 Maximum XOR of Two Numbers
- [ ] LC 1310 XOR Queries

⸻

Hard-ish but worth

- [ ] LC 982 Triples with Bitwise AND Equal To Zero
- [ ] LC 2172 Maximum AND Sum
- [ ] LC 1799 Maximize Score After N Operations
