# Python 链表：从零开始，一步步写熟

> 适合对象：刚接触 Python、第一次学习链表的大一同学。  
> 学习目标：理解链表怎样连接，能自己创建、遍历、增删改查，逐步掌握反转、快慢指针等常见写法。  
> 代码环境：Python 3，无需安装第三方库。本文主要讲**单向链表**，简称“单链表”。

配套文件：[linked_list_practice.py](linked_list_practice.py)。可以直接运行，也可以对照本文逐段手写。

## 怎么使用这份教程

不要一口气背完。每次只练一小部分，完成这三个动作：

1. **先画图**：画出每个节点和它的 `next`。
2. **再预测**：运行之前，写下你认为会输出什么。
3. **再改动**：换一组数据，尤其试试空链表和只有一个节点的情况。

建议分五次学习，每次约 30～60 分钟；如果卡住，可以拆成更多次。

| 阶段 | 阅读范围 | 这次学会什么 |
| --- | --- | --- |
| 第 1 次 | 第 1～4 节 | 理解节点、引用、`head` 和 `next`，手动建链表 |
| 第 2 次 | 第 5～6 节 | 遍历、查找、按下标访问、修改 |
| 第 3 次 | 第 7～9 节 | 插入、删除、哑节点，完成基础综合练习 |
| 第 4 次 | 第 10～11 节 | 反转、找中点、判断环 |
| 第 5 次 | 第 12～16 节 | 合并、删除倒数节点、综合练习与查漏补缺 |

**代码怎么拼起来？** 先定义第 3 节的 `ListNode`，再定义第 5 节的辅助函数，后面的函数都放在它们下面。函数只要定义一次即可。示例之间会重新创建 `head`，不需要沿用上一节的运行结果。

如果暂时不熟悉 `class` 和 `self`，先读第 3 节的小解释，不必先学完整个面向对象编程。

---

## 1. 链表到底是什么

先看一个普通 Python 列表：

```python
nums = [10, 20, 30]
print(nums[1])  # 20
```

你可以直接用下标访问其中的元素。

链表采用另一种组织方式：每个小盒子保存“一个值”和“下一个盒子在哪里”。这种盒子叫**节点**。

```text
head
 ↓
[10 | next] ──→ [20 | next] ──→ [30 | next] ──→ None
```

- `val`：节点中存放的值，比如 `10`。
- `next`：对下一个节点的引用。可以先把“引用”理解成“找到那个对象的线索”。
- `head`：引用第一个节点的变量，相当于链表入口。
- `None`：没有对象。在末尾表示“后面没有节点了”。

这些箭头表示节点之间的连接关系，不代表节点在内存里一定挨在一起。

**一句话记住：从 `head` 出发，反复沿着 `next`，就能访问整条链表。**

### 1.1 Python 的 `list` 就是链表吗

不是。常见 Python 实现 CPython 的 `list` 使用动态数组结构，保存一组对象引用。它与这里手写的链表不同。

| 需求 | Python `list` | 本文单链表 |
| --- | --- | --- |
| 按下标访问第 k 个元素 | 通常可直接定位，O(1) | 要从头走到那里，O(n) |
| 在开头插入或删除 | 要移动后面的引用，O(n) | 改变头部连接，O(1) |
| 在中间插入或删除 | 通常要移动部分引用，O(n) | 找到前一个节点后，修改连接本身 O(1) |
| 在末尾添加 | `list.append` 摊还 O(1) | 本文只有 `head`，需要找尾部，O(n) |
| 获取长度 | `len(nums)` 为 O(1) | 本文没有记录长度，需遍历，O(n) |

这里的 `n` 表示节点数量。`O(1)` 表示操作次数不会随 `n` 增长；`O(n)` 表示操作次数大致随 `n` 成比例增长。`list.append` 的“摊还 O(1)”表示一长串追加操作的平均成本是常数，偶尔扩容会更慢。

注意：链表“中间增删 O(1)”有前提——**你已经拿到了合适的位置**。如果还要从头查找，总成本通常仍然是 O(n)。

### 1.2 平时什么时候用

- 学数据结构、做算法题时，经常需要自己实现链表。
- 需要理解“通过修改连接改变顺序”的场景时，链表很有帮助。
- 日常保存一组数据，多数情况下直接用 `list` 更方便。
- 如果主要需求是在两端添加和删除，可以用标准库 `collections.deque`。

```python
from collections import deque

queue = deque()
queue.append(10)      # 右端加入
queue.append(20)
queue.appendleft(5)   # 左端加入
print(queue.popleft())  # 5，左端取出
print(queue.pop())      # 20，右端取出
```

`deque` 很实用，但不会让你直接操作本文这样的 `ListNode` 和 `next`。下面学习手写链表。

## 2. 先认识四个基础概念

### 2.1 一个节点不等于一条链表

节点只是一个小盒子。把多个节点连接起来，才形成我们要用的链表结构。

### 2.2 空链表是什么

```python
head = None
```

这表示链表没有节点。

而下面表示一个值为 `0` 的节点，**不是空链表**：

```python
# 先定义第 3 节的 ListNode 后，才能运行这一行。
# head = ListNode(0)
```

### 2.3 下标从 0 开始

```text
节点值： 10 → 20 → 30 → None
下标：   0    1    2
```

