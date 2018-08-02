## 二叉查找树和平衡二叉树
+ B+树是通过二叉树，再由平衡二叉树，B树演化而来。
### 二叉查找树
+ 深度平均值O(logN)
+ 二叉树实际上是图
+ 使二叉树成为二叉查找树的性质：对于树中的每个节点X，它的左子树中所有关键字值小于X的关键字值，而他的右子树中所有关键字值大于X的关键字值。
+ 对二叉树进行中序遍历得到的就是一个有序的序列。
### 平衡二叉树(AVL树)
+ 前提是符合二叉查找树的定义，必须满足任何节点的两个子树的高度最大差为1。
+ 但是维护平衡二叉树的代价较大。
### B+树
+ B+树由B树和索引顺序访问方法演化而来。
+ B+树是为磁盘或其他直接存取辅助设备设计的一种平衡二叉树。在B+树，所有记录节点都是按照键值的大小顺序存放在同一层的叶子节点上，由个叶子节点指针进行连接。
+ B+树的插入操作
+ B+树的插入必须保证插入后叶子节点中的记录仍然有序，每种情况都可能会导致不同的插入算法。
+ B+树插入的3种情况：
<table border="2">
<tr>
    <th align="center">Leaf Page满</th>
    <th align="center">Index Page满</th>
    <th align="center">操作</th>
</tr>
<tr>
    <td align="center">No</td>
    <td align="center">No</td>
    <td align="center">直接将记录插入到叶子节点</td>
</tr>
<tr>
    <td align="center">Yes</td>
    <td align="center">No</td>
    <td align="center">
    1)拆分Leaf Page  
    2)将中间的节点放入index Page中
    3)小于中间节点的记录放左边
    4)大于或等于中间节点的记录放右边
    </td>
</tr>
<tr>
    <td align="center">Yes</td>
    <td align="center">Yes</td>
    <td align="center">
    1)拆分Leaf Page  
    2)小于中间节点的记录放左边
    3)大于或等于中间节点的记录放右边
    4)拆分Index Page  
    5)小于中间节点的记录放左边
    6)大于或等于中间节点的记录放右边
    7)中间节点放入上一层Index Page
    </td>
</tr>
</table>
+ B+树的旋转
+ 为了保持平衡对于新插入的键值可能需要做大量的拆分页操作，因为B+树结构主要用于磁盘，页的拆分意味着磁盘的操作，所以，应该在可能的情况下，尽量减少页的拆分操作。
+ 旋转发生在Leaf Page已满，但是其的左右兄弟节点没有满的情况下，这时B+树并不会急于做拆分页的操作，而是将记录移到所在页的兄弟节点上。在通常情况下，左兄弟会被首先检查用来做检查。
+ 采用旋转操作使B+树减少了一次页的拆分操作，同时这棵B+树的高度不变。
+ B+树的删除
+ B+树使用填充因子来控制树的删除变化，50%是填充因子可设的最小值。B+树的删除操作必须保证删除后叶子节点中的记录依然排序。
+ B+树删除的三种情况：
<table border="2">
<tr>
    <th align="center">叶子节点小于填充因子</th>
    <th align="center">叶子节点大于填充因子</th>
    <th align="center">操作</th>
</tr>
<tr>
    <td align="center">No</td>
    <td align="center">No</td>
    <td align="center">直接将记录从叶子节点删除，如果该节点还是Index Page的节点，用该节点的右节点代替</td>
</tr>
<tr>
    <td align="center">Yes</td>
    <td align="center">No</td>
    <td align="center">
    合并叶子节点和它的兄弟节点，同时更新Index Page
    </td>
</tr>
<tr>
    <td align="center">Yes</td>
    <td align="center">Yes</td>
    <td align="center">
    1)合并叶子节点和它的兄弟节点 
    2)更新Index Page
    3)合并Index Page和它的兄弟节点
    </td>
</tr>
</table>