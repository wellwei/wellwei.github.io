## C++实现红黑树（插入、查找、删除）

### 红黑树性质

- **所有节点都是红或黑色**
- **根节点是黑色**
- **每个叶子节点（NIL节点，或NULL）都是黑色**
- **红色节点的两个子节点都是黑色**（不会出现相邻的两个红色节点）。不红红
- **对于每个节点，从该节点到其所有后代叶子节点的简单路径上，包含相同数量的黑色节点**。黑路同

以上特性保证了**最长路径不会超过最短路径的两倍**。保证了树的平衡性，由此插入和删除操作的复杂度为**O(log n)**。

> 本文建议搭配该[网页](https://www.cs.usfca.edu/~galles/visualization/RedBlack.html)演示理解

### 定义结点结构

首先定义红黑树的节点结构。每个节点包含数据、指向父节点和左右子节点的指针，以及一个表示节点颜色的属性（红色或黑色）。

```cpp
template <typename T>
struct Node {
    T data;
    Node<T> *parent, *left, *right;
    bool isRed;
    
    Node(const T& data) : data(data), parent(nullptr), left(nullptr), right(nullptr), isRed(true) {}
};
```

在这个结构体中：

- `data`：存储节点的数据。我们使用了模板 `typename T`，使得红黑树可以存储不同类型的数据。
- `parent`：指向父节点的指针。
- `left`：指向左子节点的指针。
- `right`：指向右子节点的指针。
- `isRed`：表示节点的颜色，`true`为红色，`false`为黑色。新插入的节点通常被染成红色。



### 定义红黑树类

定义红黑树类，包含根节点、NIL节点和一些基本操作。

```cpp
template <typename T>
class RBTree {
private:
	Node<T> *root;
	Node<T> *nil;

public:
	RBTree() {
		nil = new Node<T>(0);
		nil->isRed = false;
		nil->parent = nil->left = nil->right = nullptr;
		root = nil;
	}

	// 插入
	void insert(const T&);

	// 删除
	void erase(const T&);

	// 查找
	Node<T>* find(const T&);

	// 左旋
	void rotateLeft(Node<T>*);

	// 右旋
	void rotateRight(Node<T>*);

	// 插入后修复红黑树性质
	void insertFixUp(Node<T>*);

	// 删除后修复红黑树性质
	void eraseFixUp(Node<T>*);

	// 查找最小节点
	Node<T>* findMin(Node<T>*);

	// 查找最大节点
	Node<T>* findMax(Node<T>*);

	// 查找前驱
	Node<T>* predecessor(Node<T>*);

	// 查找后继
	Node<T>* successor(Node<T>*);
    
    // 替换子树
	void transplant(Node<T> *u, Node<T> *v);
};
```

声明一些核心操作以及一些用于维护红黑树性质的辅助函数。

`nil`节点为哨兵节点，用来代替`nullptr`，防止空指针操作异常。



### 实现查找操作

红黑树的查找操作与一般的二叉搜索树的查找相同。

```cpp
template <typename T>
Node<T> *RBTree<T>::find(const T &data) const {
	Node<T> *cur = root;
	while (cur != nil) {
		if (cur->data == data)
			return cur;
		else if (cur->data < data)
			cur = cur->right;
		else
			cur = cur->left;
	}
	return nullptr;
}
```



### 实现左旋和右旋

旋转操作是红黑树保持平衡的关键，与平衡二叉树（AVL）的旋转操作类似。

**左旋**：原根右子节点成为新根，如果新根存在左子节点，那么将其作为原根的右子节点，原根变为新根的左子节点。

**右旋**：与左旋相反，原根左子节点成为新根，如果新根存在右子节点，那么将其作为原根的左子节点，原根变为新根的右子节点。

```cpp
template <typename T>
void RBTree<T>::rotateLeft(Node<T> *node) {
	Node<T> *newNode = node->right;	// 原根的右孩子成为新根
	node->right = newNode->left;	// 新根的左孩子变为旧根的右孩子
	if (newNode->left != nil)
		newNode->left->parent = node;	// 更新左孩子的父节点为旧根
	newNode->parent = node->parent;	// 更新新根的父节点为旧根的父节点
	if (node->parent == nil)
		root = newNode;	// 如果旧根是根节点，更新根节点为新根
	else if (node == node->parent->left)
		node->parent->left = newNode;	// 如果旧根是父节点的左孩子，更新父节点的左孩子为新根
	else
		node->parent->right = newNode;	// 如果旧根是父节点的右孩子，更新父节点的右孩子为新根
	newNode->left = node;	// 旧根变为新根的左孩子
	node->parent = newNode;	// 旧根的父节点变为新根
}

template <typename T>
void RBTree<T>::rotateRight(Node<T> *node) {
	Node<T> *newNode = node->left;
	node->left = newNode->right;
	if (newNode->right != nil)
		newNode->right->parent = node;
	newNode->parent = node->parent;
	if (node->parent == nil)
		root = newNode;
	else if (node == node->parent->left)
		node->parent->left = newNode;
	else
		node->parent->right = newNode;
	newNode->right = node;
	node->parent = newNode;
}
```



### 实现插入操作

红黑树的插入操作是标准的二叉搜索树插入，然后通过`insertFixUp`函数维护红黑树的性质。每个新插入的节点都是红色。

```cpp
template <typename T>
void RBTree<T>::insert(const T &data) {
	Node<T> *newNode = new Node<T>(data);
	Node<T> *cur = root, *pre = nil;
	while (cur != nil) {
		pre = cur;
		if (cur->data == data) {	// 重复数据不插入
			delete newNode;
			return;
		} else if (cur->data < data)
			cur = cur->right;
		else
			cur = cur->left;
	}
	newNode->parent = pre;

	if (pre == nil)	// 空树
		root = newNode;
	else if (pre->data < data)
		pre->right = newNode;
	else
		pre->left = newNode;

	newNode->left = nil;
	newNode->right = nil;
	newNode->isRed = true;
	
	insertFixUp(newNode);
}
```



### 实现插入后的修复操作 `insertFixUp`

插入操作如果破坏了红黑树的性质，那么就要按照以下**变色**和**旋转**规则修复：

1. ##### 当前节点（红色）的父节点是红色且当前节点的叔叔节点也是红色

   1. **把父节点设置为黑色**
   2. **把叔叔节点也设置为黑色**
   3. **把祖父节点设置为红色**
   4. **上移至祖父节点继续修复**

2. ##### 当前节点的父节点是红色，叔叔节点不存在或叔叔节点为黑色，且父节点是祖父节点的左子节点

   - **LL型**（当前节点是父节点的左子节点）：**父节点变黑，祖父节点变红，右旋祖父节点**
   - **LR**型（当前节点是父节点的右子节点）：**左旋父节点变成LL型**

3. **当前节点的父节点是红色，叔叔节点不存在或叔叔节点为黑色，且父节点是祖父节点的右子节点**

   - **RR型**（当前节点是父节点的右子节点）：**父节点变黑，祖父节点变红，左旋祖父节点**
   - **RL型**（当前节点是父节点的左子节点）：**右旋父节点变成RR型**

```cpp
template <typename T>
void RBTree<T>::insertFixUp(Node<T> *node) {
	while (node->parent->isRed) {	// 当前节点与父节点都是红，违反 “不红红”。
		if (node->parent == node->parent->parent->left) { // 父节点是祖父节点的左子节点
			Node<T> *uncle = node->parent->parent->right;
			if (uncle->isRed) { // 叔叔节点是红色
				node->parent->isRed = false;
				uncle->isRed = false;
				node->parent->parent->isRed = true;
				node = node->parent->parent; // 上移至祖父节点继续修复
			} else {
				if (node == node->parent->right) { // LR型，先左旋转为LL型
					node = node->parent;
					rotateLeft(node);
				}
				node->parent->isRed = false; // LL型，父节点变黑、祖父节点变红，右旋转祖父节点
				node->parent->parent->isRed = true;
				rotateRight(node->parent->parent);
			}
		} else { // 父节点是祖父节点的右子节点
			Node<T> *uncle = node->parent->parent->left;
			if (uncle->isRed) {
				node->parent->isRed = false;
				uncle->isRed = false;
				node->parent->parent->isRed = true;
				node = node->parent->parent;
			} else {
				if (node == node->parent->left) { // RL型，先右旋转为RR型
					node = node->parent;
					rotateRight(node);
				}
				node->parent->isRed = false; // RR型，父节点变黑、祖父节点变红，左旋转祖父节点
				node->parent->parent->isRed = true;
				rotateLeft(node->parent->parent);
			}
		}
	}

	root->isRed = false; // 根节点始终为黑
}
```



### 实现删除操作的辅助函数*

实现查找最小、最大、后继、前驱节点以及替换子树函数。这些辅助函数再删除操作中会用到。

```cpp
template <typename T>
Node<T> *RBTree<T>::findMin(Node<T> *node) const {
	while (node != nil && node->left != nil) {
		node = node->left;
	}
	return node;
}

template <typename T>
Node<T> *RBTree<T>::findMax(Node<T> *node) const {
	while (node != nil && node->right != nil) {
		node = node->right;
	}
	return node;
}

template <typename T>
Node<T> *RBTree<T>::successor(Node<T> *node) const {
	if (node->right != nil) {
		return minimum(node->right);
	}
	Node<T> *y = node->parent;
	while (y != nil && node == y->right) {
		node = y;
		y = y->parent;
	}
	return y;
}

template <typename T>
Node<T> *RBTree<T>::predecessor(Node<T> *node) const {
	if (node->left != nil) {
		return maximum(node->left);
	}
	Node<T> *y = node->parent;
	while (y != nil && node == y->left) {
		node = y;
		y = y->parent;
	}
	return y;
}

template <typename T>
void RBTree<T>::transplant(Node<T>* u, Node<T>* v) {
    if (u->parent == nullptr) {
        root = v;
    } else if (u == u->parent->left) {
        u->parent->left = v;
    } else {
        u->parent->right = v;
    }
    if (v != nullptr) {
        v->parent = u->parent;
    }
}

```



### 实现删除操作

 删除操作也需要先执行标准的二叉搜索树删除，然后通过 `deleteFixUp` 函数来维护红黑树的性质。

```cpp
template <typename T>
void RBTree<T>::erase(const T &data) {
	Node<T> *node = find(data);
	if (node == nullptr)
		return;

	Node<T> *x, *y = node;
	bool yOriginalColor = y->isRed;

	if (node->left == nil) {	// 左子树为空，用右子树替代
		x = node->right;
		transplant(node, node->right);
	} else if (node->right == nil) {	// 右子树为空，用左子树替代
		x = node->left;
		transplant(node, node->left);
	} else {	// 左右子树均不为空，找到后继节点
		y = findMin(node->right);
		yOriginalColor = y->isRed;
		x = y->right;
		if (y->parent == node) {	// 后继节点是待删除节点的右子节点，直接用右子节点替代
			if (x != nil)
				x->parent = y;
		} else {	// 用后继节点的右子节点替代后继节点
			transplant(y, y->right);
			y->right = node->right;
			y->right->parent = y;
		}
		transplant(node, y);	// 用后继节点替代待删除节点
		y->left = node->left;
		y->left->parent = y;
		y->isRed = node->isRed;
	}
	if (!yOriginalColor) {	// 删除节点为黑色，修复红黑树性质
		eraseFixUp(x);
	}
	delete node;
}
```

### 实现删除后的修复操作`eraseFixUp`

如果删除后破坏了红黑树的性质，那么需要根据以下规则进行修复：

- 当前节点是左子节点
  - 兄弟节点是红色：父变红、兄变黑、左旋父节点，继续修复
  - 兄弟节点是黑色：
    - 兄弟的两个子节点都是黑色：兄变红，上移继续修复
    - 兄弟节点至少有一个红色子节点：
      - RL型：兄弟节点的右子节点是黑色，兄弟节点变红，其左子节点变黑，右旋兄弟节点变成RR型
      - RR型：父节点颜色给兄弟节点，父节点变黑，右子变黑，左旋父节点
- 当前节点是右子节点
  - 兄弟节点是红色：父变红、兄变黑、右旋父节点，继续修复
  - 兄弟节点是黑色：
    - 兄弟的两个子节点都是黑色：兄变红，上移继续修复
    - 兄弟至少有一个红色子节点：
      - LR型：兄弟节点的左子节点是黑色，兄弟节点变红，其右子节点变黑，左旋兄弟节点变成RR型
      - LL型：父节点颜色给兄弟节点，父节点变黑，左子变黑，右旋父节点

```cpp
template <typename T>
void RBTree<T>::eraseFixUp(Node<T> *node) {
	while (node != root && !node->isRed) { // 当前节点不是根节点且不是红色
		if (node == node->parent->left) {  // 当前节点是左子节点
			Node<T> *bro = node->parent->right;
			if (bro->isRed) { // 兄弟节点是红色, 父节点变红，兄弟节点变黑，左旋父节点，继续修复
				bro->isRed = false;
				node->parent->isRed = true;
				rotateLeft(node->parent);
				bro = node->parent->right;
			}
			if (!bro->left->isRed && !bro->right->isRed) { // 兄弟节点的两个子节点都是黑色，兄弟节点变红，上移继续修复
				bro->isRed = true;
				node = node->parent;
			} else { // 兄弟节点至少有一个红色子节点
				// 兄弟节点的右子节点是黑色，左子节点是红色，RL型，兄弟节点变红，其左子节点变黑，右旋兄弟节点变成RR型
				if (!bro->right->isRed) {
					bro->left->isRed = false;
					bro->isRed = true;
					rotateRight(bro);
					bro = node->parent->right;
				}
				// RR型，兄弟节点的右子节点是红色，父节点颜色给兄弟节点，父节点变黑，右子变黑，左旋父节点
				bro->isRed = node->parent->isRed;
				node->parent->isRed = false;
				bro->right->isRed = false;
				rotateLeft(node->parent);
				node = root;
			}
		} else {
			Node<T> *bro = node->parent->left;
			if (bro->isRed) {
				bro->isRed = false;
				node->parent->isRed = true;
				rotateRight(node->parent);
				bro = node->parent->left;
			}
			if (!bro->right->isRed && !bro->left->isRed) {
				bro->isRed = true;
				node = node->parent;
			} else {
				// LR型，兄弟节点的左子节点是黑色，兄弟节点变红，其右子节点变黑，左旋兄弟节点变成RR型
				if (!bro->left->isRed) {
					bro->right->isRed = false;
					bro->isRed = true;
					rotateLeft(bro);
					bro = node->parent->left;
				}
				// RR型，兄弟节点的左子节点是红色，父节点颜色给兄弟节点，父节点变黑，左子变黑，右旋父节点
				bro->isRed = node->parent->isRed;
				node->parent->isRed = false;
				bro->left->isRed = false;
				rotateRight(node->parent);
				node = root;
			}
		}
	}
	node->isRed = false;
}
```

### 总结&测试

```cpp
template <typename T>
struct Node {
	T data;
	Node<T> *parent, *left, *right;
	bool isRed;

	Node(const T &data) : data(data), parent(nullptr), left(nullptr), right(nullptr), isRed(true) {}
};

template <typename T>
class RBTree {
private:
	Node<T> *root;
	Node<T> *nil;

public:
	RBTree() {
		nil = new Node<T>(0);
		nil->isRed = false;
		nil->parent = nil->left = nil->right = nullptr;
		root = nil;
	}

	// 插入
	void insert(const T &);

	// 删除
	void erase(const T &);

	// 查找
	Node<T> *find(const T &) const;

	// 左旋
	void rotateLeft(Node<T> *);

	// 右旋
	void rotateRight(Node<T> *);

	// 插入后修复红黑树性质
	void insertFixUp(Node<T> *);

	// 删除后修复红黑树性质
	void eraseFixUp(Node<T> *);

	// 查找最小节点
	Node<T> *findMin(Node<T> *) const;

	// 查找最大节点
	Node<T> *findMax(Node<T> *) const;

	// 查找前驱
	Node<T> *predecessor(Node<T> *) const;

	void transplant(Node<T> *u, Node<T> *v);

	// 查找后继
	Node<T> *successor(Node<T> *) const;
};

template <typename T>
Node<T> *RBTree<T>::find(const T &data) const {
	Node<T> *cur = root;
	while (cur != nil) {
		if (cur->data == data)
			return cur;
		else if (cur->data < data)
			cur = cur->right;
		else
			cur = cur->left;
	}
	return nullptr;
}

template <typename T>
void RBTree<T>::rotateLeft(Node<T> *node) {
	Node<T> *newNode = node->right; // 原根的右孩子成为新根
	node->right = newNode->left;	// 新根的左孩子变为旧根的右孩子
	if (newNode->left != nil)
		newNode->left->parent = node; // 更新左孩子的父节点为旧根
	newNode->parent = node->parent;	  // 更新新根的父节点为旧根的父节点
	if (node->parent == nil)
		root = newNode; // 如果旧根是根节点，更新根节点为新根
	else if (node == node->parent->left)
		node->parent->left = newNode; // 如果旧根是父节点的左孩子，更新父节点的左孩子为新根
	else
		node->parent->right = newNode; // 如果旧根是父节点的右孩子，更新父节点的右孩子为新根
	newNode->left = node;			   // 旧根变为新根的左孩子
	node->parent = newNode;			   // 旧根的父节点变为新根
}

template <typename T>
void RBTree<T>::rotateRight(Node<T> *node) {
	Node<T> *newNode = node->left;
	node->left = newNode->right;
	if (newNode->right != nil)
		newNode->right->parent = node;
	newNode->parent = node->parent;
	if (node->parent == nil)
		root = newNode;
	else if (node == node->parent->left)
		node->parent->left = newNode;
	else
		node->parent->right = newNode;
	newNode->right = node;
	node->parent = newNode;
}

template <typename T>
void RBTree<T>::insert(const T &data) {
	Node<T> *newNode = new Node<T>(data);
	Node<T> *cur = root, *pre = nil;
	while (cur != nil) {
		pre = cur;
		if (cur->data == data) { // 重复数据不插入
			delete newNode;
			return;
		} else if (cur->data < data)
			cur = cur->right;
		else
			cur = cur->left;
	}
	newNode->parent = pre;

	if (pre == nil) // 空树
		root = newNode;
	else if (pre->data < data)
		pre->right = newNode;
	else
		pre->left = newNode;

	newNode->left = nil;
	newNode->right = nil;
	newNode->isRed = true;

	insertFixUp(newNode);
}

template <typename T>
void RBTree<T>::insertFixUp(Node<T> *node) {
	while (node->parent->isRed) {						  // 当前节点与父节点都是红，违反 “不红红”。
		if (node->parent == node->parent->parent->left) { // 父节点是祖父节点的左子节点
			Node<T> *uncle = node->parent->parent->right;
			if (uncle->isRed) { // 叔叔节点是红色
				node->parent->isRed = false;
				uncle->isRed = false;
				node->parent->parent->isRed = true;
				node = node->parent->parent; // 上移至祖父节点继续修复
			} else {
				if (node == node->parent->right) { // LR型，先左旋转为LL型
					node = node->parent;
					rotateLeft(node);
				}
				node->parent->isRed = false; // LL型，父节点变黑、祖父节点变红，右旋转祖父节点
				node->parent->parent->isRed = true;
				rotateRight(node->parent->parent);
			}
		} else { // 父节点是祖父节点的右子节点
			Node<T> *uncle = node->parent->parent->left;
			if (uncle->isRed) {
				node->parent->isRed = false;
				uncle->isRed = false;
				node->parent->parent->isRed = true;
				node = node->parent->parent;
			} else {
				if (node == node->parent->left) { // RL型，先右旋转为RR型
					node = node->parent;
					rotateRight(node);
				}
				node->parent->isRed = false; // RR型，父节点变黑、祖父节点变红，左旋转祖父节点
				node->parent->parent->isRed = true;
				rotateLeft(node->parent->parent);
			}
		}
	}

	root->isRed = false; // 根节点始终为黑
}

template <typename T>
Node<T> *RBTree<T>::findMin(Node<T> *node) const {
	while (node != nil && node->left != nil) {
		node = node->left;
	}
	return node;
}

template <typename T>
Node<T> *RBTree<T>::findMax(Node<T> *node) const {
	while (node != nil && node->right != nil) {
		node = node->right;
	}
	return node;
}

template <typename T>
Node<T> *RBTree<T>::successor(Node<T> *node) const {
	if (node->right != nil) {
		return minimum(node->right);
	}
	Node<T> *y = node->parent;
	while (y != nil && node == y->right) {
		node = y;
		y = y->parent;
	}
	return y;
}

template <typename T>
Node<T> *RBTree<T>::predecessor(Node<T> *node) const {
	if (node->left != nil) {
		return maximum(node->left);
	}
	Node<T> *y = node->parent;
	while (y != nil && node == y->left) {
		node = y;
		y = y->parent;
	}
	return y;
}

template <typename T>
void RBTree<T>::transplant(Node<T> *u, Node<T> *v) {
	if (u->parent == nil)
		root = v;
	else if (u == u->parent->left)
		u->parent->left = v;
	else
		u->parent->right = v;
	v->parent = u->parent;
}

template <typename T>
void RBTree<T>::erase(const T &data) {
	Node<T> *node = find(data);
	if (node == nullptr)
		return;

	Node<T> *x, *y = node;
	bool yOriginalColor = y->isRed;

	if (node->left == nil) { // 左子树为空，用右子树替代
		x = node->right;
		transplant(node, node->right);
	} else if (node->right == nil) { // 右子树为空，用左子树替代
		x = node->left;
		transplant(node, node->left);
	} else { // 左右子树均不为空，找到后继节点
		y = findMin(node->right);
		yOriginalColor = y->isRed;
		x = y->right;
		if (y->parent == node) { // 后继节点是待删除节点的右子节点，直接用右子节点替代
			if (x != nil)
				x->parent = y;
		} else { // 用后继节点的右子节点替代后继节点
			transplant(y, y->right);
			y->right = node->right;
			y->right->parent = y;
		}
		transplant(node, y); // 用后继节点替代待删除节点
		y->left = node->left;
		y->left->parent = y;
		y->isRed = node->isRed;
	}
	if (!yOriginalColor) { // 删除节点为黑色，修复红黑树性质
		eraseFixUp(x);
	}
	delete node;
}

template <typename T>
void RBTree<T>::eraseFixUp(Node<T> *node) {
	while (node != root && !node->isRed) { // 当前节点不是根节点且不是红色
		if (node == node->parent->left) {  // 当前节点是左子节点
			Node<T> *bro = node->parent->right;
			if (bro->isRed) { // 兄弟节点是红色, 父节点变红，兄弟节点变黑，左旋父节点，继续修复
				bro->isRed = false;
				node->parent->isRed = true;
				rotateLeft(node->parent);
				bro = node->parent->right;
			}
			if (!bro->left->isRed && !bro->right->isRed) { // 兄弟节点的两个子节点都是黑色，兄弟节点变红，上移继续修复
				bro->isRed = true;
				node = node->parent;
			} else { // 兄弟节点至少有一个红色子节点
				// 兄弟节点的右子节点是黑色，左子节点是红色，RL型，兄弟节点变红，其左子节点变黑，右旋兄弟节点变成RR型
				if (!bro->right->isRed) {
					bro->left->isRed = false;
					bro->isRed = true;
					rotateRight(bro);
					bro = node->parent->right;
				}
				// RR型，兄弟节点的右子节点是红色，父节点颜色给兄弟节点，父节点变黑，右子变黑，左旋父节点
				bro->isRed = node->parent->isRed;
				node->parent->isRed = false;
				bro->right->isRed = false;
				rotateLeft(node->parent);
				node = root;
			}
		} else {
			Node<T> *bro = node->parent->left;
			if (bro->isRed) {
				bro->isRed = false;
				node->parent->isRed = true;
				rotateRight(node->parent);
				bro = node->parent->left;
			}
			if (!bro->right->isRed && !bro->left->isRed) {
				bro->isRed = true;
				node = node->parent;
			} else {
				// LR型，兄弟节点的左子节点是黑色，兄弟节点变红，其右子节点变黑，左旋兄弟节点变成RR型
				if (!bro->left->isRed) {
					bro->right->isRed = false;
					bro->isRed = true;
					rotateLeft(bro);
					bro = node->parent->left;
				}
				// RR型，兄弟节点的左子节点是红色，父节点颜色给兄弟节点，父节点变黑，左子变黑，右旋父节点
				bro->isRed = node->parent->isRed;
				node->parent->isRed = false;
				bro->left->isRed = false;
				rotateRight(node->parent);
				node = root;
			}
		}
	}
	node->isRed = false;
}

#include <iostream>

int main() {
	RBTree<int> tree;
	tree.insert(10);
	tree.insert(5);
	tree.insert(15);
	tree.insert(3);
	tree.insert(7);
	tree.insert(13);
	tree.insert(17);


	std::cout << "Searching for 7: " << (tree.find(7) != nullptr ? "Found" : "Not Found") << std::endl;
	std::cout << "Searching for 8: " << (tree.find(8) != nullptr ? "Found" : "Not Found") << std::endl;

	tree.erase(7);
	
	std::cout << "Searching for 7: " << (tree.find(7) != nullptr ? "Found" : "Not Found") << std::endl;

	return 0;
}
```

结果输出：

```
Searching for 7: Found
Searching for 8: Not Found
Searching for 7: Not Found
```

