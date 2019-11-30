---
title: Ownership trong Rust
published: true
technology: true
date: 2019-07-07 20:14:20
tags: rust, learning, challenge
description: Tính năng ownership trong Rust 
image: https://theburningmonk.com/wp-content/uploads/2015/05/rust_ownership_3.png
---

## Ownership là gì
Ownership là quyền sở hữu của mỗi biến trong Rust.

Đây là một khái niệm mới và là một tính năng rất quan trọng trong Rust, phân biệt Rust với nhiều ngôn ngữ lập trình khác.

Nhiêu ngôn ngữ lập trình có một **bộ dọn rác** , hay còn gọi là garbage collection. Bộ dọn rác này có nhiệm vụ tự động giải phóng vùng nhớ khi nhận thấy chương trình không còn sử dụng nó nữa. Khi học môn lập trình hướng đối tượng mình có nhớ C# có bộ dọn này, trong khi C và *C++* thì không , ta phải tự giải phóng vùng nhớ, ví dụ sau mỗi lần xin cấp phát động thì các thầy luôn nói phải có dòng free.

Rust thì nói không với garbage collection (GC), thay vào đó Rust phát triển một phương thức mới: bộ nhớ sẽ được quản lí thông qua *ownership*, với các luật logic được compiler kiểm tra trong quá trình biên dịch, Rust compiler sẽ dự đoán được khi nào một biến hết sử dụng và tự động chèn thêm code logic để giải phóng nó. Phương thức này giúp Rust kiểm soát tài nguyên một cách an toàn và hiệu suất cao mà không cần dùng GC.

