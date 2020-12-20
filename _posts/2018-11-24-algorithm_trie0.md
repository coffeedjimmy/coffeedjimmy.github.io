---
layout: post
title: "[알고리즘] Trie(트라이)"
tags: [알고리즘, 트라이]
comments: true
---

.

- retrieval에서 유래 = prefix tree
- N개의 원소를 갖는 BST에서 원하는 원소를 찾는 데에는 O(logN)번 비교
  - 단, 이는 각 원소간의 비교를 상수시간에 할 수 있을 때.
  - 문자열의 경우에는 문자열의 최대길이 M을 곱한 O(MlogN)이 최종 시간 복잡도
- 위의 문제를 해결하기 위해 Trie가 고안
  - 집합 내에서 원하는 원소를 찾는 작업 => O(M)
- trie: 집합에 포함된 문자열의 접두사들에 대응되는 노드들이 서로 연결된 트리
- trie의 루트는 항상 길이 0인 문자열, 노드의 깊이가 깊어질 때마다 대응되는 문자엳르이 길이는 1씩 늘어남.
- 루트에서 한 노드까지 내려가는 경로에서 만나는 글자들으 모으면 해당 노드에 대응되는 접두사를 얻을 수 있음.
- 트라이의 한 노드 객체 = 자손 노드들을 가리키는 포인터 목록 + 종료노드인지를 나타내는 bool
- alphabet의 경우에는 26개짜리 포인터배열
  - 메모리를 엄청나게 낭비하지만 다음 노드를 찾는 과정에서 어떤 탐색도 필요하지 않다는 장점.
- 빠른 속도가 필요한 검색엔진이나 기타 문자열 처리 프로그램에서 자주 사용
- 알파벳의 경우 26개의 포인터를 저장
  - 포인터의 크기가 8bytes인 64비트 아키텍처라면 노드하나를 표현하는데에 200여 바이트
    - 배열로 선언하고 포인터 대신 배열에서의 인덱스를 사용하면 절반으로 줄기는 함
    - 여전히 큼

### 파이썬 구현

```python
class Node:
    def __init__(self, key, data=None):
        self.key = key
        self.data = data
        self.children = {}


class Trie:
    def __init__(self):
        self.head = Node(None)

    def insert(self, word):
        cur_node = self.head

        for char in word:
            if char not in cur_node.children:
                cur_node.children[char] = Node(char)

            cur_node = cur_node.children[char]

        if cur_node.data is None:
            cur_node.data = word

    def find(self, word):
        cur_node = self.head

        for char in word:
            if char in cur_node.children:
                cur_node = cur_node.children[char]
            else:
                return False

        if cur_node.data is not None:
            return True

    def starts_with(self, prefix):
        result = []
        cur_node = self.head
        subtrie = None

        for char in prefix:
            if char in cur_node.children:
                cur_node = cur_node.children[char]
                subtrie = cur_node
            else:
                return None
        queue = list(subtrie.children.values())

        while queue:  # BFS
            cur = queue.pop(0)
            if cur.data is not None:
                result.append(cur.data)

            queue += list(cur.children.values())

        return result

    def delete(self, word):
        cur_node = self.head
        parent_node = None

        for char in word:
            if char not in cur_node.children:
                return False

            parent_node = cur_node
            cur_node = cur_node.children[char]

        parent_node.children.pop(word[-1])
        return True



t = Trie()
words = ["romane", "romanus", "romulus", "ruben", 'rubens', 'ruber', 'rubicon', 'ruler']
for word in words:
    t.insert(word)

print(t.starts_with("ro"))
print(t.find("romane"))
print(t.delete("romane"))
print(t.find("romane"))
```
