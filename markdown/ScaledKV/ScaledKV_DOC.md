- [1. ScaledKv模块调用关系](#1-scaledkv%e6%a8%a1%e5%9d%97%e8%b0%83%e7%94%a8%e5%85%b3%e7%b3%bb)
  - [1.1. NVMBpNode类](#11-nvmbpnode%e7%b1%bb)
    - [1.1.1. 类成员](#111-%e7%b1%bb%e6%88%90%e5%91%98)
    - [1.1.2. InsertAtLeaf](#112-insertatleaf)
    - [1.1.3. InsertIndex](#113-insertindex)
    - [1.1.4. InsertAtIndex](#114-insertatindex)
    - [1.1.5. Split](#115-split)
    - [1.1.6. Get](#116-get)
    - [1.1.7. FindLeafNode](#117-findleafnode)
    - [1.1.8. GetMinLeaveNode](#118-getminleavenode)
    - [1.1.9. Search(key,value,_valueNode)](#119-searchkeyvaluevaluenode)
    - [1.1.10. Search(key,key,result,size)](#1110-searchkeykeyresultsize)
    - [1.1.11. GetValue](#1111-getvalue)
    - [1.1.12. GetRange](#1112-getrange)
    - [1.1.13. deleteIndex](#1113-deleteindex)
    - [1.1.14. borrowNext](#1114-borrownext)
    - [1.1.15. borrowPrev](#1115-borrowprev)
    - [1.1.16. mergeNext](#1116-mergenext)
    - [1.1.17. nodeDelete](#1117-nodedelete)
    - [1.1.18. DeleteAtLeaf](#1118-deleteatleaf)
    - [1.1.19. DeleteAtIndex](#1119-deleteatindex)
  - [1.2. NVMBpTree类](#12-nvmbptree%e7%b1%bb)
    - [1.2.1. Insert、Delete、Get、Search、GetRange、Get*](#121-insertdeletegetsearchgetrangeget)
  - [1.3. BLevel类](#13-blevel%e7%b1%bb)
    - [1.3.1. FindAtBLevel](#131-findatblevel)
    - [1.3.2. GetCDFOffset](#132-getcdfoffset)
    - [1.3.3. FindAtCDFLess](#133-findatcdfless)
    - [1.3.4. FindAtCDFGreate](#134-findatcdfgreate)
    - [1.3.5. BinarySearchAtBLevel](#135-binarysearchatblevel)
    - [1.3.6. Insert](#136-insert)
    - [1.3.7. Update](#137-update)
    - [1.3.8. Delete](#138-delete)
    - [1.3.9. Get*](#139-get)
    - [1.3.10. AppendBLevel](#1310-appendblevel)
  - [1.4. CNode 类](#14-cnode-%e7%b1%bb)
    - [1.4.1. SortLeafNode](#141-sortleafnode)
    - [1.4.2. Split](#142-split)
    - [1.4.3. ExpandIndexNode](#143-expandindexnode)
    - [1.4.4. InsertAtCNode](#144-insertatcnode)
    - [1.4.5. Get、Update、Delete、GetRange](#145-getupdatedeletegetrange)
  - [1.5. SuffixEntry类](#15-suffixentry%e7%b1%bb)
  - [1.6. NVMScaledKv类](#16-nvmscaledkv%e7%b1%bb)
    - [1.6.1. Inittialize](#161-inittialize)
    - [1.6.2. Insert](#162-insert)
    - [1.6.3. ExpandFromBpTree](#163-expandfrombptree)
    - [1.6.4. ExpandFromBLevel](#164-expandfromblevel)
- [2. Question](#2-question)
  - [2.1. BLevel::Insert](#21-blevelinsert)
  - [2.2. BLevel::AppendBLevel](#22-blevelappendblevel)
  - [2.3. CNode：：InsertAtCNode](#23-cnodeinsertatcnode)
  - [2.4. CNode::InsertAtLeaf](#24-cnodeinsertatleaf)
  - [2.5. NVMScaledKV::ExpandFromBpTree](#25-nvmscaledkvexpandfrombptree)
  - [2.6. NVMScaledKV::ExpandFromBLevel](#26-nvmscaledkvexpandfromblevel)


## 1. ScaledKv模块调用关系

ScaledKv的UML图如下：

![ScaledKv](./ScaledKv_UML.jpg)

### 1.1. NVMBpNode类

#### 1.1.1. 类成员

1. m_key : 存放keys
2. NVMBpNode ：存放keys对应的BpNode指针
3. prev、next : 存放BpNode的左右结点（叶子结点有用）
4. m_currentSize : 当前结点中key数量
5. nodeTyep ：标识当前结点是叶子节点（1）还是索引结点（0）
6. reserved ：对齐？

#### 1.1.2. InsertAtLeaf

目的：向叶子结点插入一个key/value对。

逻辑：如果m_currentSize为0(只有在当前结点为root结点的时候会发生)，那么直接插入。否则查找在当前结点中查找给定key插入的位置，然后插入。

> 在查找给定key插入位置的时候可以使用二分查找？是因为一致性等其他原因而没用吗？

#### 1.1.3. InsertIndex

目的：向当前结点插入一个key/value对。

和InsertAtLeaf实现一样。区别在填写的value值不一样。只不过调用方为中间结点。

#### 1.1.4. InsertAtIndex

目的：向B+Tree插入一个key/value对。

InsertIndex是向单个结点插入数据，同时也没有考虑结点key数目溢出的问题。InsertAtIndex则是向当前结点为root的B+Tree树插入数据，同时考虑插入溢出之后进行split操作。


#### 1.1.5. Split

目的：将当前结点分割成两个结点。

逻辑：调用pmem库分配一个结点的内存。将当前结点key较小的一半key/value对复制过去。如果是叶子结点同时还更新链表。


#### 1.1.6. Get

目的：根据给定的key，在当前结点为root的B+Tree中查找对应的value。


#### 1.1.7. FindLeafNode

目的：根据给定的key，查找可能包含给定key的结点指针。


#### 1.1.8. GetMinLeaveNode

目的：返回叶子结点构成的链表的头。

#### 1.1.9. Search(key,value,_valueNode)

目的：在当前结点中查找是否存在给定的key。

#### 1.1.10. Search(key,key,result,size)

目的：查找当前结点结点中是否有key落在key1到key2的范围内。

#### 1.1.11. GetValue

目的：将指向value的指针封装成string。

#### 1.1.12. GetRange

目的：查找当前结点中时候包含key1到key2的size数目的key。

#### 1.1.13. deleteIndex

目的：删除第i个key/value对应的B+Tree子树。

#### 1.1.14. borrowNext

目的：将next的第0个key/value对移到当前结点的m_currenSize的位置。

#### 1.1.15. borrowPrev

目的：将Prev的最后一个key/value对插入到当前结点。

#### 1.1.16. mergeNext

目的：将next中的key/value对都复制到当前结点的尾部。

#### 1.1.17. nodeDelete

目的：如果当前结点是叶子结点，则将当前结点从链表上取下来。

#### 1.1.18. DeleteAtLeaf

目的：将给定的key从当前叶子结点删除。


#### 1.1.19. DeleteAtIndex

目的：将指定的key从当前结点为root的B+Tree中删除。

逻辑：递归的查找，将指定的key从叶子结点删除。如果删除后叶子节点中key数目不满足要求则进行merge和其他操作。

> 其实这里调用merge的逻辑个人认为有点问题。只是因为没有批量删除的操作，所以这样的实现是正确的。

> 其次则是处理了m_pointer[i]的节点数目下溢，可能当前结点自身也发生了结点数目下溢，感觉并没有处理这种情况。

问了学长之后知道了root结点的下溢在BpTree中做了特判。

### 1.2. NVMBpTree类

问题：pmem分配的空间，new之后不用delete吗?现在对于pmem内存的使用到释放还是不清楚。

> 在结合nvm_allocator.h和scaled_kv.cc大致弄清楚了pmem的使用

#### 1.2.1. Insert、Delete、Get、Search、GetRange、Get*

其他函数则是封装了底层NVMBpNode的函数。只有在insert、delete的时候对root做了额外处理。

### 1.3. BLevel类

1. CDFTable 对应论文中的Tier A
2. levelBentry 对应论文中的Tier B
3. BEntryCount B Level中的key总数目
4. ConstBEntryCount B Level中key的最大数目
5. CDFCount B Level中当前数目的key对应的CDFCount
6. ConstCDFCount B Level中最大的CDFCount


#### 1.3.1. FindAtBLevel

输入参数：

1. key  要查找的key
2. entry_offset 存放key在B Level中位置。

目的：找到给定的key在B Level中的位置。

逻辑：根据CDF函数的计算方式（论文中的计算公式1），得到比当前key小的数据比重。如果key比B中最小的key都小，那么要查找的key在B中的位置为0。如果key比B中最大的key都大，那么要查找的key在B中的位置为B的最后一个key。否则根据公式2得到给定的key在CDFTable中的位置，然后通过CDFTable来得到key在B中对应的位置。然后通过加一来获得比传入key大的key在B中的offset。然后通过FindAtCDFLess获得比传入key小的key在B中的offset。最后通过BinarySearchAtBLevel来获得key在B中的位置。

> 这里的问题是没有按照论文中的公式加一？是直接在计算结果的基础上直接加一？

> 继续看代码之后学长是通过cdfi_offset+1来确保获得比key更大的Tier B中offset，即end_offset。然后通过FindAtCDFLess来确保获得比key更小的Tier B的offset，即start_offset。这样来确保查找的正确性。

#### 1.3.2. GetCDFOffset

目的：获得Tier A(CDFTable)中第i个entry存放的key在Tier B中的offset。

#### 1.3.3. FindAtCDFLess

目的：在Tier A中向前查找，直到遇见一个entry存储的在Tier B中对应offset比传入参数更小的项。

#### 1.3.4. FindAtCDFGreate

目的：在Tier A中向后查找，直到遇见一个entry存储的在Tier B中对应offset比传入参数更大的项。

#### 1.3.5. BinarySearchAtBLevel

目的：二分查找key在Tier B中可能对应的位置。

> 这里完全可以写一个非递归版本的二分查找。

#### 1.3.6. Insert

目的：向BLevel结点中插入一个key/value对

逻辑：因为BLevel在构造之后就不能发生变化，所以新插入的key/value只能是插入到对应的C层light B+tree中。首先调用FindAtBLevel，得到key在BLevel中对应的offset。
1. 如果对应BLevel的entry当前直接指向一个值，那么需要构造一个Light B+tree（CNode）来存放原有数据和新插入的key/value。根据论文中的图5可以知道两个Tier B entry的Pointer指向的light B+tree的key满足的范围。据此根据FindAtBLevel得到的BEntryOffset来构造CNode。然后插入对应的数据，并设置entry对应的pointer。
2. 如果对应BLevel的entry当前已经指向一个CNode，那么直接插入就行了。

> 这样做法的唯一问题在于插入新的最小值的时候会有点理解上的问题。但是这些都封装在FindAtBLevel中处理了。

> 这里简明的补充一句：FindAtBLevel就是找到比传入key大的最小的key在BLevel中的offset。

> 同时这个函数调用的是insertAtLeaf，直接忽略了溢出的问题。


#### 1.3.7. Update

目的：更新给定key的value。

> 个人认为这里的update实现的有问题。默认update的key已经存在，没有做合法性检查。

#### 1.3.8. Delete

目的：删除给定的key

#### 1.3.9. Get*

目的：获取对应key的value。

#### 1.3.10. AppendBLevel

目的：用来在Expand时向BLevel添加key/value。

参数：
1. key 插入的key
2. value 插入的value
3. cdf_i 插入的key对应的CDF value
4. entry_count 当前Blevel的数据项数目
5. CDF_Count 当前BLevel的CDF项数
6. inertrimEntryCount 当前BLevel的数据项上限数目

> 参数其实还没有看懂。

逻辑：

1. 如果cdf_i小于0，说明插入的key比当前最小的key还要小，那么更新Tier B第0项对应的light B+Tree.
2. 如果当前CDF_Count等于0，说明插入的key是插入的第1个key。更新Tier B第0项对应的light B+Tree即可
3. 如果cdf_i大于1.0，那么说明插入的key比当前最大key都大。插入到最后一项即可。
4. 如果已经插入的项数达到了上限，南无调用InsertAtCNode进行结点扩展。否则直接插入即可。
5. 如果cdf_i大于当前已经插入的项目占所有项目的总和，那么说明需要插入一项新的CDF项了。


### 1.4. CNode 类

CNode类对应论文中的light B+-tree结构。KVEntry中的pointer既可以直接指向key对应的value，也可以指向对应range的light B+-tree。

1. nodeType 结点类型：Leaf64,Leaf128,Leaf256,Leaf512,Index256,Index512,Index1024,LeafNode
2. lastindex 当前结点中的key数目
3. prefix_len 存放的公共前缀的长度
4. totalEntry 当前结点能存放的最大key数目
5. index_count 当前结点中的索引数据项数目
6. prefix 公共前缀
7. prefix_entrys  数据项

对应中间的索引结点和最终的叶子结点，都是同一个CNode结构，所以其实是共用后面的数据段，只不过通过lastindex和index_count来分别表示叶子结点的数据部分和中间结点的索引部分。但其实二者的结构是一样的，两者加起来等于最初计算出来的totalentry。后续每次要插入一个indexentry，就会向数据部分借空间（totalentry--）。

#### 1.4.1. SortLeafNode

目的：将当前LeafNode按照key排序后的offset存放在index中返回。

> 是用冒泡排序来实现的。

#### 1.4.2. Split

目的：对当前CNode的key进行排序，将当前CNode的结点分裂出较大的一半到newnode中。

#### 1.4.3. ExpandIndexNode

目的：将当前索引结点的大小扩展为两倍

#### 1.4.4. InsertAtCNode

目的：向以当前CNode为root的子树中插入key/value对。如果插入后发生了节点数目上溢，则分配新结点，并将分配结果存储在resnewNode中。

逻辑：
1. 如果当前结点是叶子结点，则获取lastindex所在的entry的指针，然后用memcpy写入数据。
   1. 如果写入后当前结点满了，
      1. 如果当前节点类型是leaf256，则调用split生成一个新节点newnode，并将newnode写入当前结点的index部分（构成链表？）。
      2. 否则声明一个大小为当前结点2倍（nodeType*2是4倍？）的新节点并指向resnewnode，并把当前结点的内容都复制到新结点。
   2. 否则直接将lastidnex和写入的内容都持久化
2. 否则当前结点是中间索引结点:
   1. 如果当前索引结点类型是Index2048，说明当前结点已经满了，返回NormalRes（0？）
   2. 否则遍历indexentry，查找要插入的key对应的子树位置。
      1. 如果key插入对应的子树是0，那说明直接插入到当前结点就行了。如果插入后当前结点的lastindex部分满了（等于totalentry）：
         1. 如果index_count>=totalEntry（等于的时候说明index和数部份各占一半），说明需要扩展结点了:
            1. 如果当前结点的类型是Index1024，那说明无法继续扩展结点大小，则持久化写入的key，并返回NeedExpandRes(1)；
            2. 否则调用ExpandIndexNode，将扩展后的结果存放在resNewNode中。
         2. 如果index_count < totalEntry ，则只需要对当前结点进行split。并将newnode写入当前结点的index部分（构成链表？）。
      2. 如果i不是0，说明key在i-1对应的子树上直接获取对应的CNode，并进行和上面类似的操作。

> 其实这个函数实现的逻辑让我对index_count更加迷惑了。index_count>1的情况我还无法理解？。

> 询问学长之后得知：CNode对应是C层的结点，是一个类似B+树的结构，但是还是有一些区别的（因为我们C层的数据规模进行了控制，所以不需要向B+树那样预设高度并进行一个维护操作）。这部分论文好像没有描述或者描述的不是很清楚。

> 每个CNode前面存放自己的key值，初始没有index。当CNode存放的数据满了之后进行split操作， 将一半的数据写入一个新节点，这个新节点是叶子结点，没有index。而之前的CNode则有一个index指向新生成的叶子结点。

> 整个CNode构成的树最大高度为2

#### 1.4.5. Get、Update、Delete、GetRange

其他的基本操作就是基于这个二层的CNode结构上的简单操作了，不进行描述了。
要注意的是Delete操作采用的是SetValuePointer(nullptr)。

### 1.5. SuffixEntry类

SuffixENtry类对应light B+-tree中存放的数据项。由直接由key和pointer组成。

> 这里判断pointer类型的操作是因为申请内存的pointer末尾几位无用？

> 现在的疑问是什么时候调用pmem_persist？

### 1.6. NVMScaledKv类

成员：

1. bptree NVMBpTree 在构造ComboTree之前存放数据的B+Tree
2. blevel Blevel* ComboTree
3. interimBlevel BLevel* 猜想是在resize时的中间结构
4. entryCount uint64_t 数据量
5. limitEntry uint64_t 数据量上限
6. key_size_ size_t key大小
7. buf_size_ size_t buf？
8. state size_t 当前状态 0：KV_NONEXPAND 1：KV_EXPAND KV_EXPANDBLevel
9. ExpandKeyBefore char * expand时还没处理的Blevel的最小的key
10. ExpandKeyAfter char * expand时已经处理完的Blevel的最大的key

#### 1.6.1. Inittialize

目的：初始化一个NVMScaledKV。分配一个bptree，状态设置为0.

#### 1.6.2. Insert

目的：向ScaledKv中插入一项key/value。

逻辑：
1. 插入数据
   1. 如果当前还是bptree阶段，则直接向bptree插入。
   2. 如果在ExpandBLevel阶段，根据key的范围，选择向interimBlevel或者blevel插入。
   3. 否则直接向blevelcharu。
2. 插入后检查数据量是否达到数据上线，否则进行相应的expand。



#### 1.6.3. ExpandFromBpTree

目的：原先的BpTree大小达到限制，转换成ComboTree。

关键变量：
1. entry_count 写入BLevel中的entry数量
2. CDF_Count 写入CDFTable中的数量
3. EveryInterval 用于生成CDFTable的参数（span）。每EveryInterval个Entry生成一个CDF项。
4. interimEntryCount entryCount：BpTree中的数据数目
5. interimCDFCount 要生成的CDFTable大小

逻辑：
1. 声明一个CDFTable和BEntry大小都为0的BLevel。
2. 对于BpTree中的每一个key/value：
   1. 如果key对应的cdf值比当前已经写入的CDF项占全部CDF项比重大，那么说明要写入一个新的CDF项。调用interimBlevel->SetCDFOffset进行写入。然后CDF_Count++。
   2. entry_count++.

> 这里对于interimCDFCount+2的判断我有点没有想明白。

#### 1.6.4. ExpandFromBLevel




## 2. Question

### 2.1. BLevel::Insert

在计算择LeafType上没明白怎么选择的。

> ans:无所谓

FindAtBLevel就是找到比传入key大的最小的key在Tier B中的offset，不知道这样的理解是否正确。

> ans：正确

### 2.2. BLevel::AppendBLevel

1. 我们的Delete操作都是把value设置为null，什么时候会真正的删除？
2. CDF_Count==0的时候，在entry->GetKey的时候难道不需要判断是否为空吗？
3. entry_count >= interimEtnryCount-1表示超过了分配空间有点没弄懂
4. 对于判断CDF_Count>(ConstCDFCOunt+2)有点没弄明白

> ans:在进行expand的时候删除
> ans：NVMScaledKv中提前插入了一个Minkey，所以不会为空
> ans：这个情况在并行访问的时候可能会出现
> ans：NVMScaledKv中提前插入的（0，0）会有影响。然后是比BEntry中最大key还要大的key在进行span的时候可能会超出一。所以一起是加2
### 2.3. CNode：：InsertAtCNode

1. nodeType*2的操作有点懵，不是+1吗
2. 对于Index1024到Index2048的操作不是很懂，

> ans:leaf64为2 leaf256为4，没有leaf128
> ans：就是表示要进行expand了
### 2.4. CNode::InsertAtLeaf

lastindex>=totalEntry的时候直接lastindex=totalEntry，并不写入，这是什么逻辑？

> ans：应对并行访问错误的
### 2.5. NVMScaledKV::ExpandFromBpTree

1. CDF_Count 和 interimCDFCount+2的逻辑
2. limitEtnry的初始化

> ans：和上面一个一样
> ans：在initialize中
### 2.6. NVMScaledKV::ExpandFromBLevel

1. 对于BLevelOldCount的处理？这个offset不是对应空吗？
2. BLevel中MaxKey的存在有点奇怪。

> ans:比BEntry中最大key还要大的key存放在BLevelOdlCount中
> ans:就是对应比最大key还大的部分。