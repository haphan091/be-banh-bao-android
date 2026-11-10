# Bé Bánh Bao – Android MVP

Ứng dụng học và chơi dành cho bé 3–4 tuổi, lấy cảm hứng từ ảnh bé do gia đình cung cấp. Đây là **bản MVP mã nguồn Android Studio**, có tương tác học số/hình/màu/xe, trò chơi quy luật và nút nói chuyện bằng giọng nói.

## Tính năng hiện có

- Trang chủ với nhân vật Bé Bánh Bao, giao diện nút lớn, tiếng Việt.
- Học số 0–10: chọn số, nghe Text-to-Speech tiếng Việt, đếm xe và câu hỏi chọn đáp án.
- Học 4 hình: tròn, vuông, tam giác, chữ nhật.
- Học 6 màu: đỏ, vàng, xanh dương, xanh lá, cam, tím.
- Trò chơi tư duy nhận ra chuỗi lặp đơn giản.
- Khám phá xe cảnh sát, xe cần cẩu, xe ben, xe trộn bê tông, máy xúc và xe cứu hỏa.
- Nút micro dùng dịch vụ nhận diện giọng nói của Android (Speech Recognizer); câu nói được chuyển thành văn bản để bé có thể gửi.
- AI chat qua server riêng. Khi chưa cấu hình server, ứng dụng dùng các câu trả lời mẫu theo chủ đề để minh họa luồng tương tác.
- TTS đọc câu trả lời thành tiếng.

## Nhân vật và thiết kế

- `app/src/main/res/drawable/mascot_banhbao.png`: hình nhân vật dùng trong app.
- `CHARACTER_SHEET.png`: bảng tạo hình ba tư thế; giữ áo dài xanh ghi, họa tiết chim hạc, tóc đen ngắn, má bầu và nụ cười từ ảnh tham chiếu.
- `UI_CONCEPT_HOME.png`, `UI_CONCEPT_FEATURES.png`: moodboard giao diện. Chúng là hình tham khảo, không phải ảnh chụp màn hình app đang chạy.

Định hướng nhân vật: giữ nét nhận diện của ảnh gốc trong trang học và màn chào; khi vào thế giới xe, Bé Bánh Bao có thể đội mũ bảo hộ hoặc mặc áo phản quang/cảnh sát cách điệu. Tránh biến nhân vật thành người lớn hoặc dùng hình ảnh xe/đèn còi gây cảm giác nguy hiểm.

## Lộ trình sư phạm gợi ý

- Vòng 1: nhận biết hình dạng và màu qua một vật quen thuộc; đếm có chỉ tay trên từng chiếc xe.
- Vòng 2: nối chữ số với nhóm đồ vật tương ứng; ghép đôi và phân loại xe theo màu/hình.
- Vòng 3: nhận ra quy luật đơn giản AB (cảnh sát–máy xúc–cảnh sát–máy xúc) và hoàn thành chuỗi.
- Số 0–5 là phần làm quen cốt lõi; số 6–10 là phần mở rộng linh hoạt theo hứng thú, không ép trẻ phải thuộc ngay.
- Mỗi lượt nên ngắn khoảng 5–8 phút, thay đổi giữa nghe–nhìn–chạm–nói và hoạt động với đồ chơi thật; người lớn đồng hành, khen nỗ lực hơn là điểm số.

## Mở dự án và tạo APK

### Cách 1: Android Studio

1. Cài Android Studio mới, Android SDK Platform 35, JDK 17 và Gradle 8.9.
2. Mở thư mục `banh-bao-android` bằng Android Studio. Nếu Android Studio yêu cầu Gradle distribution, trỏ tới bản Gradle 8.9 đã cài trên máy.
3. Đồng bộ Gradle.
4. Chọn **Build → Build Bundle(s) / APK(s) → Build APK(s)**.
5. APK debug nằm tại `app/build/outputs/apk/debug/app-debug.apk`.

Lưu ý: gói mã nguồn này chưa chứa `gradle-wrapper.jar`; cách build ít thao tác nhất là dùng workflow GitHub Actions ở Cách 2.

### Cách 2: GitHub Actions

1. Tạo repository GitHub riêng và đẩy toàn bộ thư mục dự án lên nhánh `main`.
2. Vào tab **Actions**, chọn workflow **Build Bé Bánh Bao APK** và bấm **Run workflow** (workflow cũng chạy tự động khi push).
3. Mở lần chạy thành công và tải artifact `be-banh-bao-debug-apk`.

