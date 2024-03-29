# 이진트리순회(DFS)

## 전위 순회

![전위순회](https://user-images.githubusercontent.com/78953393/168762343-99716d86-3d01-4ab7-9fb9-f2ed556b46de.png)

- 부모노드 -> 왼쪽 자식 노드 -> 오른쪽 자식 노드 순으로 순회

## 중위 순회

![중위순회](https://user-images.githubusercontent.com/78953393/168762356-886a50d3-2788-4eac-89a7-56317e772b51.png)

- 왼쪽 자식 노드 -> 부모 노드 -> 오른쪽 자식 노드 순으로 순회

## 후위 순회

![후위순회](https://user-images.githubusercontent.com/78953393/168762362-c7eea6e4-0611-4961-9638-d39dab0a0a79.png)

- 왼쪽 자식 노드 -> 오른쪽 자식 노드 -> 부모 노드 순으로 순회

## 이진트리순회 구현

```java
class Node {
    int data;
    Node lt, rt;
    
    public Node(int value) {
        data = value;
        lt = rt = null;
    }
}

public class Main {
    Node root;
    
    public void dfs(Node root) {
        if (root == null) return;
        else {
            // System.out.println("root") 전위순회
            dfs(root.lt);
            // System.out.println("root") 중위순회
            dfs(root.rt);
            // System.out.println("root") 후위순회
        }
    }
    
    public static void main(String[] args) {
        Main tree = new Main();
        tree.root = new Node(1);
		tree.root.lt = new Node(2);
        tree.root.rt = new Node(3);
        tree.root.lt.lt = new Node(4);
        tree.root.rt.rt = new Node(5);
        tree.root.rt.lt = new Node(6);
        tree.root.rt.rt = new Node(7);
        tree.dfs(tree.root);
    }
}
```

- 재귀를 활용하여 이진 트리 순회를 구현
- 출력 하는 위치에 따라 전위, 중위, 후위 순회에 맞는 값이 출력된다.

## 부분집합(DFS)

```java
public class SubsetDFS {

    static class Main {
        static int n;
        static int[] ch;

        public void solution(int l) {
            if (l == n + 1) {
                String tmp = "";
                for (int i = 1; i <= n; i++) {
                    if (ch[i] == 1) {
                        tmp += (i + " ");
                    }
                }
                if (tmp.length() > 0) {
                    System.out.println(tmp);
                }
            } else {
                ch[l] = 1;
                solution(l + 1);
                ch[l] = 0;
                solution(l + 1);
            }
        }

        public static void main(String[] args) {
            Main main = new Main();
            n = 3;
            ch = new int[n + 1];
            main.solution(1);
        }
    }
}

```

- DFS를 활용하여, 부분집합 구하기 알고리즘 예제