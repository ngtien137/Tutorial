# Tutorial about RoomDB with Kotlin
## Một vài điều cơ bản về RoomDB
* Room DB là gì thì google đầy nên sẽ không đề cập đến
* Tại sao dùng room mà không phải dùng SQLite. Vì SQLite có nhiều thứ gây khó chịu
  * Để tạo ra 1 instance tới sqlite ta phải thông qua sqlite helper, phải open, close các kiểu. Mỗi lần run xong 1 câu lệnh lấy dữ liệu tới sqlite, dù ta viết thành các hàm thì mỗi hàm vẫn phải có hàm open, dùng xong lại phải close.
  * Mặc dù có 1 chút gì đó gọi là hỗ trợ lập trình viên truy vấn đến cơ sở dữ liệu thông qua các hàm có sẵn như insert, update,... nhưng việc truyền dữ liệu vào trong câu truy vấn hết sức cồng kềnh. Câu lệnh thường trở nên dài dòng nếu câu truy vấn phức tạp. Viết khó, sửa cũng khó.
  * Kết quả trả về còn phải load ra thông qua cursor theo từng bảng, từng cột. Vì vậy lại cần 1 bước nữa để chuyển chúng thành object
* Còn với RoomDB:
  * đã có annotation hỗ trợ sẵn. Giống như ta đã học với annotation thì chỉ cần áp annotation vào trong class và khi sử dụng với room nó sẽ tự nhận diện và chuyển đổi cái cursor đó thành một cái list các đối tượng tương ứng.
  * Có sẵn lớp adapter thông qua annotation @Dao để thực hiện truy vấn tới database
  * Tạo instance cũng trở nên dễ dàng hơn
  * Và đặc biệt là còn hỗ trợ livedata.
## Sử dụng RoomDB
* Bước 1: Đương nhiên là phải implement thư viện vào, nhớ là dùng mới nhất càng tốt
```
apply plugin: 'kotlin-kapt'
dependencies {
 implementation "androidx.room:room-runtime:$room_version"
 kapt "androidx.room:room-compiler:$room_version"
 implementation "androidx.room:room-ktx:$room_version"
}
```
* Bước 2: RoomDB
