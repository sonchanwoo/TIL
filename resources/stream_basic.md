> 🚀 무엇이 불편한가?

```java
        //원하는 데이터를 다룰 때 데이터 소스마다 방식이 다르다.
        String[] strArr = {"aa","bb"};
        List<String> strList = Arrays.asList(strArr);
        
        //정렬 + 전체출력(반복하며)
        Arrays.sort(strArr);
        for(String str:strArr)
            System.out.println(str);
        
        Collections.sort(strList);
        for(String str:strList)
            System.out.println(str);
```

- Array / List(Collection)의 정렬 등을 다루는 방법이 상이하여 위 같이 해야한다.

```java
        Stream<String> strStream1 = strList.stream();
        Stream<String> strStream2 = Arrays.stream(strArr);
        
        strStream1.sorted().forEach(System.out::println);
        strStream2.sorted().forEach(System.out::println);
        
        //스트림은 데이터를 읽기만 한다.
        //스트림은 내부적으로 동작한다.
        //중간연산(sorted, filter 등)은 여러번
        //최종연산(forEach, collect 등)은 하나만 할 수 있다.
        
        //스트림은 일회용이다.(최종연산이 수행되면 스트림이 닫힌다.)
        //최종연산이 수행되기 전까지 중간연산은 수행되지 않는, 지연된 연산을 한다.
```

- 스트림은 (추상화 된) 표준 데이터 소스라고 생각하면 편할 것 같다.

- 일단 스트림을 생성하면(초기 생성 방식은 다를 수 있으나) 다루는 방식이 같아진다.

- 주석 참고

<br/>

> 🚀 기본 자료형에 대한 스트림을 제공한다.

```java
        Integer[] numArr = {1,2,3,4};
        List<Integer> numList = Arrays.asList(numArr);        
        
        //컬렉션 스트림 생성방법
        numList.stream();        
        
        //배열 스트림 생성방법
        Arrays.stream(numArr);
        Stream.of(numArr);        
        //배열 스트림 생성방법(원시 타입일 때)
        int[] numArr2 = {1,2,3,4,5};
        IntStream.of(numArr2);
        
        //range vs rangeClosed
        IntStream.range(1, 5);//1,2,3,4
        IntStream.rangeClosed(1, 5);//1,2,3,4,5    
```

> 🚀 여러 번 할 수 있는 중간연산(forEach는 최종연산)

- 1

```java
    public static void main(String[] args) {
        //다음은 forEach, 2부터 3개 중복없이 를 의미함
        IntStream.rangeClosed(1, 5).skip(1).limit(3).distinct().forEach(System.out::print);

        //filter에는 Predicate를 필요로 하지만 return이 boolean인 람다식을 넣을 수도 있다.
        IntStream.range(1, 5).filter(i -> i%2 == 0).forEach(System.out::print);
        IntStream.range(1, 5).filter(MidCal::method).forEach(System.out::print);        
        
    }
    static boolean method(int i) {
        return  i%2==0;
    }
```

- 2 - sorted

```java
	    //sorted
        //매개변수 없으면 기본적인 Comparable로 정렬한다.(해당 인터페이스를 구현하지 않으면 예외가 발생)
        //Comparator로 구현한다.
        
        List<String> strList = Arrays.asList();
        Stream<String> strStream = strList.stream();
        
        //모두 같다
        strStream.sorted().forEach(System.out::print);
        strStream.sorted(Comparator.naturalOrder()).forEach(System.out::print);
        strStream.sorted(String::compareTo).forEach(System.out::print);
        strStream.sorted((s1,s2) -> s1.compareTo(s2)).forEach(System.out::print);
```

- 3 - map, peek ...

```java
        //map : 원하는 부분만 뽑아내거나, 특정 형태로 변환할 때
        List<String> str = null;
        str.stream().map(String::toUpperCase)
        .peek(System.out::print).map(s -> s.substring(-1)).forEach(System.out::print);;
        //peek는 연산사이에 로깅하는 용도로 사용하여 보자!
        
        //Stream<T>를 IntStream으로 변환할 땐 mapToInt
        //IntStream을 Stream<T>로 변활할 땐 mapToObj
        //IntStream을 Stream<Integer>로 변활할 땐 boxed(collect()를 사용하기 위함)
```

<br/>

> 🚀 단 한 번만 수행되는 최종연산

- 1

```java
        IntStream i = IntStream.range(0, 1);

        //forEach
        i.forEach(System.out::print);
        
        //match : filter와 달리 최종연산이고 boolean값을 반환한다
        i.allMatch(e -> e%2 == 0);//모두 맞나
        i.anyMatch(e -> e%2 == 0);//하나라도 맞나
        i.noneMatch(e -> e%2 == 0);//아무것도 아닌가
        
        //find
        i.findFirst();//주로 filter와 같이 쓰여 조건에 맞는 것을 반환한다.(Optional<T>을 반환한다.)
        
        //그 밖에도 통계를 위한 메서들, 리듀싱 등이 있다.
```

- 2 

```java
        //collect()가 수집할 때 어떻게 할 것인가에 대한 collector에 대해 학습한다.
        //collect()는 매개변수로 collector를 필요로 한다.
        //Collector인터페이스를 구현한 Collectors클래스를 사용할수도, 직접 구현하여 만들어 사용할수도 있다.
        
        IntStream.range(0, 0).boxed().collect(Collectors.toList());
        IntStream.range(0, 0).boxed().collect(Collectors.toSet());
        //특정 컬렉션 지정 시
        IntStream.range(0, 0).boxed().collect(Collectors.toCollection(ArrayList::new));
        //map은 key, value이므로 아래처럼 해야한다.
        //(p -> p처럼 항등식일 경우 indentity메서드를 사용할 수 있음)
        IntStream.range(0, 0).boxed().collect(Collectors.toMap(p->p,Function.identity()));
        
        //이 밖에도 통계함수, 리듀싱, 그룹화 등이 있다.
```