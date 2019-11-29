### Exception

``` java
/**
1 自动抛出
*/
public class ThrowTest {
    public static void main(String[] args) {
        int a = 0;
        int b = 1;
        System.out.println(b / a);
    }
}

/**
2 if throw
*/
public class ThrowTest {
    public static void main(String[] args) {
        String string = "abc";
        if ("abc".equals(string)) {
            throw new NumberFormatException();
        } else {
            System.out.println(string);
        }
    }
}

/**
3 throws 到方法外，由外部try catch
*/
ublic class ThrowTest {

    public static void test() throws ArithmeticException {
        int a = 0;
        int b = 1;
        System.out.println(b / a);
    }

    public static void main(String[] args) {
        try {
            test();
        } catch (ArithmeticException e) {
            // TODO: handle exception
            System.out.println("test() -- 算术异常！");
        }
    }
}

```

