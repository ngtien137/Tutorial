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
  
  * Chuyển sang ví dụ cụ thể, ta sẽ tạo 1 class User
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
