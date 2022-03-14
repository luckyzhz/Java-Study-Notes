> 简单的二维猜方格游戏。

如图，设想有一个 7x7 的方格网，有 3 组目标方格，每组目标方格内有 3 个相连方格（水平或垂直排列）：

![一维猜方格](../_images/二维猜方格.svg ':size=450')

然后让用户来猜测这 3 组目标方格的位置。当全部猜出时，就顺利结束游戏，并给出游戏结果。在逻辑上，可以看成是简单的击沉战舰的游戏。当击中一个方格时，可以看成战舰受损。只有一组的三个目标方格都被击中时，战舰才会沉没。

下面的三个类，需要分别**保存到单独的文件** `DotCom.java`, `DotComBust.java`, `GameHelper.java`，然后这三个文件需要放在**同一文件夹**下。

## DotCom

`DotCom` 类（可以看成战舰）最重要的是要知道自己的位置，以及判断用户是否击中自己：

```java
import java.util.*;

public class DotCom {
  // DotCom 应该有位置和名字两个属性
  private ArrayList<String> locationCells;
  private String name;

  public void setLocationCells(ArrayList<String> loc) {
    locationCells = loc;
  }

  public void setName(String n) {
    name = n;
  }

  public String checkYourself(String userInput) {
    String result = "miss";
    // 如果用户的猜测不在 locationCells 里，会返回 -1
    int index = locationCells.indexOf(userInput);

    if (index >= 0) {
      // 如果击中，需要把方格移除
      locationCells.remove(index);
      // 如果 locationCells 为空，说明这个 dot com 死亡
      if (locationCells.isEmpty()) {
        result = "kill";
        System.out.println("Ouch! You sunk " + name + " : ( ");
      } else {
        result = "hit";
      }
    }

    return result;
  }
}
```

## DotComBust

`DotComBust` 类，用于控制游戏初始化、游戏开始、游戏结束：

```java
import java.util.*;

public class DotComBust {

  // 声明并初始化需要的变量
  private GameHelper helper = new GameHelper();
  private ArrayList<DotCom> dotComsList = new ArrayList<DotCom>();
  private int numOfGuesses = 0;

  private void setUpGame() {
    // first make some dot coms and give them locations

    // 用一个数组存放一系列 dot com 名
    String[] dotComNames = {
        "Pets.com",
        "eToys.com",
        "Go2.com"
    };

    // 循环添加 DotCom 对象到 dotComsList
    for (int i = 0; i < dotComNames.length; i++) {
      DotCom aDotCom = new DotCom();
      aDotCom.setName(dotComNames[i]); // 设置名字
      ArrayList<String> newLocation = helper.placeDotCom(3);
      aDotCom.setLocationCells(newLocation); // 设置位置
      dotComsList.add(aDotCom);
    }

    // 输出提示
    System.out.println("Your goal is to sink " + dotComsList.size() + " dot coms.");
    System.out.println(dotComNames);
    System.out.println("Try to sink them all in the fewest number of guesses");
  } // close setUpGame method

  // 开始游戏
  private void startPlaying() {
    while (!dotComsList.isEmpty()) {
      String userGuess = helper.getUserInput("Enter a guess");
      checkUserGuess(userGuess);
    }
  }

  // 检验用户是否击中
  private void checkUserGuess(String userGuess) {
    numOfGuesses++;
    String result = "miss";

    for (DotCom dotComTest : dotComsList) {
      result = dotComTest.checkYourself(userGuess);
      if (result.equals("hit")) {
        break;
      }
      if (result.equals("kill")) {
        dotComsList.remove(dotComTest);
        break;
      }
    }

    System.out.println(result);
  }

  // 结束游戏
  private void finishGame() {
    System.out.println("All Dot Coms are dead! Your stock is now worthless.");
    if (numOfGuesses <= 18) {
      System.out.println("It only took you " + numOfGuesses + " guesses.");
      System.out.println("You got out before your options sank.");
    } else {
      System.out.println("Took you long enough. " + numOfGuesses + " guesses.");
      System.out.println("Fish are dancing with your options.");
    }
  }

  // 主进程
  public static void main(String[] args) {
    DotComBust game = new DotComBust();
    game.setUpGame();
    game.startPlaying();
    game.finishGame();
  }
}
```

