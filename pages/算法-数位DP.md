- 将某数字的每一位作dp的单位
- 模板：
- ```
  private String[] digits;
      private char s[];
      private int dp[];
  
      public int atMostNGivenDigitSet(String[] digits, int n) {
          this.digits = digits;
          s = Integer.toString(n).toCharArray();
          dp = new int[s.length];
          Arrays.fill(dp, -1); // dp[i] = -1 表示 i 这个状态还没被计算出来
          return f(0, true, false);
      }
  
      private int f(int i, boolean isLimit, boolean isNum) {
          if (i == s.length) return isNum ? 1 : 0; // 如果填了数字，则为 1 种合法方案
          if (!isLimit && isNum && dp[i] >= 0) return dp[i]; // 在不受到任何约束的情况下，返回记录的结果，避免重复运算
          var res = 0;
          if (!isNum) // 前面不填数字，那么可以跳过当前数位，也不填数字
              // isLimit 改为 false，因为没有填数字，位数都比 n 要短，自然不会受到 n 的约束
              // isNum 仍然为 false，因为没有填任何数字
              res = f(i + 1, false, false);
          var up = isLimit ? s[i] : '9'; // 根据是否受到约束，决定可以填的数字的上限
          // 注意：对于一般的题目而言，如果此时 isNum 为 false，则必须从 1 开始枚举，由于本题 digits 没有 0，所以无需处理这种情况
          for (var d : digits) { // 枚举要填入的数字 d
              if (d.charAt(0) > up) break; // d 超过上限，由于 digits 是有序的，后面的 d 都会超过上限，故退出循环
              // isLimit：如果当前受到 n 的约束，且填的数字等于上限，那么后面仍然会受到 n 的约束
              // isNum 为 true，因为填了数字
              res += f(i + 1, isLimit && d.charAt(0) == up, true);
          }
          if (!isLimit && isNum) dp[i] = res; // 在不受到任何约束的情况下，记录结果
          return res;
      }
  ```
- 题目列表
	- 233
	- 面试17.06
	- 600
	- 788
	- 1012
	- 1067
	- 1397