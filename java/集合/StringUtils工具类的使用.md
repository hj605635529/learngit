# StringUtils工具类的使用

[TOC]

## 1.介绍

StringUtils 方法的操作对象是 java.lang.String 类型的对象，是 JDK 提供的 String 类型操作方法的补充，并且是 null 安全的(即如果输入参数 String 为 null 则不会抛出 NullPointerException ，而是做了相应处理，例如，如果输入为 null 则返回也是 null 等，具体可以查看源代码)。  除了构造器，StringUtils 有多个方法，并且都是 static 的，所以我们可以这样调用 StringUtils.xxx()  下面分别对一些常用方法做简要介绍： 

## 2.引入的maven依赖

```java
<!-- https://mvnrepository.com/artifact/org.apache.commons/commons-lang3 -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.4</version>
        </dependency>
```

## 3.常用方法源码

- isEmpty

  ```java
   * <p>Checks if a String is empty ("") or null.</p>
   *
   * <pre>
   * StringUtils.isEmpty(null)      = true
   * StringUtils.isEmpty("")        = true
   * StringUtils.isEmpty(" ")       = false
   * StringUtils.isEmpty("bob")     = false
   * StringUtils.isEmpty("  bob  ") = false
   * </pre>
   *
   * <p>NOTE: This method changed in Lang version 2.0.
   * It no longer trims the String.
   * That functionality is available in isBlank().</p>  //注意这个方法在2.0中不再去修剪字符串了， isBlank中修剪字符串了
   *
   * @param str  the String to check, may be null
   * @return <code>true</code> if the String is empty or null
   */
  public static boolean isEmpty(String str) {
      return str == null || str.length() == 0;
  }
  ```

- isBlank  和isEmpty在最大的区别就是如果字符串全是空白字符串，这个函数也是返回true的

  ```java
  /**
   * <p>Checks if a String is whitespace, empty ("") or null.</p>
   *
   * <pre>
   * StringUtils.isBlank(null)      = true
   * StringUtils.isBlank("")        = true
   * StringUtils.isBlank(" ")       = true
   * StringUtils.isBlank("bob")     = false
   * StringUtils.isBlank("  bob  ") = false
   * </pre>
   *
   * @param str  the String to check, may be null
   * @return <code>true</code> if the String is null, empty or whitespace
   * @since 2.0
   */
  public static boolean isBlank(String str) {
      int strLen;
      if (str == null || (strLen = str.length()) == 0) {
          return true;
      }
      for (int i = 0; i < strLen; i++) {
          if ((Character.isWhitespace(str.charAt(i)) == false)) {
              return false;
          }
      }
      return true;
  }
  ```

- trim

  ```java
   * <pre>
   * StringUtils.trim(null)          = null
   * StringUtils.trim("")            = ""
   * StringUtils.trim("     ")       = ""
   * StringUtils.trim("abc")         = "abc"
   * StringUtils.trim("    abc    ") = "abc"
   * </pre>
   *
   * @param str  the String to be trimmed, may be null
   * @return the trimmed string, <code>null</code> if null String input
   */
  public static String trim(String str) {
      return str == null ? null : str.trim();
  
  ```

- trimToNull

  ```java
   * <pre>
   * StringUtils.trimToNull(null)          = null
   * StringUtils.trimToNull("")            = null
   * StringUtils.trimToNull("     ")       = null
   * StringUtils.trimToNull("abc")         = "abc"
   * StringUtils.trimToNull("    abc    ") = "abc"
   * </pre>
   *
   * @param str  the String to be trimmed, may be null
   * @return the trimmed String,
   *  <code>null</code> if only chars &lt;= 32, empty or null String input
   * @since 2.0
   */
  public static String trimToNull(String str) {
      String ts = trim(str);
      return isEmpty(ts) ? null : ts;
  }
  ```

- trimToEmpty

  ```java
   * <pre>
   * StringUtils.trimToEmpty(null)          = ""
   * StringUtils.trimToEmpty("")            = ""
   * StringUtils.trimToEmpty("     ")       = ""
   * StringUtils.trimToEmpty("abc")         = "abc"
   * StringUtils.trimToEmpty("    abc    ") = "abc"
   * </pre>
   *
   * @param str  the String to be trimmed, may be null
   * @return the trimmed String, or an empty String if <code>null</code> input
   * @since 2.0
   */
  public static String trimToEmpty(String str) {
      return str == null ? EMPTY : str.trim();
  }
  ```

