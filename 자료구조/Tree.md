# 트리에 대해 설명하시오


# 답변

> 트리는 비선형 자료구조로 **사이클이 없는 연결 그래프**입니다.  
연결된 두 노드는 부모-자식 관계를 가지며, 예외적으로 가장 위의 노드는 부모를 가지지 않습니다.  
어떤 노드의 부모 노드는 **단 하나**입니다.
> 

# 내용

## 트리의 구조와 명칭

<img src=https://user-images.githubusercontent.com/31722496/201730709-a45b149b-5157-4210-8acc-1b81946d771d.png width=70%>

- **루트 노드(Root Node)**
    - 부모 노드가 없는 노드
- **리프 노드(Leaf Node)**
    - 자식 노드가 없는 노드
- **차수(Degree)**
    - 자식 노드의 개수
        - D의 차수는 2
        - B의 차수는 4
- **깊이(Depth) := 레벨(Level)**
    - 현재 노드에서 **루트 노드**까지의 거리(간선의 수)
- **높이(Height)**
    - 현재 노드에서 가장 멀리있는 **리프 노드**까지의 거리(간선의 수)
        - A의 높이는 3
        - E의 높이는 0
- **크기(Size)**
    - 자신을 포함한 모든 자식 노드의 개수
        - A의 크기는 7
        - D의 크기는 3
- **서브트리(Subtree)**
    - 어떤 노드의 부모 노드와 연결을 끊었을 때, 해당 노드를 루트 노드로 하는 트리
    - 따라서 트리의 자식도 트리인 서브트리 구성을 가진다.

## 트리의 성질

- 트리는 **계층 구조**를 갖기 때문에 계층을 형성하는 정보를 저장할 때 사용한다.
    - ex) 파일 시스템 구조, 사내 조직도 등
- 트리의 **접근과 검색 속도**는 연결 리스트보다 빠르고 배열보다 느리다.
- 트리의 노드 **삽입/삭제 속도**는 배열보다 빠르고 연결 리스트보다 느리다.
- 포인터를 사용하여 연결하기 때문에 **노드 추가에 제한이 없다.**
- 노드와 노드 사이의 **경로가 유일**하다.
- 트리의 지름은 두 리프 노드 사이의 거리가 가장 긴 것이다.

## 트리 순회

- 전위(preorder) 순회
    - 루트 → 왼쪽 서브트리 → 오른쪽 서브트리
- 중위(inorder) 순회
    - 왼쪽 서브트리 → 루트 → 오른쪽 서브트리
- 후위(postorder) 순회
    - 왼쪽 서브트리 → 오른쪽 서브트리 → 루트
- 레벨(level order) 순회
    - 레벨 순으로 순회 → 큐를 사용(BFS)

## 이진 트리(Binary Tree)

> 이진트리는 모든 노드가 최대 2개의 서브트리를 가지는 트리입니다.  
⇒ 모든 노드의 차수가 2 이하  
⇒ 모든 노드의 자식이 최대 2개
> 

### 이진 트리의 종류

<img src=https://user-images.githubusercontent.com/31722496/201730722-94472c32-9a7f-4626-9d01-35bfc25369b1.png width=100%>

- 정 이진 트리(Full Binary Tree)
    - 모든 노드의 자식 노드 개수는 0 또는 2
- 완전 이진 트리(Complete Binary Tree)
    - 마지막 레벨을 제외하고 모든 레벨이 채워지고 마지막 레벨은 왼쪽부터 채워짐
- 포화 이진 트리(Perfect Binary Tree)
    - 리프 노드를 제외한 모든 노드의 자식이 2개이며 모든 리프 노드는 동일한 레벨(깊이)을 갖는다.

### 이진 트리의 성질

- $N$ 개의 노드가 있으면 $N - 1$ 개의 간선이 존재한다.
- 레벨 $k$에서 노드의 최대 수는 $2^k$ 이다. (루트 레벨 0)
- 높이가 $h$ 일 때 최소 $h+1$ 개, 최대 $2^{h+1} - 1$ 개 노드를 가진다.
    
    
    | 높이 | 최소 | 최대 |
    | --- | --- | --- |
    | 0 (단일 노드) | 1 | 1 |
    | 1 | 2 | 3 |
    | 2 | 3 | 7 |
    
	<img src=https://user-images.githubusercontent.com/31722496/201730730-937479ac-805d-4d7d-9c99-351ee3ae71dc.png width=70%>

    
- $N$ 개의 노드를 가지는 트리의 최소 높이는 $log_2{(N + 1)}-1$, 최대  높이는 $N-1$ 이다.
    - 높이가 $h$ 일 때 노드의 수를 $N$이라고 하면 $h + 1 ≤ N ≤ 2^{h+1} - 1$
    - 노드 N개 일 때 최소 높이
        - $N ≤ 2^{h+1} - 1$
        - $N + 1 ≤ 2^{h+1}$
        - $log_2{(N+1)} ≤ h+1$
        - $log_2{(N+1)}-1 ≤ h$
    - 노드 N개 일 때 최대 높이
        - $h+1 ≤ N$
        - $h ≤ N-1$
- 이진 트리의 지름 = max(왼쪽 서브 트리의 지름, 오른쪽 서브 트리의 지름, 루트를 지나는 두 리프 노드의 가장 긴 경로)
    - 루트를 지나는 두 리프 노드의 경로는 하위 트리의 높이로 계산 가능
    - **루트에서 가장 먼 리프 노드 u를 먼저 찾고, u에서 가장 먼 리프 노드 v를 구하는 방식도 있다.**

## 이진 탐색 트리(Binary Search Tree)

> 정렬된 이진 트리를 뜻한다.  
왼쪽 서브트리는 현재 노드보다 작은 값을 가진 노드들로, 
오른쪽 서브트리는 큰 값을 가진 노드들로 이루어져 있다.  
탐색 시 시간 복잡도는 최선의 경우 $O(log_2 N)$이다.
> 

<img src=https://user-images.githubusercontent.com/31722496/201730733-c30fd436-7d0c-44ec-aef7-b07748033249.png>

- $O(log_2N)$  이 얼마나 빠른걸까?
    - $N$ = $10^9$ (1억) → $log_2N$ = 26.575425
    - 1억 개의 데이터 중에서 약 27번만에 원하는 데이터를 찾을 수 있다.
- 하지만 트리의 모양이 한쪽으로 치우친 형태(최악의 경우)라면 $O(N)$이 될 것이다.
- 그래서 **자가 균형 이진 탐색 트리(Self-Balancing Binary Search Tree)**는 높이를 낮게 유지하여  $O(log_2 N)$ 을 보장한다.
    - AVL Tree, Red Black Tree

# 추가 질문

## AVL, Red Black Tree?

다음에 주제로 선정해서 정리..

# 참고 자료

[https://www.geeksforgeeks.org/binary-tree-data-structure/](https://www.geeksforgeeks.org/binary-tree-data-structure/)

[https://www.geeksforgeeks.org/binary-search-tree-set-1-search-and-insertion/](https://www.geeksforgeeks.org/binary-search-tree-set-1-search-and-insertion/)

[http://www.yes24.com/Product/Goods/91084402](http://www.yes24.com/Product/Goods/91084402)

[http://www.yes24.com/Product/Goods/69750539](http://www.yes24.com/Product/Goods/69750539)
