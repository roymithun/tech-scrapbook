# Binary Tree Types with Diagrams

Binary trees come in various forms, each with unique properties. Below are different types of binary trees along with their structures.

## 1. Full Binary Tree
Every node has either **0 or 2** children.

```
    1
   / \
  2   3
 / \  / \
4  5 6  7
```

---

## 2. Complete Binary Tree
All levels are fully filled **except possibly the last**, with nodes **left-aligned**.

```
    1
   / \
  2   3
 / \  /
4  5 6
```

---

## 3. Perfect Binary Tree
All internal nodes have **2 children**, and all leaves exist at the **same depth**.
```
    1
   / \
  2   3
 / \  / \
4  5 6  7
```

---

## 4. Balanced Binary Tree
The difference in height between the left and right subtree of any node is **at most 1**.
```
    5
   / \
  3   7
 / \  / \
2  4 6  8
```

---

## 5. Skewed (Degenerate) Binary Tree
Each node has **only one child**, making it resemble a linked list.

### Right-Skewed
Each node has **only one child**, and all children are to the right.
```
1
 \
  2
   \
    3
     \
      4
```

### Left-Skewed
Each node has **only one child**, and all children are to the left.
```
      4
     /
    3
   /
  2
 /
1
```


---

## 6. Binary Search Tree (BST)
Nodes are arranged so that:
- Left subtree contains **smaller values**.
- Right subtree contains **greater values**.
```
    8
   / \
  3   10
 / \    \
1   6    14
```


---

## 7. Threaded Binary Tree
A **Threaded Binary Tree** is designed to optimize in-order traversal by utilizing **extra pointers**, allowing traversal without recursion or a stack.

### **Example of a Right-Threaded Binary Tree**
```
    4
   / \
  2   6
 / \   \
1   3   8
```
If this tree were **right-threaded**, the `NULL` right pointers would point to **in-order successors**:
- `1 → 2`
- `2 → 3`
- `3 → 4`
- `4 → 6`
- `6 → 8`

### **Advantages of Threaded Binary Trees**
- Eliminates recursion or stack usage during in-order traversal.
- Reduces memory overhead.
- Improves traversal efficiency in large datasets.

---

## 8. Expression Tree
Represents arithmetic expressions, with **operators** as internal nodes and **operands** as leaf nodes.

Example for `(3 × 2) + 5`:
```
      +
     / \
    *   5
   / \
  3   2
```
