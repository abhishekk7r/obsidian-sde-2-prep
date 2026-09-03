# Common C++ Operations

## 1. Sort
```cpp
sort(v.begin(), v.end());
```

## 2. Custom Sort
```cpp
sort(v.begin(), v.end(), [](auto& a, auto& b) {
    return a.second < b.second;
});
```

## 3. Max Heap
```cpp
priority_queue<int> pq;
```

## 4. Min Heap
```cpp
priority_queue<int, vector<int>, greater<>> pq;
```

## 5. Frequency Count
```cpp
unordered_map<int,int> mp;
mp[x]++;
```

## 6. Existence Check in Set
```cpp
unordered_set<int> st;
st.count(x);
```

## 7. Binary Search (Lower Bound)
```cpp
auto it = lower_bound(v.begin(), v.end(), x);
```

`lower_bound` = ≥, `upper_bound` = >.

## 8. Stack Operations
```cpp
stack<int> st;
st.push(x);
st.pop();
st.top(); // Access top element without popping.
```

## 9. Breadth-First Search (BFS)
```cpp
dqueue<int> q;
q.push(x);
q.pop();
f.front(); // Access front element.
```

## 10. Reverse a Vector or Container
```cpp
reverse(v.begin(), v.end());
default