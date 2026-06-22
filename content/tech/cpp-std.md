---
title: "C++ STL"
date: 2025-04-10
tags: ["c++"]
draft: false
---

## Binary Search

### int version

```
while (L < R) {
  int M = L + (R-L)/2;
  if (check(M)) { return M; }
  else if (A[M] < target) { L = M+1; }
  else { R = M; }
}
return L;
```

### double version

```
while (R - L > 1e-6) {
  double M = L + (R-L) / 2.0;
  if (M < target) { L = M; }
  else { R = M; }
}
return L;
```

L X X X M X T X R
 

## Vector
### Declaration:

```
vector<int> vec;
```

### 2D:
```
vector< vector<int> > vec(n, vector<int>(m, 0));
```

### Size:

```
vec.size()
```


### Add:

```
vec.push_back(v)
```


### Remove:

```
vec.erase( vec.begin() ); // remove head
vec.erase( vec.begin()+2 ); // remove idx=2, the 3rd element
```


### Read:

```
int v = vec.at(i);
int v = vec[i];
```


### Overwrite:

```
vec[i] = newVal;
```


### Sort:

```
sort(vec.begin(), vec.end()); // from small to large
sort(vec.begin(), ven.end(), greater<int>()); // from large to small
```


### Custom Sort:

```
struct Interval {
  int start;
  int end;
};

bool compar(T a, T b) { // return true if 'a' should be placed before 'b'
  if (a.start == b.start) { return a.end < b.end; }
  else { return a.start < b.start; }
}
sort(vec.begin(), vec.end(), compar);
```

### Trick: Use Two elements enumeration to check if compar() is what you want.

```
vector<struct Interval> A = { {1, 5}, {1, 3} };
sort(A.begin(), A.end(), compar);
```


### Search(sorted):

```
1 3 5 7 9
  ^
auto it = lower_bound(vec.begin(), vec.end(), 3); // first element greater or equal to 3


1 3 5 7 9
    ^
auto it = upper_bound(vec.begin(), vec.end(), 3); // first element greater than 3, which is 5


8 7 5 5 4 3 3 2 1
          ^
auto it = lower_bound(vec.begin(), vec.end(), 3, greater<int>());
// first element smaller or equal to 3.

8 7 5 5 4 3 3 2 1
        ^
auto it = upper_bound(vec.begin(), vec.end(), 5, greater<int>());
// first element smaller than 5
```


## Set(sorted)
### Declaration:

```
set<int> s;
set<int, greater<int>> s; // decreasing
```

### Size:

```
s.size();
```

### Add:

```
s.insert(v);
```

### Remove:

```
s.erase(v);
```

### Check exist:

```
s.find(v) != s.end() // faster
s.count(v);
```

### Foreach:

```
for (auto v : s) { ... }
```

### Iterator:

```
I: set<int>::iterator iter = s.begin(); // point to beginning
II: iter = s.end(); // point to end
III: set<int>::iterator nx = next(iter, 1); // point to iter's next
```
### For Loop:

```
// from small to big
for (set<int>::iterator iter = s.begin(); iter != s.end(); iter++) {
  int v = *iter;
}
// from big to small
for (set<int>::reverse_iterator iter = s.rbegin(); iter != s.rend(); iter++) {
  int v = *iter;
}
```

### Remove duplicates:

```
vector<int> vec = { 1,1,2,2,2,3 };
set<int> s(vec.begin(), vec.end()); // s has 1,2,3
```



## Set(Unsorted)
### Declaration:

```
unordered_set set<int> s;
```


### Size:

```
s.size();
```


### Add:

```
s.insert(v);
```


### Remove:

```
s.erase(v);
```


### Find:

```
s.count(v);
I: exists -> return 1
II: not exist -> return 0
```


### Foreach:

```
for (auto v : s) { ... }
```


### Remark: No iterator style



## multiset
Be careful about 'erase' element!

```
multiset<int> ms = { 2,3,3,5 };

ms.erase(3); // only {2,5}
ms.erase(ms.find(3)) // {2,3,5}
```


## Map(Sorted)
### Declaration:

