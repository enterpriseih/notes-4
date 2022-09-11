# 第八章：树

### 基础知识

树的阶数：每个节点的最大子节点数

树的深度：树的高度就是深度，根节点的。

节点的深度：该节点的祖先个数

节点的高度：**p节点的高度是p节点到其最后一个子孙节点的边数**。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/202208161053603.png" alt="二叉树" style="zoom:67%;" />

```
    1
   / \
  2   3
 / \   \
4   5   7
         \
          8
3的高度为2，2的高度为1
```



### 树的深度

```java
public int maxDepth(TreeNode root) {
    if (root == null) {
        return 0;
    } else {
        int leftHeight = maxDepth(root.left);
        int rightHeight = maxDepth(root.right);
        return Math.max(leftHeight, rightHeight) + 1;
    }
}
```

### 树的最小深度

```java
/**
 * 递归法，相比求MaxDepth要复杂点
 * 因为最小深度是从根节点到最近**叶子节点**的最短路径上的节点数量
 */
public int minDepth(TreeNode root) {
    if (root == null) {
        return 0;
    }
    int leftDepth = minDepth(root.left);
    int rightDepth = minDepth(root.right);
    if (root.left == null) {
        return rightDepth + 1;
    }
    if (root.right == null) {
        return leftDepth + 1;
    }
    // 左右结点都不为null
    return Math.min(leftDepth, rightDepth) + 1;
}
```

### [根据中序遍历和后序遍历还原二叉树](https://leetcode.cn/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/iShot2022-04-19_17.21.46.png" alt="在这里插入图片描述" style="zoom: 40%;" />

```java
int postId;// 用postId[0]也可以跟随递归
int[] inorder;
int[] postorder;
Map<Integer, Integer> mapId = new HashMap<>();
public TreeNode buildTree(int[] inorder, int[] postorder) {
    this.inorder = inorder;
    this.postorder = postorder;
    postId = postorder.length - 1;

    // 建立 元素-下标 的哈希表
    int i = 0;
    for (int num : inorder) {
        mapId.put(num, i++);
    }
    return helper(0, inorder.length - 1);
}
private TreeNode helper(int inleft, int inright) {
    // 如果这里没有节点构造二叉树了，就结束
    if (inright < inleft) return null;

    // 选择 postId 位置的元素作为当前子树根节点
    TreeNode root = new TreeNode(postorder[postId]);
    // 根据 root 所在位置分成左右两棵子树
    int index = mapId.get(postorder[postId]);
    postId--;
	
    // 后序遍历：左右根，所以根减一是右子树的根
    // 所以这里先还原右子树
    // 若是前序，根左右，根加一是左节点
    root.right = helper(index + 1, inright);
    root.left = helper(inleft, index - 1);

    return root; 
}
```

### 前后序构造二叉树

不唯一

```
首先pre[0]是根，pre[1]是左子树的根
记左子树的长度为L，pre[1] = post[L-1] // 下标问题
得出左子树的长度，左子树通过左子树的前后序得出
右子树同样

```

```java
public TreeNode constructFromPrePost(int[] pre, int[] post) {
    int N = pre.length;
    if (N == 0) return null;
    TreeNode root = new TreeNode(pre[0]);
    if (N == 1) return root;

    int L = 0;
    // 得出左子树的长度
    for (int i = 0; i < N; ++i)
        if (post[i] == pre[1])
            L = i+1;
	// preLeft[] = pre[1,L], copy和substring类似，要往下标后一位
    // postLeft[] = post[0,L-1]
    root.left = constructFromPrePost(Arrays.copyOfRange(pre, 1, L+1),
                                     Arrays.copyOfRange(post, 0, L));
    root.right = constructFromPrePost(Arrays.copyOfRange(pre, L+1, N),
                                      Arrays.copyOfRange(post, L, N-1));
    return root;
}
```





## 8.1 二叉树的深度优先搜索

树为

```
    1
   / \
  2   3
 / \ / \
4  5 6  7
```

### 中序遍历

4，2，5，1，6，3，7

````java
// 递归
public List<Integer> inorderTraversal(TreeNode root) {
    List<Integer> nodes = new LinkedList<>();
    dfs(root, nodes);
    return nodes;
}

private void dfs(TreeNode root, List<Integer> nodes) {
    if(root != null) {
        dfs(root.left, nodes);
        nodes.add(root,val);
        dfs(root.right, nodes);
    }
}
````

