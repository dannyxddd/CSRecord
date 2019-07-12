# BFS/DFS类型

<!-- GFM-TOC -->
* [1. 单词阶梯变换最短长度](#1-单词阶梯变换最短长度)
* [2. 两个字符串的最短编辑距离](#2-两个字符串的最短编辑距离)
* [3. 两个字符串的最短公共超序列](#3-两个字符串的最短公共超序列)
* [4. 一组字符串的最长公共前缀](#4-一组字符串的最长公共前缀)
<!-- GFM-TOC -->

> BFS：广度优先遍历，非常适合求最短路径长度，当BFS遍历完一个点后，不会再来更改这个点的值，常用队列实现，典型应用是树的层次遍历。
> DFS：深度优先遍历，非常适合遍历所有路径，当DFS遍历一条路径一定会走到头，那条路径不一定最短，DFS会反复更改同一个点的值，常用回溯法递归实现，典型应用是树的先序遍历、拓扑排序等。

## 1. 单词阶梯变换最短长度
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
