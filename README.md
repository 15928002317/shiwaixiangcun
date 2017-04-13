# shiwaixiangcun
public project
# aggregate

项目结构分模块编写：
- 模块统一都放在`modules`下面，建立一个`package`，以模块命名
- 模块内的目录结构
1.  `controller`
2. `domain`           存放实体
3. `dto`                存放数据传输对象
4. `mapper`
5. `repository`      编写操作数据库的接口
6. `service`

## Domain

所有 domain 都必须继承`BaseEntity`
domain的一些配置:

@Entity(table = "house")



## Controller

在`Controller`中的每个方法都只能调用一次`Service`的，不要多次调用`Service`
`Controller`中处理数据校验，已经`Service`返回的数据，做特殊处理
`Controller` 类上面的格式   => `@RequestMapping("/m/....")`

### Http的4种操作方式

- `PUT` : 更新操作
- `GET` : 查询操作
- `POST` : 创建
- `DELETE` : 删除

### 请求`URL`的方式（后期扩展）

- 以`.json`结尾（或者待参数`format=json`）的URL，表示以JSON的格式返回数据结果。eg : `http://localhost/house/form.json?one.name=SilentWu&two.age=27`
- URL没有任何结尾或者没带`format`参数，那么默认以`text/html`的方式返回。eg : `http://localhost/house/view?page.size=10&page.pn=1&sort.unitNum=desc`

### 参数注入的方式（后期扩展）
- 支持`springMVC`的所有注入方式`@RequestParam`    `@PathVariable`
- 支持通用Dto的注入方式 `MonkeyDto`
- 支持`Pageable`分页参数的注入，相关的还有  `@PageableDefaults()`配置默认的分页参数
- 支持 `FormModel`的注入方式，`@FormModel`

### 两种数据校验方式
- 采用`SpringMVC`中校验方式,创建Dto对象接受参数  `@Valid`   `BindingResult`
- 使用通用Dto对象`MonkeyDto`接受参数，使用`@ParameterConstraints`   `@ParameterConstraint`


### 允许返回的结果类型
- 直接返回实体对象
- 直接返回一个List对象
- 直接返回字符串消息
- 直接返回一个分页的对象Page
- 直接返回MonkeyResponseDto

### 前端传入fields参数，指定需要哪些属性用 逗号分隔
eg : `http://localhost/house/form.json?page.size=10&page.pn=1&sort.unitNum=desc&fields=id,name,number`

## Service

- 模块中的所有`Service`接口都需要继承`BaseService`, 已经包含了很多通用的方法。 eg:
```
public interface HouseService extends BaseService<House, Long> {
    //定义自己需要的方法
    House findHouseById(Long id);
}
```

- `Service`的实现类 继承 `BaseSeviceImpl` 并且实现 自己定义的`Service`接口。eg:
```
@Service    //必须的
public class HouseServiceImpl extends BaseServiceImpl<HouseRepository, House, Long> implements HouseService {
    /**
     * 构造函数.  必须的
     *
     * @param repository 注入的Repository
     */
    public HouseServiceImpl(HouseRepository repository) {
        super(repository);
    }

    //实现自己定义的方法
    @Override
    public House findHouseById(Long id) {
        return repository.findById(id);
    }
}
```

项目中的`Transaction`都是在`Service`层开启的，默认都开启的是 Read-Write事务，在实际使用过程中，
如果Service的方法都只做的查询操作不做任何的修改，请在方法上面添加注解`@Transactional(readOnly = true)`


## Repository
所有的`Repository`都应该继承`BaseRepository`

完整的参考文档：[reference documentation][1]

- 简单的单表操作，定义好方法名，框架能够自动生成SQL，不用自己手写SQL

- 复杂的SQL,使用`@Query`,具体参数配置可以看源码  eg:
```
public interface HouseRepository extends BaseRepository<House, Long> {

    House findById(Long id);

    @Query("findByHouseNum")  //指定SQL 的id
    List<House> findByHouseNum(@Param("houseNum") String houseNum);
}
```


- SQL映射文件 Mapper.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--
注意：
    1. namespace：只能指定成实体的路径
    2. sql查询中 需要用 as 转换成对象的属性名称，这样才能正确注入返回结果,  (后期需要扩展成 下划线 和 驼峰互转)
    3. 返回的结果类型 必须写成全路径   (后期需要扩展能支持别名)
 -->
<mapper namespace="com.shiwaixiangcun.property.modules.house.domain.House">
    <select id="findByHouseNum" resultType="com.shiwaixiangcun.property.modules.house.domain.House">
      SELECT
        house_num_desc as houseNumDesc,
        ridgepole_id as ridgepoleId,
        unit_num as unitNum,
        house_num as houseNum
      FROM house
      WHERE
      house_num = #{houseNum}
    </select>
</mapper>
```

[1]:http://www.ifrabbit.com/



