<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>

# 《并发算法与理论》期末整理

## 一、简介

- 异步计算中的两种形式化性质
	+ Safety Properties：不发生坏事
	+ Liveness Properties：好事总会发生
- 互斥（Mutual Exclusion）：Both not in simultaneously（Safety）
- 无死锁（No Deadlock）：If one wants in, it gets in; If both want in, one gets in.（Liveness）
- Amdahl's Law：$$p$$表示并发成分比例，$$n$$表示处理器个数. $$$$Speedup = \frac{1}{1-p+\frac{p}{n}}$$$$

## 二、互斥

- 形式化
	+ **event**: 线程$$A$$的一个事件（event）$$a_0$$是瞬间发生的，没有与之同时发生的其它事件.
	+ **thread**：一个线程$$A$$是一连串的事件$$a_0,a_1,\dots$$，$$a_0\rightarrow a_1$$表示了顺序.
	+ **interval**：一个间隔$$A_0=(a_0,a_1)$$是$$a_0$$和$$a_1$$之间的时间.
	+ **precedence**：$$A_0\rightarrow B_0$$表示$$A_0$$最后的事件发生在$$B_0$$第一个事件之前.
	+ **偏序（Partial Orders）**：非自反性（$$\neg(A\rightarrow A)$$）、反对称性（$$A\rightarrow B \Rightarrow \neg(B\rightarrow A)$$）、传递性（$$(A\rightarrow B\wedge B\rightarrow C)\Rightarrow (A\rightarrow C)$$）
	+ **全序（Total Orders）**：偏序 + 对任意两个不同的$$A,B$$都有$$A\rightarrow B$$或$$B\rightarrow A$$.
	+ **互斥（Mutual Exclusion）**：用$$CS_i^k$$表示线程$$i$$的第$$k$$次临界区执行，相应地有$$CS_j^m$$，那么要么$$CS_i^k\rightarrow CS_j^m$$要么$$CS_j^m\rightarrow CS_i^k$$.

### 1. 锁的使用

- 无死锁（Deadlock-Freedom）：若有线程调用了`lock()`但永不返回，则其它的线程必须能够无限次完成`lock()`和`unlock()`的调用；无饥饿（Starvation-Freedom）：如果有线程调用了`lock()`，它一定能够返回.

#### 1.1 两个线程的情形

- LockOne：给两个线程设置了大小为2的`flag`数组，`lock()`：`flag[i] = true; while (flag[j]) {}`；`unlock()`: `flag[i] = false;` 满足互斥（可结合Code给出的顺序以及Assumption推出成环，得到矛盾）；并发执行可能导致死锁（`flag`都为`true`时）.
- LockTwo：“让其它人先走”. `lock()`：`victim = i; while (victim == i) {}`；`unlock()`什么都不做. 满足互斥，串行执行会导致死锁.
- Peterson算法：`lock()`：表明自己感兴趣（`flag[i] = true;`），并先把机会让给别人（`victim = i; while (flag[j] && victim == i) {}`；`unlock()`：自己不再感兴趣（`flag[i] = false;`）. 满足互斥、无死锁、无饥饿.

#### 1.2 $$n$$（$$n>2$$）个线程的情形

- Filter算法：给定$$n-1$$个层，每一层至少有一个线程进入，如果有多个线程想进入，则至少有一个被阻塞. 只有唯一一个线程穿越每一层到达CS. 实现上用`level[i] = L`表明自己想进入第$$L$$层，用`victim[L] = i`把机会让给别人，若自己是`victim`并且有其他人在相同或更高层，就等待；（Peterson算法的推广）
	+ Claim：从$$L=0$$层开始，至多有$$n-L$$个线程进入第$$L$$层，因此在第$$n-1$$层达到互斥.
	+ 无饥饿，但不够公平（让给别人超越）
	+ $$r$$-Bounded Waiting：将`lock()`分为两个部分（总能在有限步完成的Doorway Interval $$D_A$$和有不确定步数的Waiting Interval $$W_A$$），如果$$D_A^k\rightarrow D_B^j$$，那么$$CS_A^k\rightarrow CS_B^{j+r}$$，也即$$B$$只能优先$$A$$至多$$r$$次. FCFS意味着$$r=0$$.

