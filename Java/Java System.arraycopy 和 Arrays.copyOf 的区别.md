# System.arraycopy 和 Arrays.copyOf 的区别



## System.arraycopy 

- 只有数组为一维数组且元素为基本类型、String 类型的时候是深拷贝，其它情况下都属于浅拷贝，比如元素是引用类型、二维数组的情况
- 调用的是 native 方法，性能好
- 需要传入 dest
- 可以指定起始位置和拷贝的长度，比较灵活

```java

    /**
     * Copies an array from the specified source array, beginning at the
     * specified position, to the specified position of the destination array.
     * A subsequence of array components are copied from the source
     * array referenced by <code>src</code> to the destination array
     * referenced by <code>dest</code>. The number of components copied is
     * equal to the <code>length</code> argument. The components at
     * positions <code>srcPos</code> through
     * <code>srcPos+length-1</code> in the source array are copied into
     * positions <code>destPos</code> through
     * <code>destPos+length-1</code>, respectively, of the destination
     * array.
     * <p>
     * If the <code>src</code> and <code>dest</code> arguments refer to the
     * same array object, then the copying is performed as if the
     * components at positions <code>srcPos</code> through
     * <code>srcPos+length-1</code> were first copied to a temporary
     * array with <code>length</code> components and then the contents of
     * the temporary array were copied into positions
     * <code>destPos</code> through <code>destPos+length-1</code> of the
     * destination array.
     * <p>
     * If <code>dest</code> is <code>null</code>, then a
     * <code>NullPointerException</code> is thrown.
     * <p>
     * If <code>src</code> is <code>null</code>, then a
     * <code>NullPointerException</code> is thrown and the destination
     * array is not modified.
     * <p>
     * Otherwise, if any of the following is true, an
     * <code>ArrayStoreException</code> is thrown and the destination is
     * not modified:
     * <ul>
     * <li>The <code>src</code> argument refers to an object that is not an
     *     array.
     * <li>The <code>dest</code> argument refers to an object that is not an
     *     array.
     * <li>The <code>src</code> argument and <code>dest</code> argument refer
     *     to arrays whose component types are different primitive types.
     * <li>The <code>src</code> argument refers to an array with a primitive
     *    component type and the <code>dest</code> argument refers to an array
     *     with a reference component type.
     * <li>The <code>src</code> argument refers to an array with a reference
     *    component type and the <code>dest</code> argument refers to an array
     *     with a primitive component type.
     * </ul>
     * <p>
     * Otherwise, if any of the following is true, an
     * <code>IndexOutOfBoundsException</code> is
     * thrown and the destination is not modified:
     * <ul>
     * <li>The <code>srcPos</code> argument is negative.
     * <li>The <code>destPos</code> argument is negative.
     * <li>The <code>length</code> argument is negative.
     * <li><code>srcPos+length</code> is greater than
     *     <code>src.length</code>, the length of the source array.
     * <li><code>destPos+length</code> is greater than
     *     <code>dest.length</code>, the length of the destination array.
     * </ul>
     * <p>
     * Otherwise, if any actual component of the source array from
     * position <code>srcPos</code> through
     * <code>srcPos+length-1</code> cannot be converted to the component
     * type of the destination array by assignment conversion, an
     * <code>ArrayStoreException</code> is thrown. In this case, let
     * <b><i>k</i></b> be the smallest nonnegative integer less than
     * length such that <code>src[srcPos+</code><i>k</i><code>]</code>
     * cannot be converted to the component type of the destination
     * array; when the exception is thrown, source array components from
     * positions <code>srcPos</code> through
     * <code>srcPos+</code><i>k</i><code>-1</code>
     * will already have been copied to destination array positions
     * <code>destPos</code> through
     * <code>destPos+</code><i>k</I><code>-1</code> and no other
     * positions of the destination array will have been modified.
     * (Because of the restrictions already itemized, this
     * paragraph effectively applies only to the situation where both
     * arrays have component types that are reference types.)
     *
     * @param      src      the source array.
     * @param      srcPos   starting position in the source array.
     * @param      dest     the destination array.
     * @param      destPos  starting position in the destination data.
     * @param      length   the number of array elements to be copied.
     * @exception  IndexOutOfBoundsException  if copying would cause
     *               access of data outside array bounds.
     * @exception  ArrayStoreException  if an element in the <code>src</code>
     *               array could not be stored into the <code>dest</code> array
     *               because of a type mismatch.
     * @exception  NullPointerException if either <code>src</code> or
     *               <code>dest</code> is <code>null</code>.
     */
    //src 源数组
	//srcPos 源数组起始位置
	//dest 目标数组
	//destPos 目标数组起始位置
	//length 要拷贝的长度（要拷贝的元素数量）
    public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```



