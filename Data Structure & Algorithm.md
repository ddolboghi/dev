---
sticker: emoji//1f4df
Created: 2023-10-15
---
**단계별로 풀어보기** 를 통해 특정 알고리즘 분류의 유형을 어느 정도 익힌 상황에서는 저는 아래의 방법으로 새롭게 풀 문제들을 찾았습니다.

## 문제를 찾는 방법

1. **solved.ac의 클래스 문제** 를 찾아 풀기. 알고리즘 분류 여러 개가 적절히 섞여 있는 문제집이면서, **단계별로 풀어보기** 처럼 학습자가 입문하기에도 좋은 문제들이 다수 포진해 있습니다.
2. 학습하고 싶은 알고리즘 분류를 찾아, **푼 문제 수의 내림차순**으로 설정하고 무작위로 문제 골라서 풀기. 단계별로 풀어보기 / 클래스 문제를 풀었음에도 그 알고리즘 분류의 문제들을 좀 더 풀면서 익숙해지고 싶다면 추천드립니다.
3. **랜덤 디펜스**. 랜덤 디펜스란 난이도의 범위를 어느 정도 정하고 정말 쌩무작위로 문제들을 고르고 푸는 과정을 의미합니다. 이미 핵심적인 알고리즘 학습에 익숙된 중수 이상의 유저분들은 주로 랜덤 디펜스를 활용하여 어떤 유형의 알고리즘의 문제가 나와도 커버할 수 있도록 연습하는 과정을 거치시는 경우가 많았습니다. 여러 알고리즘에 어느 정도 익숙되셨다면 추천드립니다.

## 문제를 푸는 방법

풀 문제를 고르셨고 문제를 풀 때에는 **점차 자신에게 힌트를 줘가는 방법으로** 푸는 방법이 유용한 학습법이라고 생각합니다. 아래에 제가 적은 시간은 예시일 뿐이며 학습자마다 적절한 시간은 다르므로 참고용으로만 보셔야 합니다. 입문자일 수록 힌트를 줘야 하는 시간이 빠르며 숙련자일 수록 길어야 합니다. 또한, 이미 익숙된 알고리즘 문제를 푸는 것이 아닌 **새롭게 알고리즘에 적응하기 위해 / 구현 연습을 하기 위해 문제를 푸는 경우라면 알고리즘 분류와 답지는 즉시 공개하면서 푸셔도 상관없다 생각합니다.**

1. 문제를 먼저 충분히 읽고 접근 방법을 생각해 봅니다.
2. 문제를 읽은 지 어느 정도의 시간 (+15분) 이 지났음에도 접근 방법이 떠오르지 않는다면, **알고리즘 분류** 를 공개합니다.
3. 알고리즘 분류를 생각하며 접근 방법을 계속해서 생각해 봅니다. (+15분) 풀이가 떠올라 구현을 하실 수 있다면 구현하세요. 여전히 접근 방법이 떠오르지 않는다면 블로그 등을 검색하여 **풀이만 확인하고, 소스코드는 확인하지 않습니다.**
4. 풀이를 바탕으로 구현을 시도해 봅니다. 구현에서도 여전히 막힌다면, 그제서야 **소스 코드도 확인하면서** 문제풀이를 진행해 주시면 됩니다.

