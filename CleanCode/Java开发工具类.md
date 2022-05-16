---
typora-copy-images-to: upload
---

## Java开发工具类

### 对象与`Map`转换

1. 对象转`Map<String, String>`

```java
Student student = new Student();
try {
  Map<String, String> map = org.apache.commons.beanutils.BeanUtils.describe(student);
} catch (IllegalAccessException e) {
  e.printStackTrace();
} catch (InvocationTargetException e) {
  e.printStackTrace();
} catch (NoSuchMethodException e) {
  e.printStackTrace();
}
```

### Git远程仓库回滚

> 1. 通过IDEA将本地分支重置到要回滚到的记录，已提交的代码会恢复到未提交状态
>
> ![image-20220516214045177](https://tva1.sinaimg.cn/large/e6c9d24ely1h2aktdsjs5j20ut0u0112.jpg)
>
> ![image-20220516222259269](https://tva1.sinaimg.cn/large/e6c9d24ely1h2am0zm654j20nq0gg40g.jpg)
>
> 2. 将本地未提交的代码回滚，然后git push -f 强制覆盖远程的分支
