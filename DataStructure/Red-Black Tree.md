## 2-3-4树
2-3-4树是四阶的 B树(Balance Tree)，属于一种多路查找树，它的结构有以下限制：

所有叶子节点都拥有相同的深度。
节点只能是2-节点、3-节点4节点之一。
- ﻿2节点：包含1个元素的节点，有2个子节点：
- ﻿3节点：包含2个元素的节点，有3个子节点;
- ﻿4节点：包含3 个元素的节点，有4个子节点：
所有节点必须至少包含1个元素

元素始终保特排序顺序，整体上保持二叉查找树的性质，即父结点大于左子结点，小于右子结点。而且结点有多个元素时，每个元素必须大于它左边的和它的左子树中元素。

2-3-4树的查询操作像普通的二叉搜索树一样，非常简单，但由于其结点元素数不确定，在一些编程语言中实现起来并不方便，实现一般使用它的等同--红黑树

![[234树转红黑树.png]]
![[转换分裂的情况.png]]

红黑树，Red-Black Tree是一个自平衡(不是绝对的平衡)的二叉查找树，树上的每个节点都遵循下面的规则：
1. ﻿﻿每个节点要么是黑色，要么是红色。
2. ﻿﻿根节点是黑色，
3. ﻿﻿﻿每个叶子节点 (NIL) 是黑色。
4. ﻿﻿﻿每个红色结点的两个子结点一定都是黑色。
5. ﻿﻿﻿任意一结点到每个叶子结点的路径都包含数量相同的黑结点。

红黑树能自平衡，它靠的是什么？三种操作：左旋、右旋和变色

红黑树的新增
1.插入节点(可以看成是在普通的二叉树中插入）
 a. ﻿﻿找到插入的位置（父节点）
 b. ﻿﻿将节点插入父节点相应的位置
2.节点插入进去后调整

2节点插入不需要调整
3节点插入有6种情况，4种需要调整
4节点插入有4种情况，4种需要调整
![[增加的八种情况.png]]