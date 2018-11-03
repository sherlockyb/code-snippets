# code-snippets
> 好的代码片段就如同一件美的工艺品，需要收藏，反复品味~

## tricks

1. [**用移位代替乘除法**](#用移位代替乘除法)



***

### 用移位代替乘除法

来源——`jdk-java.lang.Integer-getChars(int i, int index, char[] buf)`

```java
static void getChars(int i, int index, char[] buf) {
        ...
        // Generate two digits per iteration
        while (i >= 65536) {
            q = i / 100;
        // really: r = i - (q * 100);
            r = i - ((q << 6) + (q << 5) + (q << 2));
            i = q;
            buf [--charPos] = DigitOnes[r];
            buf [--charPos] = DigitTens[r];
        }

        // Fall thru to fast mode for smaller numbers
        // assert(i <= 65536, i);
        for (;;) {
            q = (i * 52429) >>> (16+3);
            r = i - ((q << 3) + (q << 1));  // r = i-(q*10) ...
            buf [--charPos] = digits [r];
            i = q;
            if (i == 0) break;
        }
        ...
    }
```

当然，这种代替也不是随便一个数就行的。比如上面的`i * 52429`还是老老实实的乘法。直观上讲，当乘数或除数对应的有效二进制串中1的数目远少于总位数时，这种替换带来的效率提升越明显。比如，对于100，对应的有效二进制串为`1100100`，占比为3/7，做替换是有提升的；而对于像127、255这种全是1的，就没必要做这种替换了。