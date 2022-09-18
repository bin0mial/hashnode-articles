## Rails™ AntiPatterns Summary Part 1

**Note**: Some of the **book's** **examples** and **sections** have been **modified** to **match** **modern rails** (Rails 6+)

## About the book
The book **Rails™ AntiPatterns - Best Practice Ruby on Rails™ Refactoring** was written by **Chad Pytel** and **Tammer Saleh**  to **improve** the way of **typing** rails **applications** as we will discuss in the following sections, you can find the book on **O'Reilly** [Rails™ AntiPatterns - Best Practice Ruby on Rails™ Refactoring](https://www.oreilly.com/library/view/railstm-antipatterns-best/9780321620293/).


## Introduction
**Rails** has been used widely in many applications nowadays such as [**Shopify**](https://www.shopify.com), [**Github**](https://github.com), and [**Basecamp**](https://basecamp.com/). <br />
Also, a lot of people tend to use this **framework** for **creating applications**. <br />
Before we start we have to understand some concepts:

### What Are AntiPatterns?
**AntiPatterns** are common **approaches** to **recurring problems** that ultimately **prove** to **be ineffective**. <br />
There must be at **least** **two key elements** present to formally **distinguish** an **actual AntiPattern** **from** a simple **bad habit**, **bad practice**, or **bad idea**:
- A **repeated pattern** of **action**, **process**, or **structure** that initially **appears** to be **beneficial** **but** ultimately **produces** more **bad consequences** than beneficial results
- A **refactored solution** that is **clearly documented**, **proven in actual practice**, and **repeatable**.

### What Is Refactoring?
**Refactoring** is the act of **modifying** an **application’s code** **NOT** to **change** its **functional** behavior but **instead** to **improve** the **quality** of the **application** itself. <br />
These **improvements** are intended to **improve** **readability**, **reduce** **complexity**, **increase** **maintainability**, and **improve** the **extensibility** (that is possibility for future growth) of the system.


## Chapter 1: Models
The **Model layer** of your Rails application often **provides** the **core structure** of your **application**. <br />
The **Model layer** should also **contain** the **business logic** of your application. **Therefore**, the **models** of your application will **often** **receive** the **majority** of **developer attention** throughout a project’s lifecycle. <br />
**Because** so **much attention** is **paid** to the **models**, and **because** so **much responsibility** is **contained** within them, it’s **relatively easy** for things to **get out of hand** in the models as an **application** **grows** and **evolves**.

This **chapter** **covers** many of the common pitfalls that can occur in the Model layer of the application, and we present proven **techniques** for **recovering** from these **issues**—and **avoiding** them in the first place.

### AntiPattern: Voyeuristic Models
- A **well-intentioned programmer** may **create** an **application** that **breaks** the f**undamental tenets** of **object-oriented** programming for a variety of **reasons**. For **example**, if the **programmer** is coming from a **less structured web development framework** (or un-framework) such as **Perl** or **PHP**, they may simply **not** be **aware** of the **structure** that **Ruby on Rails**, **MVC**, and **object-oriented** programming **provide**. **they** may **apply** what they **know** about their **current environment** to a program she’s building using Ruby on Rails.
- A **programmer** very **experienced** with **object-oriented** programming and **MVC** may ** first approach** the **Rails** framework and **be** **distracted** by the **dynamic nature** of the **Ruby** language and **unfamiliar** with what he might **consider** being **unique** **aspects** of the **Ruby** language, such as **modules**. **Distracted** by these things, he might **proceed** to build a **system** **without** first considering the **overall architecture** and **principles** with which he is **familiar** **because** he **perceives** **Ruby on Rails** to be **different**. Or perhaps a **programmer** is just **overwhelmed** by what the **Ruby on Rails** framework **provides**—such as **generators**, **lifecycle methods**, and the **Active Record ORM**—that she may get **distracted** by the immense **capability** and **build** a system **too quickly**, **too messily**, and **without** **foresight** or **engineering discipline**.
- The **following** **sections** present **several scenarios** that **violate** the core tenets of **MVC** and **object-oriented** programming, and they **present** **alternative** **implementations** and **procedures** for **fixing** these **violations** to **help produce** more **readable** and **maintainable** **code**.

#### Solution: Follow the Law of Demeter
An incredibly **powerful** **feature** of **Ruby on Rails** is **Active Record associations**, which are incredibly **easy to set up**, **configure**, and **use**. This ease of use **allows** you to **dive deep** down and **across** **associations**, **particularly** in **views**. However, while this **functionality** is **powerful**, it can make **refactoring** **tedious** and **error-prone**. <br />
Say that you’ve properly **encapsulated** your **application’s** **functionality** inside **different** **models**, and you’ve been **effectively** **breaking up** **functionality** into **small methods** on the **models** to so the code look like the following:
```ruby
# app/models/address.rb
class Address < ActiveRecord::Base
  belongs_to :customer
end

# app/models/customer.rb
class Customer < ActiveRecord::Base
  has_one :address
  has_many :invoices
end

# app/models/invoice.rb
class Invoice < ActiveRecord::Base
  belongs_to :customer
end
```
This code shows a simple **invoice** structure, with a **customer** who has a **single** **address**.
The view code to **display** the **address lines** for the **invoice** would be as follows:
```html
<!-- app/views/addresses/show.html.erb -->
<%= @invoice.customer.name %>
<%= @invoice.customer.address.street %>
<%= @invoice.customer.address.city %>
<%= @invoice.customer.address.state %>
<%= @invoice.customer.address.zip_code %>
```
Ruby on Rails **allows** you to easily **navigate** between the **relationships** of **objects** and therefore makes it easy to **dive deep** within and **across** related **objects**.
##### Problem
While this is **really powerful**, there are a few reasons it’s** not ideal**. For proper **encapsulation**, the invoice should **NOT** reach **across** the **customer** object to the **street attribute** of the **address** object. Because in the **future** if your application were to **change** so that a **customer** has both a **billing address** and a **shipping address**, **every place** in your **code** that reached **across** these **objects** to **retrieve** the **street** would **break** and would **need** to **change**.
##### Solution
To **avoid** the **problem** just described, it’s important to **follow** the **Law of Demeter**, also **known** as the **Principle of Least Knowledge**. <br />
The **concept** is that an **object** can **call methods** on a **related object** but that it should **NOT** **reach** **through** that **object** to **call a method** on a **third related object**.<br />
In Rails, this could be **summed** up as “**use only one dot**.” For example, `@invoice.customer.name` breaks the **Law of Demeter**, but `@invoice.customer_name` **does not**. Of course, this is an **oversimplification** of the principle, but it **can be** used as a **guideline**. <br />
To follow the **Law of Demeter**, you could **rewrite** the **code** above as follows:
```ruby
# app/models/address.rb
class Address < ActiveRecord::Base
  belongs_to :customer
end

# app/models/customer.rb
class Customer < ActiveRecord::Base
  has_one :address
  has_many :invoices

  def street
    address.street
  end

  def city
    address.city
  end

  def state
    address.state
  end

  def zip_code
    address.zip_code
  end
end

# app/models/invoice.rb
class Invoice < ActiveRecord::Base
  belongs_to :customer

  def customer_street
    customer.street
  end

  def customer_city
    customer.city
  end

  def customer_state
    customer.state
  end

  def customer_zip_code
    customer.zip_code
  end
end
```
Then the **invoice** **view** would look like:
```html
<!-- app/views/addresses/show.html.erb -->
<%= @invoice.customer_name %>
<%= @invoice.customer_street %>
<%= @invoice.customer_city %>
<%= @invoice.customer_state %>
<%= @invoice.customer_zip_code %>
```
The **downside** to this **approach** is that the **classes** have been **littered** with **many** **small wrapper** methods. If **things** were to **change**, now **all** of these **wrapper** **methods** would **need** to be **maintained**. <br />
**Fortunately**, **Ruby on Rails** includes a **function** that addresses the this concern. This **method** is the **class-level** **delegate** **method**. This method provides a **shortcut** for **indicating** that **one** or **more** methods that will be created on your **object** are actually **provided** by a related object. Using this **delegate** method, you can **rewrite** your example like this:
```ruby
# app/models/address.rb
class Address < ActiveRecord::Base
  belongs_to :customer
end

# app/models/customer.rb
class Customer < ActiveRecord::Base
  has_one :address
  has_many :invoices
  
  delegate :street, :city, :state, :zip_code, to: :address
end

# app/models/invoice.rb
class Invoice < ActiveRecord::Base
  belongs_to :customer

  delegate :street, :city, :state, :zip_code, to: :customer, prefix: true #use prefix so you add customer before the delegated method
end
```
Now **nothing** **needs** to be **changed** in the **view**.