顺着左子节点遍历，因为中序遍历，所以先不读取左子节点，将其先存入栈中
直至没有左子节点，此时的栈顶元素就是中序需要去读取的元素

```java
// 迭代
public List<Integer> inorderTraversal(TreeNode root) {
    List<Integer> nodes = new LinkedList<>();
    Stack<TreeNode> stack = new Stack<>();
    TreeNode cur = root;
    while(cur != null || !stack.isEmpty()){
        while(cur != null){
            stack.push(cur);
            cur = cur.left;
        }
        cur = stack.pop();
        nodes.add(cur.val);
        cur = cur.right;
    }
    return nodes;
}

```



### 前序遍历

1，2，4，5，3，6，7

```java
// 递归
public List<Integer> preorderTraversal(TreeNode root) {
    List<Integer> nodes = new LinkedList<>();
    dfs(root, nodes);
    return nodes;
}

private void dfs(TreeNode root, List<Integer> nodes) {
    if(root != null) {
        nodes.add(root,val);
        dfs(root.left, nodes);
        dfs(root.right, nodes);
    }
}
````

```java
// 迭代
public List<Integer> preorderTraversal(TreeNode root) {
    List<Integer> nodes = new LinkedList<>();
    Stack<TreeNode> stack = new Stack<>();
    TreeNode cur = root;
    while(cur != null || !stack.isEmpty()){
        while(cur != null){
            nodes.add(cur.val);
            stack.push(cur);
            cur = cur.left;
        }
        cur = stack.pop();
        cur = cur.right;
    }
    return nodes;
}
```



### [后序遍历](https://leetcode.cn/problems/binary-tree-postorder-traversal/)

4，5，2，6，7，3，1

```java
// 递归
public List<Integer> postorderTraversal(TreeNode root) {
    List<Integer> nodes = new LinkedList<>();
    dfs(root, nodes);
    return nodes;
}

private void dfs(TreeNode root, List<Integer> nodes) {
    if(root != null) {
        dfs(root.left, nodes);
        dfs(root.right, nodes);
        nodes.add(root,val);
    }
}
````
当到达某个节点时，若已经遍历过其右子树就遍历这个节点
否则先去遍历右子树；
设置一个prev记录前一个遍历的节点

```java
// 迭代
public List<Integer> postorderTraversal(TreeNode root) {
    List<Integer> nodes = new LinkedList<>();
    Stack<TreeNode> stack = new Stack<>();
    TreeNode cur = root;
    TreeNode prev = null;
    while(cur != null || !stack.isEmpty()){
        while(cur != null){
            stack.push(cur);
            cur = cur.left;
        }
        cur = stack.peek();
        if (cur.right == null || cur.right == prev) {
            st.pop();
            res.add(cur.val);
            prev = cur;
            cur = null;
        } else {
            cur = cur.right;
        }
    }
    return nodes;
}
```



## 补充：最近的公共祖先

### 题目

```
     1
   /   \
  2     3
 / \   / \
4   5 6   7
```

p：6，q：2，最近公共祖先：1



### 解法

根据以上定义，若 root 是 p, q 的 最近公共祖先 ，则只可能为以下情况之一：

1、p 和 q 在 root 的子树中，且分列 root 的 异侧（即分别在左、右子树中）；

2、p = root，且 q 在 root 的左或右子树中；

3、q = root，且 p 在 root 的左或右子树中；

**返回值**

1. 当 left 和 right 同时为空 ：说明 root 的左 / 右子树中都不包含 p,q，返回 null；

2. 当 left和 right 同时不为空 ：说明 p, q 分列在 root 的 异侧 （分别在 左 / 右子树），因此 root为最近公共祖先，返回 root ；

3. 当 left 为空 ，right 不为空 ：p,q 都不在 root 的左子树中，直接返回 right 。具体可分为两种情况：
	- p,q 其中一个在 root 的 右子树 中，此时 right 指向 p（假设为 p ）；
	- p,q 两节点都在 root 的 右子树 中，此时的 right 指向 最近公共祖先节点 ；

4. 当 left不为空 ， right 为空 ：与情况 3. 同理；

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/202205222144064.png" alt="image-20220522214443897" style="zoom:50%;" />