“第 1 个节点”的下标是 `0`。“倒数第 1 个节点”则指最后一个节点。

### 2.4 节点、节点的值、下一个节点，分别是什么

假设 `head` 引用值为 `10` 的第一个节点：

| 表达式 | 含义 |
| --- | --- |
| `head` | 第一个节点对象的引用 |
| `head.val` | 第一个节点保存的值，例如 `10` |
| `head.next` | 第二个节点的引用；没有则为 `None` |
| `head.next.val` | 第二个节点保存的值；必须先确保第二个节点存在 |

不要把“节点”与“节点保存的整数”当成一回事。

## 3. 定义节点：从这几行开始

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next
```

逐行理解：

| 代码 | 可以怎样理解 |
| --- | --- |
| `class ListNode:` | 定义一种叫 `ListNode` 的节点类型，相当于画盒子的模板 |
| `def __init__(...)` | 创建节点时，自动执行的初始化方法 |
| `self` | 当前正在创建的那个节点；创建时不用自己传入 |
| `val=0` | 不指定值时，默认存 `0` |
| `next=None` | 不指定下一个节点时，默认后面没有节点 |
| `self.val = val` | 把传进来的值放进当前节点 |
| `self.next = next` | 记录当前节点后面接着谁 |

`next` 是算法题里常见的属性名，也与 Python 的一个内置函数同名。这里的用法是合法的；暂时不需要用到那个内置函数。

现在就能创建节点：

```python
a = ListNode(10)
b = ListNode(20)

print(a.val)        # 10
print(a.next)       # None
print(a is b)       # False，它们是两个不同的节点
```

也可以创建时就连接：

```python
b = ListNode(20)
a = ListNode(10, b)
# a → b → None
```

## 4. 手动连接，并理解“引用”

### 4.1 建立 10 → 20 → 30

```python
a = ListNode(10)
b = ListNode(20)
c = ListNode(30)

a.next = b
b.next = c
head = a

print(head.val)            # 10
print(head.next.val)       # 20
print(head.next.next.val)  # 30
print(c.next)              # None
```

连接过程可以画成：

```text
开始：  a → [10]    b → [20]    c → [30]

连接后：
head、a
   ↓
 [10] ──→ [20] ──→ [30] ──→ None
           ↑        ↑
           b        c
```

### 4.2 `cur = head` 不会复制整条链表

```python
head = ListNode(10)
cur = head

print(cur is head)  # True，引用同一个节点

cur.val = 99
print(head.val)     # 99
```

`head` 和 `cur` 就像贴在同一个盒子上的两张标签。通过其中一个修改盒子里的值，另一个也能看到。

但给变量重新赋值，只会移动那张标签：

```python
head = ListNode(10, ListNode(20))
cur = head
cur = cur.next

print(cur.val)   # 20
print(head.val)  # 10，head 仍然指着第一个节点
```

务必区分：

- `cur = cur.next`：让变量 `cur` 向后移动，链表连接没变。
- `cur.next = other`：修改当前节点后面的连接，链表结构发生变化。

### 4.3 为什么函数改了 `head`，外面却没变

```python
def move_head(head):
    head = head.next


head = ListNode(10, ListNode(20))
move_head(head)
print(head.val)  # 10
```

函数里的参数 `head` 是函数内部的变量。给它重新赋值，不会给外面的同名变量重新赋值。

想让外面的入口也移动，常见办法是返回新入口，再接住它：

```python
def move_head(head):
    if head is None:
        return None
    return head.next


head = ListNode(10, ListNode(20))
head = move_head(head)
print(head.val)  # 20
```

因此，后面的插入、删除、反转等操作，统一采用这样的调用习惯：

```python
# 调用格式示意，具体函数在后文定义：
# head = 某个操作函数(head, 其他参数)
```

## 5. 遍历：沿着 next 一个一个看

### 5.1 最重要的遍历模板

```python
head = ListNode(10, ListNode(20, ListNode(30)))

cur = head
while cur is not None:
    print(cur.val)
    cur = cur.next
```

输出：

```text
10
20
30
```

执行过程：

| 进入循环前 `cur` 指向谁 | 这次输出 | 移动后 `cur` 指向谁 |
| --- | --- | --- |
| 值为 10 的节点 | 10 | 值为 20 的节点 |
| 值为 20 的节点 | 20 | 值为 30 的节点 |
| 值为 30 的节点 | 30 | `None` |
| `None` | 不进入循环 | 结束 |

为什么用 `cur`？因为要保留 `head` 这个入口。如果一直写 `head = head.next`，最后 `head` 就成了 `None`。

### 5.2 两个辅助函数：列表和链表互相转换

每次都手动写 `a.next = b` 很费事。定义下面两个辅助函数，后面的示例会经常用到。

```python
def from_list(values):
    dummy = ListNode()
    tail = dummy

    for value in values:
        tail.next = ListNode(value)
        tail = tail.next

    return dummy.next


def to_list(head):
    result = []
    cur = head

    while cur is not None:
        result.append(cur.val)
        cur = cur.next

    return result
```

使用：

```python
head = from_list([10, 20, 30])
print(to_list(head))  # [10, 20, 30]

