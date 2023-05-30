# Splay Tree

伸展树的主要优势就在于不需要对基本的二叉树节点结构，做任何附加的要求或改动，例如记录平衡因子或黑高度之类的额外信息，实现比较简便。同时在任何足够长的真实操作序列中，能够保持分摊意义上的高效率。



通常在任意数据结构的生命周期内，不仅执行不同操作的概率往往极不均衡，而且各操作之间具有极强的相关性，并在整体上多呈现出极强的规律性。这种现象被表述为**数据局部性**:

- **刚刚被访问过的元素，极有可能在不久之后再次被访问到**
- **将被访问的下一元素，极有可能就处于不久之前被访问过的某个元素的附近**

对任意一个数据结构来说，如果能充分利用数据局部特性，即可进一步地提高数据结构和算法的效率。Splay Tree就是这方面的一个例子。



它的基本思想如下：对于任意访问到的节点p，通过zig-zag或zig-zig等旋转操作构造出一颗等价二叉树，该二叉树以节点p作为根节点。

一种方法是**逐层伸展**：即每访问过一个节点之后，随即反复地以它的父节点为轴，经适当的旋转将其提升一层，直至最终成为树根。

![image-20230529212108590](assets/image-20230529212108590.png)

它的最坏情况如下：

![image-20230529212135662](assets/image-20230529212135662.png)

这一最坏情况的实例，完全可以推广至规模任意的二叉搜索树。于是对于规模为任意$n$的伸展树，只要按关键码单调的次序，周期性地反复进行查找，不难验证在一个周期中，旋转操作的总次数应为：
$$
(n - 1) + (n - 2) + ... + 1 = \Omega(n^2)
$$
如此分摊下来，每次访问平均需要$\Omega(n)$时间。很遗憾，这一效率不仅远远低于AVL树，而且甚至与原始的二叉搜索树的最坏情况相当。



为克服上述伸展调整策略的缺陷，一种简便且有效的方法就是：将逐层伸展改为双层伸展。每经过一次双层调整操作，节点v都会上升两层。若v的初始深度depth(v)为偶数，则最终v将上升至树根。若depth(v)为奇数，则当v上升至深度为1时，不妨最后再相应地做一次zig或zag单旋操作。无论如何，经过depth(v)次旋转后，v最终总能成为树根。

![image-20230529212755785](assets/image-20230529212755785.png)

![image-20230529212803499](assets/image-20230529212803499.png)

![image-20230529212810553](assets/image-20230529212810553.png)



我们再来考察下最坏情况：

![image-20230529212938589](assets/image-20230529212938589.png)

![image-20230529212949410](assets/image-20230529212949410.png)



Tarjan等人采用势能分析法（potential analysis）业已证明，在改用“双层伸展”策略之后，伸展树的单次操作均可在分摊的*O*(logn)时间内完成。



逐层伸展的问题根源可解释为：在持续访问的过程中，树高依算术级数逐步从$n - 1$递减至$n/2$，然后再逐步递增回到$n - 1$​。在双层伸展中一旦这类“坏”节点被“碰触”到，经过随后的双层伸展，其对应的分支都会收缩至长度大致折半。于是，即便每次都“恶意地”试图访问最底层节点，最坏情况也不会持续发生。可见，伸展树虽不能杜绝最坏情况的发生，却能有效地控制最坏情况发生的频度，从而在分摊意义下保证整体的高效率。



## 查找

- 注意 `zig-zig`与AVL树中的旋转是有所不同的，因此在这种情况下不能使用`connect34()`

~~~java
/**
 * 节点q作为p的左孩子插入
 */
private void attachAsLeftChild(BinaryTreeNode<T> p, BinaryTreeNode<T> q) {
    p.leftChild = q;
    if (q != null) q.parent = p;
}

/**
 * 节点q作为p的右孩子插入
 */
private void attachAsRightChild(BinaryTreeNode<T> p, BinaryTreeNode<T> q) {
    p.rightChild = q;
    if (q != null) q.parent = p;
}
~~~



~~~java

/**
 * 将节点v伸展至树根，会修改整棵树的高度
 * @param v 要伸展的节点
 * @return 返回伸展后的树的根节点，即参数v
 */
