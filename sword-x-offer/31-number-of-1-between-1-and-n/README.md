# 整数中1出现的次数（从1到n整数中1出现的次数）

求出1\~13的整数中1出现的次数,并算出100\~1300的整数中1出现的次数？

为此他特别数了一下1\~13中包含1的数字有1、10、11、12、13因此共出现6次,但是对于后面问题他就没辙了。

ACMer希望你们帮帮他,并把问题更加普遍化,可以很快的求出任意非负整数区间中1出现的次数（从1 到 n 中1出现的次数）。

## 遍历所有

- 时间O(N)

```cpp
class Solution {
public:
    int NumberOf1Between1AndN_Solution(int n) {
        if(n <= 0) return 0;
        int sum_count = 0;
        for(int num = 1; num <= n; num++) {
            int cur_num = num, count = 0;
            while(cur_num) {
                if(cur_num%10 == 1)
                    count++;
                cur_num /= 10;
            }
            sum_count += count;
        }
        return sum_count;
    }
};
```

## 数字特点

- 时间O(log_{10} N)

### 数字特点1

- 链接：https://www.nowcoder.com/questionTerminal/bd7f978302044eee894445e244c7eee6  
- 来源：牛客网  

设N = abcde ,其中abcde分别为十进制中各位上的数字。
如果要计算百位上1出现的次数，它要受到3方面的影响：百位上的数字，百位以下（低位）的数字，百位以上（高位）的数字。  

1. 如果百位上数字为0，百位上可能出现1的次数由更高位决定。比如：12013，则可以知道百位出现1的情况可能是：100\~199，1100\~1199,2100\~2199，，...，11100\~11199，一共1200个。可以看出是由更高位数字（12）决定，并且等于更高位数字（12）乘以 当前位数（100）。
2. 如果百位上数字为1，百位上可能出现1的次数不仅受更高位影响还受低位影响。比如：12113，则可以知道百位受高位影响出现的情况是：100\~199，1100\~1199,2100\~2199，，....，11100\~11199，一共1200个。和上面情况一样，并且等于更高位数字（12）乘以 当前位数（100）。但同时它还受低位影响，百位出现1的情况是：12100\~12113,一共114个，等于低位数字（113）+1。  
3. 如果百位上数字大于1（2\~9），则百位上出现1的情况仅由更高位决定，比如12213，则百位出现1的情况是：100\~199,1100\~1199，2100\~2199，...，11100\~11199,12100\~12199,一共有1300个，并且等于更高位数字+1（12+1）乘以当前位数（100）。

```cpp
class Solution {
public:
    int NumberOf1Between1AndN_Solution(int n) {
        int count = 0, bit = 1;
        while((n/bit) != 0) {
            int cur = (n / bit) % 10; // 当前位数字
            int high = n / bit / 10;  // 高位数字
            int low = n - n / bit * bit; //低位数字
            if(cur == 0) // 当前位为0, 1的个数仅取决于高位
                count += high * bit;
            else if(cur == 1) // 当前位为1, 1的个数取决于高位和低位（低位包括一个0）
                count += high * bit + low + 1;
            else // 当前位大于1, 1的个数取决于高位
                count += (high + 1) * bit;
            bit *= 10; // 进位
        }
        return count;
    }
};
```

### 数字特点2  

主要思路：设定整数点（如1、10、100等等）作为位置点bit（对应n的各位、十位、百位等等），分别对每个数位上有多少包含1的点进行分析。对 n 进行分割，分为两部分，高位 `left = n / bit` ，低位 `right = n % bit` 。 

- 当 bit 表示百位，且百位对应的数 >= 2，如 `n = 31456, bit = 100`，则`left = 314, right = 56`，此时百位为 1 的次数有 `left / 10 + 1 = 32`（最高两位的计算是从0到31），此外，每个 1 对应低位的 100 ，都包含 100 个连续的点，即共有`(left / 10 + 1) * 100`个点的百位为 1，即`(left / 10 + 1) * bit`
- 当 bit 表示百位，且百位对应的数为 1，计算包含两部分（高位，低位）。如 `n = 31156, bit = 100`，则`left = 311, right = 56`，此时百位对应的就是 1 ，则高位共有`left / 10`（最高两位的计算是从0到30）次，每个高位都包含低位 100 个连续点，当最高两位为 31 时 （即`left = 311`）；低位的计算只对应点`00~56`，共 `right+1` 次，所有点加起来共有 `(left / 10 * 100) + (right + 1)`，这些点百位对应为 1 ，即 `(left / 10 * bit) + (right + 1)`  
- 当 bit 表示百位，且百位对应的数为 0，如 `n = 31056, bit = 100`，则 `left = 310, right = 56` ，此时百位为 1 的次数有 `left / 10 = 31`（最高两位的计算是从0到30，即`left/10*bit`    

