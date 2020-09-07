---
title: '컬렉션 프레임웍'
date: 2020-08-29 20:43:00
category: 'java'
draft: false
---
##### 자바의 정석을 공부하면서 정리한 것

## 컬렉션 프레임웍

### 컬렉션(collection)
- 여러 객체(데이터)를 모아 놓은 것을 의미

### 프레임웍(framework)
- 표준화, 정형화된 체계적인 프로그래밍 방식
- 자유도 낮은 대신 생산성 높음
- 방식이 비슷하기에 서로 협업하기에 편함

### 컬렉션 프레임웍 (collection framework)
- 컬렉션(다수의 객체)을 다루기 위한 표준화된 프로그래밍 방식
- 컬렉션을 쉽고 편리하게 다룰 수 있는(저장, 삭제, 검색, 정렬) 다양한 클래스를 제공
- java.util 패키지에 포함. jdk1.2부터 제공

### 컬렉션 클래스(collection class)
- 다수의 데이터를 저장할 수 있는 클래스
- ex) Vector, ArrayList, HashSet

### 컬렉션 프레임웍의 핵심 인터페이스(크게 3가지)
1. List
    - ![](./images/collectionframework/1.png)
    - 순서 O, 중복 O
    - ex) 대기자 명단
    - 구현 클래스 : ArrayList, LinkedList, Stack, Vector 등
    - ArrayList와 LinkedList가 핵심(List인터페이스를 구현한 대표적인 컬렉션 클래스)
    - vector의 발전된 형태가 ArrayList

1. Set
    - ![](./images/collectionframework/2.png)
    - 순서 X, 중복 X
    - ex) 양의 정수집합, 소수의 집합
    - 구현 클래스 : HashSet, TreeSet 등
    - HashSet과 TreeSet이 핵심

1. Map
    - ![](./images/collectionframework/3.png)
    - 키(key)와 값(value)의 쌍으로 이루어진 데이터의 집합
    - 순서 X, 키는 중복 X, 값은 중복 O
    - ex) 우편번호, 지역번호, id와 passwd
    - 구현 클래스 : HashMap, TreeMap, Hashtable, properties 등
    - HashMap과 TreeMap이 핵심
    - LinkedHashMap은 순서가 O
    - Hashtable은 동기화O, HashMap은 동기화X

- 위의 List와 Set의 공통부분만 추출해서 `Collection인터페이스`를 정의(Map은 성격이 약간 달라서)


### 중요한 컬렉션 클래스 설명
#### ArrayList
- ArrayList는 기존의 Vector를 개선한 것으로 구현원리와 기능적으로 동일. ArrayList와 달리 Vector는 자체적으로 동기화처리가 되어 있다.
- 데이터의 저장공간으로 배열을 사용한다(배열기반)
- 객체 배열이므로 모든 종류의 객체 저장 가능하다.
- String의 "1"과 new Integer(1)은 다르다.
- ArrayList에는 객체만 저장가능
```
ArrayList list1 = new ArrayList(); // 크기 10인 ArrayList생성

list1.add(5); // 이것은 사실 list1.add(new Integer(5)); // autoboxing에 의해 기본형이 참조형으로 자동 변환

```

```
public class Vector extends AbstracList
    implements List, RandomAccess, Cloneable, java.io.Serializable
    {   
        ...
        protected Object[] elementData; // 객체를 담기 위한 배열
        ...
    }
```

- ArrayList의 메서드
    - 추가
        - boolean add(Object o)
        - void add(int index, Object element)
        - boolean addAll(Collection c)
        - boolean addAll(int index, Collection c)

    - 삭제
        - boolean remove(Object o)
        ```
        ArrayList list1 = new ArrayList();
        ...(중략)...
        list.remove(1); //인덱스가 1인 객체를 삭제
        ```
        - Object remove(int index)
        ```
        ArrayList list1 = new ArrayList();
        ...(중략)...
        list.remove(new Integer(1)); // 1을 삭제
        ```
        - boolean removeAll(Collection c)
        - void clear() // 모든 객체 삭제

    - 검색
        - int indexOf(Object o)
        - int lastIndexOf(Object o) // 오른쪽에서 찾기 시작
        - boolean contains(Object o) // 객체가 존재?
        - Object get(int index) //객체 읽기
        - Object set(int index, Object element) // 변경. 기존의 index위치에 있는 것은 지우고 그 자리에.

    - 기타
        - List subList(int fromIndex, int toindex) // 특정 부분만을 뗴어내서 새로운 List만드는 것
        - Object[] toArray()
        - Object[] toArray(Object[] a)
        - boolean isEmpty() //비어있는지
        - void trimToSize() // 빈공간 제거
        - int size() // 저장된 객체의 개수

