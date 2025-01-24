# codepratice-day10
代码随想录第10天

栈与队列内容。
补。

[逆波兰表达式求值](https://leetcode.cn/problems/evaluate-reverse-polish-notation/)
遇到运算符就取值计算，再将两数计算结果压回。如果遇到数值就直接压入。
最后栈里的结果就是答案。

```CPP
class Solution {
public:
    int evalRPN(vector<string>& tokens) {
        stack<long long> st;
        for(int i=0;i<tokens.size();i++)
        {
            if(tokens[i]=="+"||tokens[i]=="-"||tokens[i]=="*"||tokens[i]=="/")
            {
                long long num1=st.top();
                st.pop();
                long long num2=st.top();
                st.pop();
                if(tokens[i]=="+") st.push(num2+num1);
                if(tokens[i]=="-") st.push(num2-num1);
                if(tokens[i]=="*") st.push(num2*num1);
                if(tokens[i]=="/") st.push(num2/num1);
            }
            else
            {
                st.push(stoll(tokens[i]));
            }
        }
        
        long long result=st.top();
        st.pop();
        return result;
    }
};
```

[滑动窗口最大值](https://leetcode.cn/problems/evaluate-reverse-polish-notation/)
这道题除了代码随想录的解法，之前我还写过leetcode官网解法，也是可以学习的。

代码随想录解法用的单调队列，先定义一个单调队列的类。这个单调队列的写法是：1.弹出时，如果值和最前面相等，弹出最前面的数。2.压入时，如果值大于队列最后，弹出，直到队列最后元素小于等于值。
查询最大值时，直接返回队头。
```CPP
class Solution {
private:
    class myqueue{
    public:
        deque<int> que;
        //使用deque单调队列
        //每次弹出的时候，比较当前要弹出的数值是否等于队列出口元素的数值，如果相等则弹出。
        //同时pop之前判断队列当前是否为空
        void pop(int value)
        {
            if(!que.empty()&&value==que.front())
            {
                que.pop_front();
            }
        }
        //如果push的数值大于入口元素的数值，那么就将队列后端的数值弹出，直到push的数值小于等于队列入口元素的数值为止。
        //这样就保持了队列里的数值是从大到小的了
        void push(int value)
        {
            while(!que.empty()&&value>que.back())
            {
                que.pop_back();
            }
            que.push_back(value);
        }
        //查询当前队列里的最大值，直接返回队列前端也就是front就可以
        int front()
        {
            return que.front();
        }
    };
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        myqueue que;
        vector<int> result;
        for(int i=0;i<k;i++)
        {
            //先将前k的元素放进队列
            que.push(nums[i]);
        }
        result.push_back(que.front());//result记录前k的元素的最大值
        for(int i=k;i<nums.size();i++)
        {
            que.pop(nums[i-k]);//滑动窗口移除最前面元素
            que.push(nums[i]);//华东窗口加入最后面的元素
            result.push_back(que.front());//记录对应的最大值
        }
        return result;

    }
};
```

leetcode官方题解，大根堆写法
两个相邻滑动窗口，只有一个元素是变化的，利用这个特点优化。
大根堆，可以实时维护一系列元素的最大值。
初始时将前k个元素放入优先队列，每当右移窗口，可以把一个元素加入，此时堆顶是最大值，但是这个最大值可能不在当前滑动窗口，每次我们压入二元组(num,index)，每次移动窗口
，我们循环判断堆顶的索引是否在i-k这个滑动窗口内，不在就弹出，循环。再压入堆顶元素作为答案即可。
复杂度：放入一个元素到优先队列是o(log n)的，n个元素为o(nlog n)
```CPP
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        int n=nums.size();
        priority_queue<pair<int,int>> q;
        for(int i=0;i<k;++i)
        {
            q.emplace(nums[i],i);
        }
        vector<int> ans={q.top().first};
        for(int i=k;i<n;++i)
        {
            q.emplace(nums[i],i);
            while(q.top().second<=i-k)
            {
                q.pop();
            }
            ans.push_back(q.top().first);
        }
        return ans;
    }
};
```

leetcode官方题解，队列写法
滑动窗口的两个下标i,j。nums[i]<=nums[j]的话，i如果还在，j一定还在窗口，i在j左侧，nums[i]一定不是最大值（小于j的值）。
用一个队列存储没有移除的元素下标，队列存储的下标按从小到大存，队列下标对应元素严格从大到小递减（在每次存储前进行判断，如果大于队尾，弹出队尾，如果没大于队尾，压入）保证队尾是队列里最小值，但是是当前排序的大于队尾的值（有点绕。
初始化时先对前k个值维护队列，将这个队列的front给到第一个ans元素。
接着从k到n,继续这个操作，但是在循环时判断队头的索引是否在i-k，不是的话就不在滑动窗口内。
```CPP
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        int n=nums.size();
        deque<int> q;
        for(int i=0;i<k;++i){
            while(!q.empty()&&nums[i]>=nums[q.back()])
            {
                q.pop_back();
            }
            q.push_back(i);
        }

        vector<int> ans={nums[q.front()]};
        for(int i=k;i<n;++i)
        {
            while(!q.empty()&&nums[i]>=nums[q.back()])
            {
                q.pop_back();
            }
            q.push_back(i);
            while(q.front()<=i-k)
            {
                q.pop_front();
            }

            ans.push_back(nums[q.front()]);
        }
        return ans;

    }
};
```

[前 K 个高频元素](https://leetcode.cn/problems/top-k-frequent-elements/description/)
leetcode官方题解也类似。
需要注意重载对比。写一个类传入到Priority_queue。
先统计每个元素出现的次数，用哈希表。
维护小根堆，每次压入哈希表元素，如果小根堆大小大于k，弹出堆顶（当前最小数），最后将小根堆值倒过去输入到答案数组中。
```CPP
class Solution {
public:
    //小顶堆
    class mycomparison{
        public:
            bool operator()(const pair<int,int>& lhs,const pair<int,int>& rhs)
            {
                return lhs.second>rhs.second;
            }
    };


    vector<int> topKFrequent(vector<int>& nums, int k) {
        //要统计元素出现的频率
        unordered_map<int,int> map;//map<nums[i],对应出现的次数>
        for(int i=0;i<nums.size();i++)
        {
            map[nums[i]]++;
        }

        //对频率排序
        //定义一个小顶堆，大小为k
        priority_queue<pair<int,int>, vector<pair<int,int>>,mycomparison> pri_que;

        //用固定大小为k的小顶堆，扫描所有频率的数值
        for(unordered_map<int,int>::iterator it=map.begin();it!=map.end();it++)
        {
            pri_que.push(*it);
            if(pri_que.size()>k)
            {//如果堆的大小大于k，则队列弹出，保证堆的大小一直为k
                pri_que.pop();
            }
        }

        //找出前k个高频元素，因为小顶堆先弹出的是最小的，所以倒序来输出到数组
        vector<int> result(k);
        for(int i=k-1;i>=0;i--)
        {
            result[i]=pri_que.top().first;
            pri_que.pop();
        }
        return result;
    }
};
```