```java
public TreeNode lowestCommonAncestor(TreeNode root, 
                                     TreeNode p, 
                                     TreeNode q) {
    if (root == null || root == p || root == q) { 
        // 递归结束条件
        // 只要当前根节点是p和q中的任意一个，
        // 就返回（因为不能比这个更深了，再深p和q中的一个就没了）
        return root;
    }

    // 根节点不是p和q中的任意一个，那么就继续分别往左子树和右子树找p和q
    // 后序遍历
    TreeNode left = lowestCommonAncestor(root.left, p, q);
    TreeNode right = lowestCommonAncestor(root.right, p, q);

    if(left == null && right == null) { 
        // p和q都没找到，那就没有
        return null;
    }else if(left == null && right != null) { 
        // 左子树没有p也没有q，就返回右子树的结果
        return right;
    }else if(left != null && right == null) { 
        // 右子树没有p也没有q就返回左子树的结果
        return left;
    }else { 
        // 左右子树都找到p和q了，
        // 那就说明p和q分别在左右两个子树上，
        // 所以此时的最近公共祖先就是root
        return root;
    }
}
```



## 面试题47：二叉树剪枝
### 题目
一个二叉树的所有结点的值要么是0要么是1，请剪除该二叉树中所有结点的值全都是0 的子树。

例如，在剪除图(a)中二叉树中所有结点值都为0的子树之后的结果如图(b)所示。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0802.png" alt="图2.1">

### 参考代码
``` java
public TreeNode pruneTree(TreeNode root) {
    if (root == null) {
        return root;
    }

    root.left = pruneTree(root.left);
    root.right = pruneTree(root.right);
    if (root.left == null && root.right == null && root.val == 0) {
        return null;
    }

    return root;
}
```

## 面试题48：序列化和反序列化二叉树
### 题目
请设计一个算法能将二叉树序列化成一个字符串并能将该字符串反序列化出原来二叉树的算法。

### 参考代码

- 使用前序遍历，这样得到的序列化首位是根节点
- 用#代替null，这样不会导致两个树的序列化相同

```
     6
   /   \
  6     6
 / \   
6   6 
序列化：6,6,6,#,#,6,#,#,6,#,#
     6
   /   \
  6     6
       / \   
      6   6 
序列化：6,6,#,#,6,6,#,#,6,#,#
```



``` java
public String serialize(TreeNode root) {
    if (root == null) {
        return "#";
    }
 
    String leftStr = serialize(root.left);
    String rightStr = serialize(root.right);
    // 将当前节点、左树、右树的序列化字符串拼接
    return String.valueOf(root.val) + "," + leftStr + "," + rightStr;
}

// 反序列化
public TreeNode deserialize(String data) {
    // 将逗号去掉
    String[] nodeStrs = data.split(",");
    // i[0]可以随着递归传递
    int[] i = {0};
    return dfs(nodeStrs, i);
}

private TreeNode dfs(String[] strs, int[] i) {
    String str = strs[i[0]];
    i[0]++;

    if (str.equals("#")) {
        return null;
    }

    TreeNode node = new TreeNode(Integer.valueOf(str));
    node.left = dfs(strs, i);
    node.right = dfs(strs, i);
    return node;
}
```



## 面试题49：从根结点到叶结点的路径数字之和

### 题目
在一个二叉树里所有结点都在0-9的范围之类，从根结点到叶结点的路径表示一个数字。求二叉树里所有路径表示的数字之和。

例如在图中的二叉树有三条从根结点到叶结点的路径，它们分别表示数字395、391和302，这三个数字之和是1088。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0804.png" alt="图2.1">

### 参考代码

#### 方法一

``` java
public int sumNumbers(TreeNode root) {
    return dfs(root, 0);      
}

private int dfs(TreeNode root, int path) {
    // 到了一个不是叶子节点的地方，说明路径不成立
    if (root == null) {
        return 0;
    }

    path = path * 10 + root.val;
    if (root.left == null && root.right == null) {
        return path;
    }
	// 因为题目要求的是和，所以加在一起
    return dfs(root.left, path) + dfs(root.right, path);
}
```

#### 方法二

```java
int ans = 0;
public int sumNumbers(TreeNode root) {
    dfs(root, 0);
    return ans;
}

private void dfs(TreeNode node, int val){
    // 计算当前层的数
    int tmp = val*10 + node.val;
    // 遇见叶子节点则统计和
    if(node.left == null && node.right == null){
        ans += tmp;
        return;
    }
    // 两个判断!null可以保证到达的节点就是叶子节点
    if(node.left != null) dfs(node.left, tmp);
    if(node.right != null) dfs(node.right, tmp);
}
```





