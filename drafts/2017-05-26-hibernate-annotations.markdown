<!-- ---
layout: post
title:  "Hibernate Annotations in Spring :leaves:"
date:   2017-05-31 11:35:02 -0400
categories: hibernate spring java
--- -->

Spring is filled with annotations. It both makes your life easy and a living nightmare. Forget the right one and it'll break and cry and show only semi-useful error messages. Use the wrong one and it'll break and cry and show even less useful error messages. It can be tricky, but when used right it can save you from writing a whole lot of code, and, even better, having to think out complicated object relationships and try to weave it all together.

On a recent project I noticed that out of all the annotations my team struggled with, Hibernate annotations were by far the most difficult. Something about trying to take a very complicated relational database and figure out how to get hibernate to just do what we wanted to do, proved very difficult. (Granted some of the issues might've been from a very very complicated data schema and big design changes happening often, but I digress &#x1F910;)

Since I noticed some of the documentation out there was a little hard to read, I thought I would try and make a clear and simple example on how to use these annotations. Here I will go through and illustrate a fairly simple situation that utilizes a few hibernate annotations.

{:.highlightDarkEmphasis}
**tldr;** in this article I cover basic Hibernate annotations: `@Entity`, `@Table`, `@Column`, `@OneToOne`, `@OneToMany`. In a future follow up post, I will explain `@ManyToOne`, `@ManyToMany`, and `@SecondaryTable` (and maybe a few others).

### Getting Started

You can clone a copy of the code :point_right: __[here](https://github.com/galperie/hibernateAnnotationsTutorial)__

Included in the README of the repository are instructions on how to set up the database using Docker :whale:. If you haven’t [set up docker](https://docs.docker.com/get-started/) before, I really recommend doing so. It makes quickly spinning up things like databases super easy. If you would rather work with a local instance, then feel free to set up mysql (or something similar) and change the `application.properties` file of the project as necessary.

### The Schema

Get your barista hats, we'll be setting up a Coffee Shop :coffee:! By the end of this tutorial, our database will look like this:

**CoffeeShop**

| shop_id |   name    | owner_id |
|:-------:|:---------:|:--------:|
|    1    | Gotham Coffee | 65   |
|================================|

<br>

**Owner**

| owner_id | first_name  | last_name |
|:-----------:|:-----------:|:---------:|
|    65       | Bruce       |  Wayne    |
|=======================================|

<br>

<a name="foreignKeyExample"></a>

**CoffeeShopEmployees**

| shop_id |   employee_id    |
|:-------:|:----------------:|
|    1    |      22          |
|    1    |      23         |
|============================|


<br>

**Employees**

| employee_id | first_name  | last_name |
|:-----------:|:-----------:|:---------:|
|    22       | Richard       |  Grayson    |
|    23       | Tim           |  Drake      |
|=======================================|

<br>

*\*\*I've purposely made the ids in this example unique so you can easily see the relationships*

### Creating a Coffee Shop

This app will be as simple as possible: we have one controller that will serve as our REST endpoint and when we send information about our new coffee shop to it, it will call our repository class to make changes to the database.

{% highlight java %}
@RequestMapping("/")
@RestController
public class CoffeeShopController {

  @Autowired
  private CoffeeShopRepository coffeeShopRepository;

  @RequestMapping(method = RequestMethod.POST)
  public ResponseEntity<?> add(@RequestBody CoffeeShop coffeeShop) {
    CoffeeShop savedShop = coffeeShopRepository.save(coffeeShop);
    return new ResponseEntity<Object>(savedShop, HttpStatus.OK);
  }
}
{% endhighlight %}  
<br>

The repository will be an interface that extends from the JpaRepository. In this case we’ll be utilizing the useful default methods that the [**jpa comes with**](http://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/JpaRepository.html) (save, findAll, etc).
 The benefit of using Hibernate Annotations is that it helps keep your repository class as small and as simple as possible; hopefully eliminating any need for any mapping or custom logic.

{% highlight java %}
@Repository
public interface CoffeeShopRepository extends JpaRepository<CoffeeShop, Long> {}
{% endhighlight %}

<br>
For our objects, let's start off with a basic coffee shop with only a name and an id:

<a name="coffeeShopExample"></a>
{% highlight java %}
@Entity
@NoArgsConstructor // auto generate a constructor
@Getter // auto generate getter methods for each parameter
@Setter // auto generate setter methods for each parameter
public class CoffeeShop {

  @Id
  @GeneratedValue(strategy = GenerationType.AUTO)
  @Column(name="shop_id")
  private Long id;

  private String name;
}
{% endhighlight %}

:star2: Important things to note:

* The inclusion of the `@Entity` annotation. Marks this object as an entity bean aka an object that will be used in database transactions.

* The `@Id` marks the id parameter as the unique identifier for the object. Doesn't have to be an id, but I find that's the most common in relational databases.

* The use of [**lombok annotations**](https://projectlombok.org/features/all) (`@Getter`, `@Setter`, `@NoArgsConstructor`). This is an amazing library that helps auto generate methods and constructors that I find clutter objects. **All Entities require getters and setters**, and why have a huge file filled with standard methods, when 1 annotation can replace all of that?  &#129300; Still not convinced?
That's ok! Just write out getter and setter methods for each parameter and an empty constructor. It'll work just the same.

:warning: If the name of the object is different from the table use the `@Table(name="")` annotation.

{:.hiddenList}
* *i.e.*{:.highlightEmphasis} if we wanted the table to be called **shop** because we want to eventually expand to candy shops and pet shops, we can just say:

* {% highlight java %}
@Entity
@Table(name="shop")
@Getter
@Setter
public class CoffeeShop {...}
{% endhighlight %}

{:.hiddenList}
* **Otherwise, it will assume the object name and the table name are the same.**

* Similarly if the column in the table does not match the naming of the parameter, use the `@Column` annotation like we did for the id parameter:

{% highlight java %}
@Column(name="shop_id")
private Long id;
{% endhighlight %}


<br>
If everything is working as it should, when we run the app and send a POST to [localhost:8080] with this as the body:

<figure>
  <figcaption>json body of request</figcaption>
{% highlight json %}
  {
   "name": "gotham coffee"
  }
{% endhighlight %}
</figure>

We should get back this:

<figure>
  <figcaption>json body of the response</figcaption>
{% highlight json %}
  {
   "id": 1,
   "name": "gotham coffee"
  }
{% endhighlight %}
</figure>

If we look at our table, we should then see a new entry!

<figure>
  <figcaption>terminal view of the database</figcaption>
{% highlight shell_session %}
mysql> select * from coffee_shop;
+---------+---------------+
| shop_id | name          |
+---------+---------------+
|       1 | gotham coffee |
+---------+---------------+
1 row in set (0.00 sec)
{% endhighlight %}
</figure>


### Adding an Owner: @OneToOne

Our Coffee Shop is now up and running! Time to spice things up and add an owner &#x1F481;

For this example we will say each coffee shop will only belong to *one* owner. And each owner will only have *one* coffee shop.
So can you guess which annotation, we'll be using? `@OneToOne` :open_mouth: :exclamation:

The owner will basically be the [same as the coffee shop](#coffeeShopExample), but with its own unique parameters.

{% highlight java %}
@Entity
@NoArgsConstructor
@Getter
@Setter
public class Owner {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name="owner_id")
    private Long id;

    private String firstName;

    private String lastName;
}
{% endhighlight %}

And now we'll update our coffee shop so that it has the owner.

{% highlight java %}
@Entity
@NoArgsConstructor
@Getter
@Setter
public class CoffeeShop {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name="shop_id")
    private Long id;

    private String name;

    @OneToOne(cascade = {CascadeType.ALL})
    private Owner owner;
}
{% endhighlight %}

:star2: Things to note:

* Adding the `@OneToOne` annotation tells hibernate that the relationship as we stated before is one to one, meaning one coffeeshop has one owner and vise versa. There are other relationships we can express using similar annotations (OneToMany, ManyToOne, ManyToMany). We'll get into some of them later on.

* The addition of the `CascadeType.ALL` to the annotation says that the operation you are doing should cascade down (aka if we are creating a coffeeshop with an owner that doesn't exist, cascade and create that owner too; if you are deleting the coffeeshop, delete the owner too, etc). There are a [couple of options](https://docs.jboss.org/hibernate/jpa/2.1/api/javax/persistence/CascadeType.html) for when cascading should take place. *It's optional so if you do not want it to happen, don't include it.*

:warning: In this case the relationship between owner and coffeeshop is :arrow_right: **uni-directional** (aka coffeeshop has a reference to its owner, but the owner doesn't have a reference to its coffeeshop). This is just a design decision, if you need the relationship to be :left_right_arrow: **bi-directional** (they both have a reference to each other) then you can use the parameter `mappedBy=""`. This will look the same in the database, really depends on the business logic and what data you want each object to have. So if in your app you want the owner be able to its coffee shop just add the following along with the changes we made to CoffeeShop:

{% highlight java %}
@Entity
@NoArgsConstructor
@Getter
@Setter
public class Owner {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name="owner_id")
    private Long id;

    private String firstName;

    private String lastName;

    @OneToOne(mappedBy = "owner")
    private CoffeeShop coffeeShop;
}
{% endhighlight %}

<br>

Ok let's try this all out (I just went with the uni-directional route). Send the updated json to [localhost:8080].

<figure>
  <figcaption>json body of the request</figcaption>
{% highlight json %}
{
  "name": "gotham coffee",
  "owner": {
    "firstName": "Bruce",
    "lastName": "Wayne"
  }
}
{% endhighlight %}
</figure>

<figure>
  <figcaption>json body of the response</figcaption>
{% highlight json %}
{
  "id": 1,
  "name": "gotham coffee",
  "owner": {
    "id": 1,
    "firstName": "Bruce",
    "lastName": "Wayne"
  }
}
{% endhighlight %}
</figure>

And if we check out the database:

{% highlight shell_session %}
mysql> select * from coffee_shop;
+---------+---------------+----------+
| shop_id | name          | owner_id |
+---------+---------------+----------+
|       1 | gotham coffee |        1 |
+---------+---------------+----------+
1 row in set (0.00 sec)

mysql> select * from owner;
+----------+------------+-----------+
| owner_id | first_name | last_name |
+----------+------------+-----------+
|        1 | Bruce      | Wayne     |
+----------+------------+-----------+
1 row in set (0.00 sec)
{% endhighlight %}

### Adding Employees - @ManyToOne

Each Coffee Shop can have *many* employees, but each employee can only work for *one* shop. This means we'll use the `@OneToMany` annotation :tada:

Similar to CoffeeShop and Owner, we'll make an Employee :two_men_holding_hands:

{% highlight java %}
@Entity
@NoArgsConstructor
@Getter
@Setter
public class Employee {

    @Id
    @GeneratedValue
    @Column(name="employee_id")
    private Long id;

    private String firstName;

    private String lastName;
}
{% endhighlight %}

Nothing special here. Now we'll add employee to CoffeeShop

{% highlight java %}
@Entity
@NoArgsConstructor
@Getter
@Setter
public class CoffeeShop {

    @Id
    @GeneratedValue
    @Column(name="shop_id")
    private Long id;

    private String name;

    @OneToOne(cascade = {CascadeType.ALL})
    @JoinColumn(name="owner_id")
    private Owner owner;

    @OneToMany(cascade = {CascadeType.ALL})
    private List<Employee> employees;
}
{% endhighlight %}

Very similar to Owner, except this time we're using the `@OneToMany` annotation. Hibernate is smart enough to see this and create that table of foreign keys I showed in the [overview of how the database](#foreignKeyExample) will end up looking. Also note how the CoffeeShop has a List of employees. This represents that Many part of the relationship.

Just send an updated POST and see:

<figure>
  <figcaption>json body of the request</figcaption>
{% highlight json %}
{
  "name": "gotham coffee",
  "owner": {
    "firstName": "Bruce",
    "lastName": "Wayne"
  },
  "employees": [
    {
      "firstName": "Richard",
      "lastName": "Grayson"
    },
    {
      "firstName": "Tim",
      "lastName": "Drake"
    }
  ]
}
{% endhighlight %}
</figure>

<figure>
  <figcaption>json body of the response</figcaption>
{% highlight json %}
{
  "id": 1,
  "name": "gotham coffee",
  "owner": {
    "id": 1,
    "firstName": "Bruce",
    "lastName": "Wayne"
  },
  "employees": [
    {
      "id": 1,
      "firstName": "Richard",
      "lastName": "Grayson"
    },
    {
      "id": 2,
      "firstName": "Tim",
      "lastName": "Drake"
    }
  ]
}
{% endhighlight %}
</figure>

And if we check out the database:

{% highlight shell_session %}
mysql> select * from coffee_shop;
+---------+---------------+----------+
| shop_id | name          | owner_id |
+---------+---------------+----------+
|       1 | gotham coffee |        1 |
+---------+---------------+----------+
1 row in set (0.00 sec)

mysql> select * from coffee_shop_employees;
+---------------------+-----------------------+
| coffee_shop_shop_id | employees_employee_id |
+---------------------+-----------------------+
|                   1 |                     1 |
|                   1 |                     2 |
+---------------------+-----------------------+
2 rows in set (0.00 sec)

mysql> select * from employee;
+-------------+------------+-----------+
| employee_id | first_name | last_name |
+-------------+------------+-----------+
|           1 | Richard    | Grayson   |
|           2 | Tim        | Drake     |
+-------------+------------+-----------+
2 rows in set (0.00 sec)
{% endhighlight %}

### Recap & What We Didn't Get To

What we covered in this post was how to use hibernate to create your tables and specifically how to create the one-to-one/one-to-many relationship using annotations.
This is by far not all hibernate can do and I want to continue with this post with examples of some more complicated annotations:

* ManyToMany
* ManyToOne
* SecondaryTable

and possibly more. Feel free to reach out if any of the examples aren't clear or if there was something you wanted to see that I did not cover.

-- :woman: :computer: :wave:

[localhost:8080]: http://localhost:8080
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
