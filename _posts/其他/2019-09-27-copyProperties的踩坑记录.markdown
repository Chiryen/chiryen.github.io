---
layout: post
title:  "copyProperties的踩坑记录"
date:   2019-09-27 11:34:53 +0800
categories: 其他
---

### 问题

最近，在开发过程中使用了spring的BeanUtils.copyProperties对bean进行拷贝。在调试过程中出现异常信息:  java.lang.reflect.InvocationTargetException  
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) 

模拟一下当时场景，代码如下。

	public class MetaDTO {

	    private String name;
	
	    private List<ColumnDTO> columns;
	
	    public String getName() {
	        return name;
	    }
	
	    public void setName(String name) {
	        this.name = name;
	    }
	
	    public List<ColumnDTO> getColumns() {
	        return columns;
	    }
	
	    public void setColumns(List<ColumnDTO> columns) {
	        this.columns = columns;
	    }
	}
	
	public class ColumnDTO {
	
	    private Integer id;
	
	    private String name;
	
	    public Integer getId() {
	        return id;
	    }
	
	    public void setId(Integer id) {
	        this.id = id;
	    }
	
	    public String getName() {
	        return name;
	    }
	
	    public void setName(String name) {
	        this.name = name;
	    }
	}
    
    public class MetaBO {

	    private String name;
	
	    private List<ColumnBO> columns;
	
	    public String getName() {
	        return name;
	    }
	
	    public void setName(String name) {
	        this.name = name;
	    }
	
	    public List<ColumnBO> getColumns() {
	        return columns;
	    }
	
	    public void setColumns(List<ColumnBO> columns) {
	        this.columns = columns;
	    }
	}

    public class ColumnBO {

	    private Integer id;
	
	    private String name;
	
	    public Integer getId() {
	        return id;
	    }
	
	    public void setId(Integer id) {
	        this.id = id;
	    }
	
	    public String getName() {
	        return name;
	    }
	
	    public void setName(String name) {
	        this.name = name;
	    }
	}
	
	
以上代码实际上是两个实体类MetaDTO、MetaBO, 这两个类中分别引用了ColumnDTO、ColumnBO。下面的main方法中内容是通过spring的BeanUtils.copyProperties将metaDTO对象的属性拷贝到metaBO。
	
	
	public class BeanUtilsDemo {
	
	    public static void main(String[] args) {
	
	        MetaDTO metaDTO = new MetaDTO();
	        metaDTO.setName("metaDTO");
	        ColumnDTO columnDTO = new ColumnDTO();
	        columnDTO.setId(1);
	        columnDTO.setName("ColumnDTO");
	        metaDTO.setColumns(Lists.newArrayList(columnDTO));
	
	        MetaBO metaBO = new MetaBO();
	        BeanUtils.copyProperties(metaDTO, metaBO);  // 重点
	
	        List<ColumnBO> columns = new ArrayList<>(metaBO.getColumns());
	        ColumnBO columnBO = columns.get(0);  // 报错
	        System.out.println(columnBO);
	    }
	}
   
   
这些是从工程项目中抽取出来的主要代码，整个逻辑比较简单，重点就是BeanUtils.copyProperties(metaDTO, metaBO)这一句。在本地运行时，在ColumnBO columnBO = columns.get(0)这里报错  
java.lang.ClassCastException: bean.ColumnDTO cannot be cast to bean.ColumnBO  

### 分析

原因是BeanUtils.copyProperties使用反射机制metaDTO拷贝到目标对象metaBO，而使用反射会进行泛型擦除(java的泛型仅仅存在于编译阶段，编译成的字节码文件事实上已经没有泛型了)。调试时发现右值columns.get(0)实际上是ColumnDTO类的实例。

下面看下copyProperties的核心代码。

	private static void copyProperties(Object source, Object target, Class<?> editable, String... ignoreProperties) throws BeansException {
	    Class<?> actualEditable = target.getClass();
	
	    PropertyDescriptor[] targetPds = getPropertyDescriptors(actualEditable); 
	    List<String> ignoreList = ignoreProperties != null ? Arrays.asList(ignoreProperties) : null;
	    PropertyDescriptor[] var7 = targetPds;
	    int var8 = targetPds.length;
	
	    // 遍历目标对象中的属性
	    for(int var9 = 0; var9 < var8; ++var9) {
	        PropertyDescriptor targetPd = var7[var9];
	        Method writeMethod = targetPd.getWriteMethod();
	        if (writeMethod != null && (ignoreList == null || !ignoreList.contains(targetPd.getName()))) {
	            PropertyDescriptor sourcePd = getPropertyDescriptor(source.getClass(), targetPd.getName()); // 根据目标对象的属性名找到源对象的属性描述器
	            if (sourcePd != null) {
	                Method readMethod = sourcePd.getReadMethod();
	                if (readMethod != null && ClassUtils.isAssignable(writeMethod.getParameterTypes()[0], readMethod.getReturnType())) {
	                    try {
	                        if (!Modifier.isPublic(readMethod.getDeclaringClass().getModifiers())) {
	                            readMethod.setAccessible(true);
	                        }
	
	                        Object value = readMethod.invoke(source); // 相当于get方法，获取源对象的属性值
	                        if (!Modifier.isPublic(writeMethod.getDeclaringClass().getModifiers())) {
	                            writeMethod.setAccessible(true);
	                        }
	
	                        writeMethod.invoke(target, value); // 相当于set方法，这里会泛型擦除
	                    } catch (Throwable var15) {
	                        throw new FatalBeanException("Could not copy property '" + targetPd.getName() + "' from source to target", var15);
	                    }
	                }
	            }
	        }
	    }

	}
   
   
从BeanUtils.copyProperties的源码中可以看出，属性的赋值是通过writeMethod.invoke(target, value)实现的，没有了泛型控制。回到文中的Demo，metaBO对象的columns实际上引用的是metaDTO的columns，还是List<ColumnDTO>类型。

那为什么在工程项目中抛出的异常是  
java.lang.reflect.InvocationTargetException  
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)  

InvocationTargetException是反射异常，当被调用的方法的内部抛出了异常而没有被捕获时，将由此异常接收。猜想是工程项目中使用底层框架在执行问题代码时是使用了动态代理，而动态代理使用了反射机制。也就是说在类型强制转换时ClassCastException没有被捕获，所以抛出InvocationTargetException。

### 思考
这个问题并不难，但当时排查花了挺多时间，因为一开始找错了方向。在实际工程代码中还用到了guava的EventBus组件，它相对于一个内存中的消息队列。我一开始以为是EventBus的问题，于是按"异常信息 + EventBus" 谷歌了好久，搜到了好几种解决方案，都一一试过都不行，花费了很多时间。最后在单步调试时发现问题所在。  
这也反映了我的几点问题：
	
	+ 遇到问题时，容易怀疑是不熟的东西在搞鬼。比如EventBus，我之前并没有了解过，不知道它的底层实现，在找不到问题原因时，就总觉得是跟它有关系。这一点和群体关系比较相似，在一个小群体中，出现了摸不着头脑的问题，很多人都会怀疑是可能是那个和大家都不熟的人造成的。出现这种想法，实际上对熟悉的东西没有完成掌握清楚，却又过分信赖。
	+ 过早地去谷歌，没有对当前问题做足够的分析。拿错误信息去谷歌，然后无头绪地一个个去试，真的是一个很花费时间的解决问题的方式。