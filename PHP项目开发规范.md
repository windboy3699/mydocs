# PHP项目开发规范

### 框架选择

项目使用Laravel框架，以下规范中都是以Laravel框架为基础。

### 目录结构

单模块应用没有子模块的项目按原目录结构。分子模块的项目，按如下结构创建，以BaseService为例：

```
BaseService
 └— app
    |——Appmanager
    |      |——Http
    |      |   |——controllers
    |      |   |——Middleware
    |      |——Models
    |      |——Services
    |——Usercenter
    |      |——Http
    |      |   |——controllers
    |      |   |——Middleware
    |      |——Models
    |      |——Services
    |——Authorization
    |      |——Http
    |      |   |——controllers
    |      |   |——Middleware
    |      |——Models
    |      |——Services
    |——Http
    |   |——Controllers
    |   |——Middleware
    |——Models
    └——Services
```
BaseService域名：service.base.gate.example.com

Appmanager：应用管理，域名：service.base.gate.example.com/appmanager  
Usercenter：用户中心  域名：service.base.gate.example.com/usercenter  
Authorization：权限认证  域名：service.base.gate.example.com/authorization  
Appmanager、Usercenter、Authorization是BaseService下面的3个子模块，本来它们是互相独立的服务，不过考虑到这些服务代码不多如果每个独立成一个仓库，管理起来会有麻烦，所以把它们整合起来放到BaseService下面，以后有新的小模块也可以加进来。  
单模块的应用，URI中直接以'/'开始，不需要ModuleName前缀。

### Model、Service、Controller关系

数据库的读写全部放到Model中，Service、Controller中不允许出现数据库操作，它们通过调用Model来处理数据。  
如果Model中获取的数据需要经过复杂的处理后才能显示，这时需要把数据的处理逻辑放到Service中，Service调用Model，Controller调用Service。一个Service中可以调用多个Model。

### Controller规范

外部接口继承：App\Common\Controllers\Controller  
内部接口继承：App\Common\Controllers\IntraController  

```
class UserController extends Controller
{
    /**
     * 列表页
     * @Route /user
     * @Method GET
     */
    public function index() {}

    /**
     * 显示新建某条数据时的表单页
     * @Route /user/create
     * @Method GET
     */
    public function create() {}

    /**
     * 接收表单数据，新增一行到数据库
     * @Route /user
     * @Method POST
     */
    public function store(Request $request) {}

    /**
     * 显示某条数据
     * @Route /user/{id}
     * @Method GET
     */
    public function show($id) {}

    /**
     * 显示某条需要修改的数据
     * @Route /user/{id}/edit
     * @Method GET
     */
    public function edit($id) {}

    /**
     * 接收修改后的表单数据，更新到数据库
     * @Route /user/{id}
     * @Method PUT
     */
    public function update(Request $request, $id) {}

    /**
     * 删除某一条数据
     * @Route /user/{id}
     * @Method DELETE
     */
    public function destroy($id) {}

    // 以下是非标准，带有特殊语义的接口

    /**
     * 根据用户名获取用户
     * @Route /getUserByUsername/{username}
     * @Method GET
     */
    public function getByUsername($username) {}

    /**
     * 根据用户名更新头像
     * @Route /updateUserAvatarByUsername/{username}
     * @Method PUT
     */
    public function updateAvatarByUsername($username, $avatar) {}

    /**
     * 根据ids删除用户
     * @Route /deleteUserByIdIn/{1_2_3}
     * @Method DELETE
     */
    public function deleteByIdIn($ids) {$ids}
}
```

### Model示例

```
class UserModel
{
    public function findAll() {}

    public function findById() {}

    public function findByIdIn() {}

    public function findByUsername() {}

    public function findByUsernameAndPassword() {}

    public function findByEmailLike() {}

    public function findByLastnameOrFirstname() {}

    public function findByStartDateBetween() {}

    public function findByActiveTrue() {}

    public function findByActiveFalse() {}

    public function findTop10ByUsername() {}
}
```
