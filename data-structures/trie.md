# 트라이 (Trie, 접두사 트리)

## 분류
- **카테고리**: 자료구조
- **관련 알고리즘**: [이진 탐색](../search/binary-search.md), [KMP](../string/kmp.md)

## 개요
트라이(Trie)는 문자열 집합을 효율적으로 저장하고 검색하기 위한 트리 기반 자료구조이다. 각 노드가 문자 하나를 나타내며, 루트에서 특정 노드까지의 경로가 하나의 문자열(또는 접두사)을 형성한다. 삽입·검색·접두사 검색 모두 문자열 길이에 비례하는 \(O(L)\) 시간에 수행할 수 있어, 대량의 문자열을 다루는 문제에서 매우 강력하다.

> **네이버는 검색 기업**이므로, 검색 자동완성·사전 검색·접두사 매칭과 직접적으로 관련된 Trie는 코딩 테스트에서 특히 중요하게 출제될 수 있다. 검색어 자동완성의 핵심 자료구조가 바로 Trie이다.

## 핵심 로직
1. **노드 구조**: 각 노드는 자식 노드 맵(또는 배열)과, 해당 노드가 단어의 끝인지를 나타내는 플래그를 가진다.
2. **삽입 (insert)**: 문자열의 각 문자를 순서대로 따라가며 노드를 생성하고, 마지막 문자에서 단어 종료 플래그를 표시한다.
3. **검색 (search)**: 문자열의 각 문자를 따라가며 노드를 탐색하고, 마지막 노드의 단어 종료 플래그를 확인한다.
4. **접두사 검색 (startsWith)**: 검색과 동일하지만, 경로가 존재하기만 하면 `true`를 반환한다 (단어 종료 여부 무관).
5. **자동완성**: 접두사까지 탐색한 후, 해당 노드를 루트로 DFS를 수행하여 모든 완성 가능한 단어를 수집한다.

> 자식 노드 저장 방식으로 **HashMap** 기반(유니코드/한글 대응 가능)과 **배열** 기반(알파벳 소문자 26개, 빠름)이 있다.

## 언제 사용하는가?
- **문자열 접두사 검색**: 주어진 접두사로 시작하는 단어가 있는지 빠르게 판별
- **자동완성 구현**: 입력 중인 접두사에 대해 가능한 완성 단어 목록을 제공
- **사전(Dictionary) 구현**: 대량의 단어를 저장하고 빠르게 검색
- **문자열 집합에서 빠른 검색**: HashSet 대비 접두사 관련 연산에서 압도적 성능
- **IP 라우팅 (Longest Prefix Match)**: 가장 긴 접두사 일치를 찾는 문제
- **문자열 카운팅**: 접두사별 등장 횟수 관리

