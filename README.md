# Accessing Data in Pivotal GemFire

### 개요

Apache Geode용 Spring Data를 사용하여 POJO를 저장하고 검색합니다.

> **Apache Geode**는 `key/value` 저장소이며, 지역은 `java.util.concurrent.ConcurrentMap` 인터페이스를 구현합니다.

> **POJO(Plain Old Java Object)**는 순수한 자바 객체로 특정 기술에 종속적이지 않은 순수한 객체를 의미합니다.

### 종속성

- Spring for Apache Geode

### 엔티티 정의

몇 가지 어노테이션을 사용해 Apache Geode(지역)에 `Person` 객체를 저장합니다.

`src/main/java/com/example/gemfile/Person.java`:

```java
package hello;

import java.io.Serializable;

import org.springframework.data.annotation.Id;
import org.springframework.data.annotation.PersistenceConstructor;
import org.springframework.data.gemfire.mapping.annotation.Region;

import lombok.Getter;

@Region(value = "People")
public class Person implements Serializable {

  @Id
  @Getter
  private final String name;

  @Getter
  private final int age;

  @PersistenceConstructor
  public Person(String name, int age) {
    this.name = name;
    this.age = age;
  }

  @Override
  public String toString() {
    return String.format("%s is %d years old", getName(), getAge());
  }
}
```

- `@Region(value = "People")`은  Apache Geode가 이 클래스의 인스턴스를 저장하면 People 지역 내에 새 항목이 생성됩니다.
- `name` 필드를 `@Id`로 표시합니다. 이것은 Apache Geode 내에서 개인 데이터를 식별하고 추적하는 데 사용되는 식별자를 나타냅니다.

기본적으로 `@Id` 어노테이션 필드(`name`)가 `key`이고, Person 인스턴스는 `value` 입니다. Apache Geode에는 자동화된 키 생성이 없으므로 Apache Geode에 엔티티를 유지하기 전에 ID(`name`)을 설정해야 합니다.

### 리포지토리 정의

Spring Data for Apache Geode는 Spring을 사용하여 Apache Geode에 데이터를 저장하고 액세스하는 데 중점을 둡니다. 기본적으로 Apache Geode(OQL)의 쿼리 언어를 배울 필요는 없습니다. 몇 가지 메서드를 작성할 수 있으며 프레임워크가 쿼리를 작성합니다.

어떻게 작동하는지 보기 위해 Apache Geode에 저장된 Person 객체를 쿼리하는 인터페이스를 생성해야 합니다.

`src/main/java/com/example/gemfile/PersonRepository.java`:

```java
import org.springframework.data.gemfire.repository.query.annotation.Trace;
import org.springframework.data.repository.CrudRepository;

public interface PersonRepository extends CrudRepository<Person, String> {

  @Trace
  Person findByName(String name);

  @Trace
  Iterable<Person> findByAgeGreaterThan(int age);

  @Trace
  Iterable<Person> findByAgeLessThan(int age);

  @Trace
  Iterable<Person> findByAgeGreaterThanAndAgeLessThan(int greaterThanAge, int lessThanAge);

}
```

PersonRepository는 Spring Data Commons에서 CrudRepository 인터페이스를 상속받고, value(Person 인스턴스)와 리포지토리가 작동하는 key(ID, 문자열)에 대한 제네릭 타입 매개변수의 타입을 지정합니다.

이 인터페이스는 기본 CRUD(Create, Read, Update, Delete) 및 간단한 쿼리 데이터 액세스 작업(`findById()`)을 포함한 많은 작업과 함께 제공됩니다.