## 面试题50：向下的路径结点之和

### 题目
给定一个二叉树和一个值sum，求二叉树里结点值之和等于sum的路径的数目。

路径的定义为二叉树中沿着指向子结点的指针向下移动所经过的结点，但不一定从根结点开始，也不一定到叶结点结束。

例如，在图中的二叉树里，有两个路径的结点值之和等于8，其中第一条路径从结点5开始经过结点2到达结点1，第二条路径从结点2开始到结点6。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0805.png" alt="图2.1">



### 参考代码

map 的 key: 累加的节点值之和，value: 每个节点值之和出现的次数

path：从根节点开始到当前节点的路径和

若 map 中有 path - sum，

记 path - sum 长度的最后节点为 a，当前节点为 b

则 a -> b 就是一条所求的路径

例如：求长6，526长13，52长7，13 - 7 = 6

> 每遍历一个节点，先处理当前节点，此为前序遍历

``` java
public int pathSum(TreeNode root, int sum) {
    Map<Integer, Integer> map = new HashMap<>();
    map.put(0, 1);

    return dfs(root, sum, map, 0);
}

private int dfs(TreeNode root, int sum,
    			Map<Integer, Integer> map, int path) {
    // null说明路径结束了，返回
    if (root == null) {
        return 0;
    }

    path += root.val;
    int count = map.getOrDefault(path - sum, 0);
    map.put(path, map.getOrDefault(path, 0) + 1);

    count += dfs(root.left, sum, map, path);
    count += dfs(root.right, sum, map, path);

    // 返回上一个节点之前，需要将当前节点的路径和从map中删除
    map.put(path, map.get(path) - 1);

    return count;
}
```

## 面试题51：结点之和最大的路径
### 题目
在二叉树中定义路径为从沿着结点间的连接从任意一个结点开始到达任意一个结点所经过的所有结点。

路径中至少包含一个结点，不一定经过二叉树的根结点，也不一定经过叶结点。给你非空的一个二叉树，请求出二叉树所有路径上结点值之和的最大值。

例如在图中的二叉树中，从结点15开始经过结点20到达结点7的路径是结点值之和为42，是结点值之和最大的路径。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0806.png" alt="图2.1">

### 思路

当路径到达某个节点时，既可以前往左节点，又可以前往右节点；但是若同时经过左右节点，则不能经过其父节点，`先不累加，先不计算当前节点`

例如：经过15，20，7后，不能经过9

求出左右子树中路径节点值之和的最大值（左右子树中的路径不经过当前节点）

再求出经过根节点的路径节点的最大值，最后三者进行比较

因为最后处理根节点，所以使用后序遍历

```
  a
 / \
b   c
b -> a 直径
b -> a <- c 曲径
```



### 参考代码

#### 方法一

``` java
public int maxPathSum(TreeNode root) {
    int[] maxSum = {Integer.MIN_VALUE};
    dfs(root, maxSum);
    return maxSum[0];
}
// 返回值是经过当前节点root的直径
private int dfs(TreeNode root, int[] maxSum) {
    if (root == null) {
        return 0;
    }

    int[] maxSumLeft = {Integer.MIN_VALUE};
    int left = Math.max(0, dfs(root.left, maxSumLeft));
	// 计算分支，分支为负数的话不如不选
    
    int[] maxSumRight = {Integer.MIN_VALUE};
    int right = Math.max(0, dfs(root.right, maxSumRight));

    maxSum[0] = Math.max(maxSumLeft[0], maxSumRight[0]);
    maxSum[0] = Math.max(maxSum[0], root.val + left + right);
    // root.val + left + right是曲径

    // 因为返回后需要和根节点合并，所以不能同时经过左右，只取其最大值
    // 向上提供的一定得是直径
    return root.val + Math.max(left, right);
}
````
#### 方法二：方法一的精简版

思路一样，效率也一样

```java
int maxPath = Integer.MIN_VALUE;
public int maxPathSum(TreeNode root) {
    dfs(root);
    return maxPath;
}

