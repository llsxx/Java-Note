## 核心接口

## Repository接口的使用

>  提供了方法名称命名查询方式
>  提供了基于@Query注解查询与更新

### 方法名称命名查询方式

```java
import com.bjsxt.pojo.Users;
import org.springframework.data.repository.Repository;

import java.util.List;

/**
 * Repository接口方法名称命名查询
 *
 */
public interface UsersRepositoryByName extends Repository<Users,Integer> {
    //方法名称必须要遵循驼峰式命名规则，findBy（关键字）+属性名称（首字母大写）+查询条件（首字母大写）
    List<Users> findByName(String name);

    List<Users> findByNameAndAge(String name,Integer age);

    List<Users> findByNameLike(String name);

}
```

```java
/**
	 * Repository
	 */
	@Test
	public void UsersRepositoryByName(){
		List<Users> list=this.usersRepositoryByName.findByName("张三");
		for (Users users:list){
			System.out.println(users);
		}
	}

	@Test
	public void findByNameAndAge(){
		List<Users> list=this.usersRepositoryByName.findByNameAndAge("张三",20);
		for (Users users:list){
			System.out.println(users);
		}
	}

	@Test
	public void findByNameLike() {
		List<Users> list = this.usersRepositoryByName.findByNameLike("张%");
		for (Users users : list) {
			System.out.println(users);
		}
	}

```



| Keyword           | Sample                                  | JPQL snippet                                                 |
| :---------------- | :-------------------------------------- | :----------------------------------------------------------- |
| And               | findByLastnameAndFirstname              | … where x.lastname = ?1 and x.firstname = ?2                 |
| Or                | findByLastnameOrFirstname               | … where x.lastname = ?1 or x.firstname = ?2                  |
| Is,Equals         | findByFirstnameIs,findByFirstnameEquals | … where x.firstname = ?1                                     |
| Between           | findByStartDateBetween                  | … where x.startDate between ?1 and ?2                        |
| LessThan          | findByAgeLessThan                       | … where x.age < ?1                                           |
| LessThanEqual     | findByAgeLessThanEqual                  | … where x.age ⇐ ?1                                           |
| GreaterThan       | findByAgeGreaterThan                    | … where x.age > ?1                                           |
| GreaterThanEqual  | findByAgeGreaterThanEqual               | … where x.age >= ?1                                          |
| After             | findByStartDateAfter                    | … where x.startDate > ?1                                     |
| Before            | findByStartDateBefore                   | … where x.startDate < ?1                                     |
| IsNull            | findByAgeIsNull                         | … where x.age is null                                        |
| IsNotNull,NotNull | findByAge(Is)NotNull                    | … where x.age not null                                       |
| Like              | findByFirstnameLike                     | … where x.firstname like ?1                                  |
| NotLike           | findByFirstnameNotLike                  | … where x.firstname not like ?1                              |
| StartingWith      | findByFirstnameStartingWith             | … where x.firstname like ?1 (parameter bound with appended %) |
| EndingWith        | findByFirstnameEndingWith               | … where x.firstname like ?1 (parameter bound with prepended %) |
| Containing        | findByFirstnameContaining               | … where x.firstname like ?1 (parameter bound wrapped in %)   |
| OrderBy           | findByAgeOrderByLastnameDesc            | … where x.age = ?1 order by x.lastname desc                  |
| Not               | findByLastnameNot                       | … where x.lastname <> ?1                                     |
| In                | findByAgeIn(Collection ages)            | … where x.age in ?1                                          |
| NotIn             | findByAgeNotIn(Collection age)          | … where x.age not in ?1                                      |
| ***\*TRUE\****    | findByActiveTrue()                      | … where x.active = true                                      |
| **FALSE**         | findByActiveFalse()                     | … where x.active = false                                     |
| IgnoreCase        | findByFirstnameIgnoreCase               | … where UPPER(x.firstame) = UPPER(?1)                        |

### 基于@Query注解查询与更新

```java
import com.bjsxt.pojo.Users;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;


import java.util.List;

/**
 * 〈一句话功能简述〉<br> 
 *  Repository    @Query
 *
 * @author admin
 * @create 2019/5/22
 * @since 1.0.0
 */
public interface UsersRepositoryQueryAnnotation extends JpaRepository<Users,Integer> {
    @Query("from Users where name = ?")
    List<Users> queryByNameUseHQL(String name);

    @Query(value = "select * from t_user where name=?",nativeQuery = true)
    List<Users> queryByNameUseSQL(String name);

    @Query("update Users set name=? where id=?")
    @Modifying  //需要执行一个更新操作
    void updateUsersNameById(String name,Integer id);

}
```

## CrudRepository接口的使用



## PagingAndSortingRepository接口的使用

排序和分页

dao层

```java
import com.bjsxt.pojo.Users;
import org.springframework.data.repository.PagingAndSortingRepository;

public interface UsersRepositoryPagingAndSorting extends PagingAndSortingRepository<Users,Integer> {

}
```

```java
@Test
	public void testPagingAndSortingRepositorySort() {
		//Order	定义了排序规则
		Sort.Order order=new Sort.Order(Sort.Direction.DESC,"id");
		//Sort对象封装了排序规则
		Sort sort=new Sort(order);
		List<Users> list= (List<Users>) this.usersRepositoryPagingAndSorting.findAll(sort);
		for (Users users:list){
			System.out.println(users);
		}
	}

	@Test
	public void testPagingAndSortingRepositoryPaging() {
		//Pageable:封装了分页的参数，当前页，煤业显示的条数。注意：它的当前页是从0开始
		//PageRequest(page,size):page表示当前页，size表示每页显示多少条
		Pageable pageable=new PageRequest(1,2);
		Page<Users> page=this.usersRepositoryPagingAndSorting.findAll(pageable);
		System.out.println("数据的总条数："+page.getTotalElements());
		System.out.println("总页数："+page.getTotalPages());
		List<Users> list=page.getContent();
		for (Users users:list){
			System.out.println(users);
		}
	}

	@Test
	public void testPagingAndSortingRepositorySortAndPaging() {
		Sort sort=new Sort(new Sort.Order(Sort.Direction.DESC,"id"));
		Pageable pageable=new PageRequest(0,2,sort);
		Page<Users> page=this.usersRepositoryPagingAndSorting.findAll(pageable);
		System.out.println("数据的总条数："+page.getTotalElements());
		System.out.println("总页数："+page.getTotalPages());
		List<Users> list=page.getContent();
		for (Users users:list){
			System.out.println(users);
		}
	}
```

