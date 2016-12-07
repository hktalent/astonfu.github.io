---
layout: post
title: Rails 里的 Controller 和 Router
---

Router定义了外部访问时URL的样子，通过匹配筛选后，交给对应的Controller处理。

玩转了它们，就可以构造自己想要的url，并对相应的请求指定 Controller 进行处理。

其实，一个 Controller 不一定需要 Model， 因为它只是处理 Route 传来的请求，它也不需要 View，因为
它可以 render 任何 views 目录的模板。

Rails 是很灵活的，它里面的 MVC 可以单独活动，只是秉着“惯例胜过配制”的原则：

> CONVENTIONS OVER CONFIGURATIONS

默认了很多根据名字的方便，比如 PostsController 的 view 就在 view/posts 目录里，而它有个
Model 是 Post，数据库建的表是 posts。然后在 config/routes.rb 里：

```rb
resources :posts
```

就会建好指向 PostController 的 CURD 操作，并定义好 RESTful 的 url。

是的，很方便，也符合信息论理论，用短的编码实现常用的操作。非常用的就用参数配制嘛！

# Router

## Resourceful Routes

就是我们常见的，Rails 默认的规则。

## Non-Resourceful Routes

而往往我们构造的页面，不会像脚手架生成的那样死板。

### 定义 url

- :controller 对应app里的一个Controller
- :action 对应Controller里的Action
- :其他 可以自定义，会传到params里
- () 表示这是可选的

比如：

```rb
get ':controller(/:action(/:id))'
```

- 路径： /photos/show/1?user_id=2
- 得到： params = { controller: 'photos', action: 'show', id: '1', user_id: '2' }

```rb
get ':controller/:action/:id/:user_id'
```

- 路径： /photos/show/1/2
- 得到： params = { controller: 'photos', action: 'show', id: '1', user_id: '2' }

```rb
get ':controller/:action/:id/with_user/:user_id'
```

- 路径： /photos/show/1/with_user/2
- 得到： params = { controller: 'photos', action: 'show', id: '1', user_id: '2' }

### 改变 url 的样子 或对应的 Controller

```rb
# 关于 photes 的处理，就会交给 ImagesController 来处理，但是 path helper 还是 photos。
resources :photos, controller: 'images'

# 改变 path helper 的名字
resources :photos, as: 'images'

# 那么这个意思就是 —— 只改变 url 为 /photos/xxxx
# Controller 还是 ImagesController，helper 也是 images_path
resources :photos, controller: 'images', as: 'images'

# 上面情况的相反，url 是 /images/xxxx，其他是 photos
resources :photos, path: 'images'

# 所以上上可以写作
resources :images, path: 'photos'

# 这么表示 /:id 直接传到 PhotosController
resources :photos, path: ''

# 只有符合条件的 id 才能传给 Controller
resources :photos, constraints: { id: /[A-Z][A-Z][0-9]+/ }

# 覆盖 Actions
resources :photos, path_names: { new: 'make', edit: 'change' }

# 默认就是用 name 代替 id，比如 show action 就成了 、photos/:name
resources :photos, param: :name
# 当然相应要 通知 model，咱不用 id 了，用 name 了。就是需要覆盖 to_param 方法
class Photo < ApplicationRecode
  def to_param
    name
  end
end

```

namespace 和 scope，它们都是给 url 加给前缀，而 namespace 也给 helper 加个前缀，
scope就只是给 url 加前缀。


# Controller
Controller 根据 Router 传来的请求，操作 Model，并生成 View。

但这里注意了，生成的 View 是由模板和 Model 一起生成的。

传到 Router 的请求，也是用户在 View 上点击传来的。

因此，这是一个蛇咬尾巴的故事。

## ActionController::Parameters

为了安全，一般不在Controller里直接使用params这个变量，而是通过permit筛选一遍后只提取许可的参数。
从而保证入口参数可控可靠。

而且只有permitted的参数才可以进行mass assign，防止改变不想改变的数据，比如is_admin.

- require 提取一个或多个key的值
- permit 授权一个多个参数

```rb
params = ActionController::Parameters.new(person: { name: 'Francesco' })

# require后还是一个Parameters，并且没有permitted，只是将person提取出来。
params.require(:person)
=> <ActionController::Parameters {"name"=>"Francesco"} permitted: false>

# 现在params又嵌了一层
params
=> <ActionController::Parameters {"person"=><ActionController::Parameters {"name"=>"Francesco"} permitted: false>} permitted: false>

# 只有使用permit后，才能permitted，然后可以使用
person_params = params.require(:person).permit(:name)
=> <ActionController::Parameters {"name"=>"Francesco"} permitted: true>

person_params[:name]
=> "Francesco"

# 或一开始就只用permit，得到一个hash参数
permitted_params = params.permit(person:[:name])
=> <ActionController::Parameters {"person"=><ActionController::Parameters {"name"=>"Francesco"} permitted: true>} permitted: true>

permitted_params[:person][:name]
=> "Francesco"
```

# 参考
- http://guides.rubyonrails.org/routing.html
- http://api.rubyonrails.org/classes/ActionController/Parameters.html
- http://www.nascenia.com/ruby-on-rails-development-principles/
- http://stackoverflow.com/questions/2837102/changing-the-id-parameter-in-rails-routing