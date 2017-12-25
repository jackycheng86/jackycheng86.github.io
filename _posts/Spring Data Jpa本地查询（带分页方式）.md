#Spring Data Jpa本地查询（带分页方式）
在利用spring data jpa开发的时候为了解决一些复杂的查询需求这时候我们需要引入本地查询nativeQuery
参照官方的例子

Native queries
The @Query annotation allows to execute native queries by setting the nativeQuery flag to true.

Example 50. Declare a native query at the query method using @Query

```
public interface UserRepository extends JpaRepository<User, Long> {
  @Query(value = "SELECT * FROM USERS WHERE EMAIL_ADDRESS = ?1", nativeQuery = true)
  User findByEmailAddress(String emailAddress);
  }
```
这是官方给出的使用本地查询的例子，但是也特别说明了目前本地查询并不支持动态排序，但是可以利用本地查询进行分页
Note, that we currently don’t support execution of dynamic sorting for native queries as we’d have to manipulate the actual query declared and we cannot do this reliably for native SQL. You can however use native queries for pagination by specifying the count query yourself:
这是官方给出的本地查询的分页形式
<font color="red">注意：如果你用这种方式在进行分页查询时是会报错的（测试数据库为Oracle，其他数据库暂未做测试）</font>
Example 51. Declare native count queries for pagination at the query method using @Query

```
public interface UserRepository extends JpaRepository<User, Long> {
  @Query(value = "SELECT * FROM USERS WHERE LASTNAME = ?1",
    countQuery = "SELECT count(*) FROM USERS WHERE LASTNAME = ?1",
    nativeQuery = true)
  Page<User> findByLastname(String lastname, Pageable pageable);
}
```
我的代码如下
```
@Query(value = "select * from secondleveldesc sd left join secondlevel s on s.code=sd.code ",
            countQuery = "select count(*) from secondleveldesc",
            nativeQuery = true)
    Page<SecondleveldescEntity> findAll(Pageable pageable);
```
此时的错误信息如下
>Caused by: org.springframework.data.jpa.repository.query.InvalidJpaQueryMethodException: Cannot use native queries with dynamic sorting and/or pagination in method public abstract org.springframework.data.domain.Page com.material.cltx.dao.SecondlevelDescDao.findAll(org.springframework.data.domain.Pageable)

通过各种搜索最终找到一种解决办法
```
    @Query(value = "select * from secondleveldesc sd left join secondlevel s on s.code=sd.code  ORDER BY ?#{#pageable}",countQuery = "select count(*) from secondleveldesc",nativeQuery = true)
    Page<SecondleveldescEntity> findAll(Pageable pageable);
```

在查询语句后面加上
>ORDER BY ?#{#pageable}"

这样就能顺利的利用本地查询进行分页了
