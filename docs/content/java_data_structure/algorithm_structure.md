---
title: "算法数据结构脚手架"
layout: page
date: 2019-05-25 00:00
---

[TOC]

# Java算法数据结构用法

## 位运算

1. `x & 1 == 1 or == 0`：用来判断奇偶性
2. `x >> 1`： 相当于`x / 2`操作； x >>> 1 无符号右移
3. `x = x & (x - 1)` 清零最低位的1，其它位置不变
4. `x & -x`：整数运算x & (-x)，当x为0时结果为0；x为奇数时，结果为1；x为偶数时，结果为x中2的最大次方的因子。（一般可以用来获取某个二进制数的LowBit）
5. `x & ~x`: 0

## 栈

```java
Stack<Integer> sta = new Stack();
sta.peek(); // peek() 取栈顶元素
sta.isEmpty();
sta.pop();
sta.push(1);
```

### 经典题：《面试题 17.21. 直方图的水量》

```java
class Solution {
    // 面试题 17.21. 直方图的水量
    public int trap(int[] height) {
        if(height == null || height.length <= 2){
            return 0;
        }
        int ans = 0;
        int n = height.length;
        // 单调递减栈
        Stack<Integer> stack = new Stack<>();
        for(int i=0; i<n; i++){
            if(stack.isEmpty()){
                stack.push(i);
            }else {
                if(height[i] < height[stack.peek()]){
                   stack.push(i);
                }else{
                    while (!stack.isEmpty() && height[i] >= height[stack.peek()]){
                        int peekIndex = stack.pop();
                        if(stack.isEmpty()){
                            break;
                        }
                        int lastIndex = stack.peek();
                        // eg [2,1,0,2]，求0的时候是1 * 1， 求1的时候 2*1 (2是因为求1的时候，把之前的更小的加上)
                        int water = (i - lastIndex - 1) *
                                (Math.min(height[lastIndex], height[i]) - height[peekIndex]);
                        ans += water;
                    }
                    stack.push(i);
                }
            }
        }
        return ans;
    }
}
```

## 双端队列

```java
Deque<Integer> deque = new LinkedList<>();
deque.offer(1);
deque.offerLast(1);
deque.offerFirst(1);

deque.poll();
deque.pollFirst();
deque.pollLast();

deque.peek();
deque.peekFirst();
deque.peekLast();

deque.isEmpty();
```

### 经典题：《剑指 Offer 59 - I. 滑动窗口的最大值》

```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        if(nums == null || nums.length <= 0){
            return new int[0];
        }
        int n = nums.length;
        if(n <= k){
            int maxVal = nums[0];
            for(int i=1;i<n;i++){
                maxVal = Math.max(maxVal, nums[i]);
            }
            int[] ans = new int[1];
            ans[0] = maxVal;
            return ans;
        }
        int[] ans = new int[n - k + 1];
        Deque<Integer> deque = new LinkedList();
        // 单调递减双端队列
        for(int i=0;i<k-1;i++){
            if(deque.isEmpty()){
                deque.offerLast(i);
            }else{
                while(!deque.isEmpty() && nums[deque.peekLast()] < nums[i]){
                    deque.pollLast();
                }
                deque.offerLast(i);
            }
        }
        int index = 0;
        for(int i=k-1;i<n;i++){
            // 长度超过的k，也即队列中的 index 必须 大于等于 i-k+1 的
            while (!deque.isEmpty() && deque.peekFirst() < i - k + 1){
                deque.pollFirst();
            }
            // 队列维持单调递减，且当前是最小的元素，队列头是最大元素
            while(!deque.isEmpty() && nums[deque.peekLast()] < nums[i]){
                deque.pollLast();
            }
            deque.offerLast(i);
            ans[index ++] = nums[deque.peekFirst()];
        }
        return ans;
    }
}
```