empty = from_list([])
print(empty)          # None
print(to_list(empty)) # []
```

`dummy` 是临时放在真实头节点前面的辅助节点，叫**哑节点**，也常叫“虚拟头节点”。它的值不属于真实数据，`dummy.next` 才是真实入口。

```text
dummy → 10 → 20 → 30 → None
          ↑
      真实的 head
```

`tail` 总是指向目前最后一个节点：先在它后面接上新节点，再让 `tail` 走到新节点。第 8 节会继续解释为什么 `dummy` 很好用。

**注意：`to_list` 只用于无环链表。如果末尾又接回前面的节点，它会不停循环。** 第 11 节会讲如何判断环。

### 5.3 打印成箭头形式

```python
def print_linked_list(head):
    values = to_list(head)
    if not values:
        print("None（空链表）")
        return
    print(" -> ".join(str(value) for value in values) + " -> None")
```

```python
head = from_list([10, 20, 30])
print_linked_list(head)
# 10 -> 20 -> 30 -> None
```

如果不熟悉最后一行打印代码，可以先继续使用 `print(to_list(head))`，不影响学习链表。

### 5.4 统计长度

```python
def length(head):
    count = 0
    cur = head
    while cur is not None:
        count += 1
        cur = cur.next
    return count
```

```python
print(length(from_list([10, 20, 30])))  # 3
print(length(None))                     # 0
```

遍历一次需要 O(n) 时间。除用于输出的列表之外，这里的遍历变量只占 O(1) 额外空间。`to_list` 因为保存了所有值，需要 O(n) 额外空间。

**本节自查：** 为什么遍历条件写 `cur is not None`，而不是 `cur.next is not None`？后者会漏掉尾节点，还可能在空链表时直接报错。

## 6. 查找、按下标访问、修改

### 6.1 查找第一个值等于 target 的节点

```python
def find(head, target):
    cur = head
    while cur is not None:
        if cur.val == target:
            return cur
        cur = cur.next
    return None
```

```python
head = from_list([10, 20, 20, 30])
node = find(head, 20)

if node is not None:
    print(node.val)  # 20
else:
    print("没有找到")

print(find(head, 99))  # None
```

这里返回的是**节点**，不是下标。遇到重复值时，只返回第一次找到的那个节点。

### 6.2 按下标找到节点

本文约定：下标必须是整数；不支持 Python 列表那样的负数下标。下标不在范围内，就抛出 `IndexError`。

```python
def get_node(head, index):
    if index < 0:
        raise IndexError("下标不能为负数")

    cur = head
    for _ in range(index):
        if cur is None:
            raise IndexError("下标越界")
        cur = cur.next

    if cur is None:
        raise IndexError("下标越界")
    return cur
```

`range(index)` 让循环执行 `index` 次；`_` 是一个变量名，用在“只需要重复次数，不需要本次编号”的地方。

```python
head = from_list([10, 20, 30])
print(get_node(head, 0).val)  # 10
print(get_node(head, 2).val)  # 30
```

下标为 `2`，就从第一个节点向后走两步。不能直接写 `head[2]`，因为我们没有为节点定义这种下标访问方式。

### 6.3 修改节点的值

找到节点，再修改它的 `val`：

```python
head = from_list([10, 20, 30])
node = find(head, 20)
if node is not None:
    node.val = 99

print(to_list(head))  # [10, 99, 30]

get_node(head, 0).val = 5
print(to_list(head))  # [5, 99, 30]
```

修改值不会改变连接关系。查找位置一般需要 O(n) 时间，已经拿到节点之后，修改 `val` 是 O(1)。

## 7. 插入节点：先接后面，再接前面

### 7.1 在开头插入

目标：把 `10 → 20` 变成 `5 → 10 → 20`。

```python
def prepend(head, value):
    node = ListNode(value)
    node.next = head
    return node
```

```python
head = from_list([10, 20])
head = prepend(head, 5)
print(to_list(head))  # [5, 10, 20]
```

图解：

```text
原来：head → 10 → 20 → None
新建：node → 5
连接：node → 5 → 10 → 20 → None
返回：让外面的 head 指向 5
```

这几行可以缩写成：

```python
def prepend(head, value):
    return ListNode(value, head)
```

两种写法选一种保存即可。空链表同样适用，时间 O(1)。

### 7.2 在末尾插入

```python
def append(head, value):
    node = ListNode(value)
    if head is None:
        return node

    cur = head
    while cur.next is not None:
        cur = cur.next

    cur.next = node
    return head
```

```python
head = from_list([10, 20])
head = append(head, 30)
print(to_list(head))  # [10, 20, 30]

head = None
head = append(head, 10)
print(to_list(head))  # [10]
```

为什么这里用 `while cur.next is not None`？因为我们要停在**最后一个节点**，然后修改它的 `next`。这与“访问所有节点”的遍历目的不同。

本文没有单独保存尾节点，所以每次末尾插入都要寻找尾部，时间 O(n)。第 5 节的 `from_list` 自己维护了 `tail`，每次接入新节点为 O(1)，因此创建整条链表为 O(n)。

### 7.3 已知前一个节点，在它后面插入

假设 `prev` 引用值为 `10` 的节点，要插入 `15`：

```text
原来：prev → [10] → [20] → None
目标：prev → [10] → [15] → [20] → None
```

关键两步：

```python
head = from_list([10, 20])
prev = head
node = ListNode(15)

