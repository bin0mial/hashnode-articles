## Rails™ AntiPatterns Summary Part 2

## Chapter 1: Models Cont.
### AntiPattern: Voyeuristic Models Cont.
#### Solution: Push All Reading Calls into methods on the Model
##### Problem
**Programmers** who are **NOT familiar** with the **MVC** design pattern to which **Rails** **adheres**, as well as those who are just **unfamiliar** with the **general structures** provided by **Ruby on Rails** may **find** themselves with **code living** where it simply **doesn’t** **belong**. <br />
For **example**, if you wanted to **create** a **web page** that **displays** **all** the **users** in your web **application**, **ordered** by the **last name**, you **might** be tempted to **put** this **call** to find **directly** in the **view** code, as follows:
```html
<!-- app/views/users/index.html.erb -->
<html>
  <body>
    <ul>
      <% User.order(:last_name).each do |user| %>
        <li><%= user.last_name %> <%= user.first_name %></li>
      <% end %>
    </ul>
  </body>
</html>
```
While this may **seem** like a **straightforward** way to create the **web page**, you’ve put **calls** **directly** in the **view**; this style may **seem** **familiar** to **developers** coming from **PHP** <br />
At the very **least**, **including** the **actual logic** for what **users** will be **displayed** on this page is a **violation** of **MVC**. At **worst**, this **logic** can be **duplicated** many **times** **throughout** the **application**, causing very real **maintainability issues**. For **example**, it will be **problematic** if you want the **users** in your **application** to be **ordered** by their **last name** **whenever** they **appear** in a **list**.
##### Solution
In order to get **rid** of the **MVC violation**, you need to **move** the **logic** for which **users** are **displayed** on the page into the **Users controller** to look like:
```ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  def index
    @users = Users.order(:last_name)
  end
end
```
Then your **view** will look like the following:
```html
<!-- app/views/users/index.html.erb -->
<html>
  <body>
    <ul>
      <% @users.each do |user| %>
        <li><%= user.last_name %> <%= user.first_name %></li>
      <% end %>
    </ul>
  </body>
</html>
```
It **removed** the** MVC violation** **but** **times** have **changed**, and most **developers** now **realize** the **benefits** of going **one step further** and **moving** the **direct** **order** call down into the **model itself** 
```ruby
# app/models/user.rb
class User < ActiveRecord::Base
  def self.ordered
    order(:last_name)
  end
end
```
This will make **users controller** looks like:
```ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  def index
    @users = Users.ordered
  end
end
```
Since **Ruby on Rails** has **introduced** **scope** to **improve** **readability** and **get rid of fat models** will look like this:
```ruby
# app/models/user.rb
class User < ActiveRecord::Base
  scope :ordered, -> { order(:last_name) }
end
```
#### Solution: Keep Finders on Their Own Model
##### Problem
**Moving** the **find calls** **out** of the **Controller** layer in your **Rails** application and **into** c**ustom finders** on the **model** is a **strong step** in the **right direction** of **producing maintainable software**. A **common mistake**, however, is **to move those find calls into the closest model at hand**, **ignoring** proper **delegation** of **responsibilities**.
Say that while **working** on the **next great social networking application**, you find a **complex** **find** **call** in a controller:
```ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  def show
    @user = User.find(params[:id])
    @memberships = @user.memberships.where(active: true).limit(5).order(last_active_on: :desc)
  end
end
```
##### Solution
**Based** on what we **learned** from the **last two solutions**, we diligently **move** that **scope chain** into a **method** on the **User model**. Seems like the best place for it, since we are dealing with the UsersController:
```ruby
# app/models/user.rb
class User < ActiveRecord::Base
  has_many :memberships

  def find_recent_active_memberships
     memberships.where(active: true).limit(5).order(last_active_on: :desc)
  end
end

# app/controllers/users_controller.rb
class UsersController < ApplicationController
  def show
    @user = User.find(params[:id])
    @memberships = @user.find_recent_active_memberships
  end
end
```
This is definitely an **improvement**. **UsersController** is now much **thinner**, and the **method name reveals intent** nicely. But you can do more. <br />
The **User model** now **knows** far **too much** about the **Membership model’s implementation**, which is a clue that you still **haven’t** **pushed** the **methods** far **enough**. <br />
By **making use** of the **power** of **Active Record associations**, you can **trim up** this **example** even **further**. You can **define** a **finder** on **Membership**, and you **can** then **access** that **finder** through the **User#memberships association**, as follows:
```ruby
# app/models/user.rb
class User < ActiveRecord::Base
  has_many :memberships

  def find_recent_active_memberships
     memberships.find_recently_active
  end
end

# app/models/membership.rb
class Membership < ActiveRecord::Base
  belongs_to :user

  def self.find_recently_active
     where(active: true).limit(5).order(last_active_on: :desc)
  end
end
```
This is much better. The **application** now **honors** the **MVC boundaries** and **delegates** **domain model responsibilities** cleanly. This is a **fine** place to **stop** your **refactoring**, but you **can** also** make use** of Rails **Scopes** to look like that:
```ruby
# app/models/user.rb
class User < ActiveRecord::Base
  has_many :memberships

  def find_recent_active_memberships
     memberships.only_active.order_by_activity.limit(5)
  end
end

# app/models/membership.rb
class Membership < ActiveRecord::Base
  belongs_to :user

  scope :only_active, -> { where(active: true) }
  scope :order_by_activity, -> { order(last_active_on: :desc) }
end
```
By making the **refactoring** in this **last step**, you’ve **generalized** a lot of the **code**. Instead of only having a **Member#find_recently_active** method, you now have **three** **class methods** that you can **mix**. <br />
There are **downsides** to this **approach**, **including problems** with **readability** and **simplicity**, as well as **abuse** of the **Law of Demeter** <br />
###### Own opinion
Since we went **through** many **violations** due to **using** the **last approach** we can still **refactor** the last **code snippet** and add **scopes** to **group** the **methods** that **required** to be **called together** to be:
```ruby
# app/models/user.rb
class User < ActiveRecord::Base
  has_many :memberships

  def find_recent_active_memberships
     memberships.recently_active
  end
end

# app/models/membership.rb
class Membership < ActiveRecord::Base
  belongs_to :user

  scope :only_active, -> { where(active: true) }
  scope :order_by_activity, -> { order(last_active_on: :desc) }
  scope :only_active_ordered, -> { only_active.order_by_activity }
  scope :recently_active, -> { only_active_ordered.limit(5) } 
end
```
According to the **the last step**, we **increased** the **variety** of the **methods** the user **can** **use** and also we **improved** **readability** and we **didn't** **abuse** the **Law of Demeter**.