- equals

  ```java
   * <pre>
   * StringUtils.equals(null, null)   = true
   * StringUtils.equals(null, "abc")  = false
   * StringUtils.equals("abc", null)  = false
   * StringUtils.equals("abc", "abc") = true
   * StringUtils.equals("abc", "ABC") = false
   * </pre>
   *
   * @see java.lang.String#equals(Object)
   * @param str1  the first String, may be null
   * @param str2  the second String, may be null
   * @return <code>true</code> if the Strings are equal, case sensitive, or
   *  both <code>null</code>
   */
  public static boolean equals(String str1, String str2) {
      return str1 == null ? str2 == null : str1.equals(str2);
  }
  ```

- equalsIgnoreCase

  ```java
   * <pre>
   * StringUtils.equalsIgnoreCase(null, null)   = true
   * StringUtils.equalsIgnoreCase(null, "abc")  = false
   * StringUtils.equalsIgnoreCase("abc", null)  = false
   * StringUtils.equalsIgnoreCase("abc", "abc") = true
   * StringUtils.equalsIgnoreCase("abc", "ABC") = true
   * </pre>
   *
   * @see java.lang.String#equalsIgnoreCase(String)
   * @param str1  the first String, may be null
   * @param str2  the second String, may be null
   * @return <code>true</code> if the Strings are equal, case insensitive, or
   *  both <code>null</code>
   */
  public static boolean equalsIgnoreCase(String str1, String str2) {
      return str1 == null ? str2 == null : str1.equalsIgnoreCase(str2);
  }
  ```

- indexOf

  ```java
   *
   * <pre>
   * StringUtils.indexOf(null, *)         = -1
   * StringUtils.indexOf("", *)           = -1
   * StringUtils.indexOf("aabaabaa", 'a') = 0
   * StringUtils.indexOf("aabaabaa", 'b') = 2
   * </pre>
   *
   * @param str  the String to check, may be null
   * @param searchChar  the character to find
   * @return the first index of the search character,
   *  -1 if no match or <code>null</code> string input
   * @since 2.0
   */
  public static int indexOf(String str, char searchChar) {
      if (isEmpty(str)) {
          return -1;
      }
      return str.indexOf(searchChar);
  }
  ```

- indexOf(str,searchChar, startPos)

  ```java
   * <pre>
   * StringUtils.indexOf(null, *, *)          = -1
   * StringUtils.indexOf("", *, *)            = -1
   * StringUtils.indexOf("aabaabaa", 'b', 0)  = 2
   * StringUtils.indexOf("aabaabaa", 'b', 3)  = 5
   * StringUtils.indexOf("aabaabaa", 'b', 9)  = -1
   * StringUtils.indexOf("aabaabaa", 'b', -1) = 2
   * </pre>
   *
   * @param str  the String to check, may be null
   * @param searchChar  the character to find
   * @param startPos  the start position, negative treated as zero
   * @return the first index of the search character,
   *  -1 if no match or <code>null</code> string input
   * @since 2.0
   */
  public static int indexOf(String str, char searchChar, int startPos) {
      if (isEmpty(str)) {
          return -1;
      }
      return str.indexOf(searchChar, startPos);
  }
  ```

- indexOf(String str, String searchStr)

  ```java
   * <pre>
   * StringUtils.indexOf(null, *)          = -1
   * StringUtils.indexOf(*, null)          = -1
   * StringUtils.indexOf("", "")           = 0    //这里要注意，查找的是空字符串的话，返回0
   * StringUtils.indexOf("aabaabaa", "a")  = 0
   * StringUtils.indexOf("aabaabaa", "b")  = 2
   * StringUtils.indexOf("aabaabaa", "ab") = 1
   * StringUtils.indexOf("aabaabaa", "")   = 0
   * </pre>
   *
   * @param str  the String to check, may be null
   * @param searchStr  the String to find, may be null
   * @return the first index of the search String,
   *  -1 if no match or <code>null</code> string input
   * @since 2.0
   */
  public static int indexOf(String str, String searchStr) {
      if (str == null || searchStr == null) {
          return -1;
      }
      return str.indexOf(searchStr);
  }
  ```