node.next = prev.next  # 1. 新节点先连到原来的后继节点
prev.next = node       # 2. 前一个节点再连到新节点

print(to_list(head))   # [10, 15, 20]
```

如果先写 `prev.next = node`，再写 `node.next = prev.next`，第二行中的 `prev.next` 已经变成 `node`，最终会让新节点指向自己，形成环！

所以刚开始练习时，请明确写成上面两步，不要急着压缩代码。

### 7.4 在指定下标插入

约定：原长度是 `n`，允许的插入下标是 `0～n`，包含两端。`0` 表示插到开头，`n` 表示追加到末尾。

```python
def insert_at(head, index, value):
    if index < 0:
        raise IndexError("下标不能为负数")

    dummy = ListNode(0, head)
    prev = dummy

    for _ in range(index):
        if prev.next is None:
            raise IndexError("插入位置越界")
        prev = prev.next

    prev.next = ListNode(value, prev.next)
    return dummy.next
```

```python
head = from_list([10, 30])
head = insert_at(head, 1, 20)
print(to_list(head))  # [10, 20, 30]

head = insert_at(head, 0, 5)
head = insert_at(head, 4, 40)
print(to_list(head))  # [5, 10, 20, 30, 40]
```

其中：

```python
# insert_at 函数内部的关键连接语句：
# prev.next = ListNode(value, prev.next)
```

Python 会先计算右边，创建新节点并把它连接到旧的 `prev.next`，再把新节点交给左边的 `prev.next`。它对应上一小节的两步连接。

## 8. 删除节点：让前一个节点绕过它

### 8.1 删除的本质

```text
删除前：prev → [10] → [20] → [30] → None
删除后：prev → [10] ───────→ [30] → None
```

核心操作：

```python
head = from_list([10, 20, 30])
prev = head
prev.next = prev.next.next

print(to_list(head))  # [10, 30]
```

这表示把 `10` 的下一个节点，改成 `20` 的下一个节点。前提是 `prev` 和 `prev.next` 都存在。

“从链表删除”表示从入口出发不再经过这个节点，不是把它的值设成 `None`。如果没有任何其他引用，Python 会自动管理被移除对象的回收；如果别处仍然引用它，它仍可存在。不需要手动释放内存。

### 8.2 为什么需要 dummy

删除第一个节点时，真实链表里找不到它的“前一个节点”。如果临时添加 `dummy`，头部也有前驱，就能统一处理：

```text
dummy → 10 → 20 → 30 → None

删 10：让 dummy.next 指向 20
删 20：让 10.next 指向 30
```

最后返回 `dummy.next`，就能得到新的真实入口。即使删掉原来的头节点，也不需要另外写一整套逻辑。

`dummy.val` 的具体数值无关紧要；代码中没有用它参与查找或比较。

### 8.3 删除第一个匹配的值

```python
def delete_value(head, target):
    dummy = ListNode(0, head)
    prev = dummy

    while prev.next is not None:
        if prev.next.val == target:
            prev.next = prev.next.next
            break
        prev = prev.next

    return dummy.next
```

```python
head = from_list([10, 20, 20, 30])
head = delete_value(head, 20)
print(to_list(head))  # [10, 20, 30]，只删除第一个 20

head = delete_value(head, 10)
print(to_list(head))  # [20, 30]，头节点也可以删除

head = delete_value(head, 99)
print(to_list(head))  # [20, 30]，没找到就不修改

print(to_list(delete_value(None, 10)))  # []
```

为什么检查 `prev.next.val`？因为我们要删掉的节点是 `prev` 后面的那个，必须保留它前面的连接位置。

### 8.4 按下标删除

删除时，合法下标是 `0～n-1`。与插入不同，不能删除下标 `n`，因为那里没有节点。

```python
def delete_at(head, index):
    if index < 0:
        raise IndexError("下标不能为负数")

    dummy = ListNode(0, head)
    prev = dummy

    for _ in range(index):
        if prev.next is None:
            raise IndexError("删除位置越界")
        prev = prev.next

    if prev.next is None:
        raise IndexError("删除位置越界")

    prev.next = prev.next.next
    return dummy.next
```

```python
head = from_list([10, 20, 30])
head = delete_at(head, 1)
print(to_list(head))  # [10, 30]

head = delete_at(head, 0)
print(to_list(head))  # [30]

head = delete_at(head, 0)
print(to_list(head))  # []
```

访问非法下标会抛出异常。可以这样练习观察：

```python
try:
    delete_at(None, 0)
except IndexError as error:
    print(error)  # 删除位置越界
```

`raise` 表示主动报告错误；`try/except` 表示捕获某种错误并处理。基础操作中，先让错误信息清楚，比悄悄返回不明确的结果更容易调试。

## 9. 第一轮综合练习：自己完成增删改查

先不要看下面的预期结果，逐行画图预测。

```python
head = from_list([10, 20, 30])
head = prepend(head, 5)
head = append(head, 40)
head = insert_at(head, 2, 15)
print(to_list(head))

node = find(head, 20)
if node is not None:
    node.val = 25
print(to_list(head))

