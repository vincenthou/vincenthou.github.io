# PHP编码规范
## 基本原则
- 简洁，清晰，一致[[BR]]
- 健壮[[BR]]
- 注意细节[[BR]]
- 第一次将事情做好[[BR]]
## 格式化
- 使用UTF-8作为文件编码格式
- 使用空格代替Tab
- 运算符两端分别留一个空格

``` php
//wrong
$foo=$bar;

//right
$foo = $bar;
```
- 使用空行分隔逻辑代码块

``` php
//get request parameters
$keyword =  Yii::app()->request->getQuery('keyword');
$pageSize =  Yii::app()->request->getQuery('pageSize');
Yii::log('request: keyword->' . $keyword . ' pageSize->' . $pageSize, 'debug', 'system.trip.getSystemTags');

//get data from backend
$curl = new Curl();
$result = $curl->get('/tag/list', array('keyword'=>urlencode($keyword), 'pageSize'=>$pageSize));
echo $result;
```
- 逻辑代码块哪怕只有一行也要用{}包裹起来，关键字与条件的左括号之间要空一格

``` php
while (...)
{
    bar();
}
```
- switch语句的每一个case都要以break结束，并且有一个default的声明（同样以break结束）

``` php
switch (condition)
{
    case 1:
    {
        action1;
    }
    break;
    case 2:
    {
        action A...;
        action B...;
    }
    break;
    default:
    {
        default action;
    }
    break;
}
```
- 流程控制的不同部分，新行要以关键字开始而不是“{”

``` php
//wrong
try
{
    ...
} catch (Exception $e)
{
    ...
}

//right
try
{
    ...
}
catch (Exception $e) {
    ...
}
```
- 函数名与参数列表之间不留空格，不同的参数在“,”之后留一个空格（同样适用于数组）

``` php
//wrong
function update_records ($table,$parameters,$conditions)
{
    ...
}

//right
function update_records($table, $parameters, $conditions)
{
    ...
}

//wrong
$some_array = array ('hello','world','foo'=>'bar');

//right
$some_array = array('hello', 'world', 'foo' => 'bar');
```
- 不依赖默认的操作符优先级，使用()明确表示出运算的优先级

``` php
//wrong
if($row["col_a"] == 3 && $row["col_b"] == 1)
{
 }

//right
if((3 == $row['col_a']) && (1 == $row['col_b']))
{
}
```
- 普通字符串使用单引号包裹，避免使用双引号

``` php
//wrong
$str = "Hello world";
$jointStr = "Hello world $str";

//right
$str = 'Hello world';
$jointStr = 'Hello world' . $str;
```
- 类中的方法和变量的访问权限使用public、protected、private关键字显示标识
- 进行类型转换时，使用括号将目标类型包裹起来，并与变量之间留一个空格

``` php
(int) $mynumber
```
- 过长的控制条件，差分成若干变量，计算出变量结果后再进行逻辑组合判断

``` php
// Key is only valid if it matches the current user's ID, as otherwise other
// users could access any user's things.
$is_valid_user = (isset($key) && !empty($user->uid) && $key == $user->uid);

// IP must match the cache to prevent session spoofing.
$is_valid_cache = (isset($user->cache) ? $user->cache == ip_address() : FALSE);

// Alternatively, if the request query parameter is in the future, then it
// is always valid, because the galaxy will implode and collapse anyway.
$is_valid_query = $is_valid_cache || (isset($value) && $value >= time());

if ($is_valid_user || $is_valid_query)
{
  ...
}
```
## 命名
- 使用NULL初始化空字符串
- 文件名和类名首字母大写，使用驼峰命名法 
  
  ```
  **Example:** Circle, FilledCircled, ShadedFilledCircle
  ```
- 全局方法使用小写字母命名使用“_”分隔 
  
  ```
  **Example:** is_valid(), get_script_name()
  ```
- 类方法使用驼峰命名法，动宾结构，首字母小写 
  
  ```
  **Example:** draw(), drawLine(), getName()
  ```
- 普通变量和类成员变量使用驼峰命名法，首字母小写，私有成员以_开头 
  
  ```
  **Example:** $currentTime，$_privateVal
  ```
- 常量全部使用大写字母命名，以“_”分隔 
  
  ```
  **Example:** PI, MAX_BUFFER_SIZE
  ```
## 注释
- 使用empty标识空执行

``` php
while (condition)
{
    //empty!
}
```
- 文件开头书写注释标识作者、网站主页、版权信息

``` php
/**
 * CBooleanValidator class file.
 *
 * @author Qiang Xue <qiang.xue@gmail.com>
 * @link http://www.yiiframework.com/
 * @copyright Copyright &copy; 2008-2011 Yii Software LLC
 * @license http://www.yiiframework.com/license/
 */
```
- 类定义上方书写注释阐述类的应用场景、作者、版本号、所属的包名称

``` php
/**
 * CDefaultValueValidator sets the attributes with the specified value.
 * It does not do validation. Its existence is mainly to allow
 * specifying attribute default values in a dynamic way.
 *
 * @author Qiang Xue <qiang.xue@gmail.com>
 * @version 0.1
 * @package system.validators
 */
```
- 类成员变量上方书写注释阐述变量的类型、意义、默认值等信息

``` php
/**
 * @var boolean whether to set the default value only when the attribute value is null or empty string.
 * Defaults to true. If false, the attribute will always be assigned with the default value,
 * even if it is already explicitly assigned a value.
 */
```
- 定义方法上方书写注释阐述方法的用途，使用@param详述参数的信息（包括参数类型以及意义），使用@return详述返回值的信息，如果有异常抛出使用@throws注解

``` php
/**
 * Returns the JavaScript needed for performing client-side validation.
 * @param CModel $object the data object being validated
 * @param string $attribute the name of the attribute to be validated.
 * @throws Exception
 * @return string the client-side validation script.
 * @see CActiveForm::enableClientValidation
 * @since 1.1.7
 */
```
- 对于实现较为复杂的代码，以空行分隔逻辑块的同时，使用单行注释解释逻辑块的作用

``` php
// Put table and key name in variables for easier reading
$refTable=$fkEntry[0]; // Table name that current fk references to
$refKey=$fkEntry[1];   // Key in that table being referenced
$refClassName=$this->generateClassName($refTable);

// Add relation for this table
$relationName=$this->generateRelationName($tableName, $fkName, false);
$relations[$className][$relationName]="array(self::BELONGS_TO, '$refClassName', '$fkName')";
```
- 对于尚未完成的工作使用单行注释 TODO 标注 
## 最佳实践
- 谨慎使用'echo', 'print', 'prin_r'和'die',确保你真的需要它们
- 避免使用魔法数，所用的常量使用const定义，所有字母大写以“_”分隔
- Controller中主要完成获取用户输入，从model层获取数据，选择页面渲染的工作
  - HTML和数据字段名不能写在其中，而应该写在view中
  - SQL不能写在其中，而应该写在model中，提供方法给controller调用 
- 所有的controller不能直接继承CController，而是需要自定义一个controller来继承CController，所有其他添加的controller均继承该controller

``` php
<?php
/**
 * Controller is the customized base controller class.
 * All controller classes for this application should extend from this base class.
 */
class Controller extends CController
{
    /**
     * @var string the default layout for the controller view. Defaults to '//layouts/column1',
     * meaning using a single column layout. See 'protected/views/layouts/column1.php'.
     */
    public $layout='//layouts/main';
    /**
     * @var array context menu items. This property will be assigned to {@link CMenu::items}.
     */
    public $menu=array();
    /**
     * @var array the breadcrumbs of the current page. The value of this property will
     * be assigned to {@link CBreadcrumbs::links}. Please refer to {@link CBreadcrumbs::links}
     * for more details on how to specify this property.
     */
    public $breadcrumbs=array();

    public function beforeAction($action)
    {

    }
}
```
- View中主要存放HTML代码，从controller获取数据进行渲染，包含尽量少的逻辑代码
  - 收集用户输入（例如 $_GET 和 $_POST）不能写在其中，而应该写在controller中
  - SQL不能写在其中，而应该写在model中，提供方法给controller调用 
    模板中的控制语句使用标识性语句而非括号

``` php
//wrong
<?php if (!empty($item)) { ?>
  <p><?php print $item; ?></p>
<?php } ?>

<?php foreach ($items as $item) { ?>
  <p><?php print $item; ?></p>
<?php } ?>


//right
<?php if (!empty($item)): ?>
  <p><?php print $item; ?></p>
<?php endif; ?>

<?php foreach ($items as $item): ?>
  <p><?php print $item; ?></p>
<?php endforeach; ?>
```
- 数据库表名和字段名使用小写字母，以_分隔多个单词，使用前缀名区分一个数据库下相同表名 
  
  ```
  **Example:** product_order
  ```
- Model中主要负责对于数据库的访问，为controller提供数据接口
  - HTML不应该写在其中，而应该写在view中 
- 对于简单重复数据库操作；使用AR进行查询，当涉及到多表关联，复杂操作或者性能敏感时使用SQL查询；如果只是进行查询操作，并且兼顾易用性和性能可以使用Query Builder

``` php
//AR save
$post=new Post;
$post->title='sample post';
$post->content='post body content';
$post->save();

//SQL query
$sql="SELECT username, email FROM tbl_user";
$connection = Yii::app()->db;   // 假设你已经建立了一个 "db" 连接
// 如果没有，你可能需要显式建立一个连接：
// $connection =n ew CDbConnection($dsn,$username,$password);
$command = $connection->createCommand($sql);
// 如果需要，此 SQL 语句可通过如下方式修改：
// $command->text=$newSQL;

//Query Builder
$user = Yii::app()->db->createCommand()
    ->select('id, username, profile')
    ->from('tbl_user u')
    ->join('tbl_profile p', 'u.id=p.user_id')
    ->where('id=:id', array(':id'=>$id))
    ->queryRow();
```
- 对于log信息分级，并且明确指定所属的分类名称。调试信息使用trace，产品级信息根据价值程度选择特定的级别，具体可参考[http://www.yiiframework.com/doc/guide/1.1/en/topics.logging Logging Topic]

``` php
Yii::log('request group list: ' .$groupList , 'trace', 'system.group.search');
```
- 对于可复用的页面组件或者较为复杂的页面成分，应该抽离出widget，拥有自己独立的view，具体可参考
  - [http://www.yiiframework.com/wiki/23/how-to-create-a-breadcrumb-widget Simple Example]
  - [http://www.yiiframework.com/doc/guide/1.1/en/basics.view#widget Widget View]
- 抛出的异常要明确到某个特定的类型，所有抛出异常的部分使用catch分类捕获，根据具体情况能够处理的直接处理，不能处理的继续抛出

``` php
try
{
 ...
}
catch(AppException $e)
{
    ErrorHandler::handleDefaultError($e);
    $this->render('search', array('filterParams'=>$filterParams));
}
catch(SystemException $e)
{
    ErrorHandler::handleDefaultError($e);
}
```
- 由于PHP暂时不支持finally，在异常被捕获之后，或者异常处理结束之后的控制流程中清理资源（例如数据库连接等）
- 异常捕获后不能阻塞程序的正常执行，或者容错处理，或者跳转到错误页面
- 对异常进行合理的分层，例如不要将SQLException放置到非数据处理层，而出现在业务逻辑层 
## 参考

[Drupal Coding Standards](https://drupal.org/coding-standards#helpermod)
[Yii Code Style](http://www.yiiframework.com/wiki/102/code-style/)
[Yii Code Standard](http://vadimg.com/2009/07/yii-php-coding-standards-draft/)
[Yii Conventions](http://www.yiiframework.com/doc/guide/1.1/en/basics.convention)
[Zend Framework Coding Standards](http://framework.zend.com/manual/1.12/en/coding-standard.html)
[PHP Coding Standards](http://pear.php.net/manual/en/standards.php)
