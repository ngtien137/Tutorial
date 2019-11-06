# Tutorial about RoomDB with Kotlin x androidX
## Một vài điều cơ bản về RoomDB
* Room DB là gì thì google đầy nên sẽ không đề cập đến
* Tại sao dùng room mà không phải dùng SQLite? Vì SQLite có nhiều thứ gây khó chịu:
  * Để tạo ra 1 instance tới sqlite ta phải thông qua sqlite helper, phải open, close các kiểu. Mỗi lần run xong 1 câu lệnh lấy dữ liệu tới sqlite, dù ta viết thành các hàm thì mỗi hàm vẫn phải có hàm open, dùng xong lại phải close.
  * Mặc dù có 1 chút gì đó gọi là hỗ trợ lập trình viên truy vấn đến cơ sở dữ liệu thông qua các hàm có sẵn như insert, update,... nhưng việc truyền dữ liệu vào trong câu truy vấn hết sức cồng kềnh. Câu lệnh thường trở nên dài dòng nếu câu truy vấn phức tạp. Viết khó, sửa cũng khó.
  * Kết quả trả về còn phải load ra thông qua cursor theo từng bảng, từng cột. Vì vậy lại cần 1 bước nữa để chuyển chúng thành object
  
* Còn với RoomDB:
  * Đã có annotation hỗ trợ sẵn. Giống như ta đã học với annotation thì chỉ cần áp annotation vào trong class và khi sử dụng với room nó sẽ tự nhận diện và chuyển đổi cái cursor đó thành một cái list các đối tượng tương ứng.
  * Có sẵn lớp adapter thông qua annotation @Dao để thực hiện truy vấn tới database
  * Tạo instance cũng trở nên dễ dàng hơn
  * Và đặc biệt là còn hỗ trợ livedata.
## Sử dụng RoomDB
* Đầu tiên đương nhiên là phải import thư viện vào, ưu tiên dùng mới nhất

```
apply plugin: 'kotlin-kapt'
dependencies {
 def room_version = "2.1.0" //Ví dụ 2.1.0
 implementation "androidx.room:room-runtime:$room_version"
 kapt "androidx.room:room-compiler:$room_version"
 implementation "androidx.room:room-ktx:$room_version"
}
```

* Về bản chất vẫn là SQLite nhưng RoomDB hỗ trợ lưu dữ liệu thông qua dạng object. Vì thế ta vẫn tạo 1 class như bình thường nhưng có sử dụng những chức năng của RoomDB trong này.
  * Đó là các annotation của RoomDB:
  
  *Ghi chú: {Name} sẽ là biến Name*
  
  * @Entity(tableName={tableName}) - Định nghĩa lớp phía dưới sẽ tương tứng với bảng *tableName* trong DB, Ví dụ *@Entity(tableName="tbl_user")*
  
  * @ColumnInfo(name={columnName}) - Định nghĩa thuộc tính phía dưới sẽ tương ứng với tên cột *columnName* trong DB, Ví dụ *@ColumnInfo(name="user_id")*
  
  * @PrimaryKey() - Định nghĩa thuộc tính phía dưới là khóa chính của bảng. Nếu thuộc tính phía dưới là Int hoặc Long và muốn nó tự động được sinh ra thì có thể thêm vào thành *@PrimaryKey(autoGenerate = true)*
  
  * @Ignore: - Hàm, thuộc tính, constructor phía dưới sẽ không được lưu vào trong DB. Ví dụ ta có một hàm chuyển đổi định dạng file, hay lấy ra đuôi file từ thuộc tính đường dẫn của đối tượng chẳng hạn. Vì vậy ta sẽ sử dụng cả lớp File trong hàm đó. Hoặc 2 constructor lưuu vào DB là điều không cần thiết và những cái thừa thãi này hình như gây ra lỗi =)) Không nhớ lắm nên tốt nhất là để class ở mức đơn giản để lưu vào DB thôi.
  
  * Ngoài ra còn có vài cái annotation nữa như @TypeConverters,@TypeConverter: Cái này sẽ dùng trong trường hợp trong object có thuộc tính là object. Nó sẽ convert object sang string json để lưu vào DB bởi như đã nói ở trên nhìn thì RoomDB như đang lưu object nhưng thực ra nó vẫn là SQLite nên nó không thể nào lưu object được. Phải convert về dạng chuẩn để lưu nhưng nó support mình. Hoàn toàn tự động. Về cái annotation này trở đi thì sẽ là phần nâng cao hơn rồi nên sẽ không đề cập đến. Tự tìm hiểu =))
  
- Chuyển sang ví dụ cụ thể, ta sẽ tạo 1 class User

```
@Entity(tableName="tbl_user") //class User sẽ tương ứng với bảng tbl_user trong DB
class User{
    @PrimaryKey(autoGenerate = true) //Khóa chính 
    @ColumnInfo(name = "user_id") //Cột user_id tự động tăng từ 1 thì phải =))
    var id:Int = 0
    @ColumnInfo(name = "user_name") //Cột user_name
    var name:String =""
}
```

- Tiếp đến ta sẽ tạo 1 lớp adapter để thực hiện liên kết giữa DB với class User này cũng có nghĩa là nó sẽ sở hữu các hàm truy vấn tới dữ liệu của bảng tbl_user vừa định nghĩa ở trên. Để sử dụng phần này ta sẽ tạo ra một interface và sử dụng annotation @Dao của RoomDB:

```
@Dao
interface UserDao {
    
}
```
- Như đã thấy, ta đang sử dụng là interface như vậy các hàm của chúng ta sẽ không có phần body. Vậy room sẽ chạy hàm này dựa vào đâu? Chỗ dựa lớn nhất của nó ở đây là annotation *@Query()*. Có một thứ mà ta cần khi sử dụng room đó là ta phải có kiến thức về SQL. Bởi tham số truyền vào annotation *@Query()* chính là một câu truy vấn. Ví dụ: 
```
@Query("Select * from tbl_user")
```
- Tuy nhiên Sqlite có thì room cũng có. SQLite có những hàm insert, update, delete thì RoomDB cũng  vậy. Nó hỗ trợ sẵn các phần này. Thật ra còn xịn hơn. RoomDB support 3 chức năng này lần lượt là @INSERT, @UPDATE, @DELETE sử dụng đơn giản như sau:
*Phía dưới này có thể đúng hoặc sai =))*
```
@Insert(onConflict = OnConflictStrategy.IGNORE)
fun insert(user:User):Long
//Với hàm insert này có thể trả về hoặc không trả về nhưng nếu trả về là id autoincrement thì phải là kiểu Long
//Dùng Int sẽ crash
@Update(onConflict = OnConflictStrategy.REPLACE)
fun update(user:User)
@Delete
fun delete(user:User)
```

- Như vậy, chỉ cần gọi đến hàm insert và truyền vào 1 user thì nếu thành công trong bảng tbl_user của chúng ta đã có dữ liệu của user đó rồi. Nhưng onConflict ở đây là sao? onConflict là một cờ giúp check ta sẽ làm gì nếu như gặp trường hợp primary key trùng nhau hay nói cách khác là object tồn tại trong DB rồi.
  * Với *OnConflictStrategy.IGNORE* nó sẽ từ chối hành động này và trả ra kết quả là -1 nếu như hàm phía dưới có trả về kiểu Long. Còn nếu insert thành công sẽ trả ra là id vừa được add vào.
  *  Còn với *OnConflictStrategy.REPLACE* nó sẽ remove đi dữ liệu đang bị trùng có trong DB và thay thế bằng dữ liệu được truyền vào
- Còn @Delete thì thôi, chắc tự hiểu đi =))



