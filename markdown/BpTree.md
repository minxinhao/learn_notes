# BpTree

这里整理一下BpTree相关的内容。
 

## BpNode

```cpp
//通过将value都设置为指针，使得中间节点和叶子结点可以公用一个结构。
//我们固定了key和value的大小。
struct BpNode{
    //BP_ORDER为每个节点最大的key/value对
    //BP_KEYSIZE为key的大小
    char keys[BP_ORDER][BP_KEYSIZE];  
    //子节点指针和value共用
    BpNode* values[BP_ORDER];   
    BpNode *prev, *next;
    uint8_t current_size;
    uint8_t nodeType; // IndexNode LeafNode
}  
```

### Insert

```cpp
void BpNode::Insert(string key, char *value) {
    if(IsLeafNode()) { //判断当前Node类型
        InsertAtLeaf(key, value);
    } else {
        InsertAtIndex(key, value);
    }
}
```

### InsertAtLeaf

```cpp
//value写成char*是因为这里只写入一个指针，而不复制指针指向的内容
void BpNode::InsertAtLeaf(string key, char *value)
{
    assert(current_size < BP_ORDER);
    int i;
    for (i = 0; i < current_size; ++i){
        //找到插入点
        if (keyCompare(key, keys[i], BP_KEYSIZE) <= 0)){
           break;
        }
    }
    //后续所有节点后移一个下标,对应指针同步操作
    for (int j = current_size - 1; j >= i; --j){
        SetKey(j + 1, keys[j]);
        SetPointer(j + 1, values[j]);  // 由于value和子节点共用不需要判断
    } 
    SetKey(i, key);
    SetValuePointer(i, value);
    SetCurrentSize(current_size + 1);
}
```

### InsertAtIndex

由于设置的和一般的BpTree有点不同，同时有n个key和n个指针，而一般的BpTree是n个key和n+1个指针，所以需要对大于最大key的结构而外处理。在从根节点到叶子节点的路径上维护的是最大key信息。

```cpp
void BpNode::InsertAtIndex(string key, char *value)
{
    int res = KeyCompare(key, keys[current_size-1],BP_KEYSIZE);
    //考虑Key值大于现在最大值
    if(res > 0){
        SetKey(current_size-1, key);
        values[current_size-1]->Insert(key, value);
        //判断current_size>=BP_ORDER是否成立
        if(values[current_size-1]->IsNeedSplit()){
            BpNode* left = nullptr;
            BpNode* right = nullptr;
            //把Node中排序好的key/value的分成(大致)相等的两份
            //小的那部分放在left中，大的那部分放在right中。
            values[current_size-1]->Split(left, right);

            //取left的最大key和left指针插入到当前索引节点
            string left_key(left->GetMaxKey(), BP_KEYSIZE);
            //InsertIndex向当前节点中插入一个key/pointer对
            InsertIndex(left_key, left);
        }
        return ;
    }
    //在以当前Node为根节点的子树中查找key要插入的位置
    for (int i = 0; i < current_size; ++i){
        int res = KeyCompare(key, keys[i], BP_KEYSIZE);
        if (res <= 0){
            values[i]->Insert(key, value);//递归调用
            if(values[i]->IsNeedSplit()){
                BpNode* left = nullptr;
                BpNode* right = nullptr;
                values[i]->Split(left, right);
                string left_key(left->GetMaxKey(),BP_KEYSIZE);
                InsertIndex(left_key, left);
            }
            break;//操作完成，即可退出
        }
    }
}

```

### Get

```cpp
char *BpNode::Get(string key) {
    for (int i = 0; i < current_size; ++i){
        if (KeyCompare(key, keys[i],BP_KEYSIZE) <= 0)
        {
            if(IsLeafNode()) {
                if(res == 0) {
                    return GetValuePointer(i);
                }
                else {
                    return nullptr;
                }
            } else {
                return values[i]->Get(key);
            }
        }
    }
    return nullptr;
}
```

### Delete

```cpp
bool BpNode::Delete(string key) {
    if(IsLeafNode()) {
        return DeleteAtLeaf(key);
    } else {
        return DeleteAtIndex(key);
    }
}
```

### DeleteAtLeaf

在一个叶子节点中遍历查找给定的key，如果找到就删除。