private int dfs(TreeNode node) {
    if (node == null) return 0;

    // 计算分支，分支为负数的话不如不选
    int left = Math.max(0, dfs(node.left));
    int right = Math.max(0, dfs(node.right));
	// 内部最大路径肯定是要走当前节点的
    // 只要左右节点为正，肯定会选，所以肯定是曲径
    // 不选的时候，才为直径
    maxPath = Math.max(maxPath, node.val + left + right);
	
    // 因为返回后需要和根节点合并，所以不能同时经过左右，只取其最大值
    // 向上提供的一定得是直径
    return node.val + Math.max(left, right);
    // 换成下面这个后，效率也能跟方法三一样
    // return Math.max(node.val, node.val + Math.max(left, right));
}
```

#### 方法三

效率高于前两者

```java
int maxPath = Integer.MIN_VALUE;
public int maxPathSum(TreeNode root) {
    dfs(root);
    return maxPath;
}
// 返回值是经过当前节点root的直径
public int dfs(TreeNode node) {
    if (node == null) {
        return 0;
    }
    int left = dfs(node.left);
    int right = dfs(node.right);

    int ret = Math.max(node.val, node.val + Math.max(left, right));

    maxPath = Math.max(maxPath, Math.max(ret, node.val + left + right));
    return ret;
}

```



## 8.2 二叉搜索树

`left <= root <= right`

AVL是自平衡的二叉搜索树

## 补：验证二叉搜索树

#### 方法一：中序遍历

中序遍历二叉搜索树的结果就是递增的，如果不是二叉搜索树，则会出现当前值比前一个的值小的情况

#### 方法二：

```java
// 上下界，左子树的所有节点都必须比当前节点小
public boolean isValidBST(TreeNode root) {
    long lower = Long.MIN_VALUE;
    long upper = Long.MAX_VALUE;
    return dfs(root, lower, upper);
}
private boolean dfs(TreeNode root, long lower, long upper) {
    if (root == null) return true;
    if (root.val >= upper || root.val <= lower) return false;
    return dfs(root.left, lower, root.val) && dfs(root.right, root.val, upper);
}
```



## 面试题52：展平二叉搜索树

### 题目
给你一个二叉搜索树，请调整结点的指针使得每个结点都没有左子结点看起来像一个链表，但新的树仍然是二叉搜索树。例如把图8.8（a）中的二叉搜索树按照这个规则展平之后的结果如图8.8（b）所示。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0808.png" alt="图2.1">

图8.8：把二叉搜索树展平成链表。（a）一个有6个结点的二叉树。（b）展平成看起来是链表的二叉搜索树，每个结点都没有左子结点。

### 参考代码
``` java
public TreeNode increasingBST(TreeNode root) {
    Stack<TreeNode> stack = new Stack<>();
    TreeNode cur = root;
    TreeNode prev = null;
    TreeNode head = null;
    while (cur != null || !stack.isEmpty()) {
        while (cur != null) {
            stack.push(cur);
            cur = cur.left;
        }

        cur = stack.pop();
        if (prev != null) {
            prev.right = cur;
        } else {
            head = cur;
        }

        prev = cur;
        cur.left = null;
        cur = cur.right;
    }

    return head;
}
```

> 如果是前序展平普通二叉树，寻常遍历方法不可行
>
> 会在读取左节点的时候丢失右节点的信息

## 前序展平二叉树

```java
// 提前将右节点保存在栈中
public void flatten(TreeNode root) {
    Stack<TreeNode> st = new Stack<>();
    if (root == null) {
        return;
    }
    st.push(root);
    TreeNode prev = null;
    TreeNode cur = null;
    while (!st.isEmpty()) {
        cur = st.pop();
        // sout
        ///// 下方删除即为普通前序遍历 /////
        if (prev != null) {
            prev.left = null;
            prev.right = cur;
        }
        ////////////////////////////////
        if (cur.right != null) 
            st.push(cur.right);
        if (cur.left != null) 
            st.push(cur.left);
        ///// 下方删除即为普通前序遍历 /////
        prev = cur;
    }
}
```

- 前序遍历的倒置算法

```java
private TreeNode pre = null;

public void flatten(TreeNode root) {
    if (root == null)
        return;
    flatten(root.right);
    flatten(root.left);
    root.right = pre;
    root.left = null;
    pre = root;
}
```



- 空间复杂度`O(1)`

```
    1
   / \
  2   5
 / \   \
3   4   6

//将 1 的左子树插入到右子树的地方
    1
     \
      2         5
     / \         \
    3   4         6        
//将原来的右子树接到左子树的最右边节点
    1
     \
      2          
     / \          
    3   4  
         \
          5
           \
            6
            
 //将 2 的左子树插入到右子树的地方
    1
     \
      2          
       \          
        3       4  
                 \
                  5
                   \
                    6   
        
 //将原来的右子树接到左子树的最右边节点
    1
     \
      2          
       \          
        3      
         \
          4  
           \
            5
             \
              6         
  
  ......