综合以上三种情况：

1. 当百位为 0 ，`left/10*bit`；  
2. 当百位为 1 ，`left/10*bit + right+1`。用`right%10 == 1`来判断低位，区别开百位为0的情况；
3. 当百位 >=2 ，`(left/10+1)*bit`，因 `(left + 8) / 10` 的进位不会影响百位为0和1的情况，因而写为 `(left+8)/10 * bit`。

```cpp
class Solution {
public:
    int NumberOf1Between1AndN_Solution(int n) {
        int count = 0; 
        for(long long bit = 1; bit <= n; bit *= 10) {
            // bit 表示当前分析的是哪一个数位
            int left = n / bit, right = n % bit;// 左半边（含当前位），右半边
            count += (left+8)/10*bit + (left%10==1)*(right+1);
        }
        return count;
    }
};
```

## 归纳总结

- 时间O(log_{10} N)
- 链接：https://www.nowcoder.com/questionTerminal/bd7f978302044eee894445e244c7eee6
- 来源：牛客网，在作者基础上做了大量修改    

首先将位数分类：  

- 个位数，1会每隔10出现一次，例如1、11、21等等，发现以10为一个阶梯的话，每一个完整的阶梯里面都有一个1，例如数字22，按照10为间隔来分三个阶梯，在完整阶梯0-9，10-19之中都有一个1，但是19之后有一个不完整的阶梯，我们需要去判断这个阶梯中会不会出现1，易推断知，如果最后这个露出来的部分小于1，则不可能出现1（这个归纳换做其它数字也成立）。  

个位上1出现的个数：`count(1) = (n/10)*1 + (n%10!=0 ? 1 : 0)` 

- 十位数，出现1的情况应该是10-19。沿用阶梯理论，每隔100这个阶梯，10-19这组数就会出现，如317有三段完整阶梯：0-99，100-199，200-299，每一阶梯里都会出现10次1（即10-19），最后再分析不完整阶梯step_left：300-317中的17。
    - 如果不完整阶梯的低位大于19，那么就有10个1；
    - 如果不完整阶梯的低位小于10，那么十位数不会有1；
    - 如果不完整阶梯的低位位于10-19之间的，结果应该是step_left - 10 + 1。如317的step_left = 17，十位上1出现的个数为：17-10+1=8个。

那么现在可以归纳：十位上1出现的个数为：  
1. `step_left = n % 100`  
2. `count(10) = (n / 100) * 10 + (step_left>19) ? 10 : (step_left<10 ? 0 : step_left-10+1)`  

- 百位数，都会出现1的百位数是100-199，每个阶梯中1出现100次，阶梯间隔为1000。如2139，先算阶梯数 * 完整阶梯中1在百位出现的个数，即n/1000 * 100得到前两个阶梯中1的个数，那么再算不完整阶梯的低位，即139，沿用上述思想，百位的不完整阶梯数的1的计算：
    - step_left<100，0个百位1；
    - step_left>199，100个百位1；
    - 100<=step_left<=199，step_left - 100 + 1个百位1。

归纳出百位上出现1的个数：  

1. `step_left = n % 1000`  
2. `count(100) = (n / 1000) * 100 + (step_left>199) ? 10 : (step_left>100 ? 0 : step_left-100+1)`  
后面的依次类推....  

那么我们把个位数上算1的个数的式子也纳入归纳式中：

1. `step_left = n % 10`  
2. `count(1) = n / 10 * 1 + (step_left>1) ? 1 : (step_left<1 ? 0 : step_left-1+1)`   

