---
layout: post
title: "[翻译] Railscasts：#238 Mongoid (revised)"
date: 2014-02-03 15:58:52 +0800
comments: true
categories: ["Rails", "Railscasts", "译作"]
---

翻译自Railscast上的#238 Mongoid (revised)。由于此文为收费的episode，请自觉像原创者Ryan Bates致意，并付费。原文地址：[#238 Mongoid (revised)](http://railscasts.com/episodes/238-mongoid-revised?view=asciicast, "#238 Mongoid (revised)")

{% blockquote Ryan Bates, http://railscasts.com/episodes/238-mongoid-revised?view=asciicast, Ruby on Rails Screencasts %}
<p class="no-indent">Mongoid is a Ruby gem for interacting with MongoDB. Here I show the basics of setting up an app, querying for records, adding embedded associations, overriding the id, and more.</p>
{% endblockquote %}
<!-- More -->
如果你正在为你的Rails应用寻找一个NoSQL的数据库可选方案，那么Mongoid是一个绝佳的选择。最近发布的第三版更是比以前所有的发布都棒。在这个视频中，我们会讲解它在创建一个新的Rails应用时的基本用法。

# 安装MongoDB和Mongoid

首先我们需要安装MongoDB数据库。它的官方网站有在许多平台上的安装指南，然而如果你想要用最简单的方式在Mac OS X上跑起来的话，可以用Homebrew3来安装。我们可以用下面的命令来安装：

{% codeblock terminal lang:bash %}
$ brew install mongodb
{% endcodeblock %}

这样就会下载和安装MongoDB了。完成以后，屏幕会提示一些指南说明如何启动MongoDB。我们可以通过下列的命令还手动启动数据库。

{% codeblock terminal lang:bash %}
$ mongod run --config /usr/local/etc/mongod.conf
{% endcodeblock %}

我们可以通过浏览 http://localhost:28017 来测试数据库已经启动且正常运行。在这个地址会应该可以看到MongoDB的web交互界面。

# 用Mongoid创建一个rails应用

既然我们已经有了一个运行着的MongoDB，我们将创建一个名叫store的Rails盈盈。因为我们希望使用Mongoid而不是ActiveRecord，我们将使用--skip-active-record的选项。这样rails在新建应用时就不会载入相关的数据库配置的框架和生成文件，例如数据库配置文件。

{% codeblock terminal lang:bash %}
$ rails new store --skip-active-record
{% endcodeblock %}

接下来，我们在Gemfile中增加Mongoid的gem包。为了确定我们运行的是最新的版本，我们会具体指定版本号。像平常一样，需要在修改之后执行bundle安装Mongoid。

{% codeblock /Gemfile lang:ruby %}
gem 'mongoid', '~>3.0.4'
{% endcodeblock %}

这个gem包提供了一个generator，可以用来生成一个配置文件。

{% codeblock terminal lang:bash %}
$ rails g mongoid:config
{% endcodeblock %} 

我们将保持所有的默认配置选项，但是这个文件中有很多很好的注释是值得通读一遍以了解有什么设置可以做。接着，我们生成一个scaffolding做一个有“名称”和“价格”的“产品”的模型。注意，在MongoDB中正确的“价格”的类型是big_decimal。

{% codeblock terminal lang:bash %}
$ rails g scaffold product name price:big_decimal
{% endcodeblock %}

Mongoid覆写了模型生成器，这个定制的生成器会在我们生成模型时被调用。看一下这个模型文件，我们可以看到Mongoid的模型是什么样子的。这是一个简单的Ruby类引用了（include）Mongoid::Document模块同时调用field方法实现我们在模型里需要的各个参数。

{% codeblock /app/models/product.rb lang:ruby %}
class Product
  include Mongoid::Document
  field :name, type:String
  field :price, type:BigDecimal
end
{% endcodeblock %}

如果我们访问下我们的Rails应用就会发现scaffolding已经被生成并且正常工作了。我们能够创建新的“产品”并保存到数据库。不再需要运行任何migrations和新建、配置一个数据库。

<p class="no-indent">{% img /images/20140203/pic01.png %}</p>

# 修改一个模型

使用无语定义模式（schemaless）的一个好处在于它的灵活性。如果我们希望能够记录每个“产品”是何时发布的，我们只需要增加另一个field方法的调用。

{% codeblock /app/models/product.rb lang:ruby %}
class Product
  include Mongoid::Document
  field :name, type:String
  field :price, type:BigDecimal
  field :released_on, type:Date
end
{% endcodeblock %}

要看到这个“released_on”字段，我们需要把它加入到_form.html.erb文件中。

{% codeblock /app/views/products/_form.html.erb lang:css+erb %}
<div class="field">
  <%= f.label :released_on%>
  <br/>
  <%= f.text_field :released_on%>
</div>
{% endcodeblock %}

我们现在使用了一个输入框，然而我们试图使用一个“date_select”时，我们会碰到一个Mongoid的问题。我们需要引用一个模块到Product模型以确保date_select可以工作（文档中有详细的说明关于如何这样做。）现在，当我们编辑一个产品时，我们会有一个针对发布日期的新输入框。如果我们输入一个日期，它会被保存在数据库里。同样，不需要做任何数据迁移（migration）。

# 增加验证

Mongoid和我们所了解的ActiveRecord一样大量使用了ActiveModel模块。我们可以使用attr_accessible来定义字段（field）是否可以被调用，并且被验证，正如我们似乎在使用ActiveRecord一般。

{% codeblock /app/models/product.rb lang:ruby %}
class Product
  include Mongoid::Document
  field :name, type:String
  field :price, type:BigDecimal
  field :released_on, type:Date

  attr_accessible :name, :price, :released_on
  validates_presence_of :name
end
{% endcodeblock %}

如果我们现在更新一个product，且把姓名（name）字段留空，我们会看到一条错误信息。

<p class="no-indent">{% img /images/20140203/pic02.png %}</p>

# 用MongoDB查询

有一个Mongoid和ActiveRecord不同地方是“查询记录”。我们会在控制台（console）里演示这个不同。我们依然可以对模型类调用where方法并传递条件（conditions）给它。但是，Mongoid允许我们传递更复杂的条件。我们将用这种方式来查找成本低于40英镑的product。

{% codeblock console lang:ruby %}
>> Product.where(:price.lte => 40).first
=> #<Product _id: 502952b1a175752aa0000001, _type: nil, name: "Settlers of Catan", price: "34.99", released_on: 2012-01-01 00:00:00 UTC>
{% endcodeblock %}

我们可以直接在Product类上使用一个条件方法，写出一个同样效果的查询。

{% codeblock console lang:ruby %}
>> Product.lte(price:40).first  
=> #<Product _id: 502952b1a175752aa0000001, _type: nil, name: "Settlers of Catan", price: "34.99", released_on: 2012-01-01 00:00:00 UTC>
{% endcodeblock %}

查询调用返回的是一个Mongoid::Criteria类的实例对象。它和ActiveRecord::Relation对象非常相似。这就意味着我们可以使用链式查询的方式。

{% codeblock console lang:ruby %}
>> Product.lte(price:40).gt(released_on:1.month.ago)  
=> #<Mongoid::Criteriaselector: {"price"=>{"$lte"=>"40"}, "released_on"=>{"$gt"=>Fri, 13Jul201200:00:00UTC+00:00}},  
   options:  {},  
   class:    Product,   
   embedded: false>
{% endcodeblock %}

由于Mongoid的查询接口十分强大，我们还可以做更多工作。在Mongoid的文档中有更多详细内容。

# 关联

另一个Mongoid和ActiveRecord不同之处是“关联”（association）。我们可以将一个关联内嵌在同一个document中。为了演示这种关联如何工作，我们将生成一个新的模型“评论”（Review），它有个“内容”（content）的文本字段。

{% codeblock terminal lang:bash %}
$ rails g model review content
{% endcodeblock %}

现在，我们有了两个模型。我们希望每个“产品”（Product）有多个“评论”（Review）。可以用两种方法来定义这种关联。我们可以添加一个引用关联，就像在ActiveRecord里的那样。但是这里，我们采用内嵌的方式。要定义内嵌，我们可以使用embeds_many方法。

{% codeblock /app/models/product.rb lang:ruby %}
class Product
  include Mongoid::Document
  field :name, type:String
  field :price, type:BigDecimal
  field :released_on, type:Date

  attr_accessible :name, :price, :released_on
  validates_presence_of :name

  embeds_many :reviews
end
{% endcodeblock %}

在Review的模型里，我们可以调用embedded_in方法。

{% codeblock /app/models/review.rb lang:ruby %}
class Review
  include Mongoid::Document
  field :content, type:String

  embedded_in :product
end
{% endcodeblock %}

我们可以在控制台（console）里试验。操作的关联的方式和在ActiveRecord里面类似。

{% codeblock console lang:ruby %}
>> p = Product.first  
=> #<Product _id: 502952b1a175752aa0000001, _type: nil, name: "Settlers of Catan", price: "34.99", released_on: 2012-01-01 00:00:00 UTC>  
>> p.reviews.create! content:"great game!"  
=> #<Review _id: 5029656ca17575eaa1000001, _type: nil, content: "great game!">
{% endcodeblock %}

现在，我们可以通过一个“review”的“Product”来获取它自身，正如我们所预期的。因为这是个内嵌的关联，所有的“review”的数据都存在Product的document里，而不是在MongoDB里的独立的document。这也意味着调用方法Review.count将返回值0，因为其实并没有Review这个document。我们只能通过Product模型来获取它的评论。甚至，我们不能单独创建一个评论（Review）因其需要一个父（文档）Product。

无论何时，当我们在Mongoid中定义一个关联时，我们需要知道该被关联的记录是否需要在其父文档之外被单独使用。如果回答是的话，则需要建立一个has_many的关联，而非内嵌关联（embeded_in）。内嵌关联的优势在于所有的数据都在一个文档（document）内，因此我们的应用无须为获取其他关联的文档而进行不同的查询。

# 其他模块

接着，我们将向你展示一些可以与Mongoid配合使用的其他的模块（除了Mongoid::Document）。Mongoid::Timestamps正如其名所示与ActiveRecord提供类似的功能。如果我们将它加入Product模型，那么当我们创建或者更新一个产品（的实例）时那些timestamp的字段也将被更新。

{% codeblock console lang:ruby %}
>> Product.create! name:"Agricola"  
=> #<Product _id: 50296dc5a17575eaa1000002, _type: nil, created_at: 2012-08-13 21:12:37 UTC, updated_at: 2012-08-13 21:12:37 UTC, name: "Agricola", price: nil, released_on: nil>
{% endcodeblock %}

另一个有用的模块是Paranoia。这个模块会“软删除”记录，也就是说它可以删除一条记录而非将它从数据库中真实移除。如果我们在Product（模型）中引入Mongoid::Paranoia，那么删除一个产品仅仅会是设置一个deleted_at的属性。这个产品看上去已经被删除，但是我们可以通过调用restore的方法来把它找回来。Mongoid甚至还通过Mongoid::Versioning模块内建对版本化的支持。我们这里不展开，但是在它文档中有详细描述。

# 更加美观的URL

如果我们查看一个产品的URL，会看到如下图的这个样子，不像一个通常意义上的唯一标示符。

<p class="no-indent">{% img /images/20140203/pic03.png %}</p>

如果我们想要的话，其实是有一些方法可以把把它改得稍稍美观一些。一个选择类似于ActiveRecord，我们可以覆写to_param方法。但是这里，我们会展示另一种方式——通过覆写Mongoid的_id字段。

{% codeblock /app/models/product.rb lang:ruby %}
field :_id, type:String, default: -> { name.to_s.parameterize }
{% endcodeblock %}

这一步的关键在于给_id字段设定一个默认值。我们使用一个Proc对象来动态设定它为这个产品的名字，然后参数化（parameterized）这个产品名称以合规URL安全要求。现在我们创建一个新的产品，它的名字变成了它部分的URL了。

<p class="no-indent">{% img /images/20140203/pic04.png %}</p>

当我们使用（Mongoid）这个方法时，需要关注一些事情。其中之一是，无论何时我们创建一个新的Product实例时，我们总是需要同时设定它的名字。如果没有这样做，那么这个产品的id将会是一个空的字符串，而且，就算以后我们再试图设定它的名字时，id还是会是一个空字符串。同样地，我们需要确定id是唯一的，因此，在此种情境下，我们必须添加一些校验。

这就是我们这讲关于Mongoid的全部内容。这个项目的文档相当易懂，非常值得一看。而MongoDB本身也非常值得花些时间来研究学习。在（MongoDB的）网站上有一个交互式的命令行界面（Shell）和一个教程。通过它，我们可以更好地知道MongoDB是如何工作的。

