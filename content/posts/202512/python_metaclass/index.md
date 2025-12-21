---
title: "python理解metaclass,类/实例属性查找机制"
date: 2025-12-21T19:36:02+08:00
draft: false
summary: "python理解metaclass,类/实例属性查找机制"
tags: ["python","metaclass"]
---

# python理解metaclass,类/实例属性查找机制

## metaclass 

 ``` pyt
 class M(type):
     @classmethod
     def __prepare__(metacls, name, bases, **kwargs):
         namespace = dict()
 
         # 注入一个工具函数，类定义体里可以直接用
         namespace['add'] = lambda a, b: a + b
         namespace['BASE_VALUE'] = 100
 
         return namespace
 
 
 class Test(metaclass=M):
     result = add(1, 2)  
     total = BASE_VALUE + 50  
     
 test = Test()
 print(Test.result)  #3
 ```

这里是一个元类,这里直接塞入了一个add方法,这样所有被这个元类创建的类都可以使用这个方法

### 元类的意义

控制类的创建过程

``` python
class MyMeta(type):
  	@classmethod
    def __prepare__(metacls, name, bases):    # 1. 控制类定义体的执行环境
        ...
    def __new__(cls, name, bases, ns):    # 2. 控制类对象的创建
        ...
    def __init__(cls, name, bases, ns):   # 3. 控制类对象的初始化
        ...
    def __call__(cls, *args):             # 4. 控制实例的创建
        ...
```

例子: 自动注册搜集类

``` python
registry = {}

class RegisterMeta(type):
    def __new__(cls, name, bases, ns):
        klass = super().__new__(cls, name, bases, ns)
        if name != 'Base':
            registry[name] = klass  
        return klass

class Base(metaclass=RegisterMeta): pass
class Dog(Base): pass
class Cat(Base): pass

print(registry)  # {'Dog': <class Dog>, 'Cat': <class Cat>}
```



``` python
def register(cls):
    original_init = cls.__init__
    def new_init(self, *args, **kwargs):
        original_init(self, *args, **kwargs)
        registry.append(self) 
    cls.__init__ = new_init
    return cls

@register
class Handler:
    def __init__(self, name):
        self.name = name

h1 = Handler("h1")
h2 = Handler("h2")
print(registry)  
```





### 和装饰器的区别

装饰器是使用了闭包函数,常常用来管理实例,两者使用的场景:

| **场景**                        | **装饰器**                    | **元类**             |
| ------------------------------- | ----------------------------- | -------------------- |
| **单个类增强**                  | ✓                             | ✗  没必要,太重       |
| **可选功能 (用不用随意)**       | ✓                             | ✗ 一般也是具体的增强 |
| **修改实例行为**                | ✓                             | ✗ 实例管理           |
| **所有子类自动继承行为**        | ✗                             | ✓                    |
| **类定义时验证**                | △ 如果特定一处用,可以用装饰器 | ✓                    |
| **控制类的创建过程**            | ✗                             | ✓                    |
| **框架级别 (ORM/序列化)**       | ✗                             | ✓                    |
| **需要 `__prepare__` 注入环境** | ✗                             | ✓                    |



## 类,实例和元类的关系

### 实例查找图谱

![Diagram of normal attribute lookup on objects](./assets/object-attribute-lookup-v3.png)

解释: 

1.type(obj) 的 MRO 找 data descriptor 

2.如果查询不到再回到`obj.__dict__`获取

3.type(obj) 的 MRO 找 non-data / 普通属性

4. 如果还是获取不到,`__getattr__ `兜底

__注意:不会走metaclass查询属性方法!__

__mro会扫两遍__

### 类查询图谱

![Diagram of normal attribute lookup on classes](./assets/class-attribute-lookup-v3.png)

解释

1. Metaclass MRO 找 data descriptor [M → type]
2. Class MRO 找 任何 descriptor 或普通属性 [Class → bases → object]
3. Metaclass MRO 找 non-data descriptor 或普通属性 [M → type]
4. Metaclass.**getattr** 兜底

__会查找元类的`__dict__`属性__

__mro扫描两次__

### magic method

> One distinctive feature of Python is magic methods: they allow the programmer to override behavior for various operators and behavior of objects.  

__Magic Method 隐式调用（len(obj)、obj + x）, 直接查 type(obj)，跳过 `instance.__dict__` ,和普通属性查找不同__

### 属性查找示例代码