- subString

  ```java
   * <pre>
   * StringUtils.substring(null, *)   = null
   * StringUtils.substring("", *)     = ""
   * StringUtils.substring("abc", 0)  = "abc"
   * StringUtils.substring("abc", 2)  = "c"
   * StringUtils.substring("abc", 4)  = ""
   * StringUtils.substring("abc", -2) = "bc"  //负数的时候是从结束往前数的， 
   * StringUtils.substring("abc", -4) = "abc"
   * </pre>
   *
   * @param str  the String to get the substring from, may be null
   * @param start  the position to start from, negative means
   *  count back from the end of the String by this many characters
   * @return substring from start position, <code>null</code> if null String input
   */
  public static String substring(String str, int start) {
      if (str == null) {
          return null;
      }
  
      // handle negatives, which means last n characters
      if (start < 0) {
          start = str.length() + start; // remember start is negative
      }
  
      if (start < 0) {
          start = 0;
      }
      if (start > str.length()) {
          return EMPTY;
      }
  
      return str.substring(start);
  }
  ```

- join

  ```java
   * <pre>
   * StringUtils.join(null)            = null
   * StringUtils.join([])              = ""
   * StringUtils.join([null])          = ""
   * StringUtils.join(["a", "b", "c"]) = "abc"
   * StringUtils.join([null, "", "a"]) = "a"
   * </pre>
   *
   * @param array  the array of values to join together, may be null
   * @return the joined String, <code>null</code> if null array input
   * @since 2.0
   */
  public static String join(Object[] array) {
      return join(array, null);
  }
  ```

  

  ```java
   * <pre>
   * StringUtils.join(null, *)                = null
   * StringUtils.join([], *)                  = ""
   * StringUtils.join([null], *)              = ""
   * StringUtils.join(["a", "b", "c"], "--")  = "a--b--c"
   * StringUtils.join(["a", "b", "c"], null)  = "abc"
   * StringUtils.join(["a", "b", "c"], "")    = "abc"
   * StringUtils.join([null, "", "a"], ',')   = ",,a"
   * </pre>
   *
   * @param array  the array of values to join together, may be null
   * @param separator  the separator character to use, null treated as ""
   * @return the joined String, <code>null</code> if null array input
   */
  public static String join(Object[] array, String separator) {
      if (array == null) {
          return null;
      }
      return join(array, separator, 0, array.length);
  }
  ```

  

  ```java
   * <pre>
   * StringUtils.join(null, *)               = null
   * StringUtils.join([], *)                 = ""
   * StringUtils.join([null], *)             = ""
   * StringUtils.join(["a", "b", "c"], ';')  = "a;b;c"
   * StringUtils.join(["a", "b", "c"], null) = "abc"
   * StringUtils.join([null, "", "a"], ';')  = ";;a"
   * </pre>
   *
   * @param array  the array of values to join together, may be null
   * @param separator  the separator character to use
   * @param startIndex the first index to start joining from.  It is 
   * an error to pass in an end index past the end of the array  //如果开始坐标和结束坐标超过了数组的最后索引的话， 会报出指针越界问题
   * @param endIndex the index to stop joining from (exclusive). It is
   * an error to pass in an end index past the end of the array
   * @return the joined String, <code>null</code> if null array input
   * @since 2.0
   */
  public static String join(Object[] array, char separator, int startIndex, int endIndex) {
      if (array == null) {
          return null;
      }
      int bufSize = (endIndex - startIndex);
      if (bufSize <= 0) {
          return EMPTY;
      }
  
      bufSize *= ((array[startIndex] == null ? 16 : array[startIndex].toString().length()) + 1);
      StringBuffer buf = new StringBuffer(bufSize);
  
      for (int i = startIndex; i < endIndex; i++) {
          if (i > startIndex) {
              buf.append(separator);
          }
          if (array[i] != null) {
              buf.append(array[i]);
          }
      }
      return buf.toString();
  }
  ```

- upperCase

  ```java
   * <pre>
   * StringUtils.upperCase(null)  = null
   * StringUtils.upperCase("")    = ""
   * StringUtils.upperCase("aBc") = "ABC"
   * </pre>
   *
   * <p><strong>Note:</strong> As described in the documentation for {@link String#toUpperCase()},
   * the result of this method is affected by the current locale.
   * For platform-independent case transformations, the method {@link #lowerCase(String, Locale)}
   * should be used with a specific locale (e.g. {@link Locale#ENGLISH}).</p>
   *
   * @param str  the String to upper case, may be null
   * @return the upper cased String, <code>null</code> if null String input
   */
  public static String upperCase(String str) {
      if (str == null) {
          return null;
      }
      return str.toUpperCase();
  }
  ```

