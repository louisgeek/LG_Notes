# Pattern 和 Matcher

```java
//方法1
boolean isNumber = Pattern.matches("[0-9]*", "abc");
//方法2
Pattern pattern = Pattern.compile("[0-9]*");
Matcher matcher = pattern.matcher("123");
boolean isNumber2 = matcher.matches();
//
System.out.println("test "+isNumber);
System.out.println("test "+isNumber2);
//======= 打印结果 =======
test false
test true  
```



## Pattern#matches

```java
   /**
     * Compiles the given regular expression and attempts to match the given
     * input against it.
     *
     * <p> An invocation of this convenience method of the form
     *
     * <blockquote><pre>
     * Pattern.matches(regex, input);</pre></blockquote>
     *
     * behaves in exactly the same way as the expression
     *
     * <blockquote><pre>
     * Pattern.compile(regex).matcher(input).matches()</pre></blockquote>
     *
     * <p> If a pattern is to be used multiple times, compiling it once and reusing
     * it will be more efficient than invoking this method each time.  </p>
     *
     * @param  regex
     *         The expression to be compiled
     *
     * @param  input
     *         The character sequence to be matched
     * @return whether or not the regular expression matches on the input
     * @throws  PatternSyntaxException
     *          If the expression's syntax is invalid
     */
    public static boolean matches(String regex, CharSequence input) {
        Pattern p = Pattern.compile(regex);
        Matcher m = p.matcher(input);
        return m.matches();
    }
```

- 方法1 和 方法2 是一样的



## Matcher#replaceAll

```java
Pattern pattern = Pattern.compile("[^0-9]");
Matcher matcher = pattern.matcher("收费20元");
System.out.println(matcher.replaceAll("-"));
//
Pattern pattern2 = Pattern.compile("[0-9]");
Matcher matcher2 = pattern2.matcher("收费20元");
System.out.println(matcher2.replaceAll("*"));
//
Pattern pattern3 = Pattern.compile("[0-9]");
Matcher matcher3 = pattern3.matcher("收费20元5角");
System.out.println(matcher3.replaceFirst("*"));
//======= 打印结果 =======
--20-
收费**元
收费*0元5角
```





```

```