private BinaryTreeNode<T> splay(BinaryTreeNode<T> v) {
    if (v == null) return null;
    BinaryTreeNode<T> p;
    BinaryTreeNode<T> g;
    while ((p = v.parent) != null && (g = p.parent) != null) {
        BinaryTreeNode<T> gg = g.parent;
        if (BinaryTreeNode.isLeftChild(v)) {
            if (BinaryTreeNode.isLeftChild(p)) {            //zig-zig，不可以用connect34代替
                attachAsLeftChild(g, p.rightChild);
                attachAsLeftChild(p, v.rightChild);
                attachAsRightChild(p, g);
                attachAsRightChild(v,p);
            } else {                                        //zig-zag，这里可以用connect34代替
                attachAsLeftChild(p,v.rightChild);
                attachAsRightChild(g,v.leftChild);
                attachAsLeftChild(v, g);
                attachAsRightChild(v, p);
            }
        } else {
            if (BinaryTreeNode.isLeftChild(p)) {            //zag-zag
                attachAsRightChild(p, v.leftChild);
                attachAsLeftChild(g,v.rightChild);
                attachAsRightChild(v, g);
                attachAsLeftChild(v, p);
            } else {                                        //zag-zig
                attachAsRightChild(p,v.leftChild);
                attachAsRightChild(p,v.leftChild);
                attachAsLeftChild(p, g);
                attachAsLeftChild(v,p);
            }
        }

        v.parent = gg;
        if (gg != null) {
            if (g == gg.leftChild) {
                attachAsLeftChild(gg, v);
            } else {
                attachAsRightChild(gg, v);
            }
        }
        updateHeight(g);
        updateHeight(p);
        updateHeight(v);
    }
    //此时g一定为null，p可能为null
    if (p != null) {    //如果p不为null，那么再进行一次单旋
        if (BinaryTreeNode.isLeftChild(v)) {
            attachAsLeftChild(p,v.rightChild);
            attachAsRightChild(v, p);
        } else {
            attachAsRightChild(p,v.leftChild);
            attachAsLeftChild(v, p);
        }
        updateHeight(p);
        updateHeight(v);
        v.parent = null;
    }
    return v;
}
~~~



~~~java
/**
 * 查找指定元素，并将元素推至到树根。同时会修改整个树的高度以及成员root
 * @param element 待查找的元素
 * @return 返回树根root
 */
@Override
public BinaryTreeNode<T> search(T element) {
    BinaryTreeNode<T> p = search(root, element);
    return root = splay(p == null ? hot : p);
}
~~~



## 插入

以上接口Splay::search()已集成了splay()伸展功能，故查找返回后，树根节点要么等于查找目标（查找成功）；要么就是hot，而且恰为拟插入节点的直接前驱或直接后继（查找失败）。因此，不妨改用如下方法实现Splay::insert()接口。

![image-20230530110630950](assets/image-20230530110630950.png)



~~~java
@Override
public BinaryTreeNode<T> insert(T element) {
    if (root == null) {     //处理空树的退化情况
        size += 1;
        return root = new BinaryTreeNode<T>(element, null);
    }
    //if (search(element) != null) 是错误的，因为覆写了search方法，返回值有所变化
    if (search(element).data.compareTo(element) == 0) {      //若元素已存在
        return root;
    }

    size += 1;
    BinaryTreeNode<T> t = root;
    if (root.data.compareTo(element) < 0) {
        t.parent = root = new BinaryTreeNode<T>(element, null, t, t.rightChild);
        if (BinaryTreeNode.hasRightChild(t)) {
            t.rightChild.parent = root;
            t.rightChild = null;
        }
    } else {
        t.parent = root = new BinaryTreeNode<T>(element, null, t.leftChild, t);
        if (BinaryTreeNode.hasLeftChild(t)) {
            t.leftChild.parent = root;
            t.leftChild = null;
        }
    }
    updateHeight(t);
    return root;
}
~~~



## 删除

然而同样地，在实施删除操作之前，通常都需要调用Splay::search()定位目标节点，而该接口已经集成了splay()伸展功能，从而使得在成功返回后，树根节点恰好就是待删除节点。因此，亦不妨改用如下策略，以实现Splay::remove()接口。

![image-20230530110851221](assets/image-20230530110851221.png)

~~~java
 @Override
public boolean remove(T element) {
    if (root == null
        || search(element).data.compareTo(element) != 0) {
        //1. 空树的退化情况
        //2. 未查找到元素
        return false;
    }

    if (!BinaryTreeNode.hasLeftChild(root)) {       //左子树不存在
        root = root.rightChild;
        if (root != null)            //右子树可能不存在
            root.parent = null;
    } else if (!BinaryTreeNode.hasRightChild(root)) {       //左子树存在，但是右子树不存在
        root = root.leftChild;
        root.parent = null;
    } else {                //左子树和右子树都存在
        BinaryTreeNode<T> leftTree = root.leftChild;
        root.leftChild = null;      //暂时将左子树切除
        root = root.rightChild;
        root.parent = null;
        search(element);            //必然会查找失败，此时树根root会被修改为hot
        root.leftChild = leftTree;
        leftTree.parent = root;
    }
    size -= 1;
    if (root != null)
        updateHeight(root);     //拼接左子树后要手动更新高度
    return true;
}
~~~