```




```java
public void flatten(TreeNode root) {
    TreeNode cur = root;
    while (cur != null) { 
        //左子树为 null，直接考虑下一个节点
        if (cur.left == null) {
            cur = cur.right;
        } else {
            // 找左子树最右边的节点
            TreeNode pre = cur.left;
            while (pre.right != null) {
                pre = pre.right;
            } 
            //将原来的右子树接到左子树的最右边节点
            pre.right = cur.right;
            // 将左子树插入到右子树的地方
            cur.right = cur.left;
            cur.left = null;
            // 考虑下一个节点
            cur = cur.right;
        }
    }
}
```



## 面试题53：二叉搜索树的下一个结点

### 题目
给你一个二叉搜索时和它的一个结点p，请找出按中序遍历的顺序该结点p的下一个结点。假设二叉搜索树中结点的值都是唯一的。

例如在下图二叉搜索树中，结点8的下一个结点是结点9，结点11的下一个结点是null。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0809.png" alt="图2.1">



### 参考代码
#### 解法一

时间复杂度`O(n)`

中序遍历，并初始化标记 found 为 false，

发现节点 p 后，将 found 置 true，下一个就是目标点

``` java
public TreeNode inorderSuccessor(TreeNode root, TreeNode p) {
    Stack<TreeNode> stack = new Stack<>();
    TreeNode cur = root;
    boolean found = false;
    while (cur != null || !stack.isEmpty()) {
        while (cur != null) {
            stack.push(cur);
            cur = cur.left;
        }

        cur = stack.pop();
        if (found) {
            break;
        } else if (p == cur) {
            found = true;
        }

        cur = cur.right;
    }

    return cur;
}
```

#### 解法二

时间复杂度`O(h)`，h 为二叉树的深度

**p 的下一个节点一定是大于等于 p 的所有节点中最小的那个**

寻找二叉搜索树中，比 p 小的，每到一个根节点，比较和 p 的大小

若比 p 小等，那么下一个在其右子树中

若比 p 大，那么当前节点可能会是下一个节点

接下来前往当前节点的左子树，查看是否还有比 p 大但比当前节点小的，

（左子树比当前节点小）

``` java
public TreeNode inorderSuccessor(TreeNode root, TreeNode p) {
    TreeNode cur = root;
    TreeNode result = null;
    while (cur != null) {
        if (cur.val > p.val) {
            result = cur;
            cur = cur.left;
        } else {
            cur = cur.right;
        }
    }

    return result;
}
```



## 面试题54：所有大于等于结点的值之和

### 题目
给你一个二叉搜索树，请将它的每个结点的值替换成树中大于或者等于该结点值的所有结点值之和。假设二叉搜索树中结点的值唯一。例如，输入图8.10（a）中的二叉搜索树，由于有两个结点的值大于或者等于6（即结点6和结点7），因此值为6结点的值替换成13，其他结点的值的替换过程类似，所有结点的值替换之后的结果如图8.10（b）所示。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0810.png" alt="图2.1">

图8.10：把二叉搜索树中每个结点的值替换成树中大于或者等于该结点值的所有结点值之和。（a）一个二叉搜索树。（b）替换之后的二叉树。

### 参考代码
``` java
public TreeNode convertBST(TreeNode root) {
    Stack<TreeNode> stack = new Stack<>();
    TreeNode cur = root;
    int sum = 0;
    while (cur != null || !stack.isEmpty()) {
        while (cur != null) {
            stack.push(cur);
            cur = cur.right;
        }

        cur = stack.pop();
        sum += cur.val;
        cur.val = sum;
        cur = cur.left;
    }

    return root;
}
```

## 面试题55：二叉搜索树迭代器
### 题目
请实现二叉搜索树的迭代器BSTIterator，它主要有如下三个函数：
+ 构造函数输入一个二叉搜索树的根结点初始化该迭代器；
+ 函数next返回二叉搜索树中下一个最小的结点的值；
+ 函数hasNext返回二叉搜索树是否还有下一个结点。

例如输入图8.11中的二叉树搜索树初始化BSTIterator，第一次调用函数next将返回最小的结点值1，此时调用函数hasNext返回true。再次调用函数next将返回下一个最小的结点的值2，此时再调用函数hasNext将返回true。第三次调用函数next将返回下一个最小的结点的值3，如果此时再调用函数hasNext将返回false。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0811.png" alt="图2.1">

图8.11：一个有3个结点的二叉搜索树。

### 参考代码

迭代法本身就是迭代器

``` java
public class BSTIterator {
    TreeNode cur;
    Stack<TreeNode> stack;