```
map<int, int> mp;
```

### Add:

```
mp.insert( { key, val } ); // make a pair and insert
mp[key] = val;
```

### Remove:

```
mp.erase(key);
mp.erase(first, last); // remove all b/t first and last
```

### Check if {key} exists:

```
mp.find(key);
- Exists: mp.find(key) != mp.end()
- Not exist: mp.find(key) == mp.end();
```

### Foreach:

```
for (auto iter : mp) {
  int key = iter.first;
  int val = iter.second;
}

for (auto &[k, v] : mp) {
  // k is key, v is value
}
```

### *** Binary Search

```
// mp[start] = end, represent a segment
mp[1] = 5;
mp[6] = 8;
mp[9] = 12;

T = 7

auto it = mp.upper_bound(T); // points to mp[9]
it = prev(it); // it points to mp[6]
```


## Map(Unsorted)
### Declaration:

```
unordered_map<int, int> mp;
```

### Add:

```
mp.insert( { key, val } ); // make a pair and insert
mp[key] = val;
```

### Remove:
```
mp.erase(key);
```

### Check if {key, value} exists:
```
mp.find(key);
- Exists: mp.find(key) != mp.end()
- Not exist: mp.find(key) == mp.end();
```

### Check Key:
```
mp.count(key);
```

### For Loop:
```
for (unordered_map<int, int>::iterator iter = mp.begin(); iter != mp.end(); iter++) {
  int key = iter->first;
  int val = iter->second;
}
```

### Foreach:

```
for (auto iter : mp) {
  int key = iter.first;
  int val = iter.second;
}

for (auto &[k, v] : mp) {
  // k is key, v is value
}
```


## Priority Queue
```
// Max Heap
priority_queue<int> mx;
priority_queue<int, vector<int>, less<int>> mx;

// Min Heap
priority_queue<int, vector<int>, greater<int>> mi;
mi.push(5);
mi.push(1);
mi.push(3);
mi.top(); -> 1
mi.pop();
mi.top(); -> 3
mi.pop();
mi.top(); -> 5
```

Custom comparator:
- The comparator answers the question: Is 'a' of lower priority than 'b'?
- true: 'b' has higher priority.
- This reverses the usual intuition: to give a higher priority, we need to return false when 'a' is “larger.”


sort {l, r} such that larger r than smaller l

```
auto compar = [](const PII &a, const PII &b) {
  if (a.second != b.second) { a.second < b.second; }
  else { a.first > b.first; }
};
```


## Dijkstra
```
priority_queue< PII, vector< PII >, greater<> > pq;

pq.push( { 5, 1 } ); // weight 5, dst node 1
pq.push( { 3, 2 } ); // weight 3, dst node 2
pq.push( { 3, 5 } ); // weight 3, dst node 5

auto [w, dst] = pq.top() -> w is 3, dst is 2
pq.pop();
auto [w, dst] = pq.top() -> w is 3, dst is 5
pq.pop();
auto [w, dst] = pq.top() -> w is 5, dst is 1
pq.pop();
```

Sample implements;

```
vector<int> dist(n, INT_MAX/2);

int PFS(vector< vector<int> > &gf, int src, int dst, vector<int> &dist) {
  auto compar = [](const PII &a, const PII &b) { // return true if 'a' has lower priority
    if (a.first != b.first) {
      return !(a.first < b.first);
    }
    else {
      return !(a.second < b.second);
    }
  };
  priority_queue<PII, vector<PII>, decltype(compar)> pq; // { distance, next }
  pq.push( {0, 0} );

  while (!pq.empty()) {
    auto [distance, cur] = pq.top(); pq.pop();

    if (dist[cur] != INT_MAX/2) { continue; } // visited
    dist[cur] = distance;

    if (cur == dst) { return distance; }

    for (auto nxt : gf[cur]) {
      pq.push( { distance+1, nxt } );
    }
  }

  return -1;
}
```

