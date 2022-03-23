### ACM模式

[多行输入](####多行输入)
[数组输入](####数组输入)
[链表输入](####链表输入)
[树的输入](####树的输入)

```java
Scanner sc=new Scanner(System.in);
System.out.println("在下面一行中以空格为间隙输入元素:");
//得出的是string类型
String[] str = sc.nextLine().split(" ");
//将String类型数组转成Integer类型数组。.parseInt()是基本类型
int[] nums=new int[str.length];
for (int i = 0; i < nums.length; i++) {
	nums[i]=Integer.valueOf(str[i]);
}


int N = s.nextInt();
```




#### 多行输入

```java
输入：
5 10 9
0 5
9 1
8 1
0 1
9 100
    
//Scanner类默认的分隔符就是空格
Scanner sc = new Scanner(System.in);
int n = sc.nextInt();// 5
int full = sc.nextInt();// 10
int avg = sc.nextInt();// 9
int[][] nums = new int[n][2];
for(int i = 0; i < n; i++){
    nums[i][0] = sc.nextInt();
    nums[i][1] = sc.nextInt();
}
```



#### 数组输入

```java
// 一维数组
输入:
7 15
15 5 3 7 9 14 0
    
int n = sc.nextInt();
int l = sc.nextLong();
int[] nums = new int[n];
for(int i = 0; i < n; i++){
    nums[i] = sc.nextInt();
}
```



如果要求的测试用例需要读取二维数组，我们应该先读取二维数组的长度和宽度存在两个整数中。在下一行将一串整型数字存入二维数组中。
注意：为了换行读取可以使用`nextLine()`跳过换行;

```java
// 二维数组
Scanner cin = new Scanner(System.in);
while(cin.hasNext()) {
    int r = cin.nextInt();
    int c = cin.nextInt();
    int[][] matrix = new int[r][c];
    cin.nextLine(); // 跳过行数和列数后的换行符
    for(int i=0;i<r;i++) {
        for (int j = 0; j < c; j++) {
            matrix[i][j] = cin.nextInt();
        }
    }
}
```



#### 链表输入

```java
import java.util.Scanner;
import java.util.Stack;

public class LinkListInput {
    //题目描述
    //对于一个链表 L: L0→L1→…→Ln-1→Ln,
    //将其翻转成 L0→Ln→L1→Ln-1→L2→Ln-2→…

    //先构建一个节点类，用于链表构建
    static class LinkNode {
        int val;
        LinkNode next;
        public LinkNode(int val){
            this.val = val;
        }
    }

    public static void main(String[] args){
        //输入是一串数字，请将其转换成单链表格式之后，再进行操作
        //输入描述:
        //一串数字，用逗号分隔
        //输入
        //1,2,3,4,5
        Scanner scanner = new Scanner(System.in);
        //以字符串形式作为输入
        // String str = scanner.next().toString();
        //通过分隔符将其转为字符串数组
        // String[] arr  = str.split(",");
        String[] arr  = scanner.next().split(",");
        //初始化一个整数数组
        int[] ints = new int[arr.length];
        //给整数数组赋值
        for(int j = 0; j<ints.length;j++) {
            ints[j] = Integer.parseInt(arr[j]);
        }
        
        Stack<LinkNode> stack = new Stack<>();
        
        LinkNode dummy = new LinkNode(0);
        LinkNode p = dummy;
        //链表初始化并放入stack中
        for(int i = 0; i < ints.length; i++){
            p.next = new LinkNode(ints[i]);
            p = p.next;
            // 本题特有
            stack.add(p);
        }
        LinkNode head = dummy.next;
        
        //开始链表转换
        p = head;
        LinkNode q = stack.peek();
        while ((!p.equals(q)) && (!p.next.equals(q))) {
            q = stack.pop();
            q.next = p.next;
            p.next = q;
            p = p.next.next;
            q = stack.peek();
        }
        q.next = null;
        
        //输出
        //1,5,2,4,3
        //打印
        while (head != null) {
            if(head.next == null){
                System.out.print(head.val);
            }else{
                System.out.print(head.val + ",");
            }
            head = head.next;
        }
    }
}
```



#### 树的输入

```java
// 二叉搜索树为例
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Stack;

//题目描述
//给定一个二叉树，判断其是否是一个有效的二叉搜索树。
//假设一个二叉搜索树具有如下特征：
//节点的左子树只包含小于当前节点的数。
//节点的右子树只包含大于当前节点的数。
//所有左子树和右子树自身必须也是二叉搜索树。
//例如：
//输入：
//    5
//   / \
//  1   3
//     / \
//    4   6
//输出: false

//构造树需要的结点类
class TreeNode {
    TreeNode left, right;
    int val;

    public TreeNode(int val) {
        this.val = val;
    }
}

public class TreeInput {
    public static void main(String[] args) throws IOException {
        //输入描述:
        //第一行两个数n,root，分别表示二叉树有n个节点，第root个节点时二叉树的根
        //接下来共n行，第i行三个数val_i,left_i,right_i，
        //分别表示第i个节点的值val是val_i,左儿子left是第left_i个节点，右儿子right是第right_i个节点。
        //节点0表示空。
        //1<=n<=100000,保证是合法的二叉树
        //输入
        //5 1
        //5 2 3
        //1 0 0
        //3 4 5
        //4 0 0
        //6 0 0
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        String[] s = reader.readLine().split(" ");
        int n = Integer.parseInt(s[0]);
        int root = Integer.parseInt(s[1]);
        TreeNode[] tree = new TreeNode[n + 1];
        int[][] leaf = new int[n + 1][2];
        for (int i = 1; i <= n; i++) {
            String[] ss = reader.readLine().split(" ");
            int val_i = Integer.parseInt(ss[0]);
            int left_i = Integer.parseInt(ss[1]);
            int right_i = Integer.parseInt(ss[2]);
            TreeNode node = new TreeNode(val_i);
            leaf[i][0] = left_i;
            leaf[i][1] = right_i;
            tree[i] = node;
        }
        for (int i = 1; i <= n; i++) {
            int left = leaf[i][0];
            if (left != 0) {
                tree[i].left = tree[left];
            } else {
                tree[i].left = null;
            }
            int right = leaf[i][1];
            if (right != 0) {
                tree[i].right = tree[right];
            } else {
                tree[i].right = null;
            }
        }
        TreeNode head = tree[root];
        boolean flag = isBinarySearchTree(head);
        System.out.println(flag);

    }

    private static boolean isBinarySearchTree(TreeNode node) {
        if(node == null){
            return true;
        }
        int pre = Integer.MIN_VALUE;
        Stack<TreeNode> s = new Stack<>();

        while(!s.isEmpty() || node != null){
            while(node != null){
                s.push(node);
                node = node.left;
            }
            node = s.pop();
            if(node == null){
                break;
            }
            if(pre > node.val){
                return false;
            }
            pre = node.val;
            node = node.right;
        }
        return true;
    }
}
```

