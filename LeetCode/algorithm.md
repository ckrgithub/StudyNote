# 红黑树
红黑树是一颗自平衡的二叉搜索树。
* 二叉搜索树
二叉搜索树又叫二叉查找树或二叉排序树，首先它是一个二叉树，且必须满足下面的条件：
  * 若左子树不空，则左子树上所有结点的值均小于它的根节点的值
  * 若右子树不空，则右子树上所有结点的值均大于它的根节点的值
  * 左、右子树也分别为二叉搜索树  
![](https://upload-images.jianshu.io/upload_images/4314397-9588c432805a9570.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300/format/webp)  
* 平衡二叉树  
二叉搜索树解决了很多问题，比如快速查找最大值和最小值，可以快速找到排名第几位的值，快速搜索和排序等。但普通的二叉搜索树有可能出现极不平衡的情况(斜树)，这样时间复杂度就有可能退化成O(N)的情况。
![](https://upload-images.jianshu.io/upload_images/4314397-2f1facc1bcfc501c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/709/format/webp)  
平衡二叉搜索树又叫AVL树(有别于AVL算法)，其满足下面条件：
  * 它是一颗空树或**它的左右两个子树的高度差的绝对值不超过1**
  * 左、右子树都是一棵平衡二叉树。
如果二叉树的高度超过1，则把该树调整为一棵平衡二叉树，这涉及到左旋、右旋、先右旋再左旋、先左旋再右旋、
### 右旋：
![](https://upload-images.jianshu.io/upload_images/4314397-318ca114772ad3fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/986/format/webp) 
### 左旋
![](https://upload-images.jianshu.io/upload_images/4314397-f06bb1d69571eaf0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/944/format/webp)  
### 先右旋再左旋
![](https://upload-images.jianshu.io/upload_images/4314397-d86683fad715ce89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)
### 先左旋再右旋
![](https://upload-images.jianshu.io/upload_images/4314397-fc689643c778438f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)
## 红黑树
特性：
* 1.每个节点或者是黑色，或者是红色
* 2.根节点是黑色
* 3.每个叶子节点(NIL)是黑色。注意：这里叶子节点指为空(NIL或NULL)的叶子节点
* 4.如果一个节点是红色的，则它的字节点必须是黑色的
* 5.从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。  
![](https://upload-images.jianshu.io/upload_images/4314397-a0bff7addb21cfe8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/268/format/webp)  
假设我们现在要插入一个新的节点，如果插入的这个新节点为黑色，那么必然违反性质5，所以我们把新节点定义为红色。但如果插入的新节点为红色，就可能违反性质4，因此我们需要对其进行调整，使得整棵树依然满足红黑树的性质，也就是双红修正。
 * 1.如果没有出现双红现象，父亲是黑色的不需要修正
 * 2.叔叔是红色的，将叔叔和父亲染黑，然后爷爷染红
 * 3.叔叔是黑色的，父亲是爷爷的左节点，且当前节点是其父节点的右孩子，将"父节点"作为"新的当前节点"，以"新的当前节点"为支点进行左旋。然后将"父节点"设为"黑色"，将"祖父节点"设为"红色"，以"祖父节点"为支点进行右旋；
 * 4.叔叔是黑色的，父亲是爷爷的左节点，且当前节点是其父节点的左孩子，将"父节点"设置为"黑色"，将"祖父节点"设为"红色"，以"祖父节点"为支点进行右旋
 * 5.叔叔是黑色的，父亲是爷爷的右节点，其当前节点是其父节点的左孩子，将"父节点"作为"新的当前节点"，以"新的当前节点"为支点进行右旋。然后将"父节点"设为"黑色"，将"祖父节点"设为"红色"，以"祖父节点"为支点进行左旋
 * 6.叔叔是黑色的，父亲是爷爷的右节点，且当前节点是其父节点的右孩子，将"父节点"设为"黑色"，将"祖父节点"设为"红色"，以"祖父节点"为支点进行左旋  
 上面的双红修正现象看似比较复杂，但实际上只有三种情况，一种是没有双红现象，另一种是父亲和叔叔都是红色，最后一种是叔叔是黑色的。  
 ![](https://upload-images.jianshu.io/upload_images/4314397-fae62fda145e9e35.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/957/format/webp)  
 


















# 感谢
[数据结构算法-红黑树](https://www.jianshu.com/p/1bbb19156454)
