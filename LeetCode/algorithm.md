# 红黑树
红黑树是一颗自平衡的二叉搜索树。
* 二叉搜索树
二叉搜索树又叫二叉查找树或二叉排序树，首先它是一个二叉树，且必须满足下面的条件：
** 若左子树不空，则左子树上所有结点的值均小于它的根节点的值
** 若右子树不空，则右子树上所有结点的值均大于它的根节点的值
** 左、右子树也分别为二叉搜索树
![](https://upload-images.jianshu.io/upload_images/4314397-9588c432805a9570.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300/format/webp)  
* 平衡二叉树
二叉搜索树解决了很多问题，比如快速查找最大值和最小值，可以快速找到排名第几位的值，快速搜索和排序等。但普通的二叉搜索树有可能出现极不平衡的情况(斜树)，这样时间复杂度就有可能退化成O(N)的情况。
![](https://upload-images.jianshu.io/upload_images/4314397-2f1facc1bcfc501c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/709/format/webp)  
平衡二叉搜索树又叫AVL树(有别于AVL算法)，其满足下面条件：
** 它是一颗空树或**它的左右两个子树的高度差的绝对值不超过1**
** 左、右子树都是一棵平衡二叉树。
如果二叉树的高度超过1，则把该树调整为一棵平衡二叉树，这涉及到左旋、右旋、先右旋再左旋、先左旋再右旋、
### 右旋：
![](https://upload-images.jianshu.io/upload_images/4314397-318ca114772ad3fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/986/format/webp) 
### 左旋
![](https://upload-images.jianshu.io/upload_images/4314397-f06bb1d69571eaf0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/944/format/webp)  
### 先右旋再左旋
![](https://upload-images.jianshu.io/upload_images/4314397-d86683fad715ce89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)
### 先左旋再右旋
![](https://upload-images.jianshu.io/upload_images/4314397-fc689643c778438f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

# 感谢
[数据结构算法-红黑树](https://www.jianshu.com/p/1bbb19156454)
