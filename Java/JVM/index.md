---
title: JVM
layout: default
parent: Java核心技术
---

# JVM



你好的英文==hello world==计算机

<b><font color=#ff6200>hello world</font></b>













`abcdefg`



``` java
class Test {
    int a;
    int b;
    int c;
	public void test() {
        if (a == b) 
            if (a == c) 
                System.out.println("same");
    }
}
```







```java
class TrieNode {
    private TrieNode[] children;
    private boolean isEndOfWord;

    public TrieNode() {
        children = new TrieNode[26]; // 假设只处理英文小写字母
        isEndOfWord = false;
    }

    // 插入一个单词到Trie
    public void insert(String word) {
        TrieNode node = this;
        for (int i = 0; i < word.length(); i++) {
            char c = word.charAt(i);
            int index = c - 'a';
            if (node.children[index] == null) {
                node.children[index] = new TrieNode();
            }
            node = node.children[index];
        }
        node.isEndOfWord = true;
    }

    // 检查单词是否在Trie中
    public boolean search(String word) {
        TrieNode node = this;
        for (int i = 0; i < word.length(); i++) {
            char c = word.charAt(i);
            int index = c - 'a';
            if (node.children[index] == null) {
                return false;
            }
            node = node.children[index];
        }
        return node.isEndOfWord;
    }
}

public class Trie {
    private TrieNode root;

    public Trie() {
        root = new TrieNode();
    }

    public void insert(String word) {
        root.insert(word);
    }

    public boolean search(String word) {
        return root.search(word);
    }

    public static void main(String[] args) {
        Trie trie = new Trie();
        trie.insert("hello");
        trie.insert("world");
        System.out.println(trie.search("hello")); // 输出 true
        System.out.println(trie.search("world")); // 输出 true
        System.out.println(trie.search("java")); // 输出 false
    }
}

```

