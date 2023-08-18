# String
String是不可变
+ 可以保证 String 对象的安全性，避免被篡改
+ 保证哈希值不会频繁变更
+ 可以实现字符串常量池

StringBuffer 操作字符串的方法加了 synchronized 关键字进行了同步
StringBuilder

Java 9 以后，对 + 号操作符的解释和之前发生了变化，字节码指令已经不同了

## String常量池
使用双引号声明的字符串对象会保存在字符串常量池(内存堆)中
使用 new 关键字创建的字符串对象会先从字符串常量池中找，如果没找到就创建一个，然后再在堆中创建字符串对象；如果找到了，就直接在堆中创建字符串对象
针对没有使用双引号声明的字符串对象来说, 可以调用 intern() 方法来完成

String的String Pool是一个固定大小的Hashtable，默认值大小长度是1009，如果放进String Pool的String非常多，就会造成Hash冲突严重，从而导致链表会很长，而链表长了后直接会造成的影响就是当调用String.intern时性能会大幅下降


```java
// 如果常量池中存在当前字符串, 就会直接返回当前字符串. 如果常量池中没有此字符串, 会将此字符串放入常量池中后, 再返回
public native String intern();
String s = new String("1"); 
s.intern();
String s2 = "1";
System.out.println(s == s2); // false

String s3 = new String("1") + new String("1");
s3.intern();
String s4 = "11";
System.out.println(s3 == s4); // true

String s5 = new String("1") + new String("1");
String s6 = "11";
s3.intern();
System.out.println(s5 == s6); // false
```

# 常用方法
charAt() 用于返回指定索引处的字符

```java
public String[] split(String regex);
public String[] split(String regex, int limit);
public String(char value[], int offset, int count);
public char charAt(int index);
public boolean startsWith(String prefix);
public int indexOf(int ch, int fromIndex);
public int indexOf(String str, int fromIndex);
public int lastIndexOf(String str, int fromIndex);
public String concat(String str);
public String replace(char oldChar, char newChar);
public static String valueOf(int i);
```

# 部分源码

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    // java9 -> byte
    private final char value[];
    private final byte coder; // java9: 表示编码
    private int hash;

    public String(char value[]) {
        this.value = Arrays.copyOf(value, value.length);
    }
    public String(StringBuilder builder) {
        this.value = Arrays.copyOf(builder.getValue(), builder.length());
    }

    public byte[] getBytes() {
        return StringCoding.encode(value, 0, value.length);
    }

    // 31倍哈希法的优点在于简单易实现，计算速度快，同时也比较均匀地分布在哈希表中
    public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;
            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }

    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String) anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }

    public String substring(int beginIndex) {
        if (beginIndex < 0) {
            throw new StringIndexOutOfBoundsException(beginIndex);
        }
        int subLen = value.length - beginIndex;
        if (subLen < 0) {
            throw new StringIndexOutOfBoundsException(subLen);
        }
        return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
    }

    static int indexOf(char[] source, int sourceOffset, int sourceCount,
        char[] target, int targetOffset, int targetCount,
        int fromIndex) {
        // 如果开始搜索的位置已经超出 source 数组的范围，则直接返回-1（如果 target 数组为空，则返回 sourceCount）
        if (fromIndex >= sourceCount) {
            return (targetCount == 0 ? sourceCount : -1);
        }
        // 如果开始搜索的位置小于0，则从0开始搜索
        if (fromIndex < 0) {
            fromIndex = 0;
        }
        // 如果 target 数组为空，则直接返回开始搜索的位置
        if (targetCount == 0) {
            return fromIndex;
        }

        // 查找 target 数组的第一个字符在 source 数组中的位置
        char first = target[targetOffset];
        int max = sourceOffset + (sourceCount - targetCount);

        // 循环查找 target 数组在 source 数组中的位置
        for (int i = sourceOffset + fromIndex; i <= max; i++) {
            /* Look for first character. */
            // 如果 source 数组中当前位置的字符不是 target 数组的第一个字符，则在 source 数组中继续查找 target 数组的第一个字符
            if (source[i] != first) {
                while (++i <= max && source[i] != first);
            }

            /* Found first character, now look at the rest of v2 */
            // 如果在 source 数组中找到了 target 数组的第一个字符，则继续查找 target 数组的剩余部分是否匹配
            if (i <= max) {
                int j = i + 1;
                int end = j + targetCount - 1;
                for (int k = targetOffset + 1; j < end && source[j]
                        == target[k]; j++, k++);

                // 如果 target 数组全部匹配，则返回在 source 数组中的位置索引
                if (j == end) {
                    /* Found whole string. */
                    return i - sourceOffset;
                }
            }
        }
        // 没有找到 target 数组，则返回-1
        return -1;
    }

    public int compareTo(String anotherString) {
        int len1 = value.length;
        int len2 = anotherString.value.length;
        int lim = Math.min(len1, len2);
        char v1[] = value;
        char v2[] = anotherString.value;

        int k = 0;
        while (k < lim) {
            char c1 = v1[k];
            char c2 = v2[k];
            if (c1 != c2) {
                return c1 - c2;
            }
            k++;
        }
        return len1 - len2;
    }

    public boolean contains(CharSequence s) {
        return indexOf(s.toString()) > -1;
    }
}
```