지금 당장 하시면 좋은 행동들은 아래의 사항들이라 생각됩니다.
- solved.ac의 클래스를 2까지 취득하기.
- 핵심적인 알고리즘 분류를 고르고, 푼 사람의 오름차순으로 문제를 무작위로 골라 풀어보기.
---
# Stack
자바에서 Linked List 구조의 Stack클래스 제공함
```java
public class StackImpl {  
    public static void main(String[] args) {  
       Stack<Integer> stack = new Stack<>();  
         
       for(int i=1;i<=5;i++) {  
          stack.push(i);  
          System.out.println(stack.peek());  
       }  
         
       System.out.println(stack.capacity());  
       stack.add(6);  
       System.out.println(stack.peek());  
       System.out.println(stack.search(7)); //-1
    }
}
```
## Generic Array Stack
```java
public class ArrayStack<E> {  
    private int top;  
    private int size;  
    private E[] stack;  
      
    public boolean isStackEmpty() {  
       return this.top == -1;  
    }  
      
    public boolean isStackFull() {  
       return this.size-1 == this.top;  
    }  
      
    @SuppressWarnings("unchecked")  
    public ArrayStack(int size) {  
       this.size = size;  
       stack = (E[]) new Object[size];  
       top = -1;  
    }  
      
    public void push(E item) {  
       stack[++top] = item;  
       System.out.println(stack[top]+" push");  
    }  
      
    public E pop() throws Exception{  
       if(this.isStackEmpty() == true) {  
          throw new Exception("Stack is empty");  
       }  
       return stack[top--];  
    }  
      
    public E peek() throws Exception{  
       if(this.isStackEmpty() == true) {  
          throw new Exception("Stack is empty");  
       }  
       return stack[top];  
    }  
      
}
```

## Linked List Stack

```java
import java.util.EmptyStackException;  
  
public class LinkedListStack {  
    Node top;  
    int size;  
      
    public LinkedListStack() {  
       this.top = null;  
       this.size = 0;  
    }  
      
    public void push(int data) {  
       Node node = new Node(data);  
       node.linkNode(top);  
       top = node;  
       size++;  
    }  
      
    public void pop() {  
       if(top == null) {  
          throw new EmptyStackException();  
       }  
       top = top.getNextNode();  
       size--;  
    }  
      
    public int peek() {  
       if(top == null) {  
          throw new EmptyStackException();        
       }  
       return top.getItem();  
    }  
      
    public int getData(int data) {  
       int pos = 0;  
       Node tmp = this.top;     
       while(tmp != null) {           
          if(tmp.getItem() == data) {  
             return size-pos-1;   
          }  
          tmp = tmp.getNextNode();  
          pos++;  
       }  
       throw new RuntimeException("Data not found in stack");  
    }  
}  
  
class Node {  
    private int item;  
    private Node node;  
      
    public Node(int item) {  
       this.item = item;  
       this.node = null;  
    }  
      
    protected void linkNode(Node node) {  
       //가리킬 노드 지정  
       this.node = node;  
    }  
      
    protected int getItem() {  
       return this.item;  
    }  
      
    protected Node getNextNode() {  
       return this.node;  
    }  
}

```
# Queue
자바에서는 스택을 클래스로 구현하여 제공하지만 큐는 Queue인터페이스만 있고 ==별도의 클래스가 없다.== 그래서 Queue인터페이스를 구현한 클래스들을 사용해야 한다.
```java
import java.util.LinkedList;  
import java.util.Queue;  
  
public class QueueImpl {  
    public static void main(String[] args) {  
       Queue<Integer> queue = new LinkedList<Integer>();  
  
       for(int i=1;i<=5;i++)  
          queue.offer(i);  
  
       while(!queue.isEmpty()) {  
          System.out.println(queue.poll());  
       }
    }
}
```
## Generic Array Queue
```java
public class ArrayQueue<E> {  
    private int MAX = 10000;  
    private int front;  
    private int rear;  
    private E[] queue;  
  
    public ArrayQueue() {  
        front = rear = 0;  
        this.queue = (E[]) new Object[MAX];  
    }  
  
    public boolean isEmpty() {  
        return front == rear;  
    }  
  
    public boolean isFull() {  
        return rear == (MAX-1);  
    }  
  
    public int size() {  
        return front - rear;  
    }  
  
    public void push(E item) {  
        if(isFull())  
            System.out.println("Queue is full");  
        queue[rear++] = item;//rear가 있던 곳에 값을 넣고 rear를 증가시킴  
    }  
  
    public E pop() {  
        if(isEmpty())  
            System.out.println("Queue is empty");  
        return queue[front++];//front가 있던 곳의 값을 반환하고 front를 증가시킴  
    }  
  
    public E peek() {  
        //가장 앞에 있는 값 반환하는 메서드  
        if(isEmpty())  
            System.out.println("Queue is empty");  
        return queue[front];  
    }  
}
```
## Linked List Queue
```java
public class QueueNode {  
    int item;  
    QueueNode queueNode;  
  
    public QueueNode(int item) {  
        this.item = item;  
        queueNode = null;  
    }  
  
    public int getItem() {  
        return item;  
    }  
  
    public QueueNode getNextNode() {  
        return queueNode;  
    }  
  
    public void setNextNode(QueueNode queueNode) {  
        this.queueNode = queueNode;  
    }  
}
```

