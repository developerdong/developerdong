# Apifox导入/导出OpenAPI格式数据时的问题

Apifox虽然可以导入/导出OpenAPI格式的接口数据，但是并不是能够完美处理，有一些问题需要注意。

## 导入时的问题

Apifox的接口管理是支持子分类的。假设现在Apifox中的接口管理下有user这个分类，user下有account这个子分类，导出成OpenAPI文档时，会产生`user`和`user/account`这两个tag。

但是当再导入时，因为在user/account分类下有接口，但是user分类下没有接口，所以Apifox中只会出现以`user/account`为名的接口分类，而没有保持user和account的层级关系。

## 导出时的问题

Apifox的数据模型也是支持分类的，但是在导出时无法保留分类信息。例如有`admin/LoginRequest`和`user/LoginRequest`这两个数据模型，在Apifox中是不会冲突的，但是在导出的时候，因为无法保留分类信息，所以都叫LoginRequest会冲突。此时Apifox会自动将其中某一个进行重命名来解决冲突，但如此一来，当把导出的数据再导入时，就和导出前的数据不一致了。

## 如何避免这些问题

- 在接口管理中，只建立顶级分类，不建立子分类，所有接口放在不同的顶级分类中；
- 在数据模型中，只建立一个Schemas分类，所有数据模型放在此分类中。