head = delete_value(head, 30)
head = delete_at(head, 0)
print(to_list(head))
print(length(head))
```

预期输出：

```text
[5, 10, 15, 20, 30, 40]
[5, 10, 15, 25, 30, 40]
[10, 15, 25, 40]
4
```

完成后关掉参考代码，试着独立写出：

1. `ListNode`。
2. 遍历并打印所有值。
3. 在头部插入一个值。
4. 删除第一个匹配的值。

能独立完成这些，再继续学后面的算法。

## 10. 常见算法一：反转单链表

目标：

```text
原来：head → 1 → 2 → 3 → None
反转：head → 3 → 2 → 1 → None
```

不是把 `1、2、3` 重新填进节点，而是把连接方向逐个改过来。

### 10.1 三个变量分别负责什么

- `prev`：已经反转好的那一段的入口，开始为 `None`。
- `cur`：现在正在处理的节点。
- `nxt`：提前保存原来的下一个节点，防止改连接后找不到后面的部分。

```python
def reverse_list(head):
    prev = None
    cur = head

    while cur is not None:
        nxt = cur.next   # 1. 记住原来的后面
        cur.next = prev  # 2. 把当前箭头反过来
        prev = cur       # 3. 已反转部分的入口前进
        cur = nxt        # 4. 去处理下一个节点

    return prev
```

```python
head = from_list([1, 2, 3])
head = reverse_list(head)
print(to_list(head))  # [3, 2, 1]
```

### 10.2 手动走一遍

| 时刻 | `prev` 指向的已反转部分 | `cur` 指向的待处理部分 |
| --- | --- | --- |
| 开始 | `None` | `1 → 2 → 3 → None` |
| 第 1 次循环后 | `1 → None` | `2 → 3 → None` |
| 第 2 次循环后 | `2 → 1 → None` | `3 → None` |
| 第 3 次循环后 | `3 → 2 → 1 → None` | `None` |

结束时 `cur` 已经是 `None`，新入口在 `prev`，所以返回 `prev`。

如果漏掉 `nxt = cur.next`，先执行了 `cur.next = prev`，原来的“后面在哪里”就可能丢失。

时间 O(n)，额外空间 O(1)。空链表返回 `None`，单节点链表仍然返回那个节点。

**跟练：** 手动画出 `[7, 8]` 的反转过程，然后把空链表、单节点也运行一遍。

## 11. 常见算法二：快慢指针

“指针”在这里是算法里的习惯称呼。Python 代码实际操作的是对象引用。

让两个变量从头出发：`slow` 每次走一步，`fast` 每次走两步。它们的速度不同，可以帮助我们判断位置关系。

### 11.1 找中间节点

```python
def middle_node(head):
    slow = head
    fast = head

    while fast is not None and fast.next is not None:
        slow = slow.next
        fast = fast.next.next

    return slow
```

```python
head = from_list([1, 2, 3, 4, 5])
middle = middle_node(head)
print(middle.val if middle is not None else None)  # 3

head = from_list([1, 2, 3, 4])
middle = middle_node(head)
print(middle.val if middle is not None else None)  # 3

print(middle_node(None))  # None
```

偶数个节点时有两个中间节点，这个写法返回**后面那个**，例如 `[1, 2, 3, 4]` 返回值为 `3` 的节点。

以五个节点为例：

| 时刻 | `slow` 的值 | `fast` 的值 |
| --- | --- | --- |
| 开始 | 1 | 1 |
| 走一轮 | 2 | 3 |
| 走两轮 | 3 | 5 |

`fast` 到尾部时，`slow` 走了大约一半的路。时间 O(n)，额外空间 O(1)。

### 11.2 为什么条件里要写两个判断

```python
# 循环条件示意，完整代码见上一小节：
# while fast is not None and fast.next is not None:
```

因为接下来要访问 `fast.next.next`，必须先确保 `fast` 存在，且 `fast.next` 也存在。

Python 的 `and` 会从左到右判断：左边为假时，不会继续计算右边。这叫“短路求值”，因此这里的顺序很重要。

### 11.3 判断链表有没有环

正常链表最终到达 `None`，有环链表则会绕回前面：

```text
1 → 2 → 3 → 4
    ↑       ↓
    └───────┘
```

```python
def has_cycle(head):
    slow = head
    fast = head

    while fast is not None and fast.next is not None:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:
            return True

    return False
```

如果有环，快指针进入环后会逐步追上慢指针，就像在环形跑道上追赶。如果无环，快指针会先到达末尾。

**比较的是 `slow is fast`，也就是“是不是同一个节点”，不是两个节点的值是否相等。** 两个不同节点完全可以都存着 `2`。

创建一个有环例子：

```python
a = ListNode(1)
b = ListNode(2)
c = ListNode(3)
a.next = b
b.next = c
c.next = b  # 3 后面接回 2

print(has_cycle(a))  # True

# 有环时不要调用 to_list(a)、length(a) 或普通打印遍历，会一直绕圈。
c.next = None       # 断开环，恢复普通链表
print(to_list(a))    # [1, 2, 3]
```

判断环的时间 O(n)，额外空间 O(1)。比较必须放在移动之后；如果刚初始化就比较，两个变量都等于 `head`，会误判。

## 12. 常见算法三：合并两个有序链表

输入两条**各自已经按升序排列**的链表：

```text
a：1 → 3 → 5 → None
b：2 → 4 → 6 → None

