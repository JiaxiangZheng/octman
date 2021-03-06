---
layout: post
title: 二叉树非递归遍历
categories:
- 算法与基础
tags:
- 面试题
- 算法
- 二叉树
---

二叉树的主要遍历方式有四种，分别是前、中、后、层序遍历。其中前面三种方式定义本身就是一个递归的形式，因此很容易通过递归方式把它们实现出来，而层序遍历则需要用到一个辅助的队列不断地把一层的结点不断地添加到队列中。对于层序的方法，如果把二叉树看作有向图，那就是一个广度优先遍历算法了。本方主要是就前三种遍历的非递归方式进行分析。其中前序和中序基本相同，而后序非递归则是难度最大的。

### 前序/中序非递归-Morris算法

由于前序与中序类似，这里只以中序为例。对于每个结点，其中序遍历顺序的前一个应该是左子树的最右结点，如果我们可以在遍历的过程中从左子树最右结点（叶结点）遍历完后直接转到根结点，那么我们就可以使用线性的方式进行树的遍历了。如何将左子树的最右结点这个空指针利用起来呢？

1. 如果当前的结点没有左子树，那么显然访问当前结点并转入访问右子树；
2. 而如果左子树是存在的，那么找到左子树最右结点（看代码解释）：
    * 如果为空则表示是第一次访问到该叶结点，将该叶结点的右指针链接到当前结点并转到 当前结点的左子树进行处理
    * 而如果不为空，说明我们已经在之前通过上一步把它的最右结点穿到当前元素了，这时候说明左子树已经处理完毕，可以将最右结点的右指针重置为空然后处理当前结点，并转入右子树

举个如下的例子吧：

         3
       /  \
      2    6
     / \    \
    1   5    7
       /
      4

给定上面的二叉树（不是二叉搜索树）：

1. 当前结点为3，则我们知道它中序的前一个是左子树最右结点，因此先找到左子树最右结点，这里是5
2. 然后将5的右指针链接到3，然后当前元素转到了2
3. 对2同样进行处理，找到1并将1的右指针链接到2，然后当前元素转到了1
4. 因为1左子树为空，所以访问它并转入到右指针所指元素----这里因为前一步已经将它的右指针链接到了2，所以当前元素转入2
5. 因为通过2找到它的左子树最右结点的右指针为2本身，所以就知道肯定左子树已经处理完了，即将1的右指针恢复为空并访问2转到5
6. 同样进行遍历，假设我们现在遍历了5后，则5的右指针为3，即转到了3
7. 再次地发现3的左子树最右结点的右指针为3本身，所以设置5的右指针为空并访问3，然后转到6了，以此类推。

中序遍历如下：

    void inorder(BTNode* root) {
        if (!root) return;

        BTNode* cur = root;
        while (cur != NULL) {
            if (!cur->left) {
                visit(cur);
                cur = cur->right;
            } else {
                BTNode* pre = cur->left;
                while (pre->right != cur && pre->right != NULL) pre = pre->right;
                if (pre->right == cur) {
                    visit(cur);             //  如果是前序遍历，就将该句放在else里第一句
                    pre->right = NULL;
                    cur = cur->right;
                } else {
                    pre->right = cur;
                    cur = cur->left;
                }
            }
        }
        return;
    }

对于前序遍历，事实上我们需要修改的只是`visit(cur)`的位置，即先访问然后再转入左子树。

### 后序遍历非递归算法

由于后序遍历的时候需要先把左右子树访问完成以后才能够访问根结点，所以当根结点出现在栈顶的时候，不能立马访问它，还需要确定它的左右子树是否未被访问。因此，我们需要在遍历的过程中记录当前栈顶结点是否可以访问，如果不可以，那么还需要先把左右子树分别入栈。这意味着结点可能出现在栈顶两次，需要维护一个变量去判断当前栈顶结点是不是应该访问。

显然，如果当前结点左右子树均为空，那么应该直接访问并弹出；如果左右子树均已访问，那么也需要访问并弹出，因此维护一个变量记录当前访问的前一个结点，并且判断前一个结点是不是当前结点的孩子结点。然后就是正常的递归转非递归了。

后序遍历如下：

    void postorder(BTNode* root) {
        if (!root) return;
        stack<BTNode*> S;
        BTNode  *pre = NULL;
        S.push(root);
        while (!S.empty()) {
            cur = S.top();
            if (   (!cur->left && !cur->right) 
                || (!pre && (pre == cur->left || pre == cur->right)) 
               ){
                visit(cur);
                S.pop();
                pre = cur;
            } else {
                if (cur->right) {
                    S.push(cur->right);
                }
                if (cur->left) {
                    S.push(cur->left);
                }
            }
        }
        return;
    }