Workflow dùng Java 17, Gradle 8.9 và Android SDK. APK debug có thể cài trực tiếp để thử trên điện thoại Android; đây chưa phải bản release ký khóa riêng để phát hành Play Store.

## Bật AI chat thật

Phần app Android không chứa API key. Cần triển khai server trong thư mục `server/`:

```bash
cd server
npm install
# Thiết lập biến môi trường OPENAI_API_KEY ở môi trường máy chủ
# Có thể đặt OPENAI_MODEL để chọn model được tài khoản API hỗ trợ
npm start
```

Server cung cấp `POST /chat` và `GET /health`. Dùng URL HTTPS công khai, ví dụ `https://your-domain.example/chat`.

Khi build Android, truyền URL server:

```bash
gradle :app:assembleDebug -PCHAT_API_URL=https://your-domain.example/chat
```

Với GitHub Actions, tạo repository variable `CHAT_API_URL` có giá trị URL HTTPS của endpoint trên. **Không** đưa `OPENAI_API_KEY` vào repository variable dùng để build APK; key chỉ đặt trong môi trường deploy của server.

Nếu chưa có URL, app vẫn chạy bài học và chat mẫu offline; phần phản hồi AI thực sự chưa hoạt động cho tới khi server được cấu hình.

## Quyền riêng tư và an toàn cho trẻ

- Ứng dụng xin quyền micro đúng lúc bé bấm nút nói; có thể từ chối và dùng nhập văn bản.
- Android SpeechRecognizer có thể gửi âm thanh tới dịch vụ nhận dạng từ xa của thiết bị; app không tự lưu file ghi âm. Cần đọc chính sách của dịch vụ nhận dạng đang dùng trên điện thoại.
- Server chỉ nhận văn bản nhận dạng và lịch sử ngắn của phiên trong request; không lưu hội thoại vào DB. `store: false` được đặt khi gọi Responses API.
- Không đưa API key vào APK. Dùng HTTPS, giới hạn kích thước yêu cầu và giới hạn tần suất.
- AI chỉ là bạn học ảo, không thay thế ba mẹ/cô giáo. Người lớn nên ở cạnh khi trẻ dùng tính năng AI; không cho trẻ chia sẻ tên đầy đủ, trường học, địa chỉ, số điện thoại hoặc thông tin riêng tư.
- Trước khi phát hành công khai cần bổ sung cổng phụ huynh, điều khoản/chính sách quyền riêng tư, kiểm thử nội dung, cơ chế báo cáo và quy trình bảo vệ dữ liệu trẻ em phù hợp pháp luật nơi phát hành.

## Nguồn tham khảo nội dung học

- Kho học liệu số Giáo dục – Bộ Giáo dục và Đào tạo, lớp 3 tuổi: ghép đôi, xếp tương ứng 1–1, trò chơi hình vuông/hình tròn, nhận biết tam giác/chữ nhật: https://elearning.stid.gov.vn/hoc-lieu-so/Lop-3-Tuoi/
- UNICEF, *How to introduce maths to your toddler*: đếm đồ vật, nhận biết hình, phân loại và nhận ra mẫu: https://www.unicef.org/parenting/child-development/how-introduce-maths-your-toddler
- Chương trình giáo dục mầm non 3–4 tuổi (trang trường mầm non công lập): nhận biết bốn hình cơ bản, đếm trong phạm vi 5, xếp theo quy tắc đơn giản: https://mnanhmai.hcm.edu.vn/giao-duc-nuoi-duong/chuong-trinh-giao-duc-mam-non-moi-nhat-ke-hoach-thuc-hien-chuong-trinh-giao-duc/ctmb/20735/493864
- NAEYC, *Rocking and Rolling: Nurturing Early Math Play and Discovery*: dùng ngôn ngữ toán học qua trò chơi, chỉ tay theo từng vật khi đếm: https://www.naeyc.org/node/4685

---

**Trạng thái:** Mã nguồn MVP và workflow build đã được tạo. Bản APK chưa được biên dịch trong môi trường hiện tại vì không có Android SDK/Gradle cài sẵn; cần mở bằng Android Studio hoặc chạy workflow GitHub Actions để tạo APK.
