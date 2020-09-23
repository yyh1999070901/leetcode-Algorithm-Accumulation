#### 解决leetcode股票买卖系列问题

**一、限制最多完成一笔交易（https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/）**

![image-20200923102854061](https://typoraimg0819.oss-cn-beijing.aliyuncs.com/20200923102901.png)

思路：使用贪心法，遍历数组时候，维护前面出现的股票价格最小值，作为买入日，当天为假定卖出日，更新利润。

代码：

```c++
class Solution {
public:
    
    int maxProfit(vector<int>& prices) {
        if(prices.size()==0){
            return 0;
        }
        int minprice=prices[0];
        int ans=0;
        for(int i=0;i<prices.size();i++){
            ans=max(ans,prices[i]-minprice);
            minprice=min(minprice,prices[i]);
        }
        return ans;
    }
};
```

------

**二、无限次交易（https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/）**

![image-20200923103331701](https://typoraimg0819.oss-cn-beijing.aliyuncs.com/20200923103331.png)

思路：差分数组正数求和。

代码：

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int ans=0;
        for(int i=0;i<prices.size()-1;i++){
            if(prices[i+1]>prices[i]){
                ans+=prices[i+1]-prices[i];
            }
        }
        return ans;
    }
};
```

------

**三、限制交易次数---2次交易（https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iii/）----k次交易（https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iv/）**

![image-20200923103548421](https://typoraimg0819.oss-cn-beijing.aliyuncs.com/20200923103548.png)

思路：动态规划方法解决。设dp【i】【k】【2】表示第i天，最多交易k次，是否持有股票的最大利润。例如：dp【2】【3】【1】表示第2天，最多交易3次，持有股票的时候最大利润。dp【2】【3】【0】表示第2天，最多交易3次，未持有股票时候的最大利润。

状态转移：

**dp[i] [k] [0] = max(dp[i-1] [k] [0], dp[i-1] [k] [1]+prices[i]）**表示第i天没有股票的状态可以由两个状态推出来，分别是第i-1天没有股票，以及第i-1天持有股票，第i天出手收获prices[i]利润。

**dp[i] [k] [1] = max(dp[i-1] [k] [1], dp[i-1] [k-1] [1]-prices[i]）**表示第i天有股票的状态可以由两个状态推出来，分别是第i-1天有股票，以及第i-1天没有股票，第i天购入股票花费成本prices[i]，并且i-1天的交易次数为k-1。

特殊处理：当k很大的时候，k>=n/2，当作无限次交易进行处理，可以参考二中方法。

代码：

```c++
int dp[1005][1005][2];

class Solution {
public:
    long long inf=-1000000009;
    int maxProfit(int k, vector<int>& prices) {
        int n=prices.size();
        if(n==0){
            return 0;
        }
        if(k>=n/2){
            long long ans1=0;
            for(int i=1;i<prices.size();i++){
                if(prices[i]>prices[i-1]){
                    ans1+=prices[i]-prices[i-1];
                }
            }
            return ans1;
        }
        for(int j=0;j<=k;j++){
            dp[0][j][0]=0;
            dp[0][j][1]=inf;
        }
        long long ans=0;
        for(int i=1;i<=prices.size();i++){
            int day=i-1;
            for(int j=0;j<=k;j++){
                if(j==0){
                    dp[i][j][0]=0;
                    dp[i][j][1]=inf;
                    continue;
                }
                dp[i][j][0]=max(dp[i-1][j][0],dp[i-1][j][1]+prices[day]);
                dp[i][j][1]=max(dp[i-1][j][1],dp[i-1][j-1][0]-prices[day]);
                if(dp[i][j][0]>ans){
                    ans=dp[i][j][0];
                }
                if(dp[i][j][1]>ans){
                    ans=dp[i][j][1];
                }
            }
        }
        return ans;
    }
};
```

------

**四、包含手续费的无限次购买（https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/）**

思路：和三类似，在购买股票的时候，减去手续费即可。

代码：

```c++
class Solution {
public:
    int dp[1000005][2];
    int inf=-1000000;
    int maxProfit(vector<int>& prices, int fee) {
        dp[0][0]=0;
        dp[0][1]=inf;
        int ans=0;
        for(int i=1;i<=prices.size();i++){
            int day=i-1;
            dp[i][0]=max(dp[i-1][0],dp[i-1][1]+prices[day]);
            dp[i][1]=max(dp[i-1][1],dp[i-1][0]-prices[day]-fee);
            ans=max(ans,dp[i][0]);
            ans=max(ans,dp[i][1]);
        }
        return ans;
    }
};
```

------

**五、包含冷冻期的股票购买（https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/）**

代码：

```c++
class Solution {
public:
    int dp[100007][2];
    int inf=-10000007;
    int maxProfit(vector<int>& prices) {
        if(prices.size()==0){
            return 0;
        }
        int n=prices.size();
        dp[0][1]=inf;
        dp[0][0]=0;
        int ans=0;
        for(int i=1;i<=n;i++){
            int day=i-1;
            dp[i][0]=max(dp[i-1][0],dp[i-1][1]+prices[day]);
            dp[i][1]=dp[i-1][1];
            if(i<2){
                dp[i][1]=-prices[day];
            }
            if(i>=2&&dp[i-2][0]-prices[day]>dp[i][1]){
                dp[i][1]=dp[i-2][0]-prices[day];
            }
            if(dp[i][0]>ans){
                ans=dp[i][0];
            }
            if(dp[i][1]>ans){
                ans=dp[i][1];
            }
        }
        return ans;
    }
};
```

