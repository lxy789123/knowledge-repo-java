# Java版本新特性

[TOC]

## 一、Java9

### 1.模块化

```java
module java.module01 {
  //依赖
  requires java.module02;
  //
  exports java.module01.entity;
}
```

