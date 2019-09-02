# BFS/DFS类型

<!-- GFM-TOC -->
* [1. 矩阵中的单词搜索](#1-矩阵中的单词搜索)
* [2. 单词阶梯变换的最短长度](#2-单词阶梯变换的最短长度)
* [3. 找矩阵中被包围的岛屿数量](#3-找矩阵中被包围的岛屿数量)
* [4. 修改矩阵中被包围的字母"O"](#4-修改矩阵中被包围的字母O)
<!-- GFM-TOC -->

> BFS：广度优先遍历，非常适合求最短路径长度，当BFS遍历完一个点后，不会再来更改这个点的值，常用队列实现，典型应用是树的层次遍历。
> DFS：深度优先遍历，非常适合遍历所有路径，当DFS遍历一条路径一定会走到头，那条路径不一定最短，DFS会反复更改同一个点的值，常用回溯法递归实现，典型应用是树的先序遍历、拓扑排序等。

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