## Union Find
```
int FindRoot(int v, vector<int> &root) {
  if (root[v] != v) {
    root[v] = FindRoot(root[v], root); // Path compression
  }
  return root[v];
}

void Union(int p, int q, vector<int> &root) {
  int rp = FindRoot(p, root);
  int rq = FindRoot(q, root);

  if (rp != rq) {
    if (rp < rq) {
      root[rq] = rp;
    }
    else {
      root[rp] = rq;
    }
  }
}

vector<int> root(n);

for (int i = 0; i < n; i++) { root[i] = i; }
```


## Queue
Declaration:
```
queue<int> q;
```

Add:
```
q.push(v);
```

Remove:
```
q.pop();
```

Peek:
```
int v = q.front();
```

Emptiness:
```
bool isEmpty = q.empty();
```


## Deque
Declaration:
```
deque<int> deq;
```

Add:
```
deq.push_front(v);
deq.push_back(v);
```

Remove:
```
deq.pop_front();
deq.pop_back();
```

Peek:
```
int first = deq.front();
int last = deq.back();
```


## Stack
Declaration:
```
stack<int> st;
```

Add:
```
st.push(v);
```

Remove:
```
st.pop();
```

Peek:
```
st.top();
```


## Convert integer to string:
```
int a = 135;
string s = to_string(a);
```


## Trie

```
#define WIDTH 256


class TrieNode {
public:
  TrieNode *next[WIDTH];
  bool isWord;

  TrieNode() {
    memset(next, 0, sizeof(next));
    isWord = false;
  }

  ~TrieNode() {
    for (int i = 0; i < WIDTH; i++) {
      if (next[i]) { delete next[i]; }
    }
  }
};

class Trie {
  TrieNode *root;

public:
  Trie() {
    root = new TrieNode();
  }

  /** Inserts a word into the Trie. Update count */
  void insert(string &s) {
    TrieNode *p = root;
    for (auto c : s) {
      if (!p->next[c]) {
        p->next[c] = new TrieNode();
      }
      p = p->next[c];
    }
    p->isWord = true;
  }

  /** Returns count for s in the Trie. */
  bool search(string &s) {
    TrieNode *p = root;
    for (int i = 0; i < s.size() && p; i++) {
      p = p->next[ s[i] ];
    }
    return p && p->isWord;
  }

  bool startWith(string &prefix) {
    TrieNode *p = root;
    for (int i = 0; i < prefix.size() && p; i++) {
      p = p->next[ prefix[i] ];
    }
    return p;
  }

  ~Trie() {
    delete root;
  }
};
```


## Misc
lambda function - an embedded function without name

```
vector<int> A;

// Now sort from large to small
bool compar(int a, int b) { return a > b; }
sort(A.begin(), A.end(), compar);

// lambda
sort(A.begin(), A.end(), [](int a, int b) { return a > b; })


// lamba used in callback
void cb(int n) { cout << n << endl; }

thread t(cb, n);


thread t([](int n) { cout << n << endl; }, n);
```


Unique Pointer
```
unique_ptr<MyClass> p = make_unique<MyClass>();
```

Shared Pointer
```
shared_ptr<MyClass> p = make_shared<MyClass>();
```


Member Initializer List -
```
struct X {
  int c;
  int d;
};

class T {
  int a;
  int b;
  struct X x;

  T() : a(-1), b(0), x{3, 4} {
  }
}
```

Overflow, MOD
Safe:
```
+: (a + b) % MOD -> ( (a % MOD) + (b % MOD) ) % MOD
*: (a * b) % MOD -> ( (a % MOD) * (b % MOD) ) % MOD
```

Need process:
```
-: (a - b) % MOD -> (a - b + MOD) % MOD)
```
Meanless:
```
/: (a / b) % MOD is not ( (a % MOD) / (b % MOD) ). But (a / b) is not going to overflow.
```

Prime:
```
vector<bool> isPrime(limit+5, true);

void PrimeSieve(vector<bool> &isPrime, int limit) {
  isPrime[0] = false;
  isPrime[1] = false;

  for (int p = 2; p <= sqrt(limit); p++) {
    if (isPrime[p]) {
      for (int x = p * p; x <= limit; x += p) {
        isPrime[x] = false;
      }
    }
  }
}
```
