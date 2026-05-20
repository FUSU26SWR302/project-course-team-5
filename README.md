HỆ THỐNG ĐẶT VÉ THAM QUAN DU LỊCH

1. Hệ thống này TẬP TRUNG vào cái gì? (Core Focus)
Dự án này không tập trung vào việc tạo ra các màn hình nhập liệu thông thường (CRUD), mà tập trung giải quyết 3 bài toán kỹ thuật cốt lõi:

Xử lý đồng thời cao (High Concurrency & Anti-Overbooking): Tập trung vào việc kiểm soát số lượng vé tồn kho (Ticket Availability) theo thời gian thực. Đảm bảo khi có hàng ngàn khách hàng cùng nhấn "Đặt vé" vào một thời điểm (ví dụ: ngày lễ hoặc flash sale), hệ thống không bị lỗi Race Condition (tranh chấp dữ liệu) dẫn đến bán quá số lượng vé thực tế.

Toàn vẹn giao dịch và bảo mật (Transaction Integrity): Tập trung xử lý luồng thanh toán (Payment Flow) bất đồng bộ thông qua Webhook với bên thứ ba, đảm bảo trạng thái vé và dòng tiền đồng bộ chính xác 100%, không bị treo hay mất dữ liệu khi mạng lỗi.

Cá nhân hóa trải nghiệm bằng AI (AI Personalization): Tập trung vào việc ứng dụng AI để phân tích nhu cầu và tự động thiết kế lịch trình (Generate Visit Plan), gợi ý gói vé phù hợp thay vì để người dùng phải tự tìm kiếm thủ công.

2. Hệ thống sử dụng KIẾN TRÚC nào? (System Architecture)
Để giải quyết các bài toán trên, hệ thống áp dụng mô hình Kiến trúc phân tầng (Layered Architecture - MVC) kết hợp với Kiến trúc hướng sự kiện (Event-Driven Architecture):

Kiểu kiến trúc chính:
Modular Monolith (Kiến trúc đơn khối phân rã module): Hệ thống được thiết kế chung một mã nguồn để dễ triển khai cho team 3 người, nhưng các module (Booking, AI, Partner, Auth) được tách biệt hoàn toàn về mặt logic (lớp Controller, Service, DAL) để dễ dàng nâng cấp lên Microservices sau này.

Event-Driven Integration (Tích hợp hướng sự kiện): Đối với các tác vụ tốn thời gian và không cần phản hồi ngay (như gửi email vé QR, bắn thông báo promotion, xử lý log), hệ thống sẽ đẩy vào một Message Queue (Hàng đợi tin nhắn) để xử lý bất đồng bộ, giúp giải phóng tài nguyên cho luồng chính (đặt vé và thanh toán) luôn mượt mà.

Các Design Patterns (Mẫu thiết kế) áp dụng:
Strategy Pattern: Áp dụng cho module Thanh toán (Process Online Payment) để dễ dàng hoán đổi cấu trúc giữa các cổng thanh toán (VNPAY, MoMo, Stripe) và module Khuyến mãi (Apply Voucher) để áp dụng các thuật toán giảm giá khác nhau (giảm theo % hoặc giảm số tiền cố định).

Observer Pattern: Áp dụng cho Notification Service nhằm tự động kích hoạt gửi Email/SMS khi có sự thay đổi trạng thái từ các module khác (Đặt vé thành công, Hoàn tiền thành công).

Factory Pattern: Dùng để khởi tạo các loại Vé điện tử/Mã QR tùy thuộc vào loại hình địa điểm (Vé vào cổng một lần, Vé combo, Vé theo khung giờ).

3. Hệ thống sử dụng CÔNG NGHỆ gì? (Technology Stack)
Bộ công nghệ được chọn lọc kỹ càng để đảm bảo tính an toàn dữ liệu (ACID) của một hệ thống thương mại điện tử:

Backend & Core Logic
Ngôn ngữ chính: Java (JDK 17) – Đảm bảo hiệu năng cao, xử lý đa luồng (Multi-threading) tốt và tính hướng đối tượng chặt chẽ.

Framework: Spring Boot (hoặc Java Web servlet/JSP chạy trên nền Apache Tomcat), sử dụng Spring Security để phân quyền nghiêm ngặt (Role-based Access Control) giữa Admin, Partner, Support và Customer.

Cơ sở dữ liệu & Caching (Database Layer)
Database chính: SQL Server / PostgreSQL (Sử dụng các tính năng nâng cao của T-SQL như Transactions, Stored Procedures, Triggers) để đảm bảo tính toàn vẹn dữ liệu cho các luồng thanh toán và quản lý doanh thu.

Caching & Locking: Redis – Sử dụng để lưu trữ các thông tin ít thay đổi nhưng truy cập nhiều (danh sách địa điểm, thông tin chi tiết vé) giúp tăng tốc độ tải trang. Đặc biệt, dùng Redis Distributed Lock để khóa tạm thời số lượng vé trong kho trong 5 phút khi khách hàng đang tiến hành thanh toán, chặn đứng nguy cơ overbooking.

Tích hợp trí tuệ nhân tạo (AI Integration)
AI Engine: Tích hợp OpenAI API (GPT-4o) hoặc Gemini API thông qua framework LangChain4j (Java).

Kỹ thuật xử lý: Áp dụng kiến trúc RAG (Retrieval-Augmented Generation) để nhúng (Embedding) dữ liệu địa điểm du lịch từ database vào AI. Nhờ đó, Trợ lý AI chỉ trả lời và lên lịch trình dựa trên các địa điểm thực tế có trên sàn, không bị hiện tượng "ảo tưởng/bịa thông tin" (AI Hallucination).

Giao diện & Tiện ích (Frontend & Security)
Web App: React.js hoặc Next.js (hoặc HTML5/CSS3/Bootstrap kết hợp mã nhúng JSP nâng cao) để tối ưu hóa SEO và tốc độ hiển thị bộ lọc địa điểm du lịch.

QR Code Generator: Sử dụng thư viện ZXing (Zebra Crossing) để mã hóa thông tin vé thành QR Code động (chứa chữ ký số bảo mật JWT để chống làm giả vé).

Tóm lại cho báo cáo RBL: Hệ thống này tập trung vào Hiệu năng xử lý đồng thời và Tính thông minh (AI); vận hành trên kiến trúc Modular Monolith kết hợp Event-Driven; và được hiện thực hóa bằng sức mạnh bảo mật của Java, tính toàn vẹn của SQL Server, tốc độ của Redis, cùng khả năng tư duy đột phá của Generative AI (RAG).