---
layout: post
title: Spring Data queries using the JPA Criteria API
category: "Java"
---

Spring has nice features for data persistence and gives you simple data access abstractions. Standard queries in Spring Data repositories are easy but when it comes to complex queries, people tend to use the `@Query` annotation. For some cases, it is complex to write these queries. Therefor I prefer the JPA Criteria API and how to improve complex queries using that.

## JPA entities in Spring Data

Spring Data is actually a concept to handle data persistence in a smart way and independently from the underlying database. It knows repositories which are the abstraction of tables, collections, documents or whatever your database management system calls a list of ‘similar’ tuples of an entity. Therefor entities are defined, which are basically PoJo’s having JPA annotations. For example the `@Entity` annotation makes them a managed entity by the database and multiplicities can be stated with `@OneToMany`, `@ManyToOne` etc. The following snippet shows the class Car being a JPA entity:

``` java
@Entity
public class Car {

    @Id
    private String plate;

    @Convert(converter = ColorConverter.class)
    private CarColor color;

    @OneToOne(mappedBy = "car")
    private User owner;

    private CarBrand brand;
    private String type;
    private int horsePower;
    private int maxSpeed;
    private FuelKind fuelKind;
    private double fuelUsage;

    // Constructors, getters and setters
}
```

## Spring Data repositories – It’s magic!

To get access to this (let’s assume an SQL database for now) table, a repository has to be created. In this repository we can write simple queries by simply defining new method signatures in the interface, following Springs language to build repository queries. Spring cares about implementing these queries – you only have to wish for them in the correct syntax. Also SQL-styled queries can be written ‘the old school way’ using the `@Query` annotation. Let’s extend the Car example with a repository illustration these query possibilities:

``` java
@Component
public interface CarRepository extends Repository<Car, String> {

    List<Car> findAllByOrderByPlate(Pageable pageable);

    Long countByBrand(CarBrand brand);

    @Query("select distinct c.type from Car as c")
    List<String> findAllUsedTypes();
}
```

After defining this repository with a `@Component` annotation, we can inject the repository as a dependency everywhere in our Spring context (although I recommend to follow the Three-Layer-Architecture principle). So far, so nice – isn’t it?
Plugging together our perfect Spring Data repository

You may be asking yourself which Repository interface our CarRepository is inheriting from. That’s a good one. It is our own super-repository that defines all functionality that every repository in our application should have, so we have this definition in one place. We should inherit from PagingAndSortingRepository and JpaSpecificationExecutor and can still define own custom signatures of methods. Have a look how mighty the PagingAndSortingRepository is as again inherits lots of commonly needed query signatures from CrudRepository.

``` java
@NoRepositoryBean
public interface Repository<T, ID extends Serializable> extends PagingAndSortingRepository<T, ID>,
    JpaSpecificationExecutor<T> {

    <S extends T> List<S> save(Iterable<S> entities);

    List<T> findAll();

    List<T> findAll(Sort sort);
}
```

## Querying data using the JPA Criteria API

If you payed attention to the last interface our Repository is inheriting from, JpaSpecificationExecutor, you are now seeing the bridge to the JPA Specifications API. It provides a few powerful methods expecting a Specification<T> as parameter. Those specifications are simply bundling and combining the logical descriptiveness of JPAs Predicates. A specification contains all logic of a easy or even very complex query. If queries get more complex, the idea is to divide it in atomic pieces and plug them together at the end. Those atomic pieces can be collected in a SpecificationFactory like that:

``` java
public final class SpecificationFactory {

    public static Specification containsLike(String attribute, String value) {
        return (root, query, cb) -> cb.like(root.get(attribute), "%" + value + "%");
    }

    public static Specification isBetween(String attribute, int min, int max) {
        return (root, query, cb) -> cb.between(root.get(attribute), min, max);
    }

    public static Specification isBetween(String attribute, double min, double max) {
        return (root, query, cb) -> cb.between(root.get(attribute), min, max);
    }

    public static <T extends Enum<T>> Specification enumMatcher(String attribute, T queriedValue) {
        return (root, query, cb) -> {
            Path<T> actualValue = root.get(attribute);

            if (queriedValue == null) {
                return null;
            }

            return cb.equal(actualValue, queriedValue);
        };
    }
}
```

The above specifications could be used for any entity because ‘contains’, ‘isBetween’ and matching enums (actually only matching strings) are very commonly needed. But if for instance you want to accomplish an extended search function for cars in your application, they are helpful when plugged together correctly. The following code shows a composition of building block like specifications we defined and how it can be used in a repository service:

``` java
public static Specification extendedCarSearch(CarSearchQuery q) {
    return Specifications
            .where(containsLike("plate", q.getPlate()))
            .and(containsLike("type", q.getType())).and(enumMatcher("brand", q.getBrand()))
            .and(isBetween("usage", q.getUsage().getMin(), q.getUsage().getMax()))
            .and(isBetween("maxSpeed", q.getMaxSpeed().getMin(), q.getMaxSpeed().getMax()))
            .and(isBetween("horsePower", q.getHorsePower().getMin(), q.getHorsePower().getMax()))
            .and(enumMatcher("fuelKind", q.getFuelKind()));
}

public List extendedSearch(CarSearchQuery query) {
    return repository.findAll(CarSpecificationFactory.extendedCarSearch(query));
}
```

Defining very small but reusable specifications is really great as you see here. You could not write this query logic in a beautifully readable `@Query` annotation or query methods – it would be a mess.
Conclusion

We implemented a JPA entity called ‘Car’, a Spring Data JPA super-repository which we use as base class for the CarRepository. By doing this like I did it here, we have a wide variety of default querying methods. We can also simply write our own query methods thanks or use the `@Query` annotation for SQL-styled queries.
A third possibility which I showed you in this article is to use the JPA Criteria API. In combination with Spring we therewith have a powerful and data-specific language to write easy to complex queries while being able to combine them. Using the JPA Criteria API definitely results in well readable and understandable clean code in comparison to ugly SQL queries.
