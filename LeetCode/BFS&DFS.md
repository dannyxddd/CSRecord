# BFS/DFS类型

<!-- GFM-TOC -->
* [1. 矩阵中的单词搜索](#1-矩阵中的单词搜索)
* [2. 单词阶梯变换的最短长度](#2-单词阶梯变换的最短长度)
* [3. 找矩阵中被包围的岛屿数量](#3-找矩阵中被包围的岛屿数量)
* [4. 修改矩阵中被包围的字母"O"](#4-修改矩阵中被包围的字母O)
* [5. 矩阵中的最长递增路径](#5-矩阵中的最长递增路径)
* [6. 走有障碍物的矩阵最短路径](#6-走有障碍物的矩阵最短路径)
* [7. 数组分成k个和相等的非空子集合](#7-数组分成k个和相等的非空子集合)
* [8. 课程调度问题](#8-课程调度问题)
<!-- GFM-TOC -->

```html
BFS：广度优先遍历，非常适合求最短路径长度，当BFS遍历完一个点后，不会再来更改这个点的值，常用队列实现，典型应用是树的层次遍历。
DFS：深度优先遍历，非常适合遍历所有路径，当DFS遍历一条路径一定会走到头，那条路径不一定最短，DFS会反复更改同一个点的值。
     常用回溯法递归实现，典型应用是树的先序遍历、拓扑排序等。
```

## 1. 矩阵中的单词搜索
### [79. Word Search(Medium)](https://leetcode.com/problems/word-search/)

题意：在矩阵里搜索单词是否存在，字符来自上下左右，不能重复遍历

```html
board =
[
  ['A','B','C','E'],
  ['S','F','C','S'],
  ['A','D','E','E']
]

Given word = "ABCCED", return true.
Given word = "SEE", return true.
Given word = "ABCB", return false.
```

解法：DFS遍历思想，用一个visited数组标记节点是否遍历，回溯

```java
public boolean exist(char[][] board, String word) {
    boolean visit[][]=new boolean[board.length][board[0].length];
    for(int i=0;i<board.length;i++) {
        for(int j=0;j<board[i].length;j++) {//DFS查找单词是否存在
            if(word.charAt(0)==board[i][j]&&search(board,word,visit,i,j,0)) 
                return true;
        }
    }
    return false;        
}

public boolean search(char[][] board, String word,boolean visit[][],int i,int j,int length) {
    if(length==word.length())return true;
    if(i>=board.length||i<0||j>=board[i].length||j<0||word.charAt(length)!=board[i][j]||visit[i][j])
        return false;
    visit[i][j]=true;
    if(search(board,word,visit,i,j-1,length+1)||search(board,word,visit,i,j+1,length+1)
            ||search(board,word,visit,i-1,j,length+1)||search(board,word,visit,i+1,j,length+1)) 
        return true;

    visit[i][j]=false;
    return false;	
}
```

### [212. Word Search II(Hard)](https://leetcode.com/problems/word-search-ii/)

题意：给一个字符矩阵和一组单词的词典，在矩阵中搜索，返回矩阵中的字符能组成的所有单词

```html
Input: 
board = [
  ['o','a','a','n'],
  ['e','t','a','e'],
  ['i','h','k','r'],
  ['i','f','l','v']
]
words = ["oath","pea","eat","rain"]
Output: ["eat","oath"]
```
解法：这题其实是上题的扩展，上题只找一个单词，这题找多个单词，用字典树+DFS遍历的思想

```java
class TrieNode {
    TrieNode[] node = new TrieNode[26];
    String word;
}
	
public List<String> findWords(char[][] board, String[] words) {
    List<String> list=new ArrayList<String>();
    boolean visit[][]=new boolean[board.length][board[0].length];
    TrieNode root=new TrieNode();//建立字典树
    for(String w:words) {//把词典中每个单词插入到字典树中
        TrieNode p=root;
        for(int i=0;i<w.length();i++) {
            int c=w.charAt(i)-'a';
            if(p.node[c]==null)
                p.node[c]=new TrieNode();
            p=p.node[c];
        }
        p.word=w;//单词的最后一位字符的节点上存word
    }

    for(int i=0;i<board.length;i++) {
        for(int j=0;j<board[i].length;j++) //DFS遍历找所有单词
            searchDFS(board,i,j,visit,list,root);
    }
    return list;
}
    
public void searchDFS(char[][] board, int i, int j, boolean[][] visit, List<String> list, TrieNode p) {
    if(i>=board.length||i<0||j>=board[i].length||j<0||visit[i][j]||p.node[board[i][j]-'a']==null)
	return;
    p=p.node[board[i][j]-'a'];//字符存在
    if(p.word!=null) {
	list.add(p.word);
	p.word=null;//避免再遇到重复单词
    }
    visit[i][j]=true;
    searchDFS(board,i-1,j,visit,list,p);
    searchDFS(board,i+1,j,visit,list,p);
    searchDFS(board,i,j-1,visit,list,p);
    searchDFS(board,i,j+1,visit,list,p);
    visit[i][j]=false;
}
```


## 2. 单词阶梯变换的最短长度
### [127. Word Ladder(Medium)](https://leetcode.com/problems/word-ladder/)

题意：给两个单词和一个字典，问至少多少次变换（1次变换1字符）才能从第一个单词到第二个单词

```html
Input: beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log","cog"]
Output: 5
Explanation: As one shortest transformation is "hit" -> "hot" -> "dot" -> "dog" -> "cog", return its length 5.
```
解法：类似于走迷宫的最短路径，适合用BFS，两个Set分别存已变换的单词和未变换的单词，遍历已变换单词Set，依次修改每个字符，若修改后的单词出现在未变换的Set中，则将其加入已变换单词Set

```java
public int ladderLength(String beginWord, String endWord, List<String> wordList) {
    if(!wordList.contains(endWord))return 0;
    Set<String> reached=new HashSet<String>(),unreached=new HashSet<String>();
    reached.add(beginWord);
    for(int i=0;i<wordList.size();i++)
        unreached.add(wordList.get(i));//将字典中的单词全部加入未变换Set

    int distance = 1;
    while(!reached.contains(endWord)){
        Set<String> se=new HashSet<String>();
        for(String s:reached){//遍历已变换单词Set
            for(int i=0;i<s.length();i++){
                char[] chars=s.toCharArray();
                for(char c='a';c<='z';c++){//依次修改每个单词中的每个字符
                    chars[i]=c;
                    String word=new String(chars);
                    if(unreached.contains(word)){//若修改后的单词出现在未变换的Set中
                        se.add(word);//则将其加入已变换单词Set，并从未变换Set中删除
                        unreached.remove(word);
                    }
                }
            }
        }
        distance++;
        if(se.size()==0)return 0;
        reached=se;

    }
    return distance;

}
```
## 3. 找矩阵中被包围的岛屿数量
### [200. Number of Islands(Medium)](https://leetcode.com/problems/number-of-islands/)

题意：二维矩阵中元素为0或者1，求被0包围的1的岛屿数量，边界可以看成被0包围

```html
Input:
11000
11000
00100
00011
Output: 3
```
解法：用DFS/BFS都能解，DFS更简单，遍历将1周围的连续1都标为0

```java
int count=0;
public int numIslands(char[][] grid) {
    if(grid.length==0||grid[0].length==0)return 0;
    int m=grid.length,n=grid[0].length;
    boolean dp[][]=new boolean[m][n];

    for(int i=0;i<m;i++) {//将“1”以及与其相连的“1”全部变成“0”
        for(int j=0;j<n;j++) {
            if(grid[i][j]=='1'){
                DFS(grid,i,j);	
                count++;
            }
        }
    }		
    return count;
}

public void DFS(char[][] grid,int i,int j) {
    if(i<0||j<0||i>=grid.length||j>=grid[i].length||grid[i][j]=='0')return;
    grid[i][j]='0';
    DFS(grid,i-1,j);
    DFS(grid,i+1,j);
    DFS(grid,i,j-1);
    DFS(grid,i,j+1);
}
```

## 4. 修改矩阵中被包围的字母O
### [130. Surrounded Regions(Medium)](https://leetcode.com/problems/surrounded-regions/)

题意：二维矩阵中元素为O或者X，把所有被X包围的O改成X，边界的O不能改

```html
Example:
X X X X
X O O X
X X O X
X O X X
After running your function, the board should be:
X X X X
X X X X
X X X X
X O X X
```
解法：类似于找岛屿，用DFS/BFS两种方法都可以解决，把边界及其相连的O改为别的字符，DFS用递归，BFS用队列

```java
/*DFS解法*/
public void solve(char[][] board) {
    if(board==null||board.length==0||board[0].length==0)return;
    int row=board.length,col=board[0].length;
    for(int i=0;i<row;i++){//递归把四条边上的“O”以及与其相连的内部“O”改为别的字符
        if(board[i][0]=='O')
            DFS(board,i,0);
        if(board[i][col-1]=='O')
            DFS(board,i,col-1);
    }
    for(int j=0;j<col;j++){
        if(board[0][j]=='O')
            DFS(board,0,j);
        if(board[row-1][j]=='O')
            DFS(board,row-1,j);
    }

    for(int i=0;i<row;i++){//遍历所有，将O改为X，+改回O
         for(int j=0;j<col;j++){
             if(board[i][j]=='O')
                 board[i][j]='X';
             else if(board[i][j]=='+')
                 board[i][j]='O';
         }
    }
}

private void DFS(char[][] board, int i, int j) {
    if(i>=board.length||i<0||j>=board[i].length||j<0)return;
    if(board[i][j]=='O')
        board[i][j]='+';
    if(i>1&&board[i-1][j]=='O')
        DFS(board,i-1,j);
    if(i<board.length-2&&board[i+1][j]=='O')
        DFS(board,i+1,j);
    if(j>1&&board[i][j-1]=='O')
        DFS(board,i,j-1);
    if(j<board[i].length-2&&board[i][j+1]=='O')
        DFS(board,i,j+1);
}

/*BFS解法*/
class Point {
    int x;
    int y;
    Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
}

public void solve(char[][] board) {
    if(board==null||board.length==0||board[0].length==0)return;
    int row=board.length,col=board[0].length;
    Queue<Point> q=new LinkedList<Point>();
    for(int i=0;i<row;i++){ //先把四条边上的O改为别的字符
        if(board[i][0]=='O'){
            board[i][0]='+';
            q.offer(new Point(i,0));
        }   
        if(board[i][col-1]=='O'){
            board[i][col-1]='+';
            q.offer(new Point(i,col-1));
        }    
    }
    for(int j=0;j<col;j++){
        if(board[0][j]=='O'){
            board[0][j]='+';
            q.offer(new Point(0,j));
        } 
        if(board[row-1][j]=='O'){
            board[row-1][j]='+';
            q.offer(new Point(row-1,j));
        }
    }

    while(!q.isEmpty()){ //BFS，把和边界O相连的内部O改为别的字符
        Point p=q.poll();
        int i=p.x,j=p.y;
        if(i>=1&&board[i-1][j]=='O'){
            board[i-1][j]='+';
            q.offer(new Point(i-1,j));
        }
        if(i<=board.length-2&&board[i+1][j]=='O'){
            board[i+1][j]='+';
            q.offer(new Point(i+1,j));
        }   
        if(j>=1&&board[i][j-1]=='O'){
            board[i][j-1]='+';
            q.offer(new Point(i,j-1));
        }
        if(j<=board[i].length-2&&board[i][j+1]=='O'){
            board[i][j+1]='+';
            q.offer(new Point(i,j+1));
        }

    }
    for(int i=0;i<row;i++){ //遍历所有，将O改为X，+改回O
         for(int j=0;j<col;j++){
             if(board[i][j]=='O')
                 board[i][j]='X';
             else if(board[i][j]=='+')
                 board[i][j]='O';
         }
    }
}
```

## 5. 矩阵中的最长递增路径
### [329. Longest Increasing Path in a Matrix(Hard)](https://leetcode.com/problems/longest-increasing-path-in-a-matrix/)

题意：给定二维矩阵，找矩阵中数字递增的最长路径长度，只能上下左右四个方向走，不能回走或者超过边界。

```html
Example 1:
Input: nums = 
[
  [9,9,4],
  [6,6,8],
  [2,1,1]
] 
Output: 4 
Explanation: The longest increasing path is [1, 2, 6, 9].

Example 2:
Input: nums = 
[
  [3,4,5],
  [3,2,6],
  [2,2,1]
] 
Output: 4 
Explanation: The longest increasing path is [3, 4, 5, 6]. Moving diagonally is not allowed.
```

解法：用DP记录经过当前位置的最长长度，用DFS递归找，遇到已经有的dp直接返回，避免重复计算

```java
public int longestIncreasingPath(int[][] matrix) {
    if(matrix.length==0||matrix[0].length==0)return 0;
    int dp[][]=new int[matrix.length][matrix[0].length];
    int result=0;
    for(int i=0;i<matrix.length;i++)
        for(int j=0;j<matrix[0].length;j++)
            result=Math.max(result,DFS(matrix,i,j,Integer.MIN_VALUE,dp));
    return result;
}

public int DFS(int[][] matrix,int i,int j,int pre,int dp[][]){        
    if(i<0||i>=matrix.length||j<0||j>=matrix[0].length||pre>=matrix[i][j]) return 0;
    if(dp[i][j]!=0)return dp[i][j];
    int res1=Math.max(DFS(matrix,i-1,j,matrix[i][j],dp),DFS(matrix,i+1,j,matrix[i][j],dp));
    int res2=Math.max(DFS(matrix,i,j-1,matrix[i][j],dp),DFS(matrix,i,j+1,matrix[i][j],dp));           dp[i][j]=Math.max(res1,res2)+1;
    return dp[i][j];
}
```

## 6. 走有障碍物的矩阵最短路径
### [1091. Shortest Path in Binary Matrix(Medium)](https://leetcode.com/problems/shortest-path-in-binary-matrix/)

题意：N✖️N矩阵，有一些障碍物，0表示能走，1表示不能走，可以往8个方向走，求从(0,0)到(N-1，N-1)的最短路径

```html
Example 1:
Input: [[0,1],[1,0]]
Output: 2

Example 2:
Input: [[0,0,0],[1,1,0],[1,1,0]]
Output: 4 

Note:
1 <= grid.length == grid[0].length <= 100
grid[r][c] is 0 or 1
```

解法：找最短路径，用BFS解

```java
public int shortestPathBinaryMatrix(int[][] grid) {
    int m=grid.length,n=grid[0].length;
    if(grid[0][0]==1||grid[m-1][n-1]==1)return -1;
    int move[][]=new int[][]{{0,1},{0,-1},{-1,0},{1,0},{-1,-1},{-1,1},{1,-1},{1,1}}; 

    int sum=0;
    boolean[][] visited = new boolean[m][n];
    visited[0][0]=true;
    Queue<int[]> q=new LinkedList<>();
    q.offer(new int[]{0,0});

    while(!q.isEmpty()) {
        int size=q.size();
        for(int i=0;i<size;i++) {
            int[] point=q.poll();
            if(point[0]==m-1&&point[1]==n-1) return ++sum;
            for(int j=0;j<8;j++) {
                int nextX=point[0]+move[j][0];
                int nextY=point[1]+move[j][1];
                if(nextX>=0&&nextX<m&&nextY>=0&&nextY<n&&!visited[nextX][nextY]&&grid[nextX][nextY]==0) {
                    q.offer(new int[]{nextX,nextY});
                    visited[nextX][nextY]=true;
                }

            }
        }
        sum++;
    }
    return -1;
}
```
### [剑指offer-矩阵能遍历的格子数]

剑指offer相似题型：地上有一个m行和n列的方格，一个机器人从坐标0,0的格子开始移动，每一次只能向左，右，上，下四个方向移动一格，但是不能进入行坐标和列坐标的数位之和大于k的格子。例如，当k为18时，机器人能够进入方格（35,37），因为3+5+3+7 = 18。但是，它不能进入方格（35,38），因为3+5+3+8 = 19。请问该机器人能够达到多少个格子？

解法：遍历所有路径，用DFS解

```java
public int movingCount(int threshold, int rows, int cols){
    boolean visit[][]=new boolean[rows][cols];
    return ismoving(rows,cols,0,0,threshold,visit);
}
private int ismoving(int rows, int cols, int i, int j, int threshold, boolean visit[][]) {
    if(i<0||i>=rows||j<0||j>=cols||calnum(i)+calnum(j)>threshold||visit[i][j])return 0; 
    visit[i][j]=true;
    return ismoving(rows,cols,i-1,j,threshold,visit)+ismoving(rows,cols,i+1,j,threshold,visit)+ismoving(rows,cols,i,j-1,threshold,visit)+ismoving(rows,cols,i,j+1,threshold,visit)+1;
}
private int calnum(int i) {
    int sum=0;
    while(i!=0) {
       sum+=i%10; i/=10;
    }
    return sum;
}

```

## 7. 数组分成k个和相等的非空子集合
### [698. Partition to K Equal Sum Subsets(Medium)](https://leetcode.com/problems/partition-to-k-equal-sum-subsets/)

题意：给一个数组nums和一个数字k，问数组能不能分成k个非空子集合，使得每个子集合的和相同

```html
Input: nums = [4, 3, 2, 3, 5, 2, 1], k = 4
Output: True
Explanation: It's possible to divide it into 4 subsets (5), (1, 4), (2,3), (2,3) with equal sums.

```

解法：用DFS递归找，用visited数组来记录哪些数组已经被选中了，然后调用递归函数使得k个子集合的每个和为target = sum/k。k=1时说明此时只需要组一个子集合，当前的就是直接返回true。如果cursum等于target了，再次调用递归，此时传入k-1，继续找下一个子数组。否则就从start开始遍历数组。

```java
public boolean canPartitionKSubsets(int[] nums, int k) {
    int sum=0;
    for(int i=0;i<nums.length;i++) sum+=nums[i];
    if(sum%k!=0)return false;
    boolean visited[]=new boolean[nums.length];
    return canPartition(nums,k,sum/k,0,0,visited);
}
public boolean canPartition(int[] nums, int k, int target, int cursum, int start, boolean visited[]){
    if(k==1) return true;
    if(cursum==target)return canPartition(nums,k-1,target,0,0,visited);//找下一个子数组
    for(int i=start;i<nums.length;i++){
        if(visited[i])continue;
        visited[i]=true;
        if(canPartition(nums,k,target,cursum+nums[i],i+1,visited))return true;
        visited[i]=false;
    }
    return false;
}
```

## 8. 课程调度问题
### [207. Course Schedule(Medium)](https://leetcode.com/problems/course-schedule/)

题意：给定链表，元素是两个课程的先后顺序数组，问能不能修完全部课程

```html
Example 1:
Input: 2, [[1,0]] 
Output: true
Explanation: There are a total of 2 courses to take. 
             To take course 1 you should have finished course 0. So it is possible.

Example 2:
Input: 2, [[1,0],[0,1]]
Output: false
Explanation: There are a total of 2 courses to take. 
             To take course 1 you should have finished course 0, and to take course 0 you should
             also have finished course 1. So it is impossible.
```

解法：图+BFS，保存每个节点的后续课程列表，用队列把入度为0的加到列表中；图+DFS，保存每个节点的先修课程列表，用boolean数组判断每条节点链是否已访问过

```java
public boolean canFinish(int numCourses, int[][] prerequisites) {
    ArrayList<Integer>[] graph =new ArrayList[numCourses];
    int outdegree[] =new int[numCourses];
    Queue<Integer> queue=new LinkedList<Integer>();
    int count=0;
    for(int i=0;i<numCourses;i++) 
        graph[i]=new ArrayList<Integer>();
    for(int i=0;i<prerequisites.length;i++) {
        outdegree[prerequisites[i][1]]++;
        graph[prerequisites[i][0]].add(prerequisites[i][1]);
    }
    for(int i=0;i<numCourses;i++) {
        if(outdegree[i]==0) {
            queue.add(i); count++;
        }	
    }

    while(queue.size()!=0) {
        int course=queue.poll();
        for(int i=0;i<graph[course].size();i++) {
            int p=graph[course].get(i);
            outdegree[p]--;
            if(outdegree[p]==0) {
                queue.add(p); count++;
            }					
        }
    }
    return count==numCourses;    
}
```

### [210. Course Schedule II(Medium)](https://leetcode.com/problems/course-schedule-ii/)

题意：给定链表，元素是两个课程的先后顺序数组，给出课程学习的调度列表

```html
Example 1:

Input: 2, [[1,0]] 
Output: [0,1]
Explanation: There are a total of 2 courses to take. To take course 1 you should have finished   
             course 0. So the correct course order is [0,1] .

Example 2:
Input: 4, [[1,0],[2,0],[3,1],[3,2]]
Output: [0,1,2,3] or [0,2,1,3]
Explanation: There are a total of 4 courses to take. To take course 3 you should have finished both     
             courses 1 and 2. Both courses 1 and 2 should be taken after you finished course 0. 
             So one correct course order is [0,1,2,3]. Another correct ordering is [0,2,1,3] .
```

解法：图+BFS，保存每个节点的后续课程列表，用队列把入度为0的加到列表中

```java
public int[] findOrder(int numCourses, int[][] prerequisites) {
    ArrayList<Integer>[] graph =new ArrayList[numCourses];
    int[] re=new int[numCourses];
    int[] indegree = new int[numCourses];
    for(int i=0;i<numCourses;i++)
        graph[i]=new ArrayList<Integer>();
    for(int i=0;i<prerequisites.length;i++){
        graph[prerequisites[i][1]].add(prerequisites[i][0]);
        indegree[prerequisites[i][0]]++;
    }
    Queue<Integer> q = new LinkedList<Integer>();
    int k=0;
    for(int i=0;i<numCourses;i++){
        if(indegree[i]==0) {
            q.offer(i); re[k++]=i;
        }		
    }

    while(!q.isEmpty()) {
        int pre=q.poll();
        for(int i=0;i<graph[pre].size();i++) {
            indegree[graph[pre].get(i)]--;
            if(indegree[graph[pre].get(i)]==0) {
                q.offer(graph[pre].get(i));
                re[k++]=graph[pre].get(i);
            }
        }
    }
    return k==numCourses?re:new int[0];
}
```

