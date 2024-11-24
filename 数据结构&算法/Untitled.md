## LCR 175. 计算二叉树的深度

> Problem: [LCR 175. 计算二叉树的深度](https://leetcode.cn/problems/er-cha-shu-de-shen-du-lcof/description/)

### 思路

> - 空
>
> 	空的情况下深度肯定为0
>
> - 非空
>
> 	非空则递归去找左子树和右子树深度最深的一个，则当前深度为最大值+1

### Code

```c++
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if (!root) return 0;
        return max(maxDepth(root->left), maxDepth(root->right)) + 1;
    }
};
```

---

## LCR 176. 判断是否为平衡二叉树

> Problem: [LCR 176. 判断是否为平衡二叉树](https://leetcode.cn/problems/ping-heng-er-cha-shu-lcof/description/)

### 思路

> - 空
>
> 	空树一定是平衡的
>
> - 非空
>
> 	当前子树要平衡，那么左右子树必须平衡，同时左右子树的最大深度差也不能超过1
>
> 这里就借助前面求最大深度的函数来求了

### Code

```c++
class Solution {
    int maxDepth(TreeNode* root) {
        if (!root) return 0;
        return max(maxDepth(root->left), maxDepth(root->right)) + 1;
    }
    
public:
    bool isBalanced(TreeNode* root) {
        if (!root) return true;
        return abs(maxDepth(root->left) - maxDepth(root->right)) <= 1 && isBalanced(root->left) && isBalanced(root->right);
    }
};
```

---

### 优化——自底向上递归

> 如果每次都对每个节点求一次最大深度，那么其实是非常浪费的，我们其实可以在求最大深度的过程中顺便将当前数是否平衡判断出来
>
> 因为子树如果不平衡则直接不平衡，也不用继续判断了。假如子树平衡此时我们又有两树的深度，就可以顺便判断两树的深度差了

#### Code

```c++
class Solution {
    int maxDepth(TreeNode* root) {
        if (!root) return 0;
        int l = maxDepth(root->left);
        int r = maxDepth(root->right);
        if (l == -1 || r == -1 || abs(l - r) > 1) return -1;
        return max(l, r) + 1;
    }
    
public:
    bool isBalanced(TreeNode* root) {
        return maxDepth(root) >= 0;
    }
};
```

---

## LCR 193. 二叉搜索树的最近公共祖先

> Problem: [LCR 193. 二叉搜索树的最近公共祖先](https://leetcode.cn/problems/er-cha-sou-suo-shu-de-zui-jin-gong-gong-zu-xian-lcof/description/)

### 思路

> 两个节点有可能一个是另一个的祖先，所以在递归的过程中一旦当前节点等于二者之一那么就直接返回，否则继续寻找
>
> 根据二叉搜索树的特性，我们按照大小就可以分出之后要查询的子树，按照路径一步步查下去，直到当前节点值介于p和q之间，那么他俩就不可能在同一颗子树中了，也就是说当前节点是最近公共祖先

### Code

```c++
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if (p->val > q->val) return lowestCommonAncestor(root, q, p);
        if (root == p || root == q) return root;
        if (root->val >= p->val && root->val <= q->val) return root;
        if (root->val > p->val) return lowestCommonAncestor(root->left, p, q);
        return lowestCommonAncestor(root->right, p, q);
    }
};
```

---

## LCR 194. 二叉树的最近公共祖先

> Problem: [LCR 194. 二叉树的最近公共祖先](https://leetcode.cn/problems/er-cha-shu-de-zui-jin-gong-gong-zu-xian-lcof/description/)

### 思路

> 主要思路是通过后序遍历递归的先去判断左右子树中有没有p或q节点，假如左右子树都有，那么当前节点一定是最近公共祖先，因为继续往下分就要分子树了。如果只有一边有或者两边都没有那么就是不行

### Code

```c++
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if (!root) return nullptr;
        if (root == p || root == q) return root;
        TreeNode* l = lowestCommonAncestor(root->left, p, q);
        TreeNode* r = lowestCommonAncestor(root->right, p, q);
        if (l && r) return root;
        return l ? l : r;
    }
};
```