### 经典题：《剑指 Offer 32 - III. 从上到下打印二叉树 III》

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> ans = new ArrayList<>();
        if (root == null) {
            return ans;
        }
        Deque<TreeNode> d1 = new LinkedList();
        Deque<TreeNode> d2 = new LinkedList();
        int dir = 1;
        d1.offerLast(root);
        while(!d1.isEmpty() || !d2.isEmpty()){
            List<Integer> arr = new ArrayList();
            /*
                1  [1]
               2  3 , 遍历 3，2，加入的时候， 4 5 6 7，先取right，然后头部添加
             4 5  6 7   遍历 4， 5， 6， 7，先取left， 11 10 9 8 仍然是头部添加
            8 9 10 11
            */
            if(dir == 1){
                while(!d1.isEmpty()){
                    TreeNode node = d1.pollFirst();
                    arr.add(node.val);
                    if(node.left != null){
                        d2.offerFirst(node.left);
                    }
                    if(node.right != null){
                        d2.offerFirst(node.right);
                    }
                }
            }else{
               while(!d2.isEmpty()){
                    TreeNode node = d2.pollFirst();
                    arr.add(node.val);
                    if(node.right != null){
                        d1.offerFirst(node.right);
                    }
                    if(node.left != null){
                        d1.offerFirst(node.left);
                    }
                } 
            }
            ans.add(arr);
            dir = -dir;
        }
        return ans;
    }
}
```

## 优先队列(或堆)

```java
PriorityQueue<Integer> priorityQueue = new PriorityQueue<>((a, b)->{
    return b - a;
});
priorityQueue.offer(1);
priorityQueue.poll();
priorityQueue.peek();
priorityQueue.isEmpty();
```

### 经典题：《407. 接雨水 II》（m x n 的矩阵）

```java
class Solution {
    class Node{
        int x;
        int y;
        int h;

        public Node(int x, int y, int h) {
            this.x = x;
            this.y = y;
            this.h = h;
        }
    }
    public int trapRainWater(int[][] heightMap) {
        if(heightMap == null){
            return 0;
        }
        int n = heightMap.length;
        if(n == 0){
            return 0;
        }
        int m = heightMap[0].length;
        if(m == 0){
            return 0;
        }
        int ans = 0;
        // 高度小的
        PriorityQueue<Node> priorityQueue = new PriorityQueue<>((a,b)->{
            return a.h - b.h;
        });
        int[][] dir = {
            {1,0},{-1,0},{0,1},{0,-1}
        };
        boolean[][] vis = new boolean[n][m];
        // 边界处理
        // 两行
        for(int i=0;i<m;i++){
            vis[0][i] = true;
            priorityQueue.offer(new Node(0,i,heightMap[0][i]));
            vis[n-1][i] = true;
            priorityQueue.offer(new Node(n-1,i,heightMap[n-1][i]));
        }
        // 两列
        for(int i=1;i<n-1;i++){
            vis[i][0] = true;
            priorityQueue.offer(new Node(i,0,heightMap[i][0]));
            vis[i][m-1] = true;
            priorityQueue.offer(new Node(i,m-1,heightMap[i][m-1]));
        }
        // https://www.youtube.com/watch?v=cJayBq38VYw
        // 从最小的蔓延开来，每次用较大值替换，并收集雨水
        while (!priorityQueue.isEmpty()){
            // 取对列中最小的元素
            Node node = priorityQueue.poll();
            for(int i=0;i<4;i++){
                int nx = node.x + dir[i][0];
                int ny = node.y + dir[i][1];
                if(nx < 0 || nx >= n || ny < 0 || ny >= m){
                    continue;
                }
                if(vis[nx][ny]){
                    continue;
                }
                // 四周没有访问过的，然后比较大小
                int nh = heightMap[nx][ny];
                // 设置为访问过
                vis[nx][ny] = true;

                // 比 当前pop的node.h还要小，那么可以存水了（此为非边界，且比四周要小)
                ans += Math.max(0, node.h - nh);
                // nx,ny位置是这次新的边界，高度则取当前较大的
                priorityQueue.add(new Node(nx, ny, Math.max(node.h, nh)));
            }
        }
        return ans;
    }
}
```

## 基于红黑树的TreeMap

```java
class Solution {
    // 436. 寻找右区间
    public int[] findRightInterval(int[][] intervals) {
        if(intervals == null || intervals.length == 0){
            return new int[0];
        }
        TreeMap<int[], Integer> treeMap = new TreeMap((a, b)->{
            int[] arr1 = (int[])a;
            int[] arr2 = (int[])b;
            return arr1[0] - arr2[0];
        });
        for(int i=0;i<intervals.length;i++){
            treeMap.put(intervals[i], i);
        }
        int[] res = new int[intervals.length];
        for(int i=0;i<intervals.length;i++){
            int[] entry = treeMap.ceilingKey(new int[]{intervals[i][1], 0});
            if(entry != null){
                res[i] = treeMap.get(entry);
            }else {
                res[i] = -1;
            }
        }
        return res;
    }
}
```