- ArrayList에 저장된 객체의 삭제 과정
1. ArrayList에 저장된 첫 객체부터 삭제하는 경우(배열 복사 발생)

    data[0]=0
    data[1]=1
    data[2]=2
    data[3]=3
    data[4]=4
    data[5]=null
    data[6]=null
    ```
    for(int i = 0; i<list.size(); i++)
        list.remove(i); 
    ```
    위와 같은 방식으로 하면 다 지우지 못한다. 왜냐하면 하나씩 지울때마다 뒤의 배열들이 복사되어 앞으로 당겨지기 때문이다.

1. ArrayList에 저장된 마지막 객체부터 삭제하는 경우(배열 복사 발생안함)
    data[0]=0
    data[1]=1
    data[2]=2
    data[3]=3
    data[4]=4
    data[5]=null
    data[6]=null
    ```
    for(int i = list.size() - 1; i>=0; i--) 
        list.remove(i); 
    ```
    위와 같은 방식으로는 배열 복사가 일어나지 않고 마지막 객체부터 하나씩 다 지워나갈 수 있다.


#### LinkedList와 ArrayList 차이
##### 배열의 장단점
- 장점 
    배열은 구조가 간단, 데이터를 읽는데 시간이 짧다
- 단점
    1. 크기를 변경할 수 없다.(실행 중에 못 바꾼다. 컴파일 때는 바꿀 수 있다)
    크기를 변경해야 하는 경우 새로운 배열(더 큰 배열) 생성 후 데이터를 복사, 그리고 참조주소 변경도 해야된다. 그렇다고 애초부터 너무 큰 배열을 만드는 경우 메모리의 낭비 발생
    1. 비순차적인 데이터의 추가, 삭제에 시간이 많이 걸린다. (물론 끝에 데이터를 순차적으로 추가하는 것과 끝부터 순차적으로 삭제하는 것은 빠르다)

##### LinkedList
- LinkedList는 배열의 단점을 보완(크기 변경X, 추가삭제시 시간소모 큼)
- 배열과 달리 LinkedList는 불연속적으로 존재하는 데이터를 하나하나씩 연결(link). `기차`로 비유
- 데이터 하나씩을 `Node`라고 한다. 각 Node는 데이터와 다음노드의 주소값을 가지고 있다.
- 데이터 삭제
    - 단 한 번의 참조변경만으로 가능
    - 삭제할 노드와의 연결을 끊으면 된다. 이후 가비지 컬렉터가 삭제함
- 데이터 추가 
    - 한 번의 Node객체생성과 두 번의 참조변경만으로 가능

- 단점
    - 연결리스트, 데이터 접근성이 나쁨. 불연속적이므로(자신의 다음 Node만 알고 있으므로). `기차`로 비유하자면 1칸->3칸으로 이동하려면 반드시 2칸을 거쳐야 한다. 그리고 2칸에서 1칸으로(반대로) 접근하지 못한다. `더블리 링크드 리스트(doubly linked list), 이중 연결리스트`는 이 단점을 보완해서 각 Node의 앞, 뒤로 이동 가능(그러나 여전히 여러칸을 한번에 뛰어넘지는 못한다). `더블리 써큘러 링크드 리스트(double circular linked list), 이중 원형 연결리스트`는 한 번 더 이 단점을 보완한 것. 

##### ArrayList vs LinkedList 
1. 순차적으로 데이터를 추가/삭제
    - ArrayList가 빠름
1. 비순차적으로 데이터를 추가/삭제
    - LinkedList가 빠름
1. 접근시간(access time) - ArrayList가 빠름


### Stack & Queue
#### Stack 
- 클래스
- 밑에 막힌 box(상자). 위만 뚫려 있는 구조
- LIFO(Last in First Out)구조. 마지막에 저장된 것을 제일 먼저 꺼내게 된다.
- 저장(push), 추출(pop)
- 스택을 구현하려면 배열과 링크드리스트 중에서 배열이 더 효율적. 순차적으로 저장, 삭제가 되므로
- 메서드
    - boolean empty()
    - Ojbect peek() // 제일 위의 객체를 `확인`만 
    - Object pop() // 제일 위의 객체를 `꺼내기`
    - Object push(Object item) // 저장
    - int search(Object o) 
- 스택의 활용
    - 수식계산, 수식괄호검사, 워드프로세스의 undo/redo, 웹브라우저의 뒤로/앞으로


#### Queue
- 인터페이스(객체 생성 안됨)
    - Queue를 직접 구현 or Queue를 구현한 클래스 사용(LinkedList)
- 파이프 구조(위 아래가 다 뚫려 있음)
- FIFO(FIrst in First Out)구조. 제일 먼저 저장한 것을 제일 먼저 꺼내게 된다.
- 저자(offer), 추출(poll)
- 큐를 구현하려면 배열과 링크드리스트 중에서 링크드리스트가 더 효율적. 자리이동 없이 연결만 바꿔주면 되므로.    
- 메서드
    - boolean add()
    - Object remove() // 삭제. 예외가 발생(try ~ catch 로 처리)
    - Object element()
    - boolean offer(Object o)
    - Object poll() // 삭제. null을 반환하지 예외가 발생안된다 (if(obj=null)로 처리)
    - Object peek()
