# 트리
- 노드를 활용하여 데이터를 계층적으로 구성하고 `순환을 허용하지 않는` 비선형 자료구조
- `이진트리` : 최대 2개의 자식 

# 트리 탐색
## 1) 너비 우선 탐색
- `레벨 순회(level-order traversal)`, 최단 경로
```java
void printLevelOrder() {
	Queue<Node> queue = new ArrayDeque<>();
	queue.add(new Node(1));
	while (!queue.isEmpty()) {
		Node now = queue.poll();
		System.out.println(now.element);
		if (now.left != null) {
			queue.add(now.left);
	 	}
		if (now.right != null) {
			queue.add(now.right);
		}
	}
}
```

## 2) 깊이 우선 탐색 
- 위상정렬
### 전위순회, 중위순회, 후위 순회
```java
void printDepthOrder(Node now) {
	if (now == null)
		return;
	// System.out.println(now.element); //전위
	printDepthOrder(now.left);
	// System.out.println(now.element); //중위
	printDepthOrder(now.right);
	// System.out.println(now.element); //후위
}
```

```java
// 전위 순회 DFS (루트 → 왼쪽 → 오른쪽)
stack.push(root);

while (!stack.isEmpty()) {
    TreeNode node = stack.pop();
    System.out.println(node.val);

    // 작은 수 먼저 넣는 경우
    if (node.left != null) stack.push(node.left);
    if (node.right != null) stack.push(node.right);
}
```

# 이진 트리 
## 이진 탐색 트리 (BST; Binary Search Tree)
- `정렬 규칙`을 따르는 이진트리
- 중복 요소를 허용하지 않음
    - 별도의 해시맵에 저장하거나 카운터를 통해 트리 구조에 직접 저장할 수 있음
```
왼쪽 서브트리의 모든 값 <=  부모 노드 값 <  오른쪽 서브트리의 모든 값
```
- 탐색시간
    - 평균 O(logN), 최악(N) 
- [BinarySearchTree 구현](https://github.com/Eundms/learn-datastructure/blob/master/src/main/java/com/eundms/tree/BinarySearchTree.java)

## 균형 및 불균형 이진 트리 
- `균형 이진 트리 (Balanced Binary Tree)` : 레드 블랙 트리, AVL 트리

### AVL 트리 
- 자가 균형 이진 탐색 트리 
- 하위 트리의 높이는 최대 1만큼 다를 수 있다
- 노드 n의 균형 계수 BN은 -1, 0, 1이며 높이 h의 차이로 정의된다
    BN = h(right_subtree(n)) - h(left_subtree(n)) 혹은
    BN = h(left_subtree(n)) - h(right_subtree(n))
- 삽입, 삭제, 최솟값 검색, 최댓값 검색 : O(logN)
- [AVL Tree 구현 : 어렵다]()
    - 삽입, 삭제, 검색 연산 동작 방식 확인하기

## 이진 힙(binary heap) 
> PriorityQueue 
- 힙 속성이 있는 `완전 이진 트리`
- `정렬되지 않음`
```
- 부모 노드 값 <= 자식 노드 값, 루트에 트리에서 가장 작은 값 위치: 최소 이진 힙
```
- 삽입 : O(logN) 
    - 새로운 요소는 힙트리의 끝에 추가된다
    - 이진 힙의 조건을 만족하지 않으면,
        1. 힙의 끝을 현재 노드로 설정하여 힙화를 시작한다
        2. 현재 노드에 부모가 있고 부모가 현재 노드보다 작다면 노드를 교환하고, 부모가 현재 노드보다 클 때까지 교환을 반복한다
- 삭제 : O(logN)
    1. 힙의 루트를 현재 노드로 설정하여 힙화를 시작한다.
    2. 현재 노드의 자식 중 가장 큰 노드를 찾는다.
    3. 현재 노드가 가장 큰 자식보다 작으면 두 노드를 교환하고 2단계부터 반복한다. 그렇지 않으면 알고리즘을 중단한다.
- 조회 : O(1)
- [MaxHeap 구현](https://github.com/Eundms/learn-datastructure/blob/master/src/main/java/com/eundms/tree/MaxHeap.java)

## 그외 이진 트리 종류
1. 완전 이진 트리 (complete binary tree)
- 높이 : 항상 O(logN)
2. 정 이진 트리 (full binary tree)
- 2개의 자식이 있거나 아예 없는 이진 트리
3. 포화 이진 트리 (perfect binary tree)
- 완전 이진 트리인 동시에 정 이진 트리 

# 이진 트리 외의 트리 

## disjoint-set (Union-find)
- [Union-find 구현](https://github.com/Eundms/learn-datastructure/blob/master/src/main/java/com/eundms/tree/DisjointSet.java)

## Binary Indexed Tree(이진 색인 트리) : 구간합, 누적합 구하기
- 배열의 각 원소에 부분합을 저장
- 1-based index 사용
- 갱신, 쿼리 : O(logN)
- [BinaryIndexedTree]()

## B-Tree
## B+Tree (DB 인덱스)
## 세그먼트 트리