- Bakery算法：FCFS；每个线程有个数，等待数更小的线程完成. 可能会引发overflow，从而导致互斥被破坏.

- 优先级：优先级图（Precedence Graph）：$$x$$指向$$y$$的边表示$$x$$是更晚的时间戳. $$k$$个线程的顺序确定需要$$3^k$$个结点，$$T^k=T^2*T^{k-1}$$.

- 定理：至少需要$$N$$个MRSW/MRMW寄存器才能解决无死锁互斥问题.

## 三、并发对象

- 基于锁的队列（对整个方法上锁）：显然满足互斥；无互斥的并发队列实现：如何定义实现的正确性？
- 正确性（Correctness）：Safety Property
	+ 串行规范：Precondition、Postcondition. 对象在方法调用之间有有意义的状态
	+ 并发规范：方法（method）调用不是一个事件，而是一个间隔，可能overlap. **Idea: 用串行来描述并发！**

### 1. 可线性化

- **Linearizability（可线性化）**：一个并发对象的每个方法都在它进入和返回的间隔中某一时刻瞬间take effect. 如果这样的“串行”行为是正确的，那么对象是正确的.
	+ 在一些情况下，linearization point依赖于具体的执行.

- 形式化
	+ **Invocation（调用）**：`A q.enq(x)`；**Response（返回）**：`A q: void`, `A q: empty()`.
	+ **History（历史）**：一个执行过程的描述（用以上两个符号）
	+ **match（匹配）**：一个invocation和一个response有相同的线程和对象.
	+ **Object Projections（对象投影）**：`H|q`表示对象`q`的History；**Thread Projections（线程投影）**：`H|B`表示线程`B`的History.
	+ **Complete subhistory（完全子历史）**：去掉正在pending的invocation得到的history.
	+ **Sequential Histories（串行历史）**：不同线程的方法（match）不互相交叉.
	+ **Well-Formed Histories（良结构历史）**：每个Thread Projections都是串行的.
	+ **Equivalent Histories（等价历史）**：每个Thread Projections对应相等.
	+ **Legal Histories（合法历史）**：对于每个对象`x`，`H|x`都满足串行规范.
	+ **Precedence（优先级）**：一个方法若response在另一个invocation之前，则它precede另一个方法调用. $$m_0\rightarrow_H m_1$$是偏序的，若$$H$$串行，则为全序.

- **Linearizability（可线性化）**：若$$H$$可通过为pending invocations增添responses或将其删去来扩展到$$G$$，$$G$$与某个legal sequential history $$S$$等价，且$$\rightarrow_G\subset\rightarrow_S$$（仅允许重新排序不同线程间的操作），则$$H$$可线性化.

- 可组合性定理（Composability Theorem）：一个历史$$H$$可线性化当且仅当对于每个对象$$x$$，都有$$H|x$$可线性化.
- 基于锁的并发对象的Linearization point是锁被释放的瞬间. 对于仅有一个enqueuer和一个dequeuer时，Linearization point是`head`和`tail`被修改的瞬间.

### 2. 串行一致性

- **Sequential Consistency（串行一致性）**：在可线性化的基础上，不需要保持real-time order（删去了$$\rightarrow_G\subset\rightarrow_S$$. 无可组合性.
	+ 对硬件体系结构来说是个很强的条件，对软件来说难以运作（一般用可线性化）

- Everyone makes progress & Non-Blocking $$\rightarrow$$ Wait-free  
Everyone makes progress & Blocking $$\rightarrow$$ Starvation-free  
Someone makes progress & Non-Blocking $$\rightarrow$$ Lock-free  
Someone makes progress & Blocking $$\rightarrow$$ Deadlock-free

## 四、共享内存基础

- 寄存器（Register）：有一个值（可以是布尔值或$$m$$位整数），可被读写.
- **Safe Register**：不与任何写操作overlap的读操作返回最近一次写入的值；与某个写操作overlap的读操作可能返回寄存器允许的范围内的任意值.
- **Regular Register**：首先是safe的；若一个读操作与第$$i$$次写操作overlap，则它可能返回第$$i$$或第$$i-1$$个写入的值.
- **Atomic Register**：可线性化为串行safe register.（能找到linearization point）

