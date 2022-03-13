> 简单的一维猜方格游戏。

如图，设想有 7 个方格，排成 1 列，将其中 3 个连续的方格设为目标：

![闭包图解](../_images/一维猜方格.svg ':size=400')

然后让用户来猜测这 3 个目标方格的位置。当全部猜出时，就顺利结束游戏，并给出游戏结果。在逻辑上，可以看成是最简单的击沉战舰的游戏。当击中一个时，可以看成战舰受损。只有三个目标方格都被击中时，战舰才会沉没。

下面的三个类，需要分别**保存到单独的文件** `SimpleDotCom.java`, `SimpleDotComGame.java`, `GameHelper.java`，然后这三个文件需要放在**同一文件夹**下。

## SimpleDotCom

我们并不需要真的创建一个容量为 7 数组，而是只需要创建一个容量为 3 的数组，然后把目标位置填入。可以把 3 个目标方格合起来，看成一个对象。所以可以写出 `SimpleDotCom` 类：

```java
public class SimpleDotCom {
  private int[] locationCells; // 用于存放目标方格的位置
  private int numOfHits = 0; // 记录击中次数

  // 获取主进程随机产生的目标方格的位置
  public void setLocationCells(int[] locs) {
    locationCells = locs;
  }

  // 对用户猜测的检验
  public String checkYourself(String stringGuess) {
    int guess = Integer.parseInt(stringGuess); // 把字符串转为整数
    String result = "miss"; // 默认猜不中

    for (int cell : locationCells) { // 依次检测目标方格与用户猜测的匹配
      if (guess == cell) {
        result = "hit";
        numOfHits++; // 如果猜中，则击中次数加 1
        break; // 后面的方格就不用检测了，跳出循环
      }
    }

    if (numOfHits == locationCells.length) {
      result = "kill"; // 如果击中次数达到目标方格的长度，就说明目标已被 kill
    }

    System.out.println(result); // 打印检测结果
    return result; // 返回检测结果
  }
}
```

## SimpleDotComGame

然后我们还需要一个类，来启动游戏主进程：

```java
public class SimpleDotComGame {
  // 游戏主进程
  public static void main(String[] args) {
    boolean isAlive = true; // 记录 SimpleDotCom 对象的存活状态
    int numOfGuesses = 0; // 记录用户猜测次数
    GameHelper helper = new GameHelper(); // 用 GameHelper 类来获取用户输入

    SimpleDotCom theDotCom = new SimpleDotCom(); // 创建 SimpleDotCom 对象，用于给用户击中
    int randomNum = (int) (Math.random() * 5); // 生成 [0, 4] 的随机整数，代表第一个目标方格的位置
    // 用 {} 初始化数组
    int[] locations = { randomNum, randomNum + 1, randomNum + 2 }; // 后面两个目标方格位置依次增加 1
    theDotCom.setLocationCells(locations); // 把随机生成的目标方格位置，设置给上面生成的 SimpleDotCom 对象

    while (isAlive) { // 当 SimpleDotCom 对象还存活，就需要继续游戏
      String guess = helper.getUserInput("enter a number"); // 获取用户输入
      String result = theDotCom.checkYourself(guess); // 检测用户猜测
      numOfGuesses++; // 不论是否猜中，猜测次数加 1
      if (result.equals("kill")) {
        isAlive = false; // 如果 SimpleDotCom 对象被 kill，就说明它死了
        System.out.println("You took " + numOfGuesses + " guesses"); // 输出游戏结果
      }
    }
  }
}
```

## GameHelper

我们还需要一个辅助类，来获取用户输入：

```java
import java.io.*; // 需要用到 Java 自带的 io 类

public class GameHelper {
  public String getUserInput(String prompt) {
    String inputLine = null; // 用于存放用户输入的字符串
    System.out.print(prompt + " "); // 输出提示，让用户输入猜测

    try {
      BufferedReader is = new BufferedReader(
          new InputStreamReader(System.in));

      inputLine = is.readLine(); // 读取用户输入的一行

      if (inputLine.length() == 0) {
        inputLine = null;
      }
    } catch (IOException e) {
      System.out.println("IOException: " + e);
    }

    return inputLine; // 返回用户输入的字符串
  }
}
```

## bug

实际上，这个简单的一维猜方格游戏有严重 bug。当用户猜中一个方格时，只要继续输入两次已猜中的位置，就一定能结束游戏。在下一章，我们会使用 `ArrayList` 解决这个 bug，并将游戏升级为二维方格。



> 更新时间：{docsify-updated}