<span class = "mute sidenote"> [Rust has a static garbage collector](https://news.ycombinator.com/item?id=18186189) </span>

Vì sao Rust không chọn GC? Mọi người có thể tra và đọc thêm về [những mặt chưa tốt của GC](https://stackoverflow.com/questions/3214980/what-are-the-disadvantages-in-using-garbage-collection), mình nghĩ một trong số đó phải nhắc đến là GC làm ảnh hưởng đến hiệu suất thực thi của chương trình (đặc biệt là các chương trình real-time như game) khi phải liên tục tìm và xóa rác.

Việc không có GC còn giúp Rust dễ dàng được nhúng vào các ngôn ngữ có GC khác nữa

Khi có nhiều kinh nghiệm về ownership, chúng ta sẽ có khả năng phát triển code một cách an toàn và hiệu quả hơn nhiều. Hiểu về ownership là tối quan trọng để đi những bước tiếp theo trong Rust.

## Stack và Heap

Cả Stack và Heap là những phần của bộ nhớ được sử dụng tại thời điểm runtime. 

Stack lưu trữ dữ liệu theo thứ tự last in, first out (LIFO). Những dữ liệu đó cần phải rõ ràng, có một kích thước cố định. Còn Heap dùng để lưu trữ những dữ liệu có kích thước chưa rõ ràng, với mức tổ chức thấp hơn. Dữ liệu trong Heap được lưu trữ theo cách khi chúng ta gửi một yêu cầu lưu trữ dữ liệu, hệ điều hành sẽ tìm vùng trống trong Heap đủ lớn để chứa nó, đánh dấu là vùng đó đã được sử dụng, rồi trả về một con trỏ chỉ đến địa chỉ vùng nhớ vừa được lưu trữ trong Heap, con trỏ này sẽ được lưu trữ lại trong Stack.

Từ đó, có thể nhận thấy việc đưa dữ liệu vào Stack nhanh hơn đưa dữ liệu vào Heap. Bởi vì với Stack, hệ điều hành sẽ không cần thực hiện thao tác tìm kiếm một vùng nhớ đủ lớn, nó chỉ việc đặt dữ liệu mới vào phần trên cùng của stack. Tương tự cho việc truy xuất dữ liệu, việc lấy dữ liệu từ Heap ra vẫn chậm hơn Stack. Với Heap, hệ điều hành cần nhìn xem con trỏ trong Stack đang chỉ đến vùng nhớ nào trong Heap, rồi lần theo con trỏ đó đến và lấy dữ liệu ra. Với Stack, chỉ việc pop phần tử trên cùng ra.

## Chuyện gì xảy ra với một biến

Trong Rust, mọi giá trị được khai báo đều là **immutable**, nghĩa là không thể thay đổi được. Vì thế một biến được khai báo theo cách thông thường thì cũng **immutable** nốt.

Ví dụ, mình khai báo 
<span class = "mute sidenote">i32 ở đây thể hiện biến a là kiểu số nguyên 32 bit</span>
```
let a: i32 = 5;
```


Thì biến a là một immutable variable, có một kích thước xác định và không đổi. Máy tính sẽ cấp phát một vùng nhớ trên Stack, với giá trị mặc định là giá trị được truyền vào khi khai báo (là 5). Địa chỉ của vùng nhớ này sẽ được gán cho biến a. Khi đó, ta có thể coi là: vùng nhớ này thuộc về biến a, và a có quyền sở hữu (ownership) đối với vùng nhớ đó.

Tương tự với một chuỗi kiểu string literal, là một chuỗi kí tự đã xác định và không thay đổi kích thước được nữa. 
```
let s = "hello";
```
Chuỗi này đơn giản cũng được lưu trữ ở Stack. Nhưng thực tế kiểu string literal lại khá bất tiện vì tính cố định của nó. Không phải chuỗi nào chúng ta cũng có thể xác định được chính xác khi viết code, ví dụ chuỗi kí tự nhập từ bàn phím của người dùng. Từ đó, Rust có kiểu String thứ 2, kiểu này được cấp phát ở Heap và có thể lưu trữ được một lượng kí tự chưa xác định tại thời điểm biên dịch. 
```
let mut s = String::from("hello");

s.push_str(", world!"); // push_str() appends a literal to a String

println!("{}", s); // This will print `hello, world!`
```
Vậy khi ta khai báo một biến, tùy theo đặc điểm của biến đó mà nó có thể được lưu trữ ở Stack hay Heap.

Sau đó, khi mình không cần dùng tới biến s đó nữa, mình cần giải phóng nó đi để khỏi tốn bộ nhớ. Với những dữ liệu lưu trữ trên Stack, đơn giản sau khi ra khỏi **scope** (sẽ trình bày sau), máy tính sẽ pop nó ra khỏi Stack và bỏ đi. Với những dữ liệu trên Heap thì hơi phức tạp hơn, máy tính sẽ phải lần theo con trỏ trong Stack và giải phóng vùng nhớ đã cấp phát trên Heap. Với những ngôn ngữ có garbage collector (GC), GC sẽ làm nhiệm vụ đó và ta không cần quan tâm nữa. Tuy nhiên, khi không có GC, ta cần có trách nhiệm với những vùng nhớ mà mình đã xin cấp phát. Nếu không giải phóng đi sẽ rất lãng phí bộ nhớ, nếu ta xóa quá sớm, biến đó sẽ không còn giá trị nữa, hay khi ta quên đã xóa rồi mà lại xóa lại lần nữa, lỗi sẽ xảy ra. 

Những luật của ownership sẽ giúp Rust quản lý hiệu quả việc này.

## 3 luật của Ownership

1. **Mỗi vùng nhớ đều có một biến sỡ hữu nó, gọi là *owner*.**
2. **Tại một thời điểm, mỗi vùng nhớ chỉ có thể thuộc về duy nhất một biến.**
3. **Khi owner ra khỏi [scope](https://stackoverflow.com/questions/500431/what-is-the-scope-of-variables-in-javascript), vùng nhớ đó cũng ngay lập tức bị giải phóng đi (gọi là *drop*)**

## Hiểu 3 luật này
### Move
Qua ví dụ sau:

```
let a = 5;
let b = a;
```
Mọi chuyện sẽ xảy ra như sau: gán giá trị 5 cho biến a, máy tính sẽ cấp phát một vùng nhớ trong Stack có giá trị là 5 và địa chỉ được gắn cho biến a, ta gọi a là owner của vùng nhớ đó. Sau đó, vì biến a đã xác định và yên vị trong Stack, máy tính đọc biến a, đi vào Stack và copy giá trị của nó (là 5), gán cho biến b. Rồi lại tạo một vùng nhớ có giá trị là 5 và địa chỉ được gắn cho biến b, ta gọi b là owner của vùng nhớ mới này. (*luật thứ 1*)

Bởi vì biến số nguyên là đơn giản với kích cỡ đã biết trước và không thay đổi, 2 giá trị 5 này sẽ được push vào 2 vùng nhớ trên Stack.
Giờ thì cùng xem xét ví dụ tiếp theo:

```
let s1 = String::from("hello");
let s2 = s1;
```
Cũng khá giống ví dụ trên, mình đang muốn copy dòng chữ "hello" từ s1 sang cho s2. Chỉ khác ở chỗ giá trị của s1 bây giờ được lưu ở Heap chứ không phải Stack (vì kiểu String chứ không phải string literal). Mọi chuyện cũng khác đi.

Máy tính sẽ cấp phát một vùng nhớ trong Heap để  lữu trữ chữ "hello". Đồng thời trả về một dữ liệu được lưu trữ ở Stack gồm 3 thành phần: một con trỏ (ptr), trỏ tới vùng nhớ trong Heap, một biến chiều dài (len), lưu trữ độ dài theo bytes của chuỗi String trong Heap, và một biến capacity là tổng lượng bộ nhớ được nhận từ hệ điều hành. <span class = "mute sidenote">Nếu viết thêm vài dòng lệnh xóa đi chữ `o` trong vùng Heap thì len sẽ giảm xuống 4 còn capacity vẫn giữ là 5 </span>

![](img/ownership-01.svg)

Khi mình gán biến s1 cho s2, máy tính sẽ đi theo biến s2 vào Stack và copy vùng dữ liệu ở đó của nó, bao gồm 3 thành phần trên, kể cả con trỏ trỏ vào vùng Heap. Mọi chuyện sẽ trở thành:

![](img/ownership-02.svg)

Giờ thì, nếu Rust vẫn để mọi chuyện như hình trên, vấn đề sẽ xảy ra khi s1 và s2 đều ra khỏi scope, tức là giá trị mà s1 và s2 sở hữu sẽ bị xóa đi. Đầu tiên s2 sẽ xóa phần dữ liệu trên Stack, trong đó có con trỏ trỏ đến vùng nhớ trên Heap, vùng nhớ đó cũng được giải phóng theo. Tiếp theo đến s1, s1 cũng truy cập lại để xóa dữ liệu trên Stack, nhận ra vẫn còn con trỏ trỏ đến vùng Heap, máy tính vẫn tiếp tục giải phóng vùng nhớ đó dù vùng nhớ đó không còn gì. Đây được gọi là lỗi *double free* mình đã nhắc đến ở trên. Để đảm bảo không xảy ra lỗi này. Rust sẽ áp dụng luật thứ 2 trong 3 luật của ownership: **Tại một thời điểm, mỗi vùng nhớ chỉ có thể thuộc về duy nhất một biến.** Vì thế sau dòng lệnh thứ 2 (let s2 = s1;), vùng nhớ trên Heap chỉ còn được s2 truy cập thông qua dữ liệu trên Stack của s2. Và s1 không còn truy cập đến vùng Heap này được nữa. Hành động này gọi là **move**. 

Nếu mình tiếp tục muốn in giá trị "hello" của s1 ra màn hình sau khi move, sẽ sinh ra lỗi như sau:

```
let s1 = String::from("hello");
let s2 = s1;

println!("{}, world!", s1);
```
Kết quả:

```
error[E0382]: use of moved value: `s1`
 --> src/main.rs:5:28
  |
3 |     let s2 = s1;
  |         -- value moved here
4 |
5 |     println!("{}, world!", s1);
  |                            ^^ value used here after move
  |
  = note: move occurs because `s1` has type `std::string::String`, which does
  not implement the `Copy` trait
```

Bộ biên dịch của Rust nói rằng giá trị s1 không còn sử dụng được nữa sau khi đã chuyển qua cho s2.<span class = "mute sidenote">note: hành động move diễn ra vì kiểu String này không được hiện thực tính năng `Copy` mà những kiểu đơn giản như integer có, như ví dụ đầu, ta có thể  copy biến a = 5 rồi gán sang b được, kiểu String thì không. Mọi người có thể xem thêm những kiểu được hiện thực [trait `Copy`](https://doc.rust-lang.org/std/marker/trait.Copy.html) </span>

![](img/ownership-03.svg)

### Clone

Có một cách để vẫn có thể sử dụng lại s1 sau khi move, đó là **Clone**
```
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```
Kết quả:

```
s1 = hello, s2 = hello
```

Hành động clone này sẽ deeply copy cả vùng dữ liệu được lưu trữ trong Heap của s1, chứ không chỉ là vùng dữ liệu trong Stack. Điều này vẫn đảm bảo được bộ luật ownership: mỗi vùng nhớ chỉ có một owner tại một thời điểm

![](img/ownership-04.svg)

### Ownership và functions

Những luật của ownership còn thể hiện tính nghiêm khắc và cẩn thận của mình thông qua các function. Ví dụ qua đoạn lệnh dưới đây:

```
fn main() {
    let s = String::from("hello");      // biến s được khai báo

    in_chuoi(s);                        // giá trị của biến s được move sang 
                                        // biến trong function (là ss sau này)
                                        // và không còn dùng được nữa.

    println!("{}", s);                  // vậy thì in s ở đây sẽ xảy ra lỗi
}

fn in_chuoi(ss: String){                // biến ss được khai báo
    println!("{}", ss);                 // biến ss đã nhận được giá trị của s
                                        // vì thế việc in ss ra là hợp lí
}                                       // ss ra khỏi scope và bị xóa theo luật số 3
```

Lỗi:

```
  |
2 |     let s = String::from("hello");
  |         - move occurs because `s` has type `std::string::String`, 
            which does not implement the `Copy` trait
3 | 
4 |     in_chuoi(s);
  |              - value moved here
5 | 
6 |     println!("{}", s);
  |                    ^ value borrowed here after move

```

Điều này khá bất tiện vì nhu cầu dùng lại biến sau khi được truyền vào một hàm khác là luôn xảy ra. Vì vậy, mình sẽ đề cập đến tính năng **References** để giải quyết vấn đề này ở bài viết tiếp theo.

## Kết luận

3 luật trên của ownership có vẻ đơn giản nhưng đã đảm bảo được độ an toàn của dữ liệu và cải thiện tốc độ xử lý hơn so với dùng GC. Lúc đọc sách mình còn khá mông lung nhưng sau khi viết xong bài này thì mình đã hiểu hơn nhiều về ownership. Mình sẽ viết tiếp phần references ở mục sau vì bài này cũng đã dài, còn giờ thì tiếp tục học về  Struct. Happy Rusting!

## Tham khảo

- [1] [Understanding Ownership || Rust's Book](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html)
- [2] [Đừng đánh nhau với borrow checker](https://thefullsnack.com/posts/rust-borrow-checker-part-1.html)
- [3] [Learning Ownership || một bài blog từ dev.to](https://dev.to/domcorvasce/learning-rust-1-ownership-g0)
- [4] [Rust là gì? Có ăn được không?](https://thefullsnack.com/posts/rust-intro.html)