### 1. 寄存器的三个维度

- **SRSW safe Boolean**：最弱的寄存器
- **MRSW safe Boolean**：用SRSW safe Boolean实现. 每个线程有自己的safe SRSW寄存器，写时写所有的寄存器，读时只读自己线程的. 类似的，对Multi-Valued也成立.
- **MRSW regular Boolean**：用MRSW safe Boolean实现. 保存当前线程最近写入成功的值`old`，若欲写入的值`x`与`old`不同，则先`value.write(x)`再更新`old = x`. 读时直接`value.read()`即可.
- **MRSW regular M-valued**：用MRSW regular Boolean实现. 保存$$M$$大小的比特数组，写`x`时先将`x`位置为`true`，并从`x-1`到`0`逐个置为`false`；读时从`0`开始返回读到的第一个为`true`的索引.
- **SRSW atomic M-valued**：用SRSW reguar M-valued实现. 写时同时写当前时间戳；读时根据上次保存所读的值与时间戳，仅在时间戳更高时返回新值.
- **MRSW atomic M-valued**：用SRSW atomic M-valued实现. 方法：读时总是告诉其它线程自己读到了什么.
- **MRMW atomic M-valued**：用MRSW atomic M-valued实现. 方法很像Bakery算法，写时读所有的时间戳然后写入一个更高的时间戳. 读时返回有最高时间戳的那个.

### 2. 原子快照

- **Atomic Snapshot**：有两种操作（`int update(int v)`第`i`个线程写`v`到它的寄存器中；`int[] scan()`所有线程寄存器的瞬时快照）
	+ Collect（收集）：一次读所有值
	+ 问题在于不相容的并发收集和不可线性化的结果.

- **Clean Collect**：在收集的过程中没有值被改变
	+ **Simple Snapshot**：对每个entry给定递增的label. 在`scan()`时一遍遍collect如今每个寄存器的label，与上次collect的结果进行对比，如果两个数组完全相同才返回这个相同的结果. 可线性化，`update`是wait-free的，但`scan`可能饥饿.
	+ **Wait-Free Snapshot**：保存三种值：`int label`（递增计数器）、`int value`（实际值）、`int[] snap`（最近快照）. 对于`update(v)`，先调用`scan()`获取快照，再更新新值（将原先的`label`加一，并赋予新的最近的快照）；对于`scan()`，修改Simple Snapshot中检测到两次快照不一致时的处理，若发现其它线程move了两次，就返回它最新的快照，否则设定`move[j] = true`. 很容易被扩展为MRMW版本.

## 五、同步操作的相对能力

### 1. 一致性（Consensus）

- 一致性的要求
	+ Consistent：所有的线程决定了相同的值.
	+ Valid：这个共同的值是某个线程的输入.

