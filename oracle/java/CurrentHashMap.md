```java
final V putVal(K key, V value, boolean onlyIfAbsent){
  if(key == null || value = null) throw new NullPointerException();
  int hash = spread(key.hashCode());
  int binCount = 0;
  for(Node<K,V>[] tab = table;;){
    Node<K,V> f; int n,i,fh;
    if(tab == null || (n = tab.lenght) == 0){
      tab = initTable();//初始化一个槽
    } else if((f= tabAt(tab,i=(n-1)&hash))==null){ // tabAt tab 中指定位置对象
      if(casTabAt(tab,i,null,new Node<K,V>(hash,key,value,null))){ // CAS 插入数据
        break;
      }
    } else if ((fh = f.hash) == MOVED){ // 对象处于移动则进行转换
      tab = helpTransfer(tab,f);
    } else {
      V oldVal = null;
      synchronized(f){ //锁单个对象槽
        if(tabAt(tab,i) == f){
          if(fh >= 0 ){
            binCount =1;
            for(Node<k,V> e = f ;; ++binCount){
              K ek;
              if(e.hash == hash && ((ek = e.key)==key || (ek != null && key,equals(ek)))){
                oldVal = e.val;
                if(!onlyIfAbsent){
                  e.val = value;
                }
                break;
              }
              Node<K,V> pred = e;
              if((e = e.next) == null){
                pred.next = new Node<K,V>(hash,key,value,null);
                break;
              }
            }
          } else if(f instanceof TreeBin){ // 红黑树
            Node<K,V> p;
            binCount = 2 ;
            if((p = ((TreeBin<k,V>)f).putTreeVal(hash,key,value))!=null){ // 红黑树 add 方法
              oldVal = p.val;
              if(!onlyIfAbsent){
                p.val = value;
              }
            }
          }
        }
        if(binCount != 0){
          if(binCount > TREEIFY_THRESHOLD)
            treeifyBin(tab,i);
          if(oldVal != null)
            return oldVal;
          break;
        }
      }
    }
  }
  addCount(1L,binCount);
  return null;
}
```