front, rear사용하는 방법법
```java
public class OriginQueueNodeManager {  
    QueueNode front, rear;  
  
    public OriginQueueNodeManager() {  
        front = rear = null;  
    }  
  
    public boolean isEmpty() {  
        return front == null && rear == null;  
    }  
  
    public void clear() {  
        front = rear = null;  
    }  
  
    public QueueNode peek() {  
        if(isEmpty()) {  
            System.out.println("Queue is empty");  
            return null;  
        } else {  
            return front;  
        }  
    }  
  
    public int size() {  
        QueueNode front2 = front;  
        QueueNode rear2 = rear;  
        int count = 0;  
        while(front2 != rear2 && front2 != null) {  
            count++;  
            front2 = front2.getNextNode();  
        }  
        return count;  
    }  
  
    public void push(int item) {  
        QueueNode node = new QueueNode(item);  
        if(isEmpty()) {  
            front = rear = node;  
        } else {  
            rear.setNextNode(node);  
            rear = node;  
        }  
    }  
  
    public QueueNode pop() {  
        if(isEmpty()) {  
            System.out.println("Queue is empty");  
            return null;  
        } else {  
            QueueNode popnode = front;  
            front = front.getNextNode();  
            return popnode;  
        }  
    }  
}
```

circular linked list queue --> pop구현 실패
```java
public class QueueNodeManager { 
    QueueNode tail;  
  
    public QueueNodeManager() {  
        tail = null;  
    }  
  
    public boolean isEmpty() {  
        return tail == null;  
    }  
  
    public void clear() {  
        tail = null;  
    }  
  
    public int peek() {  
        if(isEmpty()) {  
            System.out.println("Queue is empty");  
        }  
        return tail.getNextNode().getItem();  
    }  
  
    public int size() {  
        if(isEmpty())  
            return 0;  
  
        int count = 1;  
        QueueNode node = tail.getNextNode();  
        while(node != tail) {  
            node = node.getNextNode();  
            count++;  
        }  
        return count;  
    }  
  
    public void push(int item) {  
        QueueNode node = new QueueNode(item);  
        if(isEmpty()) {  
            node.setNextNode(node);  
            tail = node;  
        } else {  
            node.setNextNode(tail.getNextNode());  
            tail.getNextNode().setNextNode(node);  
            tail.setNextNode(node);//tail은 삽입한 노드를 가리킴  
            tail = node;  
        }  
//        System.out.println(tail.getItem());  
    }  
  
    public QueueNode pop() {  
        if(isEmpty()) {  
            return null;  
        } else {  
            QueueNode popNode = tail.getNextNode();  
            QueueNode nextFrontNode = tail.getNextNode().getNextNode();  
            tail.setNextNode(nextFrontNode);  
            return popNode;  
        }  
    }  
}
```

