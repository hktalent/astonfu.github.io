---
layout: post
title: Rails Active Record
---

# 组成
ActiveModel 是 ActiveRecord 的一部分，主要负责模型相关，而后者主要负责数据库等其它工作。


# 关系

- belongs_to
- has_one
- has_many
- has_many :through
- has_one :through
- has_and_belongs_to_many

## 主要参数

- class_name: 对应的类名，比如 has_many :clients, class_name: 'Person'
- dependent: 当它的 owner 被 destroyed 时，它该如何处理。
  - :destroy causes the associated object to also be destroyed
  - :delete causes the associated object to be deleted directly from the database (so callbacks will not execute)
  - :nullify causes the foreign key to be set to NULL. Callbacks are not executed.
  - :restrict_with_exception causes an exception to be raised if there is an associated record
  - :restrict_with_error causes an error to be added to the owner if there is an associated object
  - Note that :dependent option is ignored when using :through option.
- foreign_key: 关联使用的外键，默认是 #{class_name}_id，如user_id。
- foreign_type: 如果这个类是 polymorphic，指明相关的列名。默认是as的参数加上_type，比如：一个类定义一个 taggable， has_one :tag, as: :taggable，那么会用 “taggable_type” 为默认的 :foreign_type.
- primary_key: 指定属性名，比如select where primary_key = id。
- polymorphic: 配合 belongs_to 使用，指明当前类是否是多态。
- as: 当做，配合 has_many 使用，指明多态的 polymorphic “父类”。
- through: Specifies a Join Model through which to perform the query. Options for :class_name, :primary_key, and :foreign_key are ignored, as the association uses the source reflection. You can only use a :through query through a has_one or belongs_to association on the join model.
- source: Specifies the source association name used by has_one :through queries. Only use it if the name cannot be inferred from the association. has_one :favorite, through: :favorites will look for a :favorite on Favorite, unless a :source is given.
- source_type: Specifies type of the source association used by has_one :through queries where the source association is a polymorphic belongs_to.
- validate: When set to true, validates new objects added to association when saving the parent object. false by default. If you want to ensure associated objects are revalidated on every update, use validates_associated.
- autosave: If true, always save the associated object or destroy it if marked for destruction, when saving the parent object. If false, never save or destroy the associated object. By default, only save the associated object if it's a new record.
  - Note that ActiveRecord::NestedAttributes::ClassMethods#accepts_nested_attributes_for sets :autosave to true.
- inverse_of: Specifies the name of the belongs_to association on the associated object that is the inverse of this has_one association. Does not work in combination with :through or :as options. See ActiveRecord::Associations::ClassMethods's overview on Bi-directional associations for more detail.
- required: When set to true, the association will also have its presence validated. This will validate the association itself, not the id. You can use :inverse_of to avoid an extra query during validation.

## has_many through
多对多的关系。


## polymorphic
主要解决一张表要记录多种类型的数据。这样方便筛选。

比如一张评论表，想记录照片的评论，又想记录文章的评论，那么就用polymorphic吧，而不是建两张表。

polymorphic 翻译来就是“多态”，它是面向对象的基石。
它使程序能够把不同种类的东西当作相同的东西来处理，从而做到更高层的抽象。