- deleteWhiteSpace

  ```java
   * <pre>
   * StringUtils.deleteWhitespace(null)         = null
   * StringUtils.deleteWhitespace("")           = ""
   * StringUtils.deleteWhitespace("abc")        = "abc"
   * StringUtils.deleteWhitespace("   ab  c  ") = "abc"
   * </pre>
   *
   * @param str  the String to delete whitespace from, may be null
   * @return the String without whitespaces, <code>null</code> if null String input
   */
  public static String deleteWhitespace(String str) {
      if (isEmpty(str)) {
          return str;
      }
      int sz = str.length();
      char[] chs = new char[sz];
      int count = 0;
      for (int i = 0; i < sz; i++) {
          if (!Character.isWhitespace(str.charAt(i))) {
              chs[count++] = str.charAt(i);
          }
      }
      if (count == sz) {
          return str;
      }
      return new String(chs, 0, count);
  }
  ```

- removeStart

  ```java
   *
   * <pre>
   * StringUtils.removeStart(null, *)      = null
   * StringUtils.removeStart("", *)        = ""
   * StringUtils.removeStart(*, null)      = *
   * StringUtils.removeStart("www.domain.com", "www.")   = "domain.com"
   * StringUtils.removeStart("domain.com", "www.")       = "domain.com"
   * StringUtils.removeStart("www.domain.com", "domain") = "www.domain.com"
   * StringUtils.removeStart("abc", "")    = "abc"
   * </pre>
   *
   * @param str  the source String to search, may be null
   * @param remove  the String to search for and remove, may be null
   * @return the substring with the string removed if found,
   *  <code>null</code> if null String input
   * @since 2.1
   */
  public static String removeStart(String str, String remove) {
      if (isEmpty(str) || isEmpty(remove)) {
          return str;
      }
      if (str.startsWith(remove)){
          return str.substring(remove.length());
      }
      return str;
  }
  ```

- remove

  ```java
   * <p>A <code>null</code> source string will return <code>null</code>.
   * An empty ("") source string will return the empty string.
   * A <code>null</code> remove string will return the source string.
   * An empty ("") remove string will return the source string.</p>
   *
   * <pre>
   * StringUtils.remove(null, *)        = null
   * StringUtils.remove("", *)          = ""
   * StringUtils.remove(*, null)        = *
   * StringUtils.remove(*, "")          = *
   * StringUtils.remove("queued", "ue") = "qd"
   * StringUtils.remove("queued", "zz") = "queued"
   * </pre>
   *
   * @param str  the source String to search, may be null
   * @param remove  the String to search for and remove, may be null
   * @return the substring with the string removed if found,
   *  <code>null</code> if null String input
   * @since 2.1
   */
  public static String remove(String str, String remove) {
      if (isEmpty(str) || isEmpty(remove)) {
          return str;
      }
      return replace(str, remove, EMPTY, -1);
  }
  ```