测试

```java
       String[] array = {"a", "b", "c", "b", "d"};
        System.out.println("array.length:" + array.length);
        String[] arrayCopy=new String[10];
        System.out.println("arrayCopy.length:" + arrayCopy.length);
        System.arraycopy(array,1,arrayCopy,1,4);
        System.out.println(Arrays.toString(array));
        System.out.println(Arrays.toString(arrayCopy));
        array[2] = "cc";
        System.out.println(Arrays.toString(array));
        System.out.println(Arrays.toString(arrayCopy));
//-------------------------- 结果 ---------------------
array.length:5
arrayCopy.length:10
[a, b, c, b, d]
[null, b, c, b, d, null, null, null, null, null]
[a, b, cc, b, d]
[null, b, c, b, d, null, null, null, null, null]
```



## Arrays.copyOf 

- 调用了 System.arraycopy 方法
- 适合目标数组不明确的情况下使用，自动创建新的数组

```java
    /**
     * Copies the specified array, truncating or padding with nulls (if necessary)
     * so the copy has the specified length.  For all indices that are
     * valid in both the original array and the copy, the two arrays will
     * contain identical values.  For any indices that are valid in the
     * copy but not the original, the copy will contain <tt>null</tt>.
     * Such indices will exist if and only if the specified length
     * is greater than that of the original array.
     * The resulting array is of exactly the same class as the original array.
     *
     * @param <T> the class of the objects in the array
     * @param original the array to be copied
     * @param newLength the length of the copy to be returned
     * @return a copy of the original array, truncated or padded with nulls
     *     to obtain the specified length
     * @throws NegativeArraySizeException if <tt>newLength</tt> is negative
     * @throws NullPointerException if <tt>original</tt> is null
     * @since 1.6
     */
    @SuppressWarnings("unchecked")
    public static <T> T[] copyOf(T[] original, int newLength) {
        return (T[]) copyOf(original, newLength, original.getClass());
    }
    /**
     * Copies the specified array, truncating or padding with nulls (if necessary)
     * so the copy has the specified length.  For all indices that are
     * valid in both the original array and the copy, the two arrays will
     * contain identical values.  For any indices that are valid in the
     * copy but not the original, the copy will contain <tt>null</tt>.
     * Such indices will exist if and only if the specified length
     * is greater than that of the original array.
     * The resulting array is of the class <tt>newType</tt>.
     *
     * @param <U> the class of the objects in the original array
     * @param <T> the class of the objects in the returned array
     * @param original the array to be copied
     * @param newLength the length of the copy to be returned
     * @param newType the class of the copy to be returned
     * @return a copy of the original array, truncated or padded with nulls
     *     to obtain the specified length
     * @throws NegativeArraySizeException if <tt>newLength</tt> is negative
     * @throws NullPointerException if <tt>original</tt> is null
     * @throws ArrayStoreException if an element copied from
     *     <tt>original</tt> is not of a runtime type that can be stored in
     *     an array of class <tt>newType</tt>
     * @since 1.6
     */
    public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        //创建一个新的长度为 newLength 的数组实例 
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        //进行拷贝，如果长度不足，意味着后面的保留默认值，此时是 null
        //Math.min 存在一定量的开销
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```

测试

```java
 		String[] array = {"a", "b", "c", "b", "d"};
        System.out.println("array.length:" + array.length);
        String[] arrayCopyed = Arrays.copyOf(array, 10);
        System.out.println("arrayCopyed.length:" + arrayCopyed.length);
        System.out.println(Arrays.toString(array));
        System.out.println(Arrays.toString(arrayCopyed));
        array[2] = "cc";
        System.out.println(Arrays.toString(array));
        System.out.println(Arrays.toString(arrayCopyed));
//-------------------------- 结果 ---------------------
array.length:5
arrayCopyed.length:10
[a, b, c, b, d]
[a, b, c, b, d, null, null, null, null, null]
[a, b, cc, b, d]
[a, b, c, b, d, null, null, null, null, null]
```