结果：1 → 2 → 3 → 4 → 5 → 6 → None
```

每次比较两条链表当前的头部，把较小的那个接到结果尾部，然后只移动被取走的那一边。

```python
def merge_sorted(a, b):
    dummy = ListNode()
    tail = dummy

    while a is not None and b is not None:
        if a.val <= b.val:
            tail.next = a
            a = a.next
        else:
            tail.next = b
            b = b.next
        tail = tail.next

    tail.next = a if a is not None else b
    return dummy.next
```

```python
a = from_list([1, 3, 5])
b = from_list([2, 4, 6])
merged = merge_sorted(a, b)
print(to_list(merged))  # [1, 2, 3, 4, 5, 6]
```

其中 `a if a is not None else b` 的意思是：“如果 `a` 还有剩余就用 `a`，否则用 `b`。”主循环结束时至少有一边已经用完，剩下的一整段可以直接接上。

使用前提与影响：

- 两条输入链表必须升序、无环，而且彼此不能共享节点。
- 这个实现会重新连接原来的节点，不会复制出一整套新节点。合并后，请把 `merged` 当作结果使用；原来的 `a`、`b` 不再代表两条保持原结构的独立链表。
- 重复值会保留，例如 `[1, 2]` 与 `[1, 3]` 合并成 `[1, 1, 2, 3]`。

两条链表长度分别为 `m`、`n` 时，时间 O(m+n)，额外空间 O(1)，不计输入节点本身。

## 13. 常见算法四：删除倒数第 n 个节点

这是进阶练习，第一次学可以先跳过，基础熟练后再回来。

例如 `[10, 20, 30, 40, 50]` 中，倒数第 `1` 个是 `50`，倒数第 `2` 个是 `40`。

最直接的方法是先统计长度 `L`，再调用 `delete_at(head, L - n)`；当然要先检查 `1 <= n <= L`。下面用两个指针做一次前后配合。

### 13.1 两个指针保持间隔

1. 让 `fast` 和 `slow` 都从 `dummy` 出发。
2. 让 `fast` 先走 `n` 步。
3. 两者一起每次走一步，直到 `fast` 到达最后一个节点。
4. 此时 `slow` 恰好是待删节点的前一个节点，绕过 `slow.next` 即可。

```python
def remove_nth_from_end(head, n):
    if n <= 0:
        raise ValueError("n 必须大于 0")

    dummy = ListNode(0, head)
    fast = dummy
    slow = dummy

    for _ in range(n):
        if fast.next is None:
            raise ValueError("n 不能超过链表长度")
        fast = fast.next

    while fast.next is not None:
        fast = fast.next
        slow = slow.next

    slow.next = slow.next.next
    return dummy.next
```

```python
head = from_list([10, 20, 30, 40, 50])
head = remove_nth_from_end(head, 2)
print(to_list(head))  # [10, 20, 30, 50]
```

画图验证 `n = 2`：

```text
初始：dummy(S,F) → 10 → 20 → 30 → 40 → 50 → None

F 先走两步：
      dummy(S) → 10 → 20(F) → 30 → 40 → 50 → None

一起前进，直到 F 到尾节点：
      dummy → 10 → 20 → 30(S) → 40 → 50(F) → None

删除 S 后面的 40。
```

这里 `n` 必须是整数且在 `1～链表长度` 之间；空链表没有合法的 `n`。如果链表长度记作 `L`，时间为 O(L)，额外空间 O(1)。

## 14. 最容易踩的坑

### 坑 1：还没确认节点存在，就访问属性

```python
# 错误示意，不要运行：
# head = None
# print(head.val)  # AttributeError
```

先判断 `head is not None`。要访问 `head.next.val`，还需要确认 `head.next` 存在。

### 坑 2：循环中忘记移动 cur

```python
# 错误示意：
# while cur is not None:
#     print(cur.val)
```

`cur` 一直停在同一个节点，循环无法结束。普通遍历需要 `cur = cur.next`。

### 坑 3：用移动 head 的方式遍历，最后丢了入口

把遍历变量写成 `cur = head`，通过移动 `cur` 保留入口。如果你有意消费入口可以移动 `head`，但应清楚后果。

### 坑 4：修改了头部，却没有接住新 head

```python
head = from_list([10, 20])
prepend(head, 5)
print(to_list(head))  # [10, 20]，新头没有被外面的变量接住