## Java 구현
```java
import java.util.*;

public class Trie {

    // ========================================
    // 배열 기반 Trie 노드 (알파벳 소문자 전용, 빠름)
    // ========================================
    static class TrieNode {
        TrieNode[] children = new TrieNode[26];
        boolean isEndOfWord = false;
        int prefixCount = 0; // 이 노드를 접두사로 가지는 단어 수
    }

    private TrieNode root;

    public Trie() {
        root = new TrieNode();
    }

    // 단어 삽입
    public void insert(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) {
                node.children[idx] = new TrieNode();
            }
            node = node.children[idx];
            node.prefixCount++;
        }
        node.isEndOfWord = true;
    }

    // 단어가 정확히 존재하는지 검색
    public boolean search(String word) {
        TrieNode node = findNode(word);
        return node != null && node.isEndOfWord;
    }

    // 주어진 접두사로 시작하는 단어가 있는지 확인
    public boolean startsWith(String prefix) {
        return findNode(prefix) != null;
    }

    // 주어진 접두사로 시작하는 단어의 개수
    public int countWordsWithPrefix(String prefix) {
        TrieNode node = findNode(prefix);
        return node == null ? 0 : node.prefixCount;
    }

    // 접두사에 해당하는 노드까지 탐색
    private TrieNode findNode(String prefix) {
        TrieNode node = root;
        for (char c : prefix.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) {
                return null;
            }
            node = node.children[idx];
        }
        return node;
    }

    // ========================================
    // 자동완성: 주어진 접두사로 시작하는 모든 단어 반환
    // ========================================
    public List<String> autocomplete(String prefix) {
        List<String> results = new ArrayList<>();
        TrieNode node = findNode(prefix);
        if (node == null) return results;

        dfsCollect(node, new StringBuilder(prefix), results);
        return results;
    }

    // DFS로 현재 노드 하위의 모든 완성 단어를 수집
    private void dfsCollect(TrieNode node, StringBuilder path, List<String> results) {
        if (node.isEndOfWord) {
            results.add(path.toString());
        }

        for (int i = 0; i < 26; i++) {
            if (node.children[i] != null) {
                path.append((char) ('a' + i));
                dfsCollect(node.children[i], path, results);
                path.deleteCharAt(path.length() - 1); // 백트래킹
            }
        }
    }

    // ========================================
    // HashMap 기반 Trie (유니코드/한글 대응)
    // ========================================
    static class HashTrieNode {
        Map<Character, HashTrieNode> children = new HashMap<>();
        boolean isEndOfWord = false;
    }

    static class HashTrie {
        HashTrieNode root = new HashTrieNode();

        void insert(String word) {
            HashTrieNode node = root;
            for (char c : word.toCharArray()) {
                node.children.putIfAbsent(c, new HashTrieNode());
                node = node.children.get(c);
            }
            node.isEndOfWord = true;
        }

        boolean search(String word) {
            HashTrieNode node = root;
            for (char c : word.toCharArray()) {
                if (!node.children.containsKey(c)) return false;
                node = node.children.get(c);
            }
            return node.isEndOfWord;
        }

        boolean startsWith(String prefix) {
            HashTrieNode node = root;
            for (char c : prefix.toCharArray()) {
                if (!node.children.containsKey(c)) return false;
                node = node.children.get(c);
            }
            return true;
        }
    }

    public static void main(String[] args) {
        // === 배열 기반 Trie 테스트 ===
        System.out.println("=== 배열 기반 Trie ===");
        Trie trie = new Trie();

        // 단어 삽입
        String[] words = {"apple", "app", "application", "apply", "banana", "band", "ban"};
        for (String word : words) {
            trie.insert(word);
        }

        // 검색 테스트
        System.out.println("apple 검색: " + trie.search("apple"));     // true
        System.out.println("app 검색: " + trie.search("app"));         // true
        System.out.println("appl 검색: " + trie.search("appl"));       // false (완전한 단어 아님)

        // 접두사 검색
        System.out.println("app 접두사 존재: " + trie.startsWith("app")); // true
        System.out.println("xyz 접두사 존재: " + trie.startsWith("xyz")); // false

        // 접두사로 시작하는 단어 수
        System.out.println("app 접두사 단어 수: " + trie.countWordsWithPrefix("app")); // 4

        // 자동완성
        System.out.println("\n=== 자동완성 ===");
        System.out.println("'app' 자동완성: " + trie.autocomplete("app"));
        System.out.println("'ban' 자동완성: " + trie.autocomplete("ban"));
        System.out.println("'xyz' 자동완성: " + trie.autocomplete("xyz"));

        // === HashMap 기반 Trie 테스트 ===
        System.out.println("\n=== HashMap 기반 Trie (유니코드 대응) ===");
        HashTrie hashTrie = new HashTrie();
        hashTrie.insert("hello");
        hashTrie.insert("help");
        hashTrie.insert("world");

        System.out.println("hello 검색: " + hashTrie.search("hello"));     // true
        System.out.println("hel 접두사: " + hashTrie.startsWith("hel"));    // true
        System.out.println("wor 접두사: " + hashTrie.startsWith("wor"));    // true
        System.out.println("xyz 검색: " + hashTrie.search("xyz"));         // false
    }
}
```

## 시간 복잡도 / 공간 복잡도
| | 복잡도 |
|---|---|
| **삽입** | \(O(L)\) — L은 삽입할 문자열의 길이 |
| **검색** | \(O(L)\) — L은 검색할 문자열의 길이 |
| **접두사 검색** | \(O(L)\) — L은 접두사의 길이 |
| **자동완성** | \(O(L + K)\) — L은 접두사 길이, K는 결과 단어들의 총 문자 수 |
| **공간 복잡도** | \(O(N \times L)\) — N개의 문자열, 평균 길이 L (최악의 경우) |

> 배열 기반은 노드당 고정 크기(26칸)를 사용하므로 메모리를 더 쓰지만 접근이 \(O(1)\)로 빠르다. HashMap 기반은 필요한 만큼만 메모리를 사용하지만 해시 연산 오버헤드가 있다. 코딩 테스트에서는 **배열 기반**이 일반적이다.

## 관련 문제 유형
- **문자열 접두사 검색**: 사전에서 특정 접두사로 시작하는 단어 존재 여부
- **자동완성 시스템**: 검색어 입력 시 추천 단어 목록 제공
- **문자열 집합 관리**: 대량의 문자열 삽입/검색/삭제
- **가장 긴 공통 접두사**: 문자열 배열에서 가장 긴 공통 접두사 찾기
- **단어 퍼즐 (Boggle)**: 격자에서 사전에 있는 단어 탐색
- **IP 주소 매칭**: 가장 긴 접두사가 일치하는 라우팅 규칙 탐색