``` python

# 继承链: C -> B -> A -> object
# 元类链: MyMeta -> type


print("=" * 60)
print("Part 0: 结构定义")
print("=" * 60)


# ---------- Data Descriptor ----------
class DataDescriptor:
    """Data descriptor: 有 __get__ 和 __set__"""

    def __init__(self, name):
        self.name = name

    def __get__(self, instance, owner):
        print(f"    [DataDescriptor.__get__] name={self.name}, instance={instance}, owner={owner}")
        if instance is None:
            return self
        return f"DataDesc({self.name})"

    def __set__(self, instance, value):
        print(f"    [DataDescriptor.__set__] name={self.name}, value={value}")


# ---------- Non-Data Descriptor ----------
class NonDataDescriptor:
    """Non-data descriptor: 只有 __get__"""

    def __init__(self, name):
        self.name = name

    def __get__(self, instance, owner):
        print(f"    [NonDataDescriptor.__get__] name={self.name}, instance={instance}, owner={owner}")
        if instance is None:
            return self
        return f"NonDataDesc({self.name})"


# ---------- Metaclass ----------
class MyMeta(type):
    """自定义元类"""

    # 元类上的 data descriptor
    meta_data_desc = DataDescriptor("meta_data_desc")

    # 元类上的 non-data descriptor
    meta_nondata_desc = NonDataDescriptor("meta_nondata_desc")

    # 元类上的普通属性
    meta_attr = "MyMeta.meta_attr"

    # 元类上的 __len__（magic method）
    def __len__(cls):
        print(f"    [MyMeta.__len__] called on {cls}")
        return 999

    def __getattr__(cls, name):
        print(f"    [MyMeta.__getattr__] name={name}")
        return f"MyMeta.__getattr__({name})"


# ---------- Class A (基类) ----------
class A(metaclass=MyMeta):
    # A 上的 data descriptor
    a_data_desc = DataDescriptor("a_data_desc")

    # A 上的 non-data descriptor
    a_nondata_desc = NonDataDescriptor("a_nondata_desc")

    # A 上的普通属性
    a_attr = "A.a_attr"

    # 会被 B 覆盖的属性
    inherited_attr = "A.inherited_attr"

    def __len__(self):
        print(f"    [A.__len__] called on {self}")
        return 100

    def __getattr__(self, name):
        print(f"    [A.__getattr__] name={name}")
        return f"A.__getattr__({name})"


# ---------- Class B (中间类) ----------
class B(A):
    # B 上的 data descriptor
    b_data_desc = DataDescriptor("b_data_desc")

    # 覆盖 A 的属性
    inherited_attr = "B.inherited_attr"

    # B 上的普通属性
    b_attr = "B.b_attr"


# ---------- Class C (最终类) ----------
class C(B):
    # C 上的 data descriptor
    c_data_desc = DataDescriptor("c_data_desc")

    # C 上的 non-data descriptor
    c_nondata_desc = NonDataDescriptor("c_nondata_desc")

    # C 上的普通属性
    c_attr = "C.c_attr"

    def __init__(self):
        # 实例属性
        self.instance_attr = "c.__dict__['instance_attr']"
        # 尝试覆盖 non-data descriptor
        self.__dict__['c_nondata_desc'] = "c.__dict__['c_nondata_desc']"
        # 尝试覆盖 data descriptor (不会生效)
        self.__dict__['c_data_desc'] = "c.__dict__['c_data_desc']"


# 创建实例
c = C()

print(f"""
继承链 (MRO):
  C.__mro__ = {C.__mro__}

实例化链:
  c.__class__ = {c.__class__}
  C.__class__ = {C.__class__}
  MyMeta.__class__ = {MyMeta.__class__}

c.__dict__ = {c.__dict__}
""")

# ============================================
print("=" * 60)
print("Part 1: Instance 属性查找 (c.attr)")
print("=" * 60)

print("\n--- 1.1 Data Descriptor 优先于 instance.__dict__ ---")
print("c.c_data_desc =", c.c_data_desc)
print("  → 虽然 c.__dict__ 有 'c_data_desc'，但 data descriptor 优先")
c_dict_c_data_desc=c.__dict__["c_data_desc"]
print(f"c.__dict__中c_data_desc的值是{c_dict_c_data_desc},这个值来自于instance的__dict__，但没有被访问到")

print("\n--- 1.2 instance.__dict__ 优先于 Non-Data Descriptor ---")
print("c.c_nondata_desc =", c.c_nondata_desc)
print("  → c.__dict__['c_nondata_desc'] 覆盖了 non-data descriptor")

print("\n--- 1.3 instance.__dict__ 普通属性 ---")
print("c.instance_attr =", c.instance_attr)

print("\n--- 1.4 沿 MRO 找 Non-Data Descriptor (A 上定义的) ---")
print("c.a_nondata_desc =", c.a_nondata_desc)

print("\n--- 1.5 沿 MRO 找普通类属性 ---")
print("c.a_attr =", c.a_attr)
print("c.b_attr =", c.b_attr)
print("c.inherited_attr =", c.inherited_attr, "(B 覆盖了 A)")

print("\n--- 1.6 __getattr__ 兜底 ---")
print("c.not_exist =", c.not_exist)

print("\n--- 1.7 Metaclass 属性对 instance 不可见 ---")

print("c.meta_attr =", c.meta_attr)
print("  → 报错！instance 不会查 metaclass ,走兜底检查__getattr__")

# ============================================
print("\n" + "=" * 60)
print("Part 2: Class 属性查找 (C.attr)")
print("=" * 60)

print("\n--- 2.1 Metaclass Data Descriptor 优先 ---")
print("C.meta_data_desc =", C.meta_data_desc)

print("\n--- 2.2 Class MRO 上的 Descriptor ---")
print("C.c_data_desc =", C.c_data_desc)
print("  → instance=None 说明是通过类访问")

print("\n--- 2.3 Class MRO 上的普通属性 ---")
print("C.c_attr =", C.c_attr)
print("C.a_attr =", C.a_attr)

print("\n--- 2.4 Metaclass Non-Data Descriptor ---")
print("C.meta_nondata_desc =", C.meta_nondata_desc)

print("\n--- 2.5 Metaclass 普通属性 ---")
print("C.meta_attr =", C.meta_attr)

print("\n--- 2.6 Metaclass __getattr__ 兜底 ---")
print("C.class_not_exist =", C.class_not_exist)

# ============================================
print("\n" + "=" * 60)
print("Part 3: Magic Method 隐式调用")
print("=" * 60)

print("\n--- 3.1 len(instance) 直接查 type(instance) ---")
print("len(c) =", len(c))
print("  → 直接调用 A.__len__，跳过 instance.__dict__")

print("\n--- 3.2 len(Class) 直接查 type(Class) 即 Metaclass ---")
print("len(C) =", len(C))
print("  → 直接调用 MyMeta.__len__")

print("\n--- 3.3 对比：显式调用 vs 隐式调用 ---")
c.__len__ = lambda: 777  # 塞进 instance.__dict__
print("c.__len__() =", c.__len__(), "  (显式调用，走正常属性链)")
print("len(c) =", len(c), "       (隐式调用，跳过 instance.__dict__)")

```





