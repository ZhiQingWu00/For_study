### 方法和注释

#### 1.方法

***

  **定议语法 **

```JAVA
[访问修饰符] [其他修饰符] 返回值类型 方法名([参数列表]) {
    // 方法体
    [return 返回值;]
}
```

  

**组成部分**

* `public`：公共的，任何类都可以访问
* `private`：私有的，只有本类可以访问
* `protected`：受保护的，同包和子类可以访问
* 默认（不写）：同包可以访问



  **返回值类型**

* 基本数据类型：`int`, `double`, `boolean`等

* 引用数据类型：`String`, 数组, 自定义类等

* `void`：无返回值

  

  **return**

  无返回值的方法中可以直接通过单独的return，立即结束当前方法的执行。

  ```java
  public static void div(int a,int b){
      if(b==0){
          system.out.println("除数不能为0")
              return;
      }
      system.out.println(a/b);
  }
  ```

  

  **方法重载**

  在同一个类中，方法名相同但参数列表不同（参数类型、个数或顺序不同）

  //注意：方法重载只关心方法名称相同，形参列表不同（类型不同，个数不同，顺序不同）其他都无所谓

  ```java
  public class OverloadExample {
      
      // 方法重载示例
      public int add(int a, int b) {
          return a + b;
      }
      
      public int add(int a, int b, int c) {
          return a + b + c;
      }
      
      public double add(double a, double b) {
          return a + b;
      }
      
      public String add(String a, String b) {
          return a + b;
      }
      
      public void test() {
          System.out.println(add(1, 2));        // 调用int add(int, int)
          System.out.println(add(1, 2, 3));     // 调用int add(int, int, int)
          System.out.println(add(1.5, 2.5));    // 调用double add(double, double)
          System.out.println(add("Hello", "World")); // 调用String add(String, String)
      }
  }
  ```

  

  **标准输入输出**

  ```java
  import java.util.Scanner;
  
  public class InputOutput {
      public static void main(String[] args) {
          // 创建Scanner对象用于输入
          Scanner scanner = new Scanner(System.in);
          
          // 输出到控制台
          System.out.print("请输入您的姓名: ");
          String name = scanner.nextLine();
          
          System.out.print("请输入您的年龄: ");
          int age = scanner.nextInt();
          
          System.out.print("请输入您的身高(米): ");
          double height = scanner.nextDouble();
          
          // 格式化输出
          System.out.println("\n=== 用户信息 ===");
          System.out.printf("姓名: %s\n", name);
          System.out.printf("年龄: %d 岁\n", age);
          System.out.printf("身高: %.2f 米\n", height);
          
          // 多种输出方式
          System.out.print("这是一行不换行的输出"); // 不换行
          System.out.println("这是换行输出"); // 换行
          System.out.printf("格式化输出: %d + %d = %d\n", 5, 3, 5+3);
          
          scanner.close();
      }
  }
  ```

  

  **更复杂的输入处理**

  ```java
  import java.util.Scanner;
  
  public class AdvancedInput {
      public static void main(String[] args) {
          Scanner scanner = new Scanner(System.in);
          
          System.out.println("请输入多个数据(用空格分隔):");
          
          // 读取多个整数
          int num1 = scanner.nextInt();
          int num2 = scanner.nextInt();
          int num3 = scanner.nextInt();
          
          System.out.printf("您输入的三个数字是: %d, %d, %d\n", num1, num2, num3);
          
          // 清除缓冲区
          scanner.nextLine();
          
          System.out.print("请输入一句话: ");
          String sentence = scanner.nextLine();
          System.out.println("您输入的是: " + sentence);
          
          scanner.close();
      }
  }
  ```

  

#### 2.注释

**类型**

* 单行注释：

  ```java
  // 这是单行注释
  int age = 25; // 声明年龄变量
  ```

* 多行注释

```java
/*
 * 这是多行注释
 * 可以写多行内容
 * 用于较长的说明
 */
```

* 文档注释

  ```java
  /**
   * 用户类
   * 用于表示系统用户信息
   * 
   * @author 张三
   * @version 1.0
   * @since 2024-01-01
   */
  public class User {
      /**
       * 用户名
       */
      private String username;
      
      /**
       * 用户年龄
       */
      private int age;
      
      /**
       * 构造函数
       * @param username 用户名
       * @param age 用户年龄
       */
      public User(String username, int age) {
          this.username = username;
          this.age = age;
      }
      
      /**
       * 获取用户信息
       * @return 格式化的用户信息字符串
       */
      public String getUserInfo() {
          return "用户名: " + username + ", 年龄: " + age;
      }
  }
  ```

  