```cpp
bool BpNode::DeleteAtLeaf(string key)
{
    for (int i = 0; i < current_size; ++i)
    {
        int res = KeyCompare(key, keys[i], BP_KEYSIZE);
        if (res == 0)//找到待删除元素
        {
            //后续所有节点前移一个下标,对应指针同步操作
            for (int j = i; j < current_size - 1; ++j)
            {
                SetKey(j, keys[j+1]);
                SetPointer(j, values[j+1]);
            }
            SetCurrentSize(current_size - 1);
            return true;
        }
        else if (res < 0)
        { //进入此逻辑，说明查无此KEY
            break;
        }
    }

    return false;
}
```

### DeleteAtIndex

```cpp
bool BpNode::DeleteAtIndex(string key) {
    bool deleted = false;
    for (int i = 0; i < current_size; ++i){
        int res = KeyCompare(key, keys[i], BP_KEYSIZE);
        if (res <= 0){
            deleted = values[i]->Delete(key);
            //判断节点数目是否小于BP_ORDER/2
            if (values[i]->IsNeedRecoveryStructure())
            {
                //1. 先判断兄弟结点是否有多余
                if(i == 0)   // 节点只有右兄弟节点
                {
                    if(current_size == 1) {
                        ;
                    }
                    //节点数目是否大于BP_ORDER/2
                    else if(values[i + 1]->IsRedundant())
                    {
                        //将右节点最小的key/value对移动到当前节点
                        values[i]->borrowNext(values[i+1]);
                        SetKey(i, values[i]->GetMaxKey());
                    } 
                    else {
                        //将当前节点和右节点合并成一个节点
                        values[i]->mergeNext(values[i+1]);
                        SetKey(i, values[i]->GetMaxKey());
                        //如果values[i+1]是叶子节点，那么更新底层叶子节点的链表
                        values[i+1]->nodeDelete();
                        //在当前节点删除右节点
                        deleteIndex(i + 1);
                    }
                } 
                //如果有左节点则优先考虑使用左节点
                else {
                    if(values[i - 1]->IsRedundant())
                    {
                        values[i]->borrowPrev(values[i - 1]);
                        SetKey(i - 1, values[i - 1]->GetMaxKey());
                    } else {
                        values[i-1]->mergeNext(values[i]);
                        SetKey(i-1, values[i-1]->GetMaxKey());
                        values[i]->nodeDelete();
                        deleteIndex(i);
                    }
                }

            }
            if(KeyCompare(keys[i], values[i]->GetMaxKey(), BP_KEYSIZE) != 0{
                SetKey(i, values[i]->GetMaxKey());
            }
            return deleted;
        }
    }
    return deleted;
}
```


## BpTree

```cpp
struct BpTree{
    //根节点指向，根节点可能是IndexNode，也可能是LeafNode
    BpNode* root;
    //双向链起点指向
    BpNode* first_leaf;
    int height;
    int entry_count;
}
```

### Insert

```cpp
void BpTree::Insert(string key, char *pvalue)
{
    root->Insert(key, pvalue);//一路调用BpNode的Insert
    if(root->IsNeedSplit()){
        BpNode *left = nullptr;
        BpNode *right = nullptr;
        height ++;
		root->Split(left, right);
        char* mem = node_alloc->Allocate(NVM_NodeSize);
        BpNode* newRoot = new (mem) BpNode;
        newRoot->SetNodeType(IndexType);
        string left_key(left->GetMaxKey(),NVM_KeySize);
        string right_key(right->GetMaxKey(),NVM_KeySize);
        //插入索引信息
        newRoot->InsertIndex(left_key, left);
        newRoot->InsertIndex(right_key, right);
        //更新根节点指针
        root = newRoot; 
    }
    entry_count ++;
}
```

### Get

```cpp
string BpTree::Get(string key) {
    char *pvalue;
    if((pvalue = root->Get(key)) != nullptr){
        return string(pvalue, BP_VALUESIZE);
    }
    return "";
}
```

### Delete

```cpp
bool BpTree::Delete(string key)
{
    if((root->Delete(key)) == true){
        if(root->GetSize() == 1 && !root->IsLeafNode()) {
            treeLevel --;
            root = root->GetPointer(0);
        }
        entry_count --;
        return true;
    }
    return false;   
}
```