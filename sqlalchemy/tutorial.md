---
categories: orm, python, sqlalchemy, tutorial, sql, database,
...

这个入门指导用于SQLAlchemy的快速入门，并便利SQLAlchemy的简单功能。如果你可以跳过这一部分进入 [[http://gashero.yeax.com/wp-admin/metadata.myt.html|主文档]] 会涉及更多内容。如下的例子全部是在Python交互模式下完成了，并且全部通过了 [[http://www.python.org/doc/lib/module-doctest.html|doctest]] 测试。

对应版本:   0.3.4

# 1 安装
## 1.1 安装SQLAlchemy
从 [[http://peak.telecommunity.com/DevCenter/setuptools|setuptools]] 安装是非常简单的，只要运行如下命令即可:

```
# easy_install SQLAlchemy
```
这将会在Python Cheese Shop获取SQLAlchemy的最新版本并安装。或者你也可以使用setup.py安装一个发行包:

```
# python setup.py install
```
## 1.2 安装一个数据库API
SQLAlchemy被设计用于操作一个 [[http://www.python.org/doc/peps/pep-0249/|DBAPI]] 实现，包括大多数常见的数据库。如果你有一个支持DBAPI的实现，那么可以跳入下一节。另外SQLite是一个易于使用的数据库，可以快速开始，并且他可以使用内存数据库。如果要使用SQLite，你将会需要：

 * [[http://pysqlite.org/|pysqlite]] - SQLite的Python接口
 * SQLite函数库

注意在Windows下并不需要SQLite函数库，因为Windows版的pysqlite已经内含了。pysqlite和SQLite可以被安装到Linux或FreeBSD，通过预编译或从源码安装。

预编译包的地址为：

http://initd.org/tracker/pysqlite/wiki/PysqlitePackages

# 2 快速开始
## 2.1 导入
SQLAlchemy提供了完整的命名空间，只要导入sqlalchemy即可，无需其子包。为了方便使用本教程，我们导入所有命名到本地命名空间:

```
>>> from sqlalchemy import *
```
## 2.2 连接到数据库
导入之后，下一步是连接到需要的数据库，表现为(represent)为一个Engine对象。这个对象处理了连接的管理和特定数据库的操作。下面，我们连接SQLite基于文件的数据库”tutorial.db”

```
>>> db=create_engine("sqlite:///tutorial.db")
```
创建数据库引擎的更多信息查看”Database Engines”。

# 3 SQLAlchemy是两个库的包装
现在已经完成了安装和连接数据库，可以开始做点实际的事情了。但首先需要有些解释。

SQLAlchemy的核心有两个完全不同的功能，一个在另一个之上工作。一个是'''SQL语言构造器''' ，另一个是 '''ORM''' 。SQL语言构造器允许调用 ```ClauseElements```来构造SQL表达式。这些 ```ClauseElements``` 可以在编译成字符串并绑定到数据库后用于执行，并返回一个叫做 '''ResultProxy''' 的对象，类似于一个结果集对象，但是更象dbapi高版本的 ```cursor``` 对象。

ORM是建立在SQL语言构造器之上的工具集，用于将Python对象映射到数据库的行，提供了一系列接口用于从数据库中存取对象(行)。在ORM工作时，在底层调用SQL语言构造器的API，这些通用的操作有些许的不同。不同的是，你不再使用行，而是使用自定义类的对象来操作。另外，数据库的查询方式也不同，ORM的可以生成大多数的SQL查询，除此之外还可以在类中定义更多操作。

SA功能强大，无与伦比，只是有两个混合在一起的方法有些复杂。有效的使用SA的方法是先了解这两种不同的工具集，这是两个不同的概念，而大家常常混交SQL语言构造器和ORM。关键的不同是，使用cursor形式的结果集时使用的是SQL语言构造器；而使用类实例进行管理时使用的是ORM。

本指南首先介绍SQL语言构造器，首先需要声明的数据库信息叫做 ```table metadata``` 。本指南包含了一些SQL构造的例子，包括如何有效的使用SQL语言构造器的例子。

# 4 操作数据库对象
在SQLAlchemy的核心哲学中表格和类是不同的。因为如此，SQLAlchemy提供了构造表格的方法(使用表格的元信息table metadata)。所以我们从构造表格的元信息对象和定制操作他们的对象开始。稍后我们可以看到SQLAlchemy的ORM，提供了表格元信息的高层封装，允许我们随心所欲的装载和保存Python类。

## 4.1 定义元信息，绑定到引擎
首先，你的表格必须已经在MetaData集合中。我们将要创建简单(handy)表格的MetaData，并自动连接到引擎(将一个模式(schema)对象连接到引擎成为绑定binding):

```
>>> metadata=BoundMetaData(db)
```
一个构造BoundMetaData对象的等同方法是直接使用引擎URL，这将会帮我们调用 ```create_engine```

```
>>> metadata=BoundMetaData("sqlite:///tutorial.db")
```
现在，我们告知metadata关于数据库中的表格，我们可以使用(issue)CREATE语句来创建表格，并且通过他们来创建和执行SQL语句，除非需要打开和关闭任何连接。这都是自动完成的。注意这个功能是推荐使用的。SQLAlchemy包含了使用模式进行连接管理和SQL构造的全部功能，并可以在任何引擎上进行操作。

本教程的目的，是教会大家使用”bound”对象，他可以使得代码简单和易读。

## 4.2 创建表格
使用metadata作为基本连接，我们可以创建表格:

```
>>> users_table=Table('user',metadata,
...     Column('user_id',Integer,primary_key=True),
...     Column('user_name',String(40)),
...     Column('password',String(10))
... )
```
有如你看到的，我们刚刚定义了一个叫做users的表格并拥有3个列：user_id作为主键，user_name和password。它现在只是一个对象而与数据库中的表格没有必然联系。为了让表格生效，我们使用create()方法。有趣的是，我们可以让SQLAlchemy发送SQL语句到数据库时同时显示SQL语句，只要设置BoundMetaData关联的Engine的echo选项即可:

```
>>> metadata.engine.echo=True
>>> users_table.create()
CREATE TABLE users (
    user_id INTEGER NOT NULL,
    user_name VARCHAR(40),
    password VARCHAR(10),
    PRIMARY KEY (user_id)
)
...
```
或者，如果users表格已经存在了(比如你第二次运行这些例子)，在这种情况下你可以跳过create()方法的调用。你设置可以跳过列定义，而是让SQLAlchemy自动从数据库装入定义:

```
>>> users_table=Table('users',metadata,autoload=True)
>>> list(users_table.columns)[0].name
'user_id'
```
关于表格元信息的文档在”Database Meda Data”中。

## 4.3 插入记录
插入记录是通过表格对象的insert()方法实现的，这将会定义一个子句对象(clause object)(就是CluseElement)来代理INSERT语句:

```
>>> i=users_table.insert()
>>> i
<sqlalchemy.sql._Insert object at 0x...>
>>> print i
INSERT INTO users (user_id,user_name,password) VALUES (?,?,?)
```
当我们创建这个插入语句对象时，语句本身也绑定到了Engine，并已经可以执行了。子句对象的execute()方法将会将对象编译为特定引擎的SQL方言，并且执行语句:

```
>>> i.execute(user_name='Mary',password='secure')
INSERT INTO users (user_name,password) VALUES (?,?)
['Mary','secure']
COMMIT
<sqlalchemy.engine.base.ResultProxy object at 0x...>
>>> i.execute({'user_name':'Tom'},{'user_name':'Fred'},{'user_name':'Harry'})
INSERT INTO users (user_name) VALUES (?)
[['Tom'],['Fred'],['Harry']]
COMMIT
<sqlalchemy.engine.base.ResultProxy object at 0x...>
```
注意VALUES子句会自动调整参数数量。这是因为ClauseElement的编译步骤并不依赖于特定的数据库，执行的参数也是如此。

当构造子句对象时，SQLAlchemy会绑定所有的值到参数。在构造时，参数绑定总是依靠键值对。在编译时，SQLAlchemy会转换他们到适当的格式，基于DBAPI的参数风格。这在DBAPI中描述的参数位置绑定中同样工作的很好。

这些文档继承于”Inserts”。

## 4.4 查询
我们可以检查users表中已经存在的数据。方法同插入的例子，只是你需要调用表格的select()方法:

```
>>> s=users_table.select()
>>> print s
SELECT users.user_id,users.user_name,users.password
FROM users
>>> r=s.execute()
SELECT users.user_id,users.user_name,users.password
FROM users
[]
```
这时，我们并没有忽略execute()的返回值。他是一个ResultProxy实例，保存了结果，而行为非常类似于DBAPI中的cursor对象:

```
>>> r
<sqlalchemy.engine.base.ResultProxy object at 0x...>
>>> r.fetchone()
(1,u'Mary',u'secure')
>>> r.fetchall()
[(2,u'Tom',None),(3,u'Fred',None),(4,u'Harry',None)]
```
查询条件同Python表达式，使用Column对象。所有表达式中的Column对象都是ClauseElements的实例，例如Select、Insert和Table对象本身:

```
>>> r=users_table.select(users_table.c.user_name=='Harry').execute()
SELECT users.user_id,users.user_name,users.password
FROM users
WHERE users.user_name=?
['Harry']
>>> row=r.fetchone()
>>> print row
(4,u'Harry',None)
```
所幸的是所有标准SQL操作都可以用Python表达式来构造，包括连接(join)、排序(order)、分组(group)、函数(function)、子查询(correlated subquery)、联合(union)等等。关于查询的文档”Simple Select”。

## 4.5 操作记录
你可以看到，当我们打印记录时返回的可执行对象，它以元组打印记录。这些记录实际上同时支持列表(list)和字典(dict)接口。字典接口允许通过字符串的列名定位字段，或者通过Column对象:

```
>>> row.keys()
['user_id','user_name','password']
>>> row['user_id'],row[1],row[users_table.c.password]
(4,u'Harry',None)
```
通过Column对象来定位是很方便的，因为这样避免了使用列名的方式。

结果集也是支持序列操作的。但是相对于select还有微小的差别，就是允许指定所选的列:

```
>>> for row in select([user_table.c.user_id,users_table.c.user_name]).execute():
...     print row
SELECT users.user_id,users.user_name
FROM users
[]
(1,u'Mary')
... ...
```
## 4.6 表间关系
我们可以创建第二个表格，email_addresses，这会引用users表。定义表间的关联，使用ForeignKey构造。我们将来也会考虑使用表格的CREATE语句:

```
>>> email_addresses_table=Table('email_addresses',metadata,
...     Column('address_id',Integer,primary_key=True),
...     Column('email_address',String(100),nullable=False),
...     Column('user_id',Integer,ForeignKey('users.user_id')))
>>> email_addresses_table.create()
CREATE TABLE email_addresses (
    address_id INTEGER NOT NULL,
    email_address VARCHAR(100) NOT NULL,
    user_id INTEGER,
    PRIMARY KEY (address_id),
    FOREIGN KEY(user_id) REFERENCES users (user_id)
)
...
```
上面的email_addresses表与表users通过ForeignKey(’users.user_id’)相联系。ForeignKey的构造器需要一个Column对象，或一个字符串代表表明和列名。当使用了字符串参数时，引用的表必须已经存在于相同的MetaData对象中，当然，可以是另外一个表。

下面可以尝试插入数据:

```
>>> email_addresses_table.insert().execute(
...     {'email_address':'tom@tom.com','user_id':2},
...     {'email_address':'mary@mary.com','user_id':1})
INSERT INTO email_addresses (email_address,user_id) VALUES (?,?)
[['tom@tom.com',2],['mary@mary.com',1]]
COMMIT
<sqlalchemy.engine.base.ResultProxy object at 0x...>
```
在两个表之间，我们可以使用 ```join``` 方法构造一个连接:

```
>>> r=users_table.join(email_addresses_table).select().execute()
SELECT users.user_id, users.user_name, users.password,
email_addresses.address_id, email_addresses.email_address,
email_addresses.user_id
FROM users JOIN email_addresses ON users.user_id=email_addresses.user_id
[]
>>> print [row for row in r]
[(1, u'Mary', u'secure', 2, u'mary@mary.com', 1),
(2,u'Tom', None, 1, u'tom@tom.com', 2)]
```
```join``` 方法同时也是 ```sqlalchemy``` 命名空间中的一个独立的函数。连接条件指明了表对象给定的外键。条件(condition)，也可以叫做 “ON 子句” ，可以被明确的指定，例如这个例子我们查询所有使用邮件地址作为密码的用户:

```
>>> print join(users_table, email_addresses_table,
...     and_(users_table.c.user_id==email_addresses_table.c.user_id,
...     users_table.c.password==email_addresses_table.c.email_address)
...     )
users JOIN email_addresses ON users.user_id=email_addresses.user_id AND
users.password=email_address.email_address
```
# 5 使用ORM工作
现在我们已经有了一些表格和SQL操作的知识了，让我们来看看SQLAlchemy的ORM (Object Relational Mapper) 。使用ORM，你可以将表格(和其他可以查询的对象)同Python联系起来，放入映射集(Mappers)当中。然后你可以执行查询并返回 ''对象实例'' 列表，而不是结果集。 ''对象实例'' 也被联系到一个叫做''Session'' 的对象，确保自动跟踪对象的改变，并可以使用 ''flush'' 立即保存结果。

## 5.1 创建一个映射
一个映射通常对应一个Python类，其核心意图是，“这个类的对象是用作这个表格的行来存储的”。让我们创建一个类叫做 ```User``` ，描述了一个用户对象，并保存到 ```users``` 表格。:

```
>>> class User(object):
...     def __repr__(self):
...         return "%s(%r,%r)"%(
...             self.__class__.__name__,self.user_name,self.password)
```
这个类是一个新形式(new style)的类(继承自 '''object''' )并且不需要构造器(在需要时默认提供)。我们只实现了一个 ```__repr__``` 方法，用于显示 ```User``` 对象的基本信息。注意 ```__repr__``` 方法应用了实例变量 ```user_name``` 和 ```password``` ，这是还没定义的。我们可选的定义这些属性，并可以进行处理；SQLAlchemy的 '''Mapper'''的构造器会自动管理这些，而且会自动协调到 ```users``` 表格的列名。让我们创建映射，并观察这些属性的定义:

```
>>> usermapper=mapper(User,users_table)
>>> ul=User()
>>> print ul.user_name
None
>>> print ul.password
None
```
函数 ```mapper``` 返回新建的 ```Mapper``` 实例。这也是我们为 ```User``` 类创建的第一个映射，也就是类的 '''主映射''' 。一般来说，不需要保存 ```usermapper``` 变量；SA的ORM会自动管理这个映射。

## 5.2 获取会话(Session)
创建了一个映射之后，所有对映射的操作都需要一个重要的对象叫做 ```Session```。所有对象通过映射的载入和保存都 ''必须'' 通过 ```Session``` 对象，有如对象的工作空间一样被加载到内存。特定对象在特定时间只能关联到一个 ```Session``` 。

缺省时，需要在载入和保存对象之前明确的创建 ```Session``` 对象。有多种方法来管理会话，但最简明的方法是调用 ```create_session()```

```
>>> session=create_session()
>>> session
<sqlalchemy.orm.session.Session object at 0x...>
```
## 5.3 查询对象
会话对象拥有载入和存储对象的所有方法，同时也可以查看他们的状态。会话还提供了查询数据库的方便接口，你可以获取一个 '''Query''' 对象:

```
>>> query=session.query(User)
>>> print query.select_by(user_name='Harry')
SELECT users.user_name AS users_user_name, users.password AS users_password,
users.user_id AS users_user_id
FROM users
WHERE users.user_name=? ORDER BY users.oid
['Harry']
[User(u'Harry',None)]
```
对象所有的查询操作实际上都是通过 '''Query''' 的。 '''Mapper''' 对象的多种 ```select```方法也是偷偷的在使用 '''Query''' 对象来执行操作。一个 '''Query''' 总是联系到一个特定的会话上。

让我们暂时关闭数据库回显，并尝试 '''Query''' 的几个方法。结尾是 ```_by``` 的方法主要用于对象的键参数。其他的方法允许接受 '''ClauseElement''' 对象，使用'''Column''' 对象的Python表达式产生，同样的方法我们在上一节使用过。使用'''ClauseElement''' 结构来查询更加冗长，但是更加灵活:

```
>>> metadata.engine.echo=False
>>> print query.select(User.c.user_id==3)
[User(u'Fred',None)]
>>> print query.get(2)
User(u'Tom',None)
>>> print query.get_by(user_name='Mary')
User(u'Mary',u'secure')
>>> print query.selectfirst(User.c.password==None)
User(u'Tom',None)
>>> print query.count()
4
```
Note

User类有一个特别的属性 '''c''' ，这个属性描述了User映射表格对象的列。

```User.c.user_name``` 等同于 ```users_table.c.user_name``` ，记得 ```User``` 是Python对象，而 ```users``` 是 '''Table''' 对象。

## 5.4 修改数据
作为小经验，我们看看如何做出修改。首先，创建一个新的用户”Ed”，然后加入会话:

```
>>> ed=User()
>>> ed.user_name='Ed'
>>> ed.password='edspassword'
>>> session.save(ed)
>>> ed in session
True
```
也可以修改数据库中的其他对象。使用 '''Query''' 对象载入，然后改变:

```
>>> mary=query.get_by(user_name='Mary')
>>> harry=query.get_by(user_name='Harry')
>>> mary.password='marysnewpassword'
>>> harry.password='harrysnewpassword'
```
这时，什么东西都没有保存到数据库；我们所有的修改都在内存中。如果这时程序其他部分尝试修改’Mary’会发生什么呢？因为使用相同的会话，所以再次载入’Mary’实际上定位到相同的主键’Mary’，并且 '''返回同一对象实例''' 。这个行为用在会话中确保数据库的一致性:

```
>>> mary2=query.get_by(user_name='Mary')
>>> mary is mary2
True
```
在唯一映射中，单一的会话可以确保安全的操作对象。

如果两个不同的会话同时操作一个对象，会检测到并发；SA会使用简单的并发控制来保存对象，可以选择使用拥有更强的使用ids的检查。参考 [[http://www.sqlalchemy.org/docs/adv_datamapping.myt#advdatamapping_arguments|Mapper Arguments]] 了解更多细节。

## 5.5 保存
在新建了用户”ed”并对”Mary”和”Harry”作修改之后，我们再删除”Fred”

```
>>> fred=query.get_by(user_name='Fred')
>>> session.delete(fred)
```
然后发送更改到数据库，使用会话的 ```flush()``` 方法。开启回显来查看过程:

```
>>> metadata.engine.echo=True
>>> session.flush()
BEGIN
UPDATE users SET password=? WHERE users.user_id=?
['marysnewpassword',1]
UPDATE users SET password=? WHERE users.user_id=?
['harrysnewpassword',4]
INSERT INTO users (user_name,password) VALUES (?,?)
['Ed','edspassword']
DELETE FROM users WHERE users.user_id=?
[3]
COMMIT
```
## 5.6 关系
如果一个关系包含其他信息时，例如包含邮件地址的列表，我们可以在使用```relation()``` 创建 '''Mapper''' 时声明。当然，你还可以对这个关系作很多事情，我们举几个简单的例子。首先使用 ```users``` 表，它拥有一个外键关系连接到```email_addresses``` 表。 ```email_addresses``` 表中的每一行都有列 ```user_id``` 用来引用```users``` 表中的一行；而且 ```email_addresses``` 表中的多行可以引用 ```users``` 表中的同一行，这叫一对多关系。

首先，处理 ```email_addresses``` 表。我们创建一个新的类 '''Address''' 描述了```email_addresses``` 表中的一行，并且也创建了用于 ```Address``` 类的映射对象:

```
>>> class Address(object):
...     def __init__(self,email_address):
...         self.email_address=email_address
...     def __repr__(self):
...         return "%s(%r)"%(
...             self.__class__.__name__,self.email_address)
>>> mapper(Address, email_addresses_table)
<sqlalchemy.orm.mapper.Mapper object at 0x...>
```
然后，我们通过使用 ```relation()``` 创建一个关系连接 ```User``` 和 ```Address``` 类，并且添加关系到 ```User``` 映射，使用 ```add_property``` 函数:

```
>>> usermapper.add_property('addresses',relation(Address))
```
函数 ```relation()``` 需要一个类或映射作为首参数，并且还有很多选项来控制行为。 ```User``` 映射现在给每一个 ```User``` 实例添加了一个属性叫 ```addresses``` 。SA将会自动检测这个一对多关系。并且随后创建 ```addresses``` 列表。当新的 ```User``` 对象创建时，这个列表为空 。

让我们看看数据库做了什么。当我们修改映射的配置时，最好清理一下会话，让所有载入的 ```User``` 对象可以重新载入:

```
>>> session.clear()
```
我们随之可以使用 ```User``` 对象的 ```addresses``` 属性来象列表一样处理:

```
>>> mary=query.get_by(user_name='Mary')
SELECT users.user_name AS users_user_name, users.password AS users_password,
users.user_id AS users_user_id
FROM users
WHERE users.user_name=? ORDER BY users.oid
LIMIT 1 OFFSET 0
['Mary']
>>> print [a for a in mary.address]
SELECT email_addresses.user_id AS email_address_user_id,
email_addresses.address_id AS email_addresses_address_id,
email_addresses.email_address AS email_addresses_email_address
FROM email_addresses
WHERE ?# email_addresses.user_id ORDER BY email_addresses.oid
[1]
[Address(u'mary@mary.com')]
```
向列表添加元素也很简单。新的 ```Address``` 对象将会在调用会话的flush()时保存:

```
>>> mary.addresses.append(Address('mary2@gmail.com'))
>>> session.flush()
BEGIN
INSERT INTO email_addresses (email_address,user_id) VALUEs (?,?)
['mary2@gmail.com',1]
COMMIT
```
主文档中关于使用映射的部分在如下地址:

http://www.sqlalchemy.org/docs/datamapping.myt#datamapping

## 5.7 事务
你可能已经注意到在上面例子中的 ```session.flush()``` ，SQLAlchemy使用 ```BEGIN```和 ```COMMIT``` 来使用数据库事务。 ```flush()``` 方法使用事务来对一些记录执行一系列的指令。如果希望在 ```flush()``` 之外使用更大规模的事务，可以通过'''SessionTransaction''' 对象，其生成使用 ```session.create_transaction()``` 。下面将会执行一个非常复杂的 ```SELECT``` 语句，进行大量的修改，然后再创建一个有两个邮箱的用户，这些都在事务中完成。而且将会在中间使用 ```flush()``` 来保存，然后在执行最后的 ```commit()``` 时将所有改变写入数据库。我们把事务封装在一个 try/except 语句块当中确保资源的安全释放:

```
>>> transaction=session.create_transaction()
>>> try:
...     (ed,harry,mary)=session.query(User).select(
...         User.c.user_name.in_('Ed','Harry','Mary'),
...         order_by=User.c.user_name
...     )
...     del mary.address[1]
...     harry.addresses.append(Address('harry2@gmail.com'))
...     session.flush()
...     print "***flushed the session***"
...     fred=User()
...     fred.user_name='fred_again'
...     fred.addresses.append(Address('fred@fred.com'))
...     fred.addresses.append(Address('fredsnewemail@fred.com'))
...     session.save(fred)
...     transaction.commit()
... except:
...     transaction.rollback()
...     raise
BEGIN
SELECT users.user_name AS users_user_name,
users.password AS users_password,
users.user_id AS users_user_id
FROM users
WHERE users.user_name IN (?, ?, ?) ORDER BY users.user_name
['Ed', 'Harry', 'Mary']
SELECT email_addresses.user_id AS email_addresses_user_id,
email_addresses.address_id AS email_addresses_address_id,
email_addresses.email_address AS email_addresses_email_address
FROM email_addresses
WHERE ? email_addresses.user_id ORDER BY email_addresses.oid
[4]
UPDATE email_addresses SET user_id=? WHERE email_addresses.address_id ?
[None, 3]
INSERT INTO email_addresses (email_address, user_id) VALUES (?, ?)
['harry2@gmail.com', 4]
***flushed the session***
INSERT INTO users (user_name, password) VALUES (?, ?)
['fred_again', None]
INSERT INTO email_addresses (email_address, user_id) VALUES (?, ?)
['fred@fred.com', 6]
INSERT INTO email_addresses (email_address, user_id) VALUES (?, ?)
['fredsnewemail@fred.com', 6]
COMMIT
```
对应的主文档：


# 来源
http://gashero.yeax.com/?p=6

