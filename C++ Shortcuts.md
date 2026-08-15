// 1. Sort
sort(v.begin(), v.end());

// 2. Custom sort
sort(v.begin(), v.end(), [](auto& a, auto& b) {
    return a.second < b.second;
});

// 3. Max heap
priority_queue<int> pq;

// 4. Min heap
priority_queue<int, vector<int>, greater<>> pq;

// 5. Frequency
unordered_map<int,int> mp;
mp[x]++;

// 6. Existence
unordered_set<int> st;
st.count(x);

// 7. Binary search
auto it = lower_bound(v.begin(), v.end(), x);

// 8. Stack
stack<int> st;
st.push(x);
st.pop();
st.top();

// 9. BFS
queue<int> q;
q.push(x);
q.pop();
q.front();

// 10. Reverse
reverse(v.begin(), v.end());