head = prepend(head, 5)
print(to_list(head))  # [5, 10, 20]
```

统一养成 `head = 操作函数(...)` 的习惯。删除头节点、反转等情况也需要这个习惯。

### 坑 5：把变量赋为 None，当作删除节点

```python
head = from_list([10, 20, 30])
cur = head.next
cur = None
print(to_list(head))  # [10, 20, 30]，连接没有改变
```

给变量重新赋值不会自动修改前一个节点的 `next`。`del cur` 同样只是删除变量名，不会替你把链表连接起来。

### 坑 6：把值相等当成同一个节点

```python
a = ListNode(7)
b = ListNode(7)
print(a.val == b.val)  # True
print(a is b)         # False
```

找某个值，用 `node.val == target`。判断是不是同一个节点，用 `a is b`。

### 坑 7：不保存后继，先把原连接覆盖了

反转时先保存 `nxt = cur.next`；插入时先让新节点接住原后继。每次改 `next` 前都问一句：**改完之后，我还能找到后面那一段吗？**

### 坑 8：反复按下标遍历，把 O(n) 写成 O(n²)

```python
# 不推荐的遍历思路：
# for i in range(length(head)):
#     print(get_node(head, i).val)
```

`get_node` 每次从头走，累计步数大约是 `0 + 1 + ... + (n-1)`。直接用 `cur` 一路向后走，整体才是 O(n)。

### 坑 9：以为普通打印会自动展开链表

`print(head)` 通常显示对象的类型与标识信息，不会自动显示所有节点值。使用 `to_list(head)` 或 `print_linked_list(head)`。

### 坑 10：只用正常数据验证

每个函数至少考虑这些输入：

| 情况 | 示例 | 要检查什么 |
| --- | --- | --- |
| 空链表 | `None` | 会不会直接访问不存在的节点 |
| 单节点 | `[8]` | 头和尾是同一个节点时是否正确 |
| 双节点 | `[8, 9]` | 连接方向、边界是否正确 |
| 重复值 | `[2, 2, 2]` | 删除一个还是全部，是否符合约定 |
| 头部变化 | 删除头、头插、反转 | 外面是否接住返回的新头 |
| 越界位置 | `-1`、长度、长度加一 | 区分插入与访问/删除的合法范围 |

## 15. 分级练习：先自己写，再看提示

### A 级：建立直觉

**A1.** 不调用 `from_list`，手动创建 `3 → 6 → 9 → None`，并遍历输出。

**A2.** 写函数 `sum_values(head)`，返回所有节点值之和。空链表返回 `0`。

**A3.** 写函数 `count_value(head, target)`，统计某个值出现了多少次。

自测数据：

| 题目 | 输入 | 预期结果 |
| --- | --- | --- |
| A2 | `[]` | `0` |
| A2 | `[3, 6, 9]` | `18` |
| A3 | `[2, 1, 2, 2]`，目标 `2` | `3` |
| A3 | `[]`，目标 `2` | `0` |

### B 级：熟练修改连接

**B1.** 写函数删除所有值等于 `target` 的节点，不只是第一个。分别试试 `[2, 2, 3, 2]` 和 `[2, 2]`。

提示：删除后先不要移动 `prev`，因为新的 `prev.next` 也可能需要删除；只有当前后继不需要删除时，才移动 `prev`。

**B2.** 不看第 7 节，重新实现 `insert_at`，测试在头、中、尾插入，以及非法位置。

**B3.** 不看第 8 节，重新实现 `delete_at`，连续删除直到链表为空。

### C 级：常见算法

**C1.** 默写迭代反转，对同一链表反转两次，检查是否回到原来的值顺序。

**C2.** 写找中间节点，解释为什么偶数个节点时返回后面那个。

**C3.** 合并 `[1, 2, 4]` 和 `[1, 3, 4]`，得到 `[1, 1, 2, 3, 4, 4]`。

**C4.** 删除 `[1, 2, 3, 4, 5]` 的倒数第 `5` 个节点，结果应为 `[2, 3, 4, 5]`。再试单节点删除。

### 参考答案：A2、A3、B1

先完成自己的版本，再往下看。

```python
def sum_values(head):
    total = 0
    cur = head
    while cur is not None:
        total += cur.val
        cur = cur.next
    return total


def count_value(head, target):
    count = 0
    cur = head
    while cur is not None:
        if cur.val == target:
            count += 1
        cur = cur.next
    return count


def delete_all(head, target):
    dummy = ListNode(0, head)
    prev = dummy

    while prev.next is not None:
        if prev.next.val == target:
            prev.next = prev.next.next
        else:
            prev = prev.next

    return dummy.next
```

```python
print(sum_values(from_list([3, 6, 9])))  # 18
print(count_value(from_list([2, 1, 2, 2]), 2))  # 3

head = from_list([2, 2, 3, 2])
head = delete_all(head, 2)
print(to_list(head))  # [3]

head = delete_all(from_list([2, 2]), 2)
print(to_list(head))  # []
```

B2、B3 的参考实现在第 7、8 节；C 级实现在第 10～13 节。A1 可按第 4 节的方法替换节点值。

## 16. 常用代码速查与下一步

### 16.1 操作速查

以下复杂度以无环单链表、仅有 `head` 的本文实现为准；`n` 表示节点数量。

| 需求 | 调用方式 | 返回什么 | 时间 / 额外空间 |
| --- | --- | --- | --- |
| 创建节点 | `ListNode(10)` | 一个节点 | O(1) / O(1) |
| 列表建链表 | `head = from_list([1, 2, 3])` | 头节点或 `None` | O(n) / O(n)，包含新建节点 |
| 转回列表 | `to_list(head)` | Python 列表 | O(n) / O(n) |
| 长度 | `length(head)` | 整数 | O(n) / O(1) |
| 查找值 | `find(head, 2)` | 首个匹配节点或 `None` | O(n) / O(1) |
| 按下标访问 | `get_node(head, 1)` | 对应节点，非法下标报错 | 最坏 O(n) / O(1) |
| 修改值 | `node.val = 9` | 赋值语句无返回结果 | 已有节点时 O(1) / O(1) |
| 头插 | `head = prepend(head, 0)` | 新头 | O(1) / O(1) |
| 尾插 | `head = append(head, 4)` | 当前头 | O(n) / O(1) |
| 指定下标插入 | `head = insert_at(head, 1, 9)` | 当前头 | 最坏 O(n) / O(1) |
| 删除第一个匹配值 | `head = delete_value(head, 2)` | 当前头 | O(n) / O(1) |
| 按下标删除 | `head = delete_at(head, 1)` | 当前头 | 最坏 O(n) / O(1) |
| 反转 | `head = reverse_list(head)` | 新头 | O(n) / O(1) |
| 找中点 | `middle_node(head)` | 中间节点或 `None` | O(n) / O(1) |
| 判断环 | `has_cycle(head)` | `True` 或 `False` | O(n) / O(1) |
| 删除倒数第 k 个 | `head = remove_nth_from_end(head, k)` | 当前头 | O(n) / O(1) |

单次插入至多新建常数个节点，因此额外空间是 O(1)。合并两条链表的复杂度见第 12 节。

### 16.2 五个必须熟悉的模板

```python
# 1. 遍历
cur = head
while cur is not None:
    # 在这里处理 cur.val
    cur = cur.next