    public BSTIterator(TreeNode root) {
        cur = root;
        stack = new Stack<>();
    }

    public boolean hasNext() {
        return cur != null || !stack.isEmpty();
    }

    public int next() {
        while (cur != null) {
            stack.push(cur);
            cur = cur.left;
        }
        
        cur = stack.pop();
        int val = cur.val;
        cur = cur.right;
        
        return val;
    }
}
```

## 面试题56：二叉搜索树中两个结点之和
### 题目
给你一个二叉搜索树和一个值k，请判断该二叉搜索树中是否存在两个结点它们的值之和等于k。假设二叉搜索树中结点的值均唯一。

例如在图8.12中的二叉搜索树里，存在两个两个结点它们的和等于12（结点5和结点7），但不存在两个结点值之和为22的结点。 

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0812.png" alt="图2.1">



### 参考代码

### 法一：哈希表

```java
public boolean findTarget(TreeNode root, int k) {
    Set<Integer> set = new HashSet<>();
    Stack<TreeNode> stack = new Stack<>();
    TreeNode cur = root;
    while (cur != null || !stack.isEmpty()) {
        while(cur != null) {
            stack.push(cur);
            cur = cur.left;
        }
        cur = stack.pop();
        
        if (set.contains(k - cur.val)) {
            return true;
        }
        set.add(cur.val);
        
        cur = cur.right;
    }
    return false;
}
```



### 法二：迭代器、双指针

`O(h)`

``` java
public class BSTIterator {
    TreeNode cur;
    Stack<TreeNode> stack;

    public BSTIterator(TreeNode root) {
        cur = root;
        stack = new Stack<>();
    }

    public boolean hasNext() {
        return cur != null || !stack.isEmpty();
    }

    public int next() {
        while (cur != null) {
            stack.push(cur);
            cur = cur.left;
        }
        
        cur = stack.pop();
        int val = cur.val;
        cur = cur.right;
        
        return val;
    }
}

public class BSTIteratorReversed {
    TreeNode cur;
    Stack<TreeNode> stack;

    public BSTIteratorReversed(TreeNode root) {
        cur = root;
        stack = new Stack<>();
    }

    public boolean hasPrev() {
        return cur != null || !stack.isEmpty();
    }

    public int prev() {
        while (cur != null) {
            stack.push(cur);
            cur = cur.right;
        }
        
        cur = stack.pop();
        int val = cur.val;
        cur = cur.left;
        
        return val;
    }
}

public boolean findTarget(TreeNode root, int k) {
    if (root == null) {
        return false;
    }

    BSTIterator iterNext = new BSTIterator(root);
    BSTIteratorReversed iterPrev = new BSTIteratorReversed(root);
    int next = iterNext.next();
    int prev = iterPrev.prev();
    while (next != prev) {
        if (next + prev == k) {
            return true;
        }

        if (next + prev < k) {
            next = iterNext.next();
        } else {
            prev = iterPrev.prev();
        }
    }

    return false;
}
```



## 8.3 TreeSet和TreeMap的应用

利用红黑树这种平衡二叉搜索树实现 TreeSet 和 TreeMap

红黑树：

- 根节点黑色
- NIL黑色
- 一个节点红则其子节点必黑
- 从一个节点到该节点子孙节点的所有路径上包含相同个数的黑节点

```
TreeSet常用函数

ceiling
返回键大于等于给定值的最小键；
如果没有，返回null

floor
返回键小于等于给定值的最大键；
如果没有，返回null

higher
返回键大于给定值的最小键；
如果没有，返回null

lower
返回键小于给定值的最大键；
如果没有，返回null
```



```
TreeMap常用函数

ceilingEntry/ceilingKey
返回键大于等于给定值的最小映射/键；
如果没有，返回null

floorEntry/floorKey
返回键小于等于给定值的最大映射/键；
如果没有，返回null

higherEntry/higherKey
返回键大于给定值的最小映射/键；
如果没有，返回null