- Replace

  ```java
  /**
   * <p>Replaces all occurrences of a String within another String.</p>
   *
   * <p>A <code>null</code> reference passed to this method is a no-op.</p>
   *
   * <pre>
   * StringUtils.replace(null, *, *)        = null
   * StringUtils.replace("", *, *)          = ""
   * StringUtils.replace("any", null, *)    = "any"
   * StringUtils.replace("any", *, null)    = "any"
   * StringUtils.replace("any", "", *)      = "any"
   * StringUtils.replace("aba", "a", null)  = "aba"
   * StringUtils.replace("aba", "a", "")    = "b"
   * StringUtils.replace("aba", "a", "z")   = "zbz"
   * </pre>
   *
   * @see #replace(String text, String searchString, String replacement, int max)
   * @param text  text to search and replace in, may be null
   * @param searchString  the String to search for, may be null
   * @param replacement  the String to replace it with, may be null
   * @return the text with any replacements processed,
   *  <code>null</code> if null String input
   */
  public static String replace(String text, String searchString, String replacement) {
      return replace(text, searchString, replacement, -1);
  }
  
  /**
   * <p>Replaces a String with another String inside a larger String,
   * for the first <code>max</code> values of the search String.</p>
   *
   * <p>A <code>null</code> reference passed to this method is a no-op.</p>
   *
   * <pre>
   * StringUtils.replace(null, *, *, *)         = null
   * StringUtils.replace("", *, *, *)           = ""
   * StringUtils.replace("any", null, *, *)     = "any"
   * StringUtils.replace("any", *, null, *)     = "any"
   * StringUtils.replace("any", "", *, *)       = "any"
   * StringUtils.replace("any", *, *, 0)        = "any"
   * StringUtils.replace("abaa", "a", null, -1) = "abaa"
   * StringUtils.replace("abaa", "a", "", -1)   = "b"
   * StringUtils.replace("abaa", "a", "z", 0)   = "abaa"
   * StringUtils.replace("abaa", "a", "z", 1)   = "zbaa"
   * StringUtils.replace("abaa", "a", "z", 2)   = "zbza"
   * StringUtils.replace("abaa", "a", "z", -1)  = "zbzz"
   * </pre>
   *
   * @param text  text to search and replace in, may be null
   * @param searchString  the String to search for, may be null
   * @param replacement  the String to replace it with, may be null
   * @param max  maximum number of values to replace, or <code>-1</code> if no maximum      //-1表示替换所有的
   * @return the text with any replacements processed,
   *  <code>null</code> if null String input
   */
  public static String replace(String text, String searchString, String replacement, int max) {
      if (isEmpty(text) || isEmpty(searchString) || replacement == null || max == 0) {
          return text;
      }
      int start = 0;
      int end = text.indexOf(searchString, start);
      if (end == -1) {
          return text;
      }
      int replLength = searchString.length();
      int increase = replacement.length() - replLength;
      increase = (increase < 0 ? 0 : increase);
      increase *= (max < 0 ? 16 : (max > 64 ? 64 : max));
      StringBuffer buf = new StringBuffer(text.length() + increase);
      while (end != -1) {
          buf.append(text.substring(start, end)).append(replacement);
          start = end + replLength;
          if (--max == 0) {
              break;
          }
          end = text.indexOf(searchString, start);
      }
      buf.append(text.substring(start));
      return buf.toString();
  }
  ```

- isNumeric

  ```java
  /**
   * <p>Checks if the String contains only unicode digits.
   * A decimal point is not a unicode digit and returns false.</p>
   *
   * <p><code>null</code> will return <code>false</code>.
   * An empty String ("") will return <code>true</code>.</p>
   *
   * <pre>
   * StringUtils.isNumeric(null)   = false
   * StringUtils.isNumeric("")     = true
   * StringUtils.isNumeric("  ")   = false
   * StringUtils.isNumeric("123")  = true
   * StringUtils.isNumeric("12 3") = false
   * StringUtils.isNumeric("ab2c") = false
   * StringUtils.isNumeric("12-3") = false
   * StringUtils.isNumeric("12.3") = false
   * </pre>
   *
   * @param str  the String to check, may be null
   * @return <code>true</code> if only contains digits, and is non-null
   */
  //小数点不是数字，所以对于浮点数返回的是false.
  public static boolean isNumeric(String str) {
      if (str == null) {
          return false;
      }
      int sz = str.length();
      for (int i = 0; i < sz; i++) {
          if (Character.isDigit(str.charAt(i)) == false) {
              return false;
          }
      }
      return true;
  }
  ```