## GameHelper

`GameHelper` 类，是辅助类。功能包括：获取用户输入、随机产生目标方格的位置（要保证不冲突，不出边界）：

```java
import java.io.*;
import java.util.*;

public class GameHelper {

  private static final String alphabet = "abcdefg";
  private int gridLength = 7;
  int comSize;
  int[][] grid = new int[gridLength][gridLength];

  // 获取用户输入
  public String getUserInput(String prompt) {
    String inputLine = null;
    System.out.print(prompt + "  ");
    try {
      BufferedReader is = new BufferedReader(
          new InputStreamReader(System.in));
      inputLine = is.readLine();
      if (inputLine.length() == 0) {
        inputLine = null;
      }
    } catch (IOException e) {
      System.out.println("IOException: " + e);
    }
    return inputLine.toLowerCase();
  }

  // 产生随机整数的方法。不包含边界
  private int randomInt(int border) {
    return (int) (Math.random() * border);
  }

  // 根据水平延伸，还是垂直延伸，产生 dotCom 对象的第一个方格坐标
  private int[] firstCell(boolean horizontal) {
    int[] coords = new int[2];
    if (horizontal) {
      // 如果是水平延伸，则列的位置受到限制
      coords[0] = randomInt(gridLength);
      coords[1] = randomInt(gridLength - comSize + 1);
    } else {
      // 如果是垂直延伸，则行的位置受到限制
      coords[0] = randomInt(gridLength - comSize + 1);
      coords[1] = randomInt(gridLength);
    }
    return coords;
  }

  // 根据第一个方格的坐标和延伸方向，判断会不会和已有的方格冲突
  private boolean canPlace(int[] coords, boolean horizontal) {
    boolean can = true;
    if (horizontal) {
      for (int i = 0; i < comSize; i++) {
        if (grid[coords[0]][coords[1] + i] == 1) {
          can = false;
          break;
        }
      }
    } else {
      for (int i = 0; i < comSize; i++) {
        if (grid[coords[0] + i][coords[1]] == 1) {
          can = false;
          break;
        }
      }
    }
    return can;
  }

  public ArrayList<String> placeDotCom(int comSize) {
    this.comSize = comSize;
    boolean successPlaceDotCom = false; // 记录随机产生的位置是否能放入方格网
    ArrayList<String> locations = new ArrayList<String>();
    String location;

    while (!successPlaceDotCom) {
      // 随机决定要水平延伸，还是垂直延伸
      boolean horizontal = true;
      if (Math.random() < 0.5) {
        horizontal = false;
      }

      // 产生第一个方格的坐标
      int[] coords = firstCell(horizontal);
      // 检测是否可以放置
      successPlaceDotCom = canPlace(coords, horizontal);

      // 如果可以放置，就真的放进去
      if (successPlaceDotCom) {
        if (horizontal) {
          for (int i = 0; i < comSize; i++) {
            grid[coords[0]][coords[1] + i] = 1; // 置 1 表示放入
            // 生成字符串坐标
            location = String.valueOf(alphabet.charAt(coords[1] + i)) + coords[0];
            locations.add(location);
          }
        } else {
          for (int i = 0; i < comSize; i++) {
            grid[coords[0] + i][coords[1]] = 1; // 置 1 表示放入
            // 生成字符串坐标
            location = String.valueOf(alphabet.charAt(coords[1])) + (coords[0] + i);
            locations.add(location);
          }
        }
      }
    }
    // 把生成的方格打印出来。用于测试
    // for (int i = 0; i < 7; i++) {
    // for (int j = 0; j < 7; j++) {
    // System.out.print(grid[i][j] + " ");
    // }
    // System.out.println();
    // }
    // System.out.println();
    return locations;
  }
}
```

## 总结

* 用 `ArrayList` 存放目标方格的位置。
  * 用户击中时便移除该方格，避免重复击中同一位置的 bug。
* 目标方格可能是水平排列，也可能是垂直排列。
  * 这两种情况下，第一个方格可能的范围是不同的，因此需要分类讨论。



> 更新时间：{docsify-updated}