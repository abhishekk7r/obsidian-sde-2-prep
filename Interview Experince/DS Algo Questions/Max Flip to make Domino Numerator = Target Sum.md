
Set of N dominos laid down vertically in a straight line one after another.

Find the min no of flips(rotate vertically) required to meet the below condition:

Sum of numerators >= targetSum

Eg: Set of 3 dominos - 1/2, 2/5, 5/6 
Target Sum = 10, Ans =1


int minNoOfFlips(int dominos[][], int targetSum) {
}


```c++
Solution: 
#include <bits/stdc++.h>
using namespace std;

int minNoOfFlips(vector<vector<int>>& dominos, int targetSum) {
    int numeratorSum = 0;
    vector<int> difference;

    for (auto& domino : dominos) {
        int numerator = domino[0];
        int denominator = domino[1];

        numeratorSum += numerator;

        if (denominator > numerator) {
            difference.push_back(denominator - numerator);
        }
    }

    if (numeratorSum >= targetSum)
        return 0;

    sort(difference.begin(), difference.end(), greater<int>());

    int count = 0;

    for (int diff : difference) {
        numeratorSum += diff;
        count++;

        if (numeratorSum >= targetSum)
            return count;
    }

    return -1;
}

int main() {
    vector<vector<int>> dominos = {{1, 2}, {2, 5}, {5, 6}};
    int targetSum = 10;

    cout << minNoOfFlips(dominos, targetSum) << endl;

    return 0;
}
```