- 定理（来自Fischer, Lynch, Paterson）：There is no wait-free implementation of $$n$$-thread consensus, $$n>1$$, from read-write registers.
- **Generic Consensus Protocol**：实现仅拥有一个抽象方法`decide(value)`的`Consensus`接口，每个线程有一个`proposed`值.
- **FIFO Consensus**：队列中预先存一个红球和一个黑球，`deq()`出红球的线程决定自己的值，否则决定另一个线程的值. (因此一致数至少为2）

- **Consensus Number（一致数）**：若一个对象$$X$$能够用任意数量的自身实例以及原子读写寄存器来实现$$n$$线程consensus，但不能实现$$n+1$$线程consensus，则它有一致数$$n$$.
- **同步能力**：若能够从$$Y$$实现$$X$$，并且$$X$$有一致数$$c$$，那么$$Y$$的一致数至少为$$c$$. 反之，若$$X$$有一致数$$c$$，而$$Y$$有一致数$$d<c$$，则不可能从$$Y$$实现无等待的$$X$$.
- **多重赋值**：原子地写多个值，读任意的元素. **多重赋值定理**：原子寄存器无法实现多重赋值.
- **二重赋值**：原子地，$$A$$写0、1；$$B$$写1、2. 若另一个线程没动或者晚动，就决定自己的值，否则决定另一个的值. 达到2-Consensus.
- **Read-Modify-Write**：原子地，返回某个对象先前的值`x`，将该值`x`用`mumble(x)`取代. 例子：`read()`, `getAndSet(v)`（2-Consensus）, `getAndIncrement()`（2-Consensus）, `getAndAdd(a)`, `compareAndSet(expected, update)`
	+ `boolean compareAndSet(expected, update)`若当前的值`value`与`expected`相等，则将其更新为`update`，返回`true`，否则返回`false`. 【有**无穷一致数**，将原子整数`r`初始化为-1，通过`r.compareAndSet(-1, i)`返回`proposed[r.get()]`】
- All the results we presented hold for lock-free algorithms also. 

## 六、自旋锁与争用

### 1. 设计性能牛逼的锁

- **Test-and-Set Lock**：维护一个锁的当前状态`state`初始化为`false`表示锁当前空闲；`lock()`即为`while (state.getAndSet(true)) {}`，`unlock()`为`state.set(false);`.
	+ 空间复杂读很小，仅为$$O(1)$$（使用了RMW操作）.

- **Test-and-Test-and-Set Lock**：潜伏+抢夺. 
```
while (true) { while (state.get()) {} if (!state.getAndSet(true)) return; }
```
	+ 性能比TAS更好，但也不接近理想性能.

- **缓存一致性（Cache Coherence）**：一个数据原始拷贝在内存中，每个处理器有它的缓存拷贝，某个处理器修改它自己的拷贝时，必须使其它缓存拷贝无效化，需要复杂的协议.
	+ TAS：拿到锁的处理器进入总线，其它想访问内存的处理器只能等待，使所有的缓存拷贝无效化，那么那些自旋的处理器会遭遇cache miss于是得通过总线寻值.
	+ TTAS：一旦有锁的处理器释放锁，在等锁的处理器们会发生cache miss，都想通过TAS进总线，导致总线拥塞.
	+ 解决方法：Delay！（看着要堵了，我让你一会）

- **Exponential Backoff Lock（指数后移锁）**：在TTAS的基础上，若没有抢夺到锁，则睡眠当前最大`delay`之内的任意时间，每次将最大`delay`翻倍. 性能比TTAS更优，且容易实现. 缺点是依赖参数和平台.
- **Anderson Queue Lock**：维护一个队列`flags[]`，每个元素是一个线程的槽，起初`flag[0]=true`，线程等待自己的槽为`true`时获得锁，释放时将它下一个槽置为`true`.
	+ 优点：性能比后移锁更好；曲线更平滑；有FIFO的公平性.（减小了一个线程释放锁和另一个线程获得锁的间隔）
	+ 缺点：每个锁需要$$O(N)$$的空间，$$L$$个锁需要$$O(LN)$$的空间，开销大；不适用于无缓存体系结构.
- **CLH Lock**：使用隐含链表降低空间开销. 每个线程将它的状态记录在`QNode`对象中（仅存储一个布尔值），并且记录它前一个线程的`QNode`到`pred`本地变量中. 整个锁维护一个`tail`. 在`lock()`时，线程先将自己的状态置为`true`，等待前面一个线程状态变`false`（用`pred = tail.getAndSet(myNode)`更新）. 在`unlock()`时将自己的状态置为`false`，并`myNode = pred`回收前驱结点.
	+ 优点：空间开销为$$O(L+N)$$；当释放锁时，只影响后继结点；FCFS.
	+ 缺点；不适用于无缓存的NUMA体系结构.（Spinning on remote memory is slow）
- **MCS Lock**：使用明确的链表，确保处理器在固定的局部区域自旋. `QNode`保存`locked`状态和后继`next`. 在`lock()`时，若前驱非空，则将自己的`locked`置为`true`并`pred.next = qnode`. 等待自己的`locked`变`false`. 在`unlock()`时若`qnode.next == null`，进一步用`tail.CAS(qnode, null)`判断是否无后继结点，若无，则返回，否则等待直到`qnode.next`非空，便`qnode.next.locked = false;`.

### 2. 设计能够中止的锁

- Back-off Lock：直接从`lock()`返回，显然wait-free.
- Queue Lock：不能直接退出，会导致后面的线程饥饿.
- CLH Lock：想要中止的线程将其的node进行标记，让后继重用这个结点，并等待这个结点的前驱.
- **Time-out Lock**：一个锁有全局的`Qnode`变量为`AVAILABLE`，表明空闲的锁.
	+ `lock()`：初始化自己的结点；更新`tail`，获取前驱结点；若前驱为空或前驱的前驱为`AVAILABLE`，则返回`true`；在循环中判断当前时间与`timeout`的大小，若尚未超时，如果前驱结点的前驱为`AVAILABLE`，则返回`true`，否则若它非空，说明前驱结点被中止了，将前驱结点的前驱更新为自己的前驱. 最后，若超时了，判断是否有后继，若有，则`qnode.prev = myPred`，最后返回`false`.
	+ `unlock()`：判断自己是否有后继，若有，则`qnode.prev = AVAILABLE`；若没有，`tail`被置为`null`.

## 七、链表：锁的作用

- 比粗粒度锁更优的同步技术
	+ **细粒度同步**：将对象分解为独立的同步组件，当多方法试图同时访问同一组件才会冲突.
	+ **乐观同步**：查找时无需获取任何锁，当找到所需的部分时再加锁，并确认该部分在检测和上锁期间没有变化，若变化则重来.（成功次数高于失败次数时意义大）
	+ **惰性同步**：先设置标记位逻辑删除，后面再物理删除.
	+ **非阻塞同步**：完全消除锁，使用类似`compareAndSet`的原子操作进行同步.
- 基于链表的集合：`Set<T>`: `boolean add(T x), boolean remove(T x), boolean contains(T x)`; `Node`: `T item, int key, Node next`.
- **细粒度同步**：从`head`开始，两个两个锁结点，直到锁住要`remove`的结点. 每次先释放前面的锁，再锁新的结点. 若该`item`存在，则可线性化点即为`pred.next = curr.next`；若不存在，则可线性化点即为每次执行的`curr.lock()`. 对于`add`操作，仅需锁住前驱即可.
	+ 比粗粒度更优（线程可以并行遍历链表）；Long chain of acquire/release，不够效率.
- **乐观同步**：不获取锁的情况下遍历链表，当`curr`的`key`大于等于`a`的`key`值时，锁住`pred`和`curr`，调用`validate()`验证`pred`是否可达以及它的`next`是否仍指向`curr`，其余不变.
	+ 遍历时无争抢，并且遍历是wait-free的；性能得到提升；但需要遍历两次（若无锁遍历两次要比有锁遍历一次开销小，才值得）；`contains()`方法需要锁，而它在许多应用中占了90%的调用.
- **惰性同步**：根据每个结点的`marked`来判断它是否被逻辑删除，以此来`validate`；`remove`时先逻辑删除再物理删除.
	+ 只遍历一次，`contains()`不用锁.
	+ 竞争的`add()`和`remove()`仍然会重新遍历，一个线程延迟会引起traffic jam.（互斥的固有问题）
- **无阻塞同步**：结合Mark-Bit和指针，使用`AtomicMarkableReference`.

## 八、并发队列和栈

- **池（Pool）**：接口. 有`void put(T item)`和`T get()`两个接口方法.（生产者-消费者缓冲区）
	+ bounded或unbounded：是否有有限的容量
	+ 池的方法可能是total、partial或synchronous的（方法不需要等待某条件为真才返回；方法需要等待某条件为真才返回；需要等待另一个方法调用与它的调用间隔相重叠）
	+ 提供各种公平性保证（FIFO, LIFO, ...）

### 1. 并发队列	
	
- **有界部分队列（Bounded, Partial）**：`enq`和`deq`各有一把锁，也对应着`notFullCondition`和`notEmptyCondition`两个条件. 入队时先拿`enqLock`，若队列满了，等待`notFullCondition`被提醒，直到队列非满后，添加新结点，此时若原先队列为空，就拿`deqLock`，提醒所有`notEmptyCondition`. 出队对称地如此.
	+ 入队和出队共享了原子计数器，容易造成串行瓶颈：可划分为两个计数器（入列者递减`enqSidePermits`，出队者递增`deqSidePermits`.

- **无界完全队列（Unbounded, Total）**：最无脑的实现，出队时若队列为空就抛出异常.
- **无界无锁队列（Unbounded, Lock-free）**：使用CAS.
	+ `enqueue()`时，先将`tail.next`指向新结点，然后将`tail`设置为新结点，这其中`tail`既可能确实是最后的结点，也可能是倒数第二个结点：因此若发现`tail.next`非空，就用CAS使`tail`指向`tail.next`. 如果CAS在第一步（逻辑入队）失败了，重来！如果在第二步（物理入队）失败了，可以忽略（一些其它线程抢先执行了）.
	+ `dequeue()`时，先读取队首结点的值并存储，然后将它标记为`head`哨兵，删去原先的哨兵（Clever！我们没有物理删除结点，只是把它变成了哨兵）
	+ 如何重用内存（无GC时）？每个线程保存free list of unused queue nodes. 这样做容易引发ABA问题. **ABA问题：**线程1准备将变量的值由$$A$$替换为$$B$$，而在此之前，其它线程已将$$A$$替换为$$C$$再替换为$$A$$，此时线程1的CAS仍能成功，但实际情况与当初不同（可能会引用free list里面的变量）. **解决方法：**用时间戳标记每个原子引用（`AtomicStampedReference<T>`）

### 2. 并发栈

- **无锁栈（Lock-free）**：先创建一个新结点`node`，然后持续尝试`push`（`node.next = oldTop`，再用CAS更新`top`），若CAS成功，直接结束；否则回退然后重试.
	+ 优点：无锁；缺点：无GC时面临ABA问题，无backoff时在顶端有争抢；不存在并行……
- **Elimination-Backoff Stack（消除-后退栈）**
	+ insight：给定一个Elimination Array，push和pop线程随机选取slot，若slot相同，则同时发生的`push`和`pop`可相互抵消，直接把前者的参数传递给后者；CAS失败时可后退一段时间；concern：如何严格控制只有两个线程交换，而不会一对多？用`Exchanger`实现.
	+ Linearizability：无消除时显然满足；有消除时，发生匹配即满足；结合起来也是可线性化的.
	+ 副作用（好处）：消除促进了并行并减少了栈的访问；后退减少了栈顶竞争.
- 实现
	+ `Elimination Array`：有一定容量的`Exchanger`. `T visit(T value, int range)`随机在`range`中选取`slot`，返回`exchanger[slot].exchange(value, nanodur)`.
	+ `Exchanger`：拥有`slot`（引用+时间戳）；三种状态`{EMPTY, WAITING, BUSY}`，分别表示这个slot空闲；一个线程在这个slot上等待配对；其它线程正在使用这个slot进行交换. 首先进入`exchange()`后，拿到这个槽上另一个家伙的`item`和时间戳. 然后分情况：
		- `EMPTY`：首先尝试插入自己的item并将状态改变为`WAITING`（若失败则重试），循环等待直到所在slot状态被改变为`BUSY`，将状态改为`EMPTY`并返回伙伴的值；如果循环等待超时了，尝试将状态变为`EMPTY`然后抛出超时异常，如果这里的CAS又失败了，说明伙伴出现了，设`EMPTY`并返回伙伴的值.
		- `WAITING`：尝试将自己的值CAS，并将状态变为`BUSY`. 若这一步成功了，那么返回在等自己的那个家伙的值就好，否则说明有别人抢先配对了，重试.
		- `BUSY`：说明有其它线程在交换，直接重试.
	+ `push`先尝试`push`，不能成功时`otherValue = eliminationArray.visit(value, range)`，若返回值为空（因为只有`pop`才会有非空返回值）则说明`push`成功；不成功的话重试. `pop`也是相应的操作（除了返回值为非空）.
	