lowerEntry/lowerKey
返回键小于给定值的最大映射/键；
如果没有，返回null
```



> 如果数据集合是动态的，即题目要求逐步在数据集合中添加更多的数据，并且需要根据数据大小实现快速查找，则可能需要用到



## 面试题57：值和下标之差都在给定的范围内

### 题目
给你一个整数数组`nums`，请判断是否存在两个不同的下标`i`和`j`（i和j之差的绝对值不大于给定的k）使得两个数值`nums[i]`和`nums[j]`的差的绝对值不大于给定的t。

例如，如果输入数组{1, 2, 3, 1}，k为3，t为0，由于下标0和下标3，它们对应的数字之差的绝对值为0，因此返回true。如果输入数组{1, 5, 9, 1, 5, 9}，k为2，t为3，由于不存在两个下标之差小于等于2的数字它们差的绝对值小于等于3，此时应该返回false。

### 参考代码
#### 解法一：TreeSet

`o(nlogk)`

``` java
public boolean containsNearbyAlmostDuplicate(int[] nums,
    int k, int t) {
    // long是怕溢出
    TreeSet<Long> set = new TreeSet<>();
    // set里只存k个数
    // 那么只需要判断里面有没有数值差满足条件的就可以了
    for (int i = 0; i < nums.length; ++i) {
        // |[i]-[?]| <= t
        // floor是小于x的max
        // lower是小于等于[i]的最大数字
        // [i]-[?] <= t
        Long lower = set.floor((long)nums[i]);
        if (lower != null && lower >= (long)nums[i] - t) {
            return true;
        }
		// upper是大于等于[i]的最小数字
        // [?]-[i] <= t
        Long upper = set.ceiling((long)nums[i]);
        if (upper != null && upper <= (long)nums[i] + t) {
            return true;
        }

        set.add((long)nums[i]);
        if (i >= k) {
            set.remove((long)nums[i - k]);
        }
    }

    return false;
}
```



#### 解法二：TreeMap

`O(n)`

``` java
// 还是k个k个的找
// 将数放入若干大小为t+1的桶中，0~t放入0号桶
// 如果两个数在同一个桶内，则差值小于等于t
// 遍历数字num，放入id桶
// 1、如果桶中有数字，则找到
// 2、桶中无数字，则判断id-1桶和id+1桶中是否存在差值小于等于t的
public boolean containsNearbyAlmostDuplicate(int[] nums, int k, int t) {
   // 桶：key-id,val-num
    Map<Integer, Integer> buckets = new HashMap<>();
    int bucketSize = t + 1;
    for (int i = 0; i < nums.length; i++) {
        int num = nums[i];
        int id = getBucketID(num, bucketSize);
		
        if (buckets.containsKey(id)
            || (buckets.containsKey(id-1) && buckets.get(id-1) + t >= num)
            || (buckets.containsKey(id+1) && buckets.get(id+1) - t <= num)) {
            return true;
        }

        buckets.put(id, num);
        if (i >= k) {
            buckets.remove(getBucketID(nums[i - k], bucketSize));
        }
    }

    return false;
}

// 用来确定数字该放入的桶的编号
private int getBucketID(int num, int bucketSize) {
    return num >= 0
        ? num / bucketSize
        : (num + 1) / bucketSize - 1; 
}

```

## 面试题58：日程表
### 题目
请实现一个类型MyCalendar用来记录你的日程安排，该类型有一个方法book(int start, int end)往日程安排表里添加一个时间区域为[start, end)的事项（这是个半开半闭区间，即start<=x<end）。如果[start, end)内没有事先安排其他事项，则成功添加该事项并返回true。否则不能添加该事项，并返回false。

例如，在下面的三次调用book方法中，第二次调用返回false，这是因为时间[15, 20)已经被第一次调用预留了。由于第一次占用的时间是个半开半闭区间，并没有真正占用时间20，因此不影响第三次调用预留时间区间[20, 30)。 
``` java
MyCalendar cal = new MyCalendar()；
cal.book(10, 20); // returns true
cal.book(15, 25); // returns false
cal.book(20, 30); // returns true
```

### 参考代码
``` java
class MyCalendar {
    private TreeMap<Integer, Integer> events;
    
    public MyCalendar() {
        events = new TreeMap<>();
    }
    
    public boolean book(int start, int end) {
        Map.Entry<Integer, Integer> event = events.floorEntry(start);
        if (event != null && event.getValue() > start) {
            return false;
        }
        
        event = events.ceilingEntry(start);
        if (event != null && event.getKey() < end) {
            return false;
        }
        
        events.put(start, end);
        return true;
    }
}
```