- 큐의 활용
    - 최근사용문서, 인쇄작업 대기목록, 버퍼

```
public class Test {
	public static void main(String[] args) {
		Stack st = new Stack();
		Queue q = new LinkedList(); //Queue 인터페이스  구현체인 LinkedList
		
		st.push("0");
		st.push("1");
		st.push("2");
		
		q.offer("0");
		q.offer("1");
		q.offer("2");
		
		System.out.println("= Stack = ");
		while(!st.empty()) {
			System.out.println(st.pop()); // 스택에서 요소 하나를 꺼냄
		}
		
		System.out.println("= Queue = ");
		while(!q.isEmpty()) {
			System.out.println(q.poll());
		}
	}
}
```
그럼 결과는
```
= Stack = 
2
1
0
= Queue = 
0
1
2
```


### Iterator, ListIterator, Enumeration
- 컬렉션에 저장된 데이터를 접근(읽는)하는데 사용되는 인터페이스
- Enumeration은 Iterator의 구버전. `Iterator`를 사용하기
- Iterator의 핵심 메서드 2개
    - boolean hasNext() // 확인, 읽어올 요소가 있으면 true(없다면 false)
    - Object next() // 읽기, 다음요소를 읽어온다.
- Enumeration의 핵심 메서드 2개
    - boolean hasMoreElements() // Iteartor의 hasNext()와 동일
    - Object nextElement() // Iterator의 next()와 동일
- ListIterator는 Iterator의 접근성을 향상시킨 것(단방향 -> 양방향)
- 컬렉션에 저장된 요소들을 읽어오는 방법을 표준화한 것이다. 그래서 구조가 어떻든 간에(List, Set, Map) hasNext()와 next()로 접근하여 사용하면 된다.
- 사용법
    - 컬렉션에 iterator()를 호출해서 Iterator를 구현한 객체를 얻어서 사용
    ```
    List list = new ArrayList(); // 다른 컬렉션으로 변경할 때는 이 이 부분만 고치면 된다.

    Iterator it = list.iterator();

    while(it.hasNext()){ //boolean hasNext() 읽어올 요소가 있는지 확인
        System.out.println(it.next()); // Object next() 다음 요소를 읽어옴
    }
    ```
    - iterator()메서드는 Collection인터페이스에 정의되어 있는 것이라서 Collection인터페이스의 자손인 List, Set이 모두 가지고 있는 메서드이다. (Map에는 iterator()가 없다)
    ```
    public interface Collection {
        ...
        public Iterator iterator();
        ...

    }

    ```

    ```
    public class test {
	public static void main(String[] args) {
		List list = new ArrayList();
		list.add(1);
		list.add(2);
		list.add(3);
		list.add(4);
		list.add(5);
		
		Iterator it = list.iterator();
		while(it.hasNext()) {
			Object obj = it.next(); // 읽음
			System.out.println(obj);
		}
	}
}
    ```
    결과는
    ```
    1
    2
    3
    4
    5
    ```
    - Iterator를 다 쓰고 나면 (pointer가 다 이동했으면) 다시 얻어서 생성해야 한다.
    ```
    public class Ex11_5 {
	public static void main(String[] args) {
		List list = new ArrayList();
		list.add(1);
		list.add(2);
		list.add(3);
		list.add(4);
		list.add(5);
		
		Iterator it = list.iterator(); //1회용임
		while(it.hasNext()) {
			Object obj = it.next(); // 읽음
			System.out.println(obj);
		}

        Iterator it = list.iterator();
		while(it.hasNext()) { // false가 된다.
			Object obj = it.next(); 
			System.out.println(obj);
		}
	}
}
    ```
    결과는
    ```
    1
    2
    3
    4
    5
    ```

- Map에는 iterator()가 없다. 그래서 keySet(), entrySet(), values()를 호출해야 한다. 각각의 반환타입은 Set, Set, Collection이다. Map을 통해서 바로 iterator()를 가져오지 않고 이들을(KeySet(), entrySet(), values()) 통해서 Set이나 Collection을 얻은 다음에 이를 통해 iterator() 호출
```
Map map = new HashMap();
    ...
Iterator it = map.entrySet().iterator();
```
위의 두번째 줄은 사실 아래와 같다.
```
Set eSet = map.entrySet();
Iterator it = eSet.iterator();
```

### Arrays 클래스
- 배열을 다루기 편리한 메서드(static) 제공. 마치 Math, Objects, Collections 클래스와 비슷(모두 static메서드 제공). util 메서드라고도 한다.

1. 배열의 출력 - toString()
1. 배열의 복사 - copyOf(), copyOfRange()
    - 새로운 배열을 만들어서 반환
1. 배열 채우기 - fill(), setAll()










참고 : 자바의 정석(기초편)