# 2. 在 prev 后插入 node，要求 prev 是有效节点
# node.next = prev.next
# prev.next = node

# 3. 删除 prev 后的节点，要求 prev.next 存在
# prev.next = prev.next.next

# 4. 用 dummy 统一头部操作
# dummy = ListNode(0, head)
# ...修改 dummy 后面的连接...
# return dummy.next

# 5. 反转时的四步，要求 cur 是当前有效节点
# nxt = cur.next
# cur.next = prev
# prev = cur
# cur = nxt
```

### 16.3 配套 Python 文件怎么运行

1. 安装好 Python 3。
2. 把 `linked_list_practice.py` 保存到电脑上。
3. 用 VS Code 等编辑器打开，点击运行；或者在文件所在文件夹打开终端，输入：

```bash
python linked_list_practice.py
```

Windows 上如果 `python` 命令不可用，也可以试试 `py linked_list_practice.py`。若两个都不可用，先检查 Python 是否安装、是否配置了命令路径。

它会演示基础操作和几个常见算法。例如，“插入、删除、反转”部分输出：

```text
=== 插入、删除、反转 ===
插入后： [5, 10, 15, 25, 30, 40]
删除后： [10, 15, 30, 40]
反转后： [40, 30, 15, 10]
```

这段演示接着文件前面的“把 20 改成 25”运行，所以与第 9 节的独立练习数据略有不同。修改配套文件中 `demo()` 的输入列表，即可练习自己的数据。

也可以新建一个文件 `my_practice.py`，与配套文件放在同一文件夹，写：

```python
from linked_list_practice import from_list, to_list, reverse_list

head = from_list([7, 8, 9])
head = reverse_list(head)
print(to_list(head))  # [9, 8, 7]
```

配套文件最后的 `if __name__ == "__main__":` 表示“直接运行这个文件时才执行演示”。通过 `import` 导入函数时，不会自动打印整段演示。

### 16.4 在算法平台上怎么写

算法平台常常已经提供 `ListNode`，并要求把函数写进 `Solution` 类。以反转为例：

```python
class Solution:
    def reverseList(self, head):
        prev = None
        cur = head
        while cur is not None:
            nxt = cur.next
            cur.next = prev
            prev = cur
            cur = nxt
        return prev
```

平台通常会替你准备输入节点和调用方法。按题目给出的函数名与参数写，不需要在提交区自己解析 `[1, 2, 3]`，也不需要重复定义平台已经提供的节点类。

以后看到 `Optional[ListNode]`，可以先把它理解为“可能是节点，也可能是 `None`”的类型提示。它帮助阅读，不会自动为你创建链表。

推荐顺序：LeetCode 203「移除链表元素」→ 206「反转链表」→ 876「链表的中间结点」→ 141「环形链表」→ 21「合并两个有序链表」→ 19「删除链表的倒数第 N 个结点」。先自己画图，再参考本文对应模板。

### 16.5 双向链表和循环链表，先认识即可

- **双向链表**：除了 `next`，每个节点还记录前一个节点 `prev`。可以前后走，插入和删除时要同时维护两个方向。
- **循环链表**：尾节点不指向 `None`，而是接回头部或前面的某个节点。不能再用“走到 `None`”作为普通遍历的停止条件。

先把本文单链表练熟，再学习这两种结构。

### 16.6 掌握程度自查

- [ ] 我能解释 `head`、`head.val`、`head.next` 的区别。
- [ ] 我能画出 `cur = cur.next` 与 `cur.next = other` 的不同效果。
- [ ] 我能独立写出节点定义、创建和遍历。
- [ ] 我能完成头插、尾插、按位置插入和删除。
- [ ] 我知道为什么很多操作后要写 `head = ...`。
- [ ] 我能解释 `dummy` 如何统一处理头部操作。
- [ ] 我能手动演示反转链表的四步。
- [ ] 我能写出快慢指针，并保证访问属性前节点存在。
- [ ] 我会用空链表、单节点、重复值和头部变化检查代码。
- [ ] 我能说明为什么查找位置可能让链表增删变成 O(n)。

如果某项还不熟，就用 3 个节点画一次图、手写一次，再换成空链表验证。能说清楚每一根箭头为什么变化，比背下一大段代码更有用。
