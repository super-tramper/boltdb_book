# 第二节 元数据页

每页有一个meta\(\)方法，如果该页是元数据页的话，可以通过该方法来获取具体的元数据信息。

```go
// meta returns a pointer to the metadata section of the page.
func (p *page) meta() *meta {
    // 将p.ptr转为meta信息
    return (*meta)(unsafe.Pointer(&p.ptr))
}
```

详细的元数据信息定义如下：

```go
type meta struct {
    magic    uint32 //魔数
    version  uint32 //版本
    pageSize uint32 //page页的大小，该值和操作系统默认的页大小保持一致
    flags    uint32 //保留值，目前貌似还没用到
    root     bucket //所有小柜子bucket的根
    freelist pgid //空闲列表页的id
    pgid     pgid //数据库文件中最大的page id + 1
    txid     txid //最大的事务id
    checksum uint64 //用作校验的校验和
}
```

下图展现的是元信息存储方式。

![../imgs/&#x5143;&#x4FE1;&#x606F;&#x5B58;&#x50A8;.png](../.gitbook/assets/元信息存储.png)

下面我们重点关注该meta数据是如何写入到一页中的，以及如何从磁盘中读取meta信息并封装到meta中

**1. meta-&gt;page**

```go
db.go

// write writes the meta onto a page.
func (m *meta) write(p *page) {
    if m.root.root >= m.pgid {
        panic(fmt.Sprintf("root bucket pgid (%d) above high water mark (%d)", m.root.root, m.pgid))
    } else if m.freelist >= m.pgid {
        panic(fmt.Sprintf("freelist pgid (%d) above high water mark (%d)", m.freelist, m.pgid))
    }

    // Page id is either going to be 0 or 1 which we can determine by the transaction ID.
    //指定页id和页类型
    p.id = pgid(m.txid % 2)
    p.flags |= metaPageFlag

    // Calculate the checksum.
    m.checksum = m.sum64()
  // 这儿p.meta()返回的是p.ptr的地址，因此通过copy之后，meta信息就放到page中了
    m.copy(p.meta())
}


// copy copies one meta object to another.
func (m *meta) copy(dest *meta) {
    *dest = *m
}


// generates the checksum for the meta.
func (m *meta) sum64() uint64 {
    var h = fnv.New64a()
    _, _ = h.Write((*[unsafe.Offsetof(meta{}.checksum)]byte)(unsafe.Pointer(m))[:])
    return h.Sum64()
}
```

**2. page-&gt;meta**

```go
page.go

// meta returns a pointer to the metadata section of the page.
func (p *page) meta() *meta {
    // 将p.ptr转为meta信息
    return (*meta)(unsafe.Pointer(&p.ptr))
}
```

