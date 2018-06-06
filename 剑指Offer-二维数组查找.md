在第一行进行查找

<table>
    <tr>
        <td>1</td>
        <td>2</td>
        <td>8</td>
        <td>9</td>
    </tr>
    <tr>
        <td>2</td>
        <td>4</td>
        <td>9</td>
        <td>12</td>
    </tr>
    <tr>
        <td>4</td>
        <td>7</td>
        <td>10</td>
        <td>13</td>
    </tr>
    <tr>
        <td>6</td>
        <td>8</td>
        <td>11</td>
        <td>15</td>
    </tr>
</table>

假设要查找的值为 6，则首先对第一行进行二分查找
情况如下： 
```c 
1. start1 = 0;  
   end1 = 3;  
   middle1 = 2;  
   arr[0][2] = 8;  
2. start1 = 0;  
   end1 = 1;  
   middle1 = 1;  
   arr[0][1] = 2;
3. start1 = 2;  
   end1 = 1;  
   middle1 = 2;  
   跳出循环。
   我们要找的值即为 middle1 - 1 的值。
```
为什么我们要寻找的值为 middle1 - 1 的值？  
&emsp;&emsp;当二分查找到最后一步（即 start 与 end 相差仅为 1）时，此时 middle 的值与 end 的值相等，有两种可能：  
1. 要查找的值小于 middle 位置的值  
   这种情况下，我们要寻找的位置就是 start 的位置，而此时 start 保持不变，由于 middle == end，而 start = end - 1，所以我们要找的位置就是 start = middle - 1。
1. 要查找的值大于 middle 位置的值   
   这种情况下，我们要寻找的位置为 end 所在的位置，而此时 end 保持不变，又令 start = middle + 1，故 middle = (start + end + 1) / 2 就等于 start，也等于 end + 1，所以最终 end = middle - 1。
```c
bool search_(int arr[ROW][COL], int K) {
	int start1 = 0;
	int end1 = COL - 1;
	int middle1 = (start1 + end1 + 1) / 2;

	for (int row = 2, col = 0;;) {

		if (arr[row][middle1] > K) {
			end1 = middle1 - 1;
		} else if (arr[row][middle1] < K) {
			start1 = middle1 + 1;
		} else {
			return true;
		}

		middle1 = (start1 + end1 + 1) / 2;

		if (start1 > end1) {
			int col = middle1 - 1;
			std::cout << col << std::endl;
			return true;
		}
	}
	return false;
}
```