- split

  ```java
  /**
   * <p>Splits the provided text into an array, separators specified.
   * This is an alternative to using StringTokenizer.</p>
   *
   * <p>The separator is not included in the returned String array.
   * Adjacent separators are treated as one separator.  //相邻的分割符是作为一个分割符号的
   * For more control over the split use the StrTokenizer class.</p>
   *
   * <p>A <code>null</code> input String returns <code>null</code>.
   * A <code>null</code> separatorChars splits on whitespace.</p>
   *
   * <pre>
   * StringUtils.split(null, *)         = null
   * StringUtils.split("", *)           = []
   * StringUtils.split("abc def", null) = ["abc", "def"]
   * StringUtils.split("abc def", " ")  = ["abc", "def"]
   * StringUtils.split("abc  def", " ") = ["abc", "def"]
   * StringUtils.split("ab:cd:ef", ":") = ["ab", "cd", "ef"]
   * </pre>
   *
   * @param str  the String to parse, may be null
   * @param separatorChars  the characters used as the delimiters,
   *  <code>null</code> splits on whitespace
   * @return an array of parsed Strings, <code>null</code> if null String input
   */
  public static String[] split(String str, String separatorChars) {
      return splitWorker(str, separatorChars, -1, false);
  }
  ```

  

  ```java
  /**
   * Performs the logic for the <code>split</code> and 
   * <code>splitPreserveAllTokens</code> methods that return a maximum array 
   * length.
   *
   * @param str  the String to parse, may be <code>null</code>
   * @param separatorChars the separate character  如果separatorChars是null, 则空白字符作为分割符号
   * @param max  the maximum number of elements to include in the
   *  array. A zero or negative value implies no limit.
   * @param preserveAllTokens if <code>true</code>, adjacent separators are
   * treated as empty token separators; if <code>false</code>, adjacent
   * separators are treated as one separator.
    //如果preserverAllTokens 作为true,相邻分隔符被视为空标记分隔符,如果是false,相邻分割符作为一个分割符号使用
   * @return an array of parsed Strings, <code>null</code> if null String input
   */
  private static String[] splitWorker(String str, String separatorChars, int max, boolean preserveAllTokens) {
      // Performance tuned for 2.0 (JDK1.4)
      // Direct code is quicker than StringTokenizer.
      // Also, StringTokenizer uses isSpace() not isWhitespace()
  
      if (str == null) {
          return null;
      }
      int len = str.length();
      if (len == 0) {
          return ArrayUtils.EMPTY_STRING_ARRAY;   //返回一个含有0个元素的空数组
      }
      List list = new ArrayList();
      int sizePlus1 = 1;
      int i = 0, start = 0;
      boolean match = false;
      boolean lastMatch = false;
      if (separatorChars == null) {
          // Null separator means use whitespace
          while (i < len) {
              if (Character.isWhitespace(str.charAt(i))) {
                  if (match || preserveAllTokens) {
                      lastMatch = true;
                      if (sizePlus1++ == max) {
                          i = len;
                          lastMatch = false;
                      }
                      list.add(str.substring(start, i));
                      match = false;
                  }
                  start = ++i;
                  continue;
              }
              lastMatch = false;
              match = true;
              i++;
          }
      } else if (separatorChars.length() == 1) {
          // Optimise 1 character case
          char sep = separatorChars.charAt(0);
          while (i < len) {
              if (str.charAt(i) == sep) {
                  if (match || preserveAllTokens) {
                      lastMatch = true;
                      if (sizePlus1++ == max) {
                          i = len;
                          lastMatch = false;
                      }
                      list.add(str.substring(start, i));
                      match = false;
                  }
                  start = ++i;
                  continue;
              }
              lastMatch = false;
              match = true;
              i++;
          }
      } else {
          // standard case
          while (i < len) {
              if (separatorChars.indexOf(str.charAt(i)) >= 0) {
                  if (match || preserveAllTokens) {
                      lastMatch = true;
                      if (sizePlus1++ == max) {
                          i = len;
                          lastMatch = false;
                      }
                      list.add(str.substring(start, i));
                      match = false;
                  }
                  start = ++i;
                  continue;
              }
              lastMatch = false;
              match = true;
              i++;
          }
      }
      if (match || (preserveAllTokens && lastMatch)) {
          list.add(str.substring(start, i));
      }
      return (String[]) list.toArray(new String[list.size()]);
  }
  ```



## 4.总结

> - isEmpty与isBlank可以用于参数的校验，isBlank在字符串全是空白字符的时候也返回true。

> - StringUtils.Empty可以一眼就看出这是空字符串， 比直接赋值"",更直观。
> - 判断两个字符串相等是比较常用的方法了，用StringUtils.equals可以避免空指针异常。
> - trimToNull和trimToEmpty是经常用到的方法，用来处理参数url带来的字符串。
> - 我们平时进行简单的字符串分割的时候，尽量不要用String自身的split方法，它是匹配正则表达式的，如果遇到$这种特殊字符，需要转义一下。用StringUtils.split()方法会更方便 ,它就是匹配字符串的。