## 类创建流程

> - `Metaclass.__prepare__` just returns the namespace object (a dictionary-like object as explained before).
> - `Metaclass.__new__` returns the `Class` object.
> - `MetaMetaclass.__call__` returns whatever `Metaclass.__new__` returned (and if it returned an instance of `Metaclass` it will also call `Metaclass.__init__` on it).

![Diagram of class creation workflow](./assets/class-creation.png)

``` python				

class MyMeta(type):

    # 1. 类定义前：准备 namespace
    @classmethod
    def __prepare__(metacls, name, bases, **kwargs):
        print(f"[1] __prepare__: 准备 {name} 的 namespace")
        print(f"    metacls = {metacls}")
        namespace = {'injected': 100}
        return namespace

    # 2. 类定义后：创建类对象
    def __new__(metacls, name, bases, namespace):
        print(f"[2] __new__: 创建类 {name}")
        print(f"    namespace = {dict(namespace)}")
        cls = super().__new__(metacls, name, bases, namespace)
        return cls

    # 3. 类对象创建后：初始化类对象
    def __init__(cls, name, bases, namespace):
        print(f"[3] __init__: 初始化类 {name}")
        cls.added_by_init = "元类 __init__ 添加的"
        super().__init__(name, bases, namespace)

    # 4. 类被调用时：控制实例创建
    def __call__(cls, *args, **kwargs):
        print(f"[4] __call__: 创建 {cls.__name__} 的实例")
        print(f"    args = {args}")
        instance = super().__call__(*args, **kwargs)
        instance.added_by_call = "元类 __call__ 添加的"
        return instance

```



## 总结

实例只能看到类定义的属性，但看不到元类定义的属性。类既能看到自己定义的，也能看到元类定义的。唯一的例外是魔术方法，它永远只看 type(obj)。

双轨机制

## 参考

[Understanding Python metaclasses](https://blog.ionelmc.ro/2015/02/09/understanding-python-metaclasses/)
