Java的4种类型，Class、Interface、Enum、Annotation 都属于Class类型



## 注解简介

- 也有人把它叫做注释、元数据，是一种代码级的说明
- 注解很像一个接口，多了一个 @ 符号，采用 @interface 作为关键字
- 用于生成文档
- 配合编译器分析检查代码，提升代码逻辑严谨性
- 代码功能的增强
- 修改程序的行为



## 元注解

- 用于定义注解

### @Target

- 声明注解的目标，就是可以在什么地方使用

- ElementType

```java
	/** Class, interface (including annotation type), or enum declaration 
	类，接口(包括注解类型)，枚举声明
	*/
    TYPE,

    /** Field declaration (includes enum constants) 
    字段(包括枚举常量)
    */
    FIELD,

    /** Method declaration */
    METHOD,

    /** Formal parameter declaration */
    PARAMETER,

    /** Constructor declaration */
    CONSTRUCTOR,

    /** Local variable declaration */
    LOCAL_VARIABLE,

    /** Annotation type declaration 用在元注解上 */
    ANNOTATION_TYPE,

    /** Package declaration */
    PACKAGE,

    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE
```

### @Retention

- 声明注解保留的策略级别

- RetentionPolicy

```java
/**
     * Annotations are to be discarded by the compiler.
     */
    SOURCE,

    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     * 默认值 
     */
    CLASS,

    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     *
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
```

### @Documented

- 声明解信息是否包含到 javadoc 文档中

  

### @Inherited





## 标记注解

@Deprecated

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})
public @interface Deprecated {
}
```

- 代表被标记的元素已经过时的、废弃的
- IDE 中一般会用删除线提示该元素不推荐使用了

@Override

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```


- 只能用在方法上
- 标记该方法进行了重写操作



@SuppressWarnings

```java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    /**
     * The set of warnings that are to be suppressed by the compiler in the
     * annotated element.  Duplicate names are permitted.  The second and
     * successive occurrences of a name are ignored.  The presence of
     * unrecognized warning names is <i>not</i> an error: Compilers must
     * ignore any warning names they do not recognize.  They are, however,
     * free to emit a warning if an annotation contains an unrecognized
     * warning name.
     *
     * <p> The string {@code "unchecked"} is used to suppress
     * unchecked warnings. Compiler vendors should document the
     * additional warning names they support in conjunction with this
     * annotation type. They are encouraged to cooperate to ensure
     * that the same names work across multiple compilers.
     * @return the set of warnings to be suppressed
     */
    String[] value();
}
```

- 抑制 IDE 的黄色下划线警告，用于忽略编译器的警告信息
- 需要传警告的类型



## 自定义注解



SOURCE

@Deprecated

@Override





RUNTIME

@Nullable

@NonNull





