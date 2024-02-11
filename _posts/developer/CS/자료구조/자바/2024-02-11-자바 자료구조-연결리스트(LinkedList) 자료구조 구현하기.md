---
layout: post
date: 2024-02-11 23:46 +0900
title: 자바 자료구조-연결리스트(LinkedList) 자료구조 구현하기
summary: 연결리스트(LinkedList) 자료구조를 자바로 구현합니다.
image:
tags: 개발 자료구조 자바 java.util
categories: 개발 자료구조 자바
sitemap: true
---



## 🧐 LinkedList 자료구조 구현

이제 대망의 LinkedList 자료구조를 구현할 차례네요.

LinkedList는 지금까지 완성된 인터페이스를 이용해서 구현해볼겁니다.  
Deque 인터페이스를 구현할겁니다!

나머지 자료구조와 관련없는 인터페이스는 java.util에서 상속받아 구현해보겠습니다.

추가로 LinkedList는 기본적인 내부 로직을 상속하기 위해 java.util의 AbstractSequentialList를 상속해요


## 👻 LinkedList

```java
package 연결리스트;

import java.io.Serializable;
import java.util.AbstractSequentialList;
import java.util.Collection;
import java.util.ConcurrentModificationException;
import 큐.Deque;
import java.util.Iterator;
import java.util.ListIterator;
import java.util.NoSuchElementException;
import java.util.Objects;
import java.util.function.Consumer;


/**
 * 일단 제네릭 클래스로 선언되어 있습니다.
 * 또한 ArrayList와 동일하기 List 인터페이스를 가지고 있어요. 별도로 Deque가 있고
 * Cloneable과 Serializable(ArrayList에선 생략했던) 인터페이스도 둘 다 구현해보고자 합니다.
 * 그런데 List<E>가 이미 AbstractList 내부에서 구현되고 있는데 한 번더 상속하고 있는 문제가 있어요.
 * 이런 중복 상속을 자바의 전처리가 자동으로 하나의 코드만을 이용하게 전처리합니다.
 *  */ 
public class LinkedList<E> extends AbstractSequentialList<E>
    implements Deque<E>, Cloneable, Serializable{

    /**
     * size는 직렬화된 오브젝트가 인스턴스화 활 때 다시 정해지므로
     * transient 키워드가 사용된다.
     */
    transient int size = 0;

    /**
     * 첫번째 노드의 주소 포인터.
     * java.util에선 Node가 정적 이너 클래스로 선언되어 있어요.
     */
    transient Node<E> first;

    /**
     * 마지막 노드에 대한 주소 포인터.
     */
    transient Node<E> last;

    /**
     * 노드에 대한 선언이다.
     * 노드들은 각각 자신의 데이터와
     * 이전과 다음에 대한 주소 포인터를 가지고 있다.
     */
    private static class Node<E> {
        E data;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.data = element;
            this.next = next;
            this.prev = prev;
        }
    } 

    /**
     * 빈 리스트에 대한 생성자이다.
     * 이 선언이 없으면 자동으로 빈리스트에 대한 선언을 추가해버린다.
     */
    public LinkedList(){

    }

    /**
     * 특정 자료형의 요소들을 가진 콜렉션으로 LinkedList를 초기화 함
     * @param c
     * @throws NullPointerException addAll에서 null포인트 오류를 체크함
     */
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }

    @Override
    public int size() {
        return size;
    }

    /**
     * 인덱스의 값이 범위를 벗어나는지 체크함
     * @param index
     */
    private void checkElementIndex(int index) {
        if (!(index >= 0 && index < size))
            throw new IndexOutOfBoundsException("인덱스: "+index+", 사이즈: "+size);
    }

    /**
     * 특정 인덱스 값의 노드를 반환한다.
     * 
     * @param  index 반환할 노드의 인덱스
     * @return {@link Node} 노드 반환
     */
    Node<E> node(int index) {
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }

    // 포지션 연산
    /**
     * 이 리스트의 특정 인덱스의 노드 값을 반환함.
     *
     * @param index 반환할 값의 인덱스
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E get(int index) {
        checkElementIndex(index);
        return node(index).data;
    }

    /**
     * 이 리스트의 특정 인덱스의 노드 값을 지정함.
     *
     * @param index 지정할 값의 인덱스
     * @param element element to be stored at the specified position
     * @return the element previously at the specified position
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E set(int index, E element) {
        checkElementIndex(index);
        Node<E> x = node(index);
        E oldVal = x.data;
        x.data = element;
        return oldVal;
    }

    /**
     * LinkedList의 super클래스인 AbstractSequentialList의 clone클래스를 가져옵니다. 여기선 복잡한 과정이 있어요.
     * clone은 기본적으로 Object의 native clone 메서드를 활용하며 이는 C++로 구현된 부분이다.
     * jdk17 아카이브에서 공개된 것은 다음뿐이며
     * https://github.com/openjdk/jdk17/blob/master/src/java.base/share/native/libjava/Object.c
     * 
     * https://github.com/cscott/Harpoon/blob/bcec08dbbaed226fe653203e18d6a2c3a8b105a9/Runtime/src/sunjdk/java.lang/java_lang_Object.h#L31
     * https://github.com/cscott/Harpoon/blob/bcec08dbbaed226fe653203e18d6a2c3a8b105a9/Runtime/src/sunjdk/java.lang/java_lang_Object.c#L39C27-L39C54
     * https://github.com/cscott/Harpoon/blob/bcec08dbbaed226fe653203e18d6a2c3a8b105a9/Runtime/src/java.lang/object.h#L12
     * 자바 객체의 복제는 fni_object_cloneHelper라는 함수에 의해서 핵심 기능이 수행된다.
     * 1. FNI_Alloc을 통해 jobject clone 변수에 변수공간을 할당한다.
     * FNI_Alloc에서부터 어렵게 느껴지는데 JNIEnv *env, struct FNI_classinfo *info, struct claz *claz, void *(*allocfunc)(size_t length), size_t length
     * 를 매개변수로 받고 info는 null이어도 되지만, claz는 assert로 체크되며 null이면 안된다. claz는 클래스 정보를 포함하는 구조체다.
     * void *(*allocfunc)(size_t length) 커스텀 allocfunc를 쓸 때 사용되는 매개변수이고 null을 넣으면 기본적으로 FNI_RawAlloc가 사용된다.
     * FNI_RawAlloc는 FNI_RawAlloc의 선언부 위에 구현되어 있고 여러 조건을 다 체크해서 메모리 할당이 이루어진다.
     * 기본적으로 java는 GC를 활용하기 때문에 할당하는 메모리를 기억해야하기 때문이다.
     * 
     * WITH_CLUSTERED_HEAPS 플래그가 있으면 NGBL_malloc함수를 호출해서 전역 힙에 저장을 한다.
        NGBL_malloc 함수에는 UPDATE_NIFTY_STATS매크로가 있는데 이는 gbl이라는 텍스트를 매크로와 size를 전달해 통계 변수들을 지정한다.
        NGBL_malloc은 여러 스레드에서 호출될 수 있고 따라서 UPDATE_NIFTY_STATS매크로의 gbl이란 통계변수는 공유변수이므로 Lock 매커니즘을 활용한다.
        
        그 다음 WITH_PRECISE_GC 플래그가 정의되어 있다면 precise_malloc함수를 호출하는데 이는 자바를 위한 정확한 할당용 함수이다.
        이는 여러 WITH_COPYING_GC, WITH_MARKSWEEP_GC, WITH_GENERATIONAL_GC와 같은 매크로를 검사해 internal_malloc가 가르키는 함수를 치환시킨다.
        각 GC의 구현부는 나중에 따로 더 정확히 공부해봐야할 듯 하다. 웹에서 GC에 대한 설명을 추상적으로 보여주긴 하지만 한 번도 코드로 정확히 본 적은 없다.
        
        WITH_PRECISE_GC 플래그가 정의되어 있지 않다면 BDW_CONSERVATIVE_GC 플래그를 체크하고 BDW Garbage Collector 오픈소스를 활용한다.
        GC_malloc이라고 하는 함수를 호출하며 주어진 포인터 영역에서 유효한 포인터로 추정하는 방식을 활용한다고 한다.
        
        그것도 정의되어 있지 않다면 시스템 기본인 C++의 malloc(size)를 사용한다.
     * 
     * WITH_CLUSTERED_HEAPS가 없고 WITH_REALTIME_JAVA 플래그가 있다면
        RTJ_malloc(length) 란 함수를 호출한다.
        이는 RealTime(실시간)에서 활용되기 위해 고안된 자바 전용 malloc 함수이며 스레드의 현재 메모리 블록을 찾아서 할당한다.
        자바 파일이 컴파일될 때 --with-realtime-java라는 옵션이 지정되어 있어야 한다.
        하지만 이는 RTSJ가 적용되어 있는 JVM에서만 구동하며 이는 매우 옛날버전에서만 동작한다. 따라서 걍 못 쓴다고 보면 된다.
     * 
     * WITH_REALTIME_JAVA도 없고 WITH_PRECISE_GC 플래그가 있다면
        precise_malloc(length) 함수를 호출한다. WITH_CLUSTERED_HEAPS과 다른 점이 통계변수를 저장하는 것 말곤 거의 모두 같다.

     * WITH_PRECISE_GC도 없고 BDW_CONSERVATIVE_GC 플래그가 있다면
     * WITH_GC_STATS플래그가 있다면 GC_malloc_stats(length)를 호출하고 없다면 GC_malloc(length)를 호출한다
     * 
     * 위 모든 과정이 거짓이라면 그냥 시스템 기본 할당인 malloc(length)를 활용한다.
     * 
     * 대부분의 경우엔 WITH_PRECISE_GC 플래그가 활성화되고 저것이 실행된다고 보면 된다.
     * 그렇지 않으면 할당된 메모리가 해제되지 않는 메모리 누수 문제가 발생할 수 있다.
     * 
     * 살펴본 바로는 C++ 기본 malloc으로 할당되었고 WITH_GENERATIONAL_GC플래그가 없다면 논리상으론 메모리 누수가 발생할 수 있는 것으로 보인다.
     * 기본적으로 WITH_GENERATIONAL_GC플래그가 있다면 
     * fni_object_cloneHelper함수에서 add_to_curr_obj_list(FNI_UNWRAP_MASKED(clone));를 사용해 할당된 포인터를 추적하지만
     * WITH_GENERATIONAL_GC플래그가 없다면 하지 않는다. 해당 플래그들을 전부 사용하지 않는 JVM이 있다면? 메모리 누수가 발생할 것이다.     * 
     * @return Object에서 상속된 clone메서드로 복사된 {@code LinkedList<E>}의 인스턴스를 반환한다.
     * @throw CloneNotSupportedException 
     */
    @SuppressWarnings("unchecked")
    private LinkedList<E> superClone() {
        try {
            return (LinkedList<E>) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new InternalError(e);
        }
    }

    /**
     * 이 링크드리스트에 대한 얕은 복사를 반환한다.
     * 각각의 요소들 자체가 복사되는 것은 아니다.
     *
     * @return 얕은 복사가 된 {@code LinkedList} instance 반환
     */
    @Override
    public Object clone() {
        LinkedList<E> clone = superClone();

        // clone 변수에 초기 상태를 추가한다.
        clone.first = clone.last = null;
        clone.size = 0;
        clone.modCount = 0;

        // clone 변수에 add메서드로 전체 요소를 끝까지 추가한다.
        for (Node<E> x = first; x != null; x = x.next)
            clone.add(x.data);

        return clone;
    }

    /**
     * 첫번째 노드에 입력된 값을 추가한다.
     * @param e 추가될 노드의 값
     */
    @Override
    public void addFirst(E e) {
        final Node<E> f = first; // 첫 번째의 포인터를 임시 저장함.
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode; // 첫 번째를 미리 치환한다.
        
        if (f == null) // 첫번째 노드일 경우
            last = newNode; // 마지막 노드도 추가할 노드로 지정한다.
        else // 첫번째가 아닐경우
            f.prev = newNode; // 임시 저장된 첫 번째의 앞 노드를 추가할 노드로 지정
        size++; 
        modCount++;
    }

    /**
     * 마지막 노드에 입력된 값을 추가한다.
     * @param e 추가될 노드의 값
     */
    @Override
    public void addLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }

    /**
     * 마지막 노드에 입력된 값을 추가한다.
     * {@link LinkedList.addLast}와 같은 로직을 가진다.'
     * 아래의 {@link LinkedList.linkBefore}와 일관된 명명규칙을 위해 사용하는 것으로 보임
     * 
     * @param e 추가될 노드의 값
     */
    void linkLast(E e) {
        addLast(e);
    }

    /**
     * null이 아닌 노드의 전에 요소를 추가한다.
     * 
     * @param e 추가될 노드의 값
     */
    void linkBefore(E e, Node<E> node) {
        final Node<E> nodePrev = node.prev;
        final Node<E> newNode = new Node<>(nodePrev, e, node);
        node.prev = newNode;
        if (nodePrev == null)
            first = newNode; //노드의 앞에 null이 있다면 추가되는게 첫 번째 노드임
        else
            // 기존 노드의 앞에 있던 노드의 다음을 추가할 노드의 주소를 가르키게 변경
            nodePrev.next = newNode;
        size++;
        modCount++;
    }

    /**
     * 입력된 노드의 값을 제거하고 해당 노드의 앞 뒤 노드들을 서로 이어줌
     * 
     * @param x 제거할 노드
     * @return {@code E} 제거된 노드의 값
     */
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.data;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.data = null;
        size--;
        modCount++;
        return element;
    }

    /**
     * 입력된 노드의 값을 제거하고 입력된 노드의 다음 노드로 첫 번째 노드를 지정한다.
     * 
     * @param f 제거할 노드
     * @return {@code E} 매개변수로 입력된 노드의 값
     */
    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.data;
        final Node<E> next = f.next;
        f.data = null;
        f.next = null;
        first = next;
        if (next == null)
            last = null;
        else
            next.prev = null;
        size--;
        modCount++;
        return element;
    }

    /**
     * 입력된 노드의 값을 제거하고  입력된 노드의 이전 노드를 마지막 노드로 지정한다.
     * 
     * @param f 제거할 노드
     * @return {@code E} 매개변수로 입력된 노드의 값
     */
    private E unlinkLast(Node<E> l) {
        // assert l == last && l != null;
        final E element = l.data;
        final Node<E> prev = l.prev;
        l.data = null;
        l.prev = null;
        last = prev;
        if (prev == null)
            first = null;
        else
            prev.next = null;
        size--;
        modCount++;
        return element;
    }

    /**
     * 리스트의 마지막에 입력된 값의 노드를 추가한다.
     *
     * @param e 추가될 노드의 값
     * @return {@code true} 추가가 정상적으로 될 경우 true를 반환
     */
    public boolean add(E e) {
        addLast(e);
        return true;
    }


    /**
     * 노드들 중 첫 번째 요소의 값을 반환한다.
     * 왜 똑같은 기능을 하는 메서드를 두 명명규칙을 이용해서 명명한지는 모르겠다.
     * 일단 이 메서드는 오래되었고 get과 set이라는 명명규칙이 정해지고 나서부터 점점 바뀌고 있다고 한다.
     * @return 첫 번째 노드의 요소의 값
     * @throws NoSuchElementException 리스트가 비어있는 경우 에러
     */
    @Override
    public E element() {
        return getFirst();
    }
    
    /**
     * 노드들 중 첫 번째 요소의 값을 반환한다.
     * @return 첫 번째 노드의 요소의 값
     * @throws NoSuchElementException 리스트가 비어있는 경우 에러발생
     */
    @Override
    public E getFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return f.data;
    }

    /**
     * 노드들 중 마지막 요소의 값을 반환한다.
     * @return 마지막 노드의 요소의 값
     * @throws NoSuchElementException 리스트가 비어있는 경우 에러발생
     */
    @Override
    public E getLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return l.data;
    }

    /**
     * 리스트의 마지막에 입력된 값의 노드를 추가한다.
     * {@link LinkedList#add}와 동일한 기능을 수행한다
     *
     * @param e 추가될 노드의 값
     * @return {@code true} 추가가 정상적으로 될 경우 true를 반환
     */
    @Override
    public boolean offer(E e) {
        return add(e);
    }

    // Deque 연산
    /**
     * 노드들의 맨 앞에 새로운 입력된 값의 노드를 추가한다.
     * {@code true}를 반환하는 것 말고 {@link LinkedList#addFirst}와 같은 동작을 수행한다.
     * @param e 추가될 노드의 값
     * @return {@code true} ({@link Deque#offerFirst}의 정의를 따라 true를 반환함)
     */
    @Override
    public boolean offerFirst(E e) {
        addFirst(e);
        return true;
    }

    /**
     * 노드들의 맨 앞에 새로운 입력된 값의 노드를 추가한다.
     * {@code true}를 반환하는 것 말고 {@link LinkedList#addLast}와 같은 동작을 수행한다.
     * @param e 추가될 노드의 값
     * @return {@code true} ({@link Deque#offerLast}의 정의를 따라 true를 반환함)
     */
    @Override
    public boolean offerLast(E e) {
        addLast(e);
        return true;
    }

    /**
     * 맨 앞 노드의 값을 반환한다.
     * 
     * @return {@code E}인 값 혹은 비어있을 경우 {@code null}을 반환 ({@link Deque#peek}의 정의를 따름 )
     */
    @Override
    public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.data;
    }

    /**
     * 맨 앞 노드의 값을 반환한다.
     * {@link LinkedList#peek}와 같은 동작을 수행한다.
     *
     * @return {@code E}인 값 혹은 비어있을 경우 {@code null}을 반환 ({@link Deque#peekFirst}의 정의를 따름 )
     */
    @Override
    public E peekFirst() {
        final Node<E> f = first;
        return (f == null) ? null : f.data;
    }

    /**
     * 맨 마지막 노드의 값을 반환한다.
     *
     * @return {@code E}인 값 혹은 비어있을 경우 {@code null}을 반환 ({@link Deque#peekLast}의 정의를 따름 )
     */
    @Override
    public E peekLast() {
        final Node<E> l = last;
        return (l == null) ? null : l.data;
    }

    @Override
    public E poll() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }

    /**
     * 맨 첫 번째 노드의 값을 반환한다.
     *
     * @return {@code E}인 값 혹은 비어있을 경우 {@code null}을 반환 ({@link Deque#pollFirst}의 정의를 따름 )
     */
    @Override
    public E pollFirst() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }

    /**
     * 맨 마지막 노드의 값을 반환한다.
     *
     * @return {@code E}인 값 혹은 비어있을 경우 {@code null}을 반환 ({@link Deque#pollLast}의 정의를 따름 )
     */
    @Override
    public E pollLast() {
        final Node<E> l = last;
        return (l == null) ? null : unlinkLast(l);
    }

    /**
     * 첫 번째 노드의 값을 제거하고 반환한다.
     * 이 메소드는 {@link LinkedList#removeFirst} 와 동일하다.
     *
     * @return 현재 리스트의 첫 번째 노드의 값
     * @throws NoSuchElementException 만약 리스트가 비어있다면 에러 발생
     */
    @Override
    public E pop() {
        return removeFirst();
    }

    /**
     * 첫번째 노드에 입력된 값을 추가한다.
     * 이 메소드는 {@link LinkedList#addFirst} 와 동일하다.
     * 
     * @param e 추가될 노드의 값
     */
    @Override
    public void push(E e) {
        addFirst(e);
    }

    /**
     * 노드의 특정 위치부터 지정된 컬렉션의 모든 원소들을 삽입함
     * 기존 원소들을 오른쪽으로 시프트 시킴
     * 새 요소는 지정된 컬렉션의 Iterator가 반환한 순서대로 목록에 추가됨
     *
     * @param index 지정된 컬렉션의 첫 번째 요소를 삽입할 인덱스
     * @param c 이 목록에 추가할 요소가 포함된 컬렉션
     * @return {@code true} 호출의 결과로 이 목록이 변경된 경우 반환
     * @throws IndexOutOfBoundsException {@inheritDoc}
     * @throws NullPointerException 지정된 컬렉션이 null인 경우
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        checkElementIndex(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0)
            return false;

        Node<E> pred, succ;
        if (index == size) {
            succ = null;
            pred = last;
        } else {
            succ = node(index);
            pred = succ.prev;
        }

        // 컬렉션의 Iterator를 불러냄
        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;
            Node<E> newNode = new Node<>(pred, e, null);
            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }

        if (succ == null) {
            last = pred;
        } else {
            pred.next = succ;
            succ.prev = pred;
        }

        size += numNew;
        modCount++;
        return true;
    }

    /**
     * 노드의 마지막에 지정된 컬렉션의 모든 원소들을 삽입함
     * 새 요소는 지정된 컬렉션의 Iterator가 반환한 순서대로 목록에 추가됨
     *
     * @param index 지정된 컬렉션의 첫 번째 요소를 삽입할 인덱스
     * @param c 이 목록에 추가할 요소가 포함된 컬렉션
     * @return {@code true} 호출의 결과로 이 목록이 변경된 경우 반환
     * @throws NullPointerException 지정된 컬렉션이 null인 경우
     */
    public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }

    /**
     * 첫 번째 노드의 값을 제거하고 반환한다.
     * 이 메소드는 {@link LinkedList#removeFirst} 와 동일하다.
     *
     * @return 현재 리스트의 첫 번째 노드의 값
     * @throws NoSuchElementException 만약 리스트가 비어있다면 에러 발생
     */
    @Override
    public E remove() {
        return removeFirst();
    }

    /**
     * 첫 번째 노드의 값을 제거하고 반환한다.
     *
     * @return 현재 리스트의 첫 번째 노드의 값
     * @throws NoSuchElementException 만약 리스트가 비어있다면 에러 발생
     */
    @Override
    public E removeFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }

    /**
     * 입력된 오브젝트의 값과 동일한 값을 가진 노드를 탐색하여 처음 일치하는 노드를 제거
     * 
     * @param o 제거할 노드의 값
     * @return 삭제를 성공할 경우 {@code true}, 실패할 경우 {@code false} 반환
     */
    @Override
    public boolean removeFirstOccurrence(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.data == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.data)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }

    /**
     * 마지막 노드의 값을 제거하고 반환한다.
     *
     * @return 현재 리스트의 마지막 노드의 값
     * @throws NoSuchElementException 만약 리스트가 비어있다면 에러 발생
     */
    @Override
    public E removeLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return unlinkLast(l);
    }

    /**
     * 입력된 오브젝트의 값과 동일한 값을 가진 노드를 뒤에서부터 탐색하여 
     * 처음 일치하는 노드를 제거
     * 
     * @param o 제거할 노드의 값
     * @return 삭제를 성공할 경우 {@code true}, 실패할 경우 {@code false} 반환
     */    
    @Override
    public boolean removeLastOccurrence(Object o) {
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (x.data == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (o.equals(x.data)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }

    @Override
    public ListIterator<E> listIterator(int index) {
        checkElementIndex(index);
        return new ListItr(index);
    }

    /**
     * LinkedList는 Itr 클래스를 가지고 있지 않고 모든 것을 ListItr에서 구현한다.
     * modCount는 이터레이터를 돌 때 내부 노드의 변경이 있는지를 체크합니다!
     */
    private class ListItr implements ListIterator<E> {
        private Node<E> lastReturned;
        private Node<E> next;
        private int nextIndex;
        private int expectedModCount = modCount;

        /**
         * ListItr의 생성자, 항상 index가 주어져야만 함
         * @param index
         */
        ListItr(int index) {
            next = (index == size) ? null : node(index);
            nextIndex = index;
        }

        /**
         * size보다 nextIndex가 작으면 다음을 가지고 있음
         * @return 작다면 {@code true} 반환, 크다면 {@code false} 반환
         */
        public boolean hasNext() {
            return nextIndex < size;
        }

        /**
         * 다음 노드의 데이터를 반환하고 next를 그 다음으로 치환
         * 
         * @return {@code E} 다음 노드의 데이터
         */
        public E next() {
            checkForComodification();
            if (!hasNext())
                throw new NoSuchElementException();

            lastReturned = next;
            next = next.next;
            nextIndex++;
            return lastReturned.data;
        }

        /**
         * nextIndex가 0보다 크다면 다음을 가지고 있음
         * @return 크다면 {@code true} 반환, 작다면 {@code false} 반환
         */
        public boolean hasPrevious() {
            return nextIndex > 0;
        }

        /**
         * 반복자를 이전으로 넘기고 값의 이전 값을 next로 치환
         * 
         * @return {@code E} 이전 노드의 데이터 혹은 {@code next}가 0일 경우 마지막 값
         */
        public E previous() {
            checkForComodification();
            if (!hasPrevious())
                throw new NoSuchElementException();

            lastReturned = next = (next == null) ? last : next.prev;
            nextIndex--;
            return lastReturned.data;
        }

        public int nextIndex() {
            return nextIndex;
        }

        public int previousIndex() {
            return nextIndex - 1;
        }

        public void remove() {
            checkForComodification();
            if (lastReturned == null)
                throw new IllegalStateException();

            Node<E> lastNext = lastReturned.next;
            unlink(lastReturned);
            if (next == lastReturned)
                next = lastNext;
            else
                nextIndex--;
            lastReturned = null;
            expectedModCount++;
        }

        public void set(E e) {
            if (lastReturned == null)
                throw new IllegalStateException();
            checkForComodification();
            lastReturned.data = e;
        }

        public void add(E e) {
            checkForComodification();
            lastReturned = null;
            if (next == null)
                linkLast(e);
            else
                linkBefore(e, next);
            nextIndex++;
            expectedModCount++;
        }

        /**
         * 함수형 프로그래밍을 구현하기 위해 도입된 것
         * action은 함수나 람다식의 일종이고 action의 동작을 이터레이터의 데이터를 통해 에러가 나거나 남는 데이터가 없을 때까지 처리합니다.
         * 
         * @param action 수행할 동작 함수
         */
        public void forEachRemaining(Consumer<? super E> action) {
            Objects.requireNonNull(action);
            while (modCount == expectedModCount && nextIndex < size) {
                action.accept(next.data);
                lastReturned = next;
                next = next.next;
                nextIndex++;
            }
            checkForComodification();
        }

        /**
         * 반복자로 객체를 여러 스레드에서 동시에 수정하고자 하는경우 발생하는 에러
         */
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

    @Override
    public Iterator<E> descendingIterator() {
        return new DescendingIterator();
    }

    /**
     * {@link ListItr#previous}를 통해 내림차순 이터레이터를 제공하는 어댑터
     * {@link Iterator}를 확장해 최소한의 기능만을 가지고 있습니다.
     */
    private class DescendingIterator implements Iterator<E> {
        private final ListItr itr = new ListItr(size());
        public boolean hasNext() {
            return itr.hasPrevious();
        }
        public E next() {
            return itr.previous();
        }
        public void remove() {
            itr.remove();
        }
    }
    
    // ToString() 오버라이드
    @Override
    public String toString() {
        Iterator<E> it = listIterator(0);
        if (! it.hasNext())
            return "[]";

        StringBuilder sb = new StringBuilder();
        sb.append('[');
        for (;;) {
            E e = it.next();
            sb.append(e == this ? "(this Collection)" : e);
            if (! it.hasNext())
                return sb.append(']').toString();
            sb.append(',').append(' ');
        }
    }

    /**
     * 모든 노드의 값을 배열로 변환해서 반환합니다.
     *
     * @return 리스트의 모든 값을 {@link Object} 배열로 변환해서 반환
     */
    public Object[] toArray() {
        Object[] result = new Object[size];
        int i = 0;
        for (Node<E> x = first; x != null; x = x.next)
            result[i++] = x.data;
        return result;
    }

    /**
     * 내부 배열을 복사하고 매개 변수로 들어온 배열의 값을 수정한 배열을 반환합니다.
     *
     * @param a 복사될 값이 저장될 충분히 사이즈가 커야하는 배열,
     *          그렇지 않으면 해당 클래스의 배열을 새롭게 할당해버립니다.
     * @return 리스트의 요소들이 담긴 배열을 반환
     * @throws ArrayStoreException 저장될 매개 변수의 타입이 리스트의 클래스의 슈퍼타입이 아님.
     * @throws NullPointerException 지정된 배열의 값이 null임
     */
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            a = (T[])java.lang.reflect.Array.newInstance(
                                a.getClass().getComponentType(), size);
        int i = 0;
        Object[] result = a;
        for (Node<E> x = first; x != null; x = x.next)
            result[i++] = x.data;

        if (a.length > size)
            a[size] = null;

        return a;
    }

    @java.io.Serial
    private static final long serialVersionUID = 876323262645176354L;

    /**
     * {@code LinkedList} 이 인스턴스의 스트림을 저장합니다.
     *
     * @serialData 리스트의 직렬화된 데이터
     */
    @java.io.Serial
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        // 현재 클래스의 저장될 필드들을 저장함
        s.defaultWriteObject();

        // size를 별도로 저장함
        s.writeInt(size);

        // 모든 노드들의 값을 순차적으로 저장함
        for (Node<E> x = first; x != null; x = x.next)
            s.writeObject(x.data);
    }

    /**
     * {@code LinkedList} 직렬화된 인스턴스의 스트림을 불러와서 새롭게 인스턴스화합니다.
     */
    @SuppressWarnings("unchecked")
    @java.io.Serial
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        // 현재 클래스의 저장된 필드들을 불러옵니다.
        s.defaultReadObject();

        // 사이즈를 읽어옵니다.
        int size = s.readInt();

        // 모든 노드들의 값을 순차적으로 읽어옵니다.
        for (int i = 0; i < size; i++)
            linkLast((E)s.readObject());
    }

    
}
```

## 이번에 배운 것

- Java의 LinkedList 구현
- modCount가 이터레이터의 사용 도중 변경을 감지하는 방법
- Cloneable과 Serializable의 구현
  