各个位数的归纳式已规整。现在对位数进一步抽象出来，设 bit 为所在的位数，`bit = 1`表示个位的位数，`bit = 10`表示十位的位数，则最终的归纳式：  

1. `step_left = n % (bit * 10)`  
2. `count(bit) = (n / (bit * 10)) * bit + (step_left>bit*2-1) ? bit : (step_left<bit ? 0 : step_left-bit+1)`   
3. `res += count(bit), bit = Math.pow(10, bit_idx), 0<=bit_idx<=log10(n)`  

然后实现代码，通过：

```cpp
class Solution {
public:
    int NumberOf1Between1AndN_Solution(int n) {
        int count = 0; 
        for(long long bit = 1; bit <= n; bit *= 10) {
            // i 表示当前分析的是哪一个数位
            int step_left = n % (bit * 10);
            count += (n / (bit * 10)) * bit +
                (
                    (step_left>bit*2-1) ? bit :
                                          (step_left<bit ? 0 : step_left-bit+1)
                );
        }
        return count;
    }
};
```

`step_left > bit*2-1 ? 0`，等同于 `step_left-bit+1 > bit ? 0`，即保证 `step_left-bit+1` 在区间 `(bit, +∞]` 则结果为0，
否则`step_left-bit+1`不在区间(-∞, bit]，结果需要根据后面的if-else进一步计算，仔细观察，不难发现根据`step_left>bit*2-1`，
即`step_left-bit+1` 是否在区间 `(bit, +∞]` 的判断，可以完美地将这两个if-else，进行合并，划分三段：

```cpp
int step_left = n % (bit * 10);
count += (n / (bit * 10)) * bit +
	 (
             (step_left-bit+1>bit) ? bit :
	                             (step_left<bit ? 0 : step_left-bit+1)
         );
```

下面是简化两个if-else为min(max())的分析（这里特别难想，好不容易想明白了，记录如下，同时佩服作者）：

第二个if-else可写为`(step_left-bit+1<1 ? 0 : step_left-bit+1)`，其含义就是`step_left-bit+1`比1小，就取0，否则`step_left-bit+1`大于1，则取自身，那么第二个if-else可写为`max(0, step_left-bit+1)`：

```cpp
(step_left-bit+1>bit) ? bit : max(0, step_left-bit+1)
```

现在就剩下第一个if-else，由于`max(0, step_left-bit+1)`的存在，无非两种情况：0或者step_left_bit+1，为0表示`step_left-bit+1 <=0`。

1. 当`step_left-bit+1 <= 0`，则两个判断变为一个：`(step_left-bit+1>bit) ? bit : 0`  
	因为bit=10^(log_{10} n)，所以bit>=1，又因为这种情况的前提是`step_left-bit+1 <= 0`，因而，结果为0，即外层if-else判断的是两个数取小，因而外层if-else可以写成min；  
2. 当`step_left-bit+1 > 0`，即`step_left+1 > bit`，则两个判断变为一个：`(step_left-bit+1>bit) ? bit : step_left-bit+1`  
	因为`step_left+1 > bit`，bit=10^n，n>=1，那么bit即使乘以2，也不会比step_left大，所以这种情况下的条件判断，结果为小的，即bit。因而外层if-else可以写成min。

```
min( max(step_left−bit+1, 0),
     bit)
```

将step_left放到count的计算公式中，即为下面的代码：

```cpp
class Solution {
    inline int max(int a, int b) { return a > b ? a : b; }
    inline int min(int a, int b) { return a < b ? a : b; }
public:
    int NumberOf1Between1AndN_Solution(int n) {
        if(n <= 0) return 0;
        int count = 0;
        for(long bit = 1; bit <= n; bit *= 10) {
            long diviver = bit * 10;
            count += (n / diviver) * bit + min(max(n % diviver - bit + 1, 0), bit);
        }
        return count;
    }
};
```

## 转为字符串

- 时间：O(N)

无论是一次开辟好所有字符串所需空间或是每次开辟，时间复杂度均为O(N)。

```cpp
class Solution {
public:
    int NumberOf1Between1AndN_Solution(int n) {
        int ones = 0; char table[10];
        for(int num = 1; num <= n; num++) {
            sprintf(table, "%d", num);
            ones += count(begin(table), end(table), '1');
        }
        return ones;
    }
};
```