所以它还能实现继承，
也就是[单表继承](http://api.rubyonrails.org/classes/ActiveRecord/Inheritance.html)让所有子类在一张表里。

只不过继承里 xxxxable 是父类（就不叫 xxxxable 了），而 xxxxable 是组合关系，而且是被多个类拥有，比如
文章和照片都可以有评论。其实都可以理解为一对多的关系，被多个子类继承，或被多个类拥有。在设计模式里，更倾向于
使用组合，因为这样更耦合。

只有理解了各个类之间的关系，比如是继承还是耦合，才能更好的给类起名字。一个好的类名说明你对它们的
关系理解了。

实现上Java用接口，Ruby用了Duck Type。

那么在表数据中，如何来表示这条记录是哪个态呢？多加一个type字段，一般是xxxxable_type.

那如果这条数据和其它数据关联，如何记录？多加一个那个关联数据的id，一般是xxxxable_id.

这就是Rails里model里的polymorphic的功能。

具体例子参考[RailsCasts](1)。在这个例子中comment可以是Article、Photo和Event的comment。

它们model如下：

```ruby
class Comment < ActiveRecord::Base
  belongs_to :commentable, :polymorphic => true
end

class Article < ActiveRecord::Base
  has_many :comments, :as => :commentable
end

class Photo < ActiveRecord::Base
  has_many :comments, :as => :commentable
  #...
end

class Event < ActiveRecord::Base
  has_many :comments, :as => :commentable
end
```


在数据库migration里，可以用 t.references 来创建表结构：

```rb
class CreatePictures < ActiveRecord::Migration[5.0]
  def change
    create_table :comments do |t|
      t.string :content
      t.references :commentable, polymorphic: true, index: true
      t.timestamps
    end
  end
end
```

比如 a_phote.comments 将会进行如下查询：

```sql
 SELECT "comments".* FROM "comments" WHERE "comments"."commentable_id" = $1 AND "comments"."commentable_type" = $2  [["commentable_id", 1], ["commentable_type", "Phote"]]
```

上面的 t.references :commentable, polymorphic: true, index: true
已经把 commentable_id 和 commentable_type 一起加入到数据框的索引了，相当于在 migration里：

```rb
add_index :comments, [:commentable_id, :commentable_type]
```

使用belongs_to和has_many这么“通俗”的方法就搞定了。可以看到不像非多态时，belongs_to 后的
参数是具体的 Owner，比如 user，这里直接是 commentable，并且 polymorphic 为 true。
正式这样将其多态了，因为没有指明具体的类，而是一个需要有评论功能的类，比如 Photo。当然，也就不用
belongs_to 给每个可能的 Owner 了，都在 commentable 里了。

而这个 Photo 类通过 has_many（拥有） comments，并且 as（当做）commentable 来变成可以被评论的。
然后这个 commentable 可以通过 comments 表里的 commentable_id 和 commentable_type 来定位。如：

```rb
comment.commentable # 将得到一个实例，比如 photo
```

创建评论的话，用：

```rb
# has_many
phote.comments.create

# has_one 的情况
phote.create_comment
```

> 一次 belongs_to，多次被 has_many. 值！

这里也可以理解为Comment通过：

```rb
belongs_to :commentable, :polymorphic => true
```

创建了commentable父类，下面的Article、Phote和Event里评论就是commentable的子类：

ArticleComment、PhoteComment和EventComment，只不过它们都存在comments这张表里，靠表里的
commentable_type属性来区分，同时通过commentable_id来记录是哪个article、phote或event的
comment。如果是article的，那么commentable_type就是"atricle"，commentable_id就是article
的id。

comments 的 controller 如下：

```ruby
def index
  @commentable = find_commentable
  @comments = @commentable.comments
end

def create
  @commentable = find_commentable
  @comment = @commentable.comments.build(params[:comment])
  if @comment.save
    flash[:notice] = "Successfully created comment."
    redirect_to :id => nil
  else
    render :action => 'new'
  end
end

private

def find_commentable
  params.each do |name, value|  # 查找表单里的内容
    if name =~ /(.+)_id$/ # 比如是，photo_id
      return $1.classify.constantize.find(value) # value 就是phote的id
    end
  end
  nil
end
```

这里的 constantize 方法能获得字符串的常量，即在变成环境里能用的变量。

## Mixin —— 混入
其实上面的多态，可以理解为一种混入方式。只不过这次需要存入数据库里。

在 Rails 里，一般的混入是在 models/concerns 里定义的 module，并 extend ActiveSupport::Concern，
后者让 module 更规范化，更容易被 extended。

对，混入有一个好记的特征 —— 名字以 able 结尾。表示有了它就有了某种能力。

混入是一种多父类继承的一种实现方式，对于类里共同的功能进行抽象。


# 数据验证

# 参考

[1]: http://railscasts.com/episodes/154-polymorphic-association
[2]: http://guides.rubyonrails.org/association_basics.html#polymorphic-associations

- http://api.rubyonrails.org/classes/ActiveRecord/Associations/ClassMethods.html
- https://tomafro.net/2009/08/using-indexes-in-rails-index-your-associations