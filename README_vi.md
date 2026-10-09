<div align="center">

[한국어](README.md) · [简体中文](README_zh.md) · [繁體中文](README_zh-TW.md) · [日本語](README_ja.md) · [Türkçe](README_tr.md) · [Русский](README_ru.md) · **Tiếng Việt** · [ไทย](README_th.md) · [Deutsch](README_de.md) · [Español](README_es.md) · [Français](README_fr.md) · [Українська](README_uk.md) · [Polski](README_pl.md) · [Português (Brasil)](README_pt-BR.md) · [العربية](README_ar.md) · [فارسی](README_fa.md)

# 🫛 PEA

### Tác nhân nông trại thông minh — đọc dữ liệu tại khu canh tác, đưa ra phán đoán và hành động.

**Cảm biến vốn đã lên tiếng. PEA giúp bạn hiểu được điều chúng nói.**

[![license](https://img.shields.io/badge/license-MIT-f4c430?style=flat-square)](LICENSE)
[![status](https://img.shields.io/badge/status-concept-5c4ee5?style=flat-square)](#trạng-thái-dự-án)

[Bắt đầu nhanh](#bắt-đầu-nhanh) · [PEA làm được gì](#pea-làm-được-gì) · [Cách hoạt động](#cách-hoạt-động) · [Nguyên tắc an toàn](#nguyên-tắc-an-toàn) · [Lộ trình](#lộ-trình)

</div>

---

PEA (đậu) là một **tác nhân nông trại thông minh** dành cho nhà kính và khu canh tác ngoài trời. PEA thu thập các giá trị cảm biến như độ ẩm đất, nhiệt độ – độ ẩm, bức xạ mặt trời, CO2, EC/pH nước thoát, để giải thích *điều đang diễn ra ngay lúc này*, rồi điều khiển các thiết bị chấp hành như tưới nước, thông gió, che nắng, chiếu sáng bổ sung — **trong giới hạn an toàn**.

Mọi phán đoán đều đi kèm căn cứ. PEA phân biệt và báo cáo rõ **giá trị quan sát được**, **giá trị suy luận**, và **giá trị còn chưa biết**. "Đất đang khô" là dữ liệu từ lịch sử cảm biến, "chiều nay nguy cơ úng cao" là suy luận, còn "van tưới có thực sự đã mở hay chưa" vẫn là ẩn số cho tới khi đồng hồ lưu lượng xác nhận.

Công việc diễn ra trọn vẹn trong nông trại. Dữ liệu ra bên ngoài chỉ gồm những mục mà người dùng chủ động chọn.

## Bắt đầu nhanh

> **Các lệnh và tên gói dưới đây chưa phải là bản cuối cùng.** Hiện chưa có gói `pea` nào được phát hành trên npm. Khi phát hành, tài liệu này sẽ được cập nhật bằng các lệnh thực tế.

```bash
# 1) install (package not published yet)
npx pea-agents setup

# 2) readiness check — gateway, sensors and actuators
pea doctor

# 3) ask about today's greenhouse
pea analyze ./farm.toml
```

Sau khi kết nối với tác nhân, bạn chỉ cần hỏi như sau.

```text
Nhà kính số 3 sáng nay tưới nhiều hơn mục tiêu 12%. Nguyên nhân là gì?
Muốn đưa tỷ lệ nước thoát về 25% thì chiều nay phải đổi lịch tưới thế nào?
Cho tôi xem thời gian lá ẩm ướt kéo dài bao nhiêu giờ theo nhiệt độ thấp nhất đêm qua.
```

## PEA làm được gì

| Lĩnh vực | PEA làm gì | Dữ liệu sử dụng |
| --- | --- | --- |
| Môi trường sinh trưởng | Hiển thị diễn biến nhiệt độ – độ ẩm trong nhà kính, VPD, bức xạ tích lũy và các khoảng bất thường | Cảm biến nhiệt ẩm, nhật xạ kế |
| Tưới nước – dung dịch dinh dưỡng | Tính lượng tưới dựa trên bức xạ tích lũy, cảnh báo độ lệch so với mục tiêu tỷ lệ nước thoát | Cảm biến độ ẩm – khối lượng đất, EC/pH nước thoát |
| Thông gió – làm mát/sưởi ấm | Điều khiển giữ VPD theo mục tiêu, đề xuất mức mở cửa sổ và quạt | Phản hồi độ mở cửa sổ, thời tiết bên ngoài |
| Cảnh báo sâu bệnh | Tính mức rủi ro bệnh nấm từ thời gian lá ẩm ướt kết hợp nhiệt độ – độ ẩm | Lịch sử nhiệt ẩm, cảm biến độ ẩm lá |
| Nhật ký sinh trưởng | Tự động ghi lại lịch sử tưới, môi trường, công việc; hỗ trợ nhập số liệu khảo sát sinh trưởng | Nhật ký công việc, ảnh hiện trường |
| Dự đoán thu hoạch | Ước tính thời điểm thu hoạch theo nhiệt độ tích lũy, so sánh giữa các vụ | Lịch sử nhiệt độ, tham số giống cây |

Các mục và ngưỡng trong bảng là **những giá trị cần hiệu chuẩn lại theo từng cây trồng, giống và vùng**. PEA chỉ đề xuất giá trị mặc định, chứ không mặc định coi đó là giá trị chuẩn của nông trại đó.

## Cách hoạt động

Khi tác nhân hỏi PEA qua MCP, PEA sẽ thu thập dữ liệu từ gateway rồi trả về kết quả kèm căn cứ. Từ kết quả đó, tác nhân giải thích tình hình hoặc đưa ra phương án điều khiển. Cùng một quy trình công việc cũng được dùng y nguyên trên CLI trong terminal.

```text
tác nhân → MCP → PEA → gateway → cảm biến
                     ← giá trị quan sát + suy luận + chưa biết
```

- **Ưu tiên cục bộ** — việc thu thập và phán đoán đều được xử lý ngay trong nông trại.
- **Ưu tiên căn cứ** — mỗi kết luận đều lưu lại giá trị gốc, thời điểm và nguồn.
- **Chưa biết thì nói là chưa biết** — những khoảng không có cảm biến hoặc mất kết nối không bị lấp bằng phỏng đoán mà được nêu rõ.
- **Kết quả như nhau** — CLI và MCP dùng chung một quy trình và một hợp đồng kết quả.

## Phạm vi hỗ trợ

| Kết nối | Cách thức | Trạng thái |
| --- | --- | --- |
| Gateway MQTT | Thu thập topic cảm biến bằng cách đăng ký với broker | Dự kiến |
| Modbus RTU / TCP | Poll thanh ghi của bộ điều khiển | Dự kiến |
| LoRaWAN | Cảm biến ngoài trời qua gateway | Dự kiến |
| Dữ liệu khí tượng | API khí tượng khu vực, nhiệt độ – độ ẩm – lượng mưa bên ngoài | Dự kiến |
| Hệ thống quản lý canh tác hiện có | Nhập tệp (CSV/JSON) để hợp nhất lịch sử | Dự kiến |

## Nguyên tắc an toàn

Việc điều khiển thiết bị chấp hành là không thể hoàn tác. Vì vậy PEA tuân thủ những điều sau.

1. **Mặc định cần con người phê duyệt** — các hành động thực tế như tưới, thông gió, chiếu sáng bổ sung đều phải in kế hoạch ra trước và chờ phê duyệt.
2. **Giới hạn cứng** — các ngưỡng đặt cho nông trại (lượng tưới tối đa mỗi lần, dải nhiệt độ cho phép, v.v.) không thể bị vượt qua bởi phán đoán của tác nhân.
3. **An toàn khi có sự cố** — khi mất kết nối hoặc mất điện, hệ thống rơi về phía an toàn (dừng tưới, mở thông gió, v.v.).
4. **Không tự động phun rải** — PEA không thực hiện việc phun thuốc bảo vệ thực vật hay phân bón, mà chỉ đề xuất thời điểm và căn cứ.
5. **Ghi lại toàn bộ** — mọi lệnh điều khiển đều lưu lại đối tượng, thời điểm, căn cứ và kết quả thực thi.

## Lộ trình

- **M0 — Ý tưởng và tài liệu** *(hiện tại)*: định nghĩa sản phẩm, nguyên tắc an toàn, thiết kế hợp đồng kết quả
- **M1 — Thu thập**: kết nối gateway, lưu lịch sử cảm biến, tra cứu diễn biến môi trường
- **M2 — Tư vấn**: cố vấn tưới tiêu – thông gió, rủi ro sâu bệnh, tự động hóa nhật ký sinh trưởng
- **M3 — Điều khiển có phê duyệt**: điều khiển tự động và cơ chế an toàn khi có sự cố dưới sự phê duyệt của con người
- **M4 — Vận hành**: dự đoán thu hoạch, so sánh nhiều nông trại, báo cáo theo vụ

## Trạng thái dự án

Tài liệu này mô tả **ý tưởng sản phẩm của PEA**. Mã nguồn hiện có trong bản checkout này là công cụ dịch ngược (`rea-agents`) và chưa triển khai các tính năng thu thập – điều khiển nêu trên. Tên lệnh, danh sách kết nối và các ngưỡng sẽ được chốt cùng với quá trình triển khai. Các README bản dịch vẫn dựa trên tài liệu trước đó, nên sẽ lần lượt được chỉnh lại cho khớp với việc đổi thương hiệu.

## Đóng góp

Chúng tôi hoan nghênh các issue và pull request. Những giá trị quan sát và ca thất bại thu được tại hiện trường, cùng các giá trị hiệu chuẩn theo từng cây trồng, đặc biệt hữu ích.

## Giấy phép

[MIT](LICENSE)