## Deque
[참고](https://st-lab.tistory.com/185)
- deque는 queue의 앞과 뒤 모두 삽입, 삭제 가능한 양방향 구조
- ~~Queue인터페이스를 상속받은 Deque인터페이스를 만들어 필요한 메서드 구현~~
	--> Queue인터페이스가 Collection인터페이스를 상속하고 Collection은 Iterable인터페이스를 상속해서 구현해야할 메서드 너무 많아 Queue상속 안받고 만듬 
- offer, poll계열은 삽입과 삭제 과정에서 저장공간이 넘치거나 삭제할 원소가 없을때 특정 값(null, false등)을 반환하지만 add, remove계열은 예외를 던짐

|front|rear|
|---|---|
|offerFirst(addFirst)|offerLast(addLast)|
|pollFirst(removeFirst)|pollLast(removeLast)|
|peekFirst(getFirst)|peekLast(getLast)|
- offer, poll계열은 Circular queue 형태로 구현하고 add, remove계열은 큐에 원소가 다 차면 Exception throw함
```java
import java.util.NoSuchElementException;  
  
public class ArrayDeque<E> {  
    private static final int DEFAULT_CAPACITY = 64;  
    private Object[] array;  
    private int size;  
    private int front;  
    private int rear;  
  
    public ArrayDeque() {  
        this.array = new Object[DEFAULT_CAPACITY];  
        this.size = 0;  
        this.front = 0;  
        this.rear = 0;  
    }  
  
    public ArrayDeque(int capacity) {  
        this.array = new Object[capacity];  
        this.size = 0;  
        this.front = 0;//첫번째 원소 앞의 빈 칸  
        this.rear = 0;  
    }  
  
    private void resize(int newCapacity) {  
        int arrayCapacity = array.length;  
        Object[] newArray = new Object[newCapacity];  
        /*  
         * i = new array index         * j = original array         * 기존 배열의 원소들을 새 배열에 복사  
         */        for(int i=1, j=front+1;i<=size;i++, j++) {  
            newArray[i] = array[j % arrayCapacity];  
        }  
  
        this.array = null;  
        this.array = newArray;  
        front = 0;  
        rear = size;  
    }  
  
//    public boolean offer(E item) {  
//        return offerLast(item);  
//    }  
  
    public boolean offerLast(E item) {  
        if((rear+1)%array.length == front) {//배열이 꽉찼다면  
            resize(array.length*2);//용량 늘리기  
        }  
  
        rear = (rear+1)%array.length;  
        array[rear] = item;  
        size++;  
        return true;  
    }  
  
    public boolean offerFirst(E item) {  
        if((front-1+array.length)%array.length == rear) {  
            resize(array.length*2);  
        }  
  
        array[front] = item;//빈 큐는 0번째 자리부터 채워짐  
        front = (front-1+array.length)%array.length;  
        size++;  
        return true;  
    }  
  
//    public boolean add(E item) {  
//        return addLast(item);  
//    }  
    public boolean addLast(E item) {  
        if((rear+1)%array.length == front)  
            throw new IllegalStateException();  
  
        rear = (rear+1)%array.length;  
        array[rear] = item;  
        size++;  
        return true;  
    }  
  
    public boolean addFirst(E item) {  
        if((front-1+array.length)%array.length == rear)  
            throw new IllegalStateException();  
  
        array[front] = item;  
        front = (front-1+array.length)%array.length;  
        size++;  
        return true;  
    }  
  
//    public E poll() {  
//        return pollFirst();  
//    }  
  
    public E pollFirst() {  
        if(size == 0) {  
            return null;  
        }  
  
        front = (front+1)%array.length;//front를 삭제할 원소 위치로 옮기기  
  
        @SuppressWarnings("unchecked")  
        E popItem = (E) array[front];  
        array[front] = null;  
        size--;  
  
        if(array.length>DEFAULT_CAPACITY && size<(array.length/4)) {  
            resize(Math.max(DEFAULT_CAPACITY, (array.length/2)));  
        }  
        return popItem;  
    }  
  
//    public E remove() {  
//        return removeFirst();  
//    }  
  
    public E removeFirst() {  
        E item = pollFirst();  
  
        if(item == null) {  
            throw new NoSuchElementException();  
        }  
  
        return item;  
    }  
  
    public E pollLast() {  
        if(size == 0)  
            return null;  
  
        @SuppressWarnings("unchecked")  
        E popItem = (E) array[rear];  
        array[rear] = null;  
        rear = (rear-1+array.length)%array.length;  
        size--;  
  
        if(array.length>DEFAULT_CAPACITY && size<(array.length/4)) {  
            resize(Math.max(DEFAULT_CAPACITY, (array.length/2)));  
        }  
        return popItem;  
    }  
  
    public E removeLast() {  
        E item = pollLast();  
        if(item == null)  
            throw new NoSuchElementException();  
        return item;  
    }  
  
//    public E peek() {  
//        return peekFirst();  
//    }  
    public E peekFirst() {  
        if(size == 0) {  
            return null;  
        }  
  
        @SuppressWarnings("unchecked")  
        E item = (E) array[(front+1)%array.length];  
        return item;  
    }  
  
    public E peekLast() {  
        if(size == 0) {  
            return null;  
        }  
  
        @SuppressWarnings("unchecked")  
        E item = (E) array[rear];  
        return item;  
    }  
  
    public E element() {  
        return getFirst();  
    }  
    public E getFirst() {  
        E item = peekFirst();  
        if(item == null) {  
            throw new NoSuchElementException();  
        }  
        return item;  
    }  
  
    public E getLast() {  
        E item = peekLast();  
        if(item == null) {  
            throw new NoSuchElementException();  
        }  
        return item;  
    }  
  
    public int size() {  
        return size;  
    }  
  
    public boolean isEmpty() {  
        return size == 0;  
    }  
  
    public void clear() {  
        for(int i=0;i<array.length;i++)  
            array[i] = null;  
  
        front = rear = 0;  
    }  
  
    public boolean contains(Object item) {  
        int start = (front+1)%array.length;  
  
        /*  
        * i : 원소 개수만큼 반복  
        * idx : 원소 위치, (idx+1)%array.length로 갱신  
        */        for(int i=0, idx=start;i<size;i++, idx=(idx+1)%array.length) {  
            if(array[idx].equals(item))  
                return true;  
        }  
        return false;  
    }  
}
```
# DFS
[참고](https://scshim.tistory.com/241)
## 재귀 구현
```java
public class DFSImpl {  
    public static int[][] graph = {  
            {},  
            {2,3,8},  
            {1,7},  
            {1,4,5},  
            {3,5},  
            {3,4},  
            {7},  
            {2,6,8},  
            {1,7}  
    };  
  
    public static boolean[] visited = new boolean[graph.length];  
  
    public static void main(String[] args) {  
        int start =1;  
        dfs(start);  
    }  
  
    public static void dfs(int v) {  
        visited[v] = true;  
  
        System.out.print(v+" ");  
  
        for(int i : graph[v]) {  
            if(!visited[i]) {  
                dfs(i);  
            }  
        }  
    }  
}
```

## Stack 구현
- `java.util.Deque<>`인터페이스 구현하는 `LinkedList<>()`사용
```java
import java.util.Deque;  
import java.util.LinkedList;  
  
public class DfsStackImpl {  
    public static void main(String[] args) {  
        int[][] graph = {  
                {},  
                {2,3,8},  
                {1,7},  
                {1,4,5},  
                {3,5},  
                {3,4},  
                {7},  
                {2,6,8},  
                {1,7}  
        };  
        boolean[] visited = new boolean[graph.length];  
        int start = 1;  
  
        dfs(graph, start, visited);  
    }  
  
    public static void dfs(int[][] graph, int start, boolean[] visited) {  
        visited[start] = true;  
        System.out.print(start+" ");  
        Deque<Integer> stack = new LinkedList<>();  
        stack.push(start);  
  
        while(!stack.isEmpty()) {  
            int top = stack.peek();  
  
            boolean hasNearNode = false; //방문 안한 인접노드가 있는지  
  
            for(int i : graph[top]) {  
                if(!visited[i]) {  
                    visited[i] = true;  
                    stack.push(i);  
                    hasNearNode = true;  
  
                    System.out.print(i+" ");  
                    break; //방금 스택에 넣은 방문 안한 인접 노드부터 탐색하기 위해 break                }  
            }  
            if(!hasNearNode) {  
                stack.pop();//인접노드를 모두 방문했다면 stack에서 꺼냄  
            }  
        }  
    }  
}
```

# BFS
## graph가 주어졌을때
```java
import java.util.LinkedList;  
import java.util.Queue;  
  
public class BfsImpl {  
    public static void main(String[] args) {  
        int[][] graph = {  
                {},  
                {2,3,8},  
                {1,7},  
                {1,4,5},  
                {3,5},  
                {3,4},  
                {7},  
                {2,6,8},  
                {1,7}  
        };  
        boolean[] visited = new boolean[graph.length];  
        int start = 1;  
        bfs(graph, start, visited);  
    }  
  
    public static void bfs(int[][] graph, int start, boolean[] visited) {  
        Queue<Integer> queue = new LinkedList<>();  
        queue.add(start);  
  
        visited[start] = true;  
  
        while(!queue.isEmpty()) {  
            int v = queue.poll();  
            System.out.print(v+" ");  
  
            for(int i:graph[v]) {  
                if(!visited[i]) {  
                    visited[i] = true;  
                    queue.add(i);  
                }  
            }  
        }  
    }  
}
```

## 좌표 사용해야할때
```java
import java.util.LinkedList;  
import java.util.Queue;  
  
public class CoordinateDfsImpl {  
    public static void main(String[] args) {  
        int[][] coordinate = {  
                {1,1,1,1,1},  
                {1,0,0,1,0},  
                {1,1,0,1,0},  
                {1,1,1,0,0},  
                {0,1,1,1,1}  
        };  
  
        int areaCount = 0;  
  
        for(int i=0;i<coordinate.length;i++) {  
            for(int j=0;j<coordinate[i].length;j++) {  
                if(coordinate[i][j] == 0) {  
                    int[] start = {i, j};  
                    bfs(start, coordinate);  
                    areaCount++;  
                }  
            }  
        }  
        System.out.println(areaCount);  
    }  
  
    public static void bfs(int[] start, int[][] coordinate) {  
        int rowLength = coordinate.length;  
        int colLength = coordinate[0].length;  
  
        int[] dx = {0,0,-1,1};  
        int[] dy = {1,-1,0,0};  
  
        Queue<int[]> queue = new LinkedList<>();  
        queue.offer(start);  
  
        while(!queue.isEmpty()) {  
            int[] v = queue.poll();  
            int x = v[1];  
            int y = v[0];  
  
            for(int i=0;i<4;i++) {  
                int xx = x + dx[i];  
                int yy = y + dy[i];  
  
                if(0<=xx && xx<colLength && 0<=yy && yy<rowLength && coordinate[yy][xx] == 0) {  
                    coordinate[yy][xx] = 1;  
                    queue.offer(new int[] {yy, xx});  
                }  
            }  
        }  
    }  
}
```

# Binary Search

## 재귀 구현
```java
public class BinarySearchImpl {  
    public static void main(String[] args) {  
        int[] array = {1,2,3,4,5,6,7,8,9,10};  
        int result = binarySearch(0, array.length-1, array, 3);  
        System.out.println(result);  
    }  
  
    public static int binarySearch(int start, int end, int[] array, int value) {  
        if(start > end) {  
            return -1;  
        }  
  
        int mid = (start+end)/2;//자바에서 /는 몫 반환  
        if(array[mid] == value) {  
            return mid;  
        } else if (array[mid] < value) {  
            start = mid+1;  
        } else {  
            end = mid-1;  
        }  
        return binarySearch(start, end, array, value);  
    }  
}
```

## 반복문 구현
- 사용법은 재귀와 같음
- 재귀 구현에서는 메서드를 호출할때 start, end 값을 넘겨주지만, 반복문 구현 방식은 반복문 내에서만 start, end값들이 변하기 때문에 메서드를 호출할때 넘겨줄 필요 없음
```java
public static int binarySearch2(int[] array, int value) {  
    int start = 0;  
    int end = array.length-1;  
  
    while(start <= end) {  
        int mid = (start+end)/2;  
  
        if(array[mid] == value) {  
            return mid;  
        } else if(array[mid] > value) {  
            end = mid-1;  
        } else {  
            start = mid+1;  
        }  
    }  
    return -1;  
}
```