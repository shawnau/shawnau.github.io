# Binary Tree Traverse

![](http://my-imgshare.oss-cn-shenzhen.aliyuncs.com/50592758_p0.jpg)

茴字有四种写法, 二叉树有三种遍历手段(递归, 栈, morris遍历)

<!--more-->

### 先序遍历
1. root进栈
2. 栈非空:
    1. 栈顶元素(父节点)出栈并处理
    2. 右子节点, 左子节点进栈, 这样下一轮会先处理左子节点, 符合父->左->右的顺序

```c++
vector<int> preorderTraversal(TreeNode *root) {
    vector<int> result;
    stack<TreeNode*> s;
    if(root == NULL) return result;
    s.push(root);
    while(!s.empty())
    {
        TreeNode* top= s.top();
        s.pop();
        result.push_back(top->val);
        if(top->right) s.push(top->right);
        if(top->left)  s.push(top->left);
    }
    return result;
}
```

### 后序遍历
为了按后序遍历的顺序处理每个节点, 在考察每个节点时需要先判断前一轮循环处理的节点, 用变量`last_pop`维护.

1. 是否处理完左右子树? 通过查看`last_pop`是否为当前节点的子节点判断. 因为如果已经处理了左/右子树, 他们都已出栈, 现在栈顶的元素会是他们的父节点. (左右皆可能, 因为不一定有右子节点, 故`last_pop`也可能是左子节点)
    - 若是, 处理当前节点. 出栈并打印, 更新`last_pop`
    - 若不是, 先处理左右子树, 即按右/左顺序压栈, 下一轮循环就会先处理左子节点(栈顶)
2. 是否到达叶子节点? 若到达, 出栈并打印, 更新`last_pop`

#### 变量解释与实现:
`cur`: 当前遍历指针, 初始状态指向root
`last_pop`: 上次处理的节点
`finishedSubtrees`: 是否处理完左右子树
`isLeaf`: 是否到达叶子节点
```c++
vector<int> postorderTraversal(TreeNode* root) {
    if (root == NULL) return result;
    stack<TreeNode*> s;
    vector<int> result;  
    
    TreeNode* last_pop = root;
    s.push(root);        
    while (!s.empty()) {
        TreeNode* cur = s.top();
        bool finishedSubtrees = (cur->right == last_pop || cur->left == last_pop);
        bool isLeaf = (cur->left == NULL && cur->right == NULL);
        
        if (finishedSubtrees || isLeaf) {
            s.pop();
            result.push_back(cur->val);
            last_pop = cur;
        } else {
            if (cur->right) s.push(cur->right);
            if (cur->left)  s.push(cur->left);
        }
    }
    return result;
}
```

#### 例子
```bash
       0
      / \
     1   2
    /   / \
   3   4   5
```

|last_pop|cur|finishedSubtrees|isLeaf|op|stack|
|-|-|:-:|:-:|-|-|
|0|0|F|F|init|0|
|0|0|F|F|br2: push 2,1|021|
|0|1|F|F|br2: push 3|0213|
|0|3|F|T|br1: pop 3|021|
|3|1|T|F|br1: pop 1|02|
|1|2|F|F|br2: push 5,4|0254|
|1|4|F|T|br1: pop 4|025|
|4|5|F|T|br1: pop 5|02|
|5|2|T|F|br1: pop 2|0|
|2|0|T|F|br1: pop 0||

`br`代表进入了哪个判断分支.
Result: `3, 1, 4, 5, 2, 0`


### 中序遍历
同后序遍历, 只不过处理顺序需要稍微改一改, 遍历(压栈)顺序依然维持不变

1. 是否处理完左子树? 通过查看`last_pop`是否为当前节点的左子节点判断. 若无左子节点也一并视为已处理完
    - 若是, 处理当前节点. 出栈并打印, 更新`last_pop`, 并将右子节点压栈, 等待下一轮处理
    - 若不是, 先处理左子树, 即左节点压栈, 等待下一轮处理
2. 是否到达叶子节点? 若到达, 出栈并打印, 更新`last_pop`

#### 变量解释与实现
`cur`: 当前遍历指针, 初始状态指向root
`last_pop`: 上次处理的节点
`finishedLeftTrees`: 是否处理完左子树
`isLeaf`: 是否到达叶子节点
```c++
vector<int> inorderTraversal1(TreeNode* root) {
    vector<int> result;
    stack<TreeNode*> s;
    if(root == NULL) return result;
    
    TreeNode* last_pop = root;
    s.push(root);
    while (!s.empty()) {
        TreeNode* cur = s.top();
        bool finishedLeftTrees = (cur->left == NULL || cur->left == last_pop);
        bool isLeaf = (cur->left == NULL && cur->right == NULL);
        
        if (finishedLeftTrees || isLeaf) {
            s.pop();
            result.push_back(cur->val);
            last_pop = cur;
            if (cur->right) s.push(cur->right);
        } else {
            if (cur->left)  s.push(cur->left);
        }
    }
    return result;
}
```

### 线索树的构造

利用已存在的空节点保存前驱/后继节点, 这方面的文章很多, 中文资料参考
https://www.cnblogs.com/AnnieKim/archive/2013/06/15/MorrisTraversal.html
