# cua-chrome (skill)

Biến MCP `cua_direct` thành **skill**: Claude điều khiển Chrome thật của bạn bằng lệnh Bash (`scripts/cua.py`), không cần đăng ký MCP, không cần `.venv`, không cần cài thư viện (chỉ Python 3.9+ chuẩn).

MCP cũ ở `../cua-direct-mcp/` **không bị thay đổi**. Skill dùng một **bản sao** của proxy (`scripts/cua_proxy.py`, giống hệt `cua_direct_proxy.py`).

## Cài đặt (mỗi máy một lần)

Chép thư mục `cua-chrome/` vào thư mục skill của Claude:

| Phạm vi | Chép vào |
|---|---|
| Mọi dự án (khuyên dùng) | `C:\Users\<tên>\.claude\skills\cua-chrome\` |
| Chỉ một dự án | `<dự án>\.claude\skills\cua-chrome\` |

PowerShell:

```powershell
Copy-Item -Recurse ".\cua-chrome" "$env:USERPROFILE\.claude\skills\cua-chrome"
```

Hoặc dùng file `cua-chrome.skill` (zip của thư mục trên). Mở phiên Claude Code mới là skill xuất hiện; gõ `/cua-chrome` hoặc nói "mở Chrome ..." là nó được dùng.

## Yêu cầu

- Ứng dụng Codex đã cài plugin **unified-computer-use** (proxy lấy runtime từ `~/.codex/plugins/cache/openai-bundled/unified-computer-use`).
- Chrome mở với extension **ChatGPT** bật, đúng profile bạn muốn điều khiển.
- Kiểm tra: `python ~/.claude/skills/cua-chrome/scripts/cua.py doctor`

## Khác biệt so với MCP

| | MCP `cua_direct` | Skill `cua-chrome` |
|---|---|---|
| Cách Claude dùng | Gọi tool (60 tool) | Chạy lệnh Bash `cua.py <lệnh>` |
| Cài đặt | Sửa file cấu hình MCP | Chép thư mục |
| Chi phí ngữ cảnh | Schema của 60 tool luôn nằm trong ngữ cảnh | Chỉ nạp hướng dẫn khi cần |
| Giữ tab giữa các lệnh | Tiến trình MCP sống | Daemon nền tự khởi động (tự tắt sau 20 phút rảnh, `cua.py stop`) |
| Ảnh chụp | Trả ảnh về cuộc trò chuyện | Lưu ra file `.jpg`, Claude mở bằng công cụ Read |
| Chọn profile Chrome | `browser_select` | `--browser ext:<mã>` / `use-browser` (mã ổn định, id số thì đổi mỗi lần) |

## Cấu trúc

```
cua-chrome/
├── SKILL.md                  # hướng dẫn Claude (ngắn)
├── scripts/
│   ├── cua.py                # CLI: 75 lệnh
│   ├── cua_proxy.py          # bản sao proxy (không sửa)
│   └── extract_meta_cards.js # trích thẻ quảng cáo từ Meta Ad Library
└── references/
    ├── commands.md           # toàn bộ lệnh và tham số
    ├── recipes.md            # quy trình nhiều bước (nghiên cứu, form, Ad Library, tab...)
    └── limits.md             # giới hạn và cách xử lý lỗi
```

## Nghiên cứu quảng cáo và xu hướng (playbook)

Mỗi playbook tự mở tab riêng, đọc, đóng tab và trả về số liệu đã xử lý:

| Lệnh | Lấy gì |
|---|---|
| `market-scan --query Q --advertiser BRAND --out DIR` | chạy tất cả bên dưới cho một từ khóa, ra thư mục JSON + `summary.md` |
| `meta-ads` | Meta Ad Library: quảng cáo đang chạy, nhà quảng cáo, ngày bắt đầu, nội dung, CTA, video + khung hình |
| `tiktok-ads` | TikTok Ad Library: nhà quảng cáo, ngày hiển thị, số người xem, nhắm mục tiêu, đơn vị trả tiền, link đích; `--top-ads`: likes/CTR/ngân sách từ Creative Center |
| `google-ads` | Google Ads Transparency: nhà quảng cáo (số QC, xác minh), creative, lần hiển thị gần nhất, định dạng, biến thể, quốc gia |
| `tiktok-trends` | video TikTok: thích/bình luận/chia sẻ/lưu, tương tác mỗi ngày, hashtag hàng đầu, hashtag thịnh hành |
| `yt-search` | YouTube: lượt xem, tuổi video, lượt xem/ngày (độ hot) |
| `google-trends` | gợi ý từ khóa Google + YouTube, từ khóa dài a–z, từ bổ nghĩa phổ biến, xu hướng tìm kiếm trong ngày theo nước |
| `meta-trends` | video Facebook: bài, caption, hashtag |
| `play-profile` | Google Play: điểm, đánh giá, lượt tải, nhà phát triển |
| `appstore-profile` | App Store iOS (API công khai của Apple): điểm, số lượt đánh giá, nhà phát triển, thể loại, ngày cập nhật |
| `ads-watch run NAME` | theo dõi theo thời gian: gộp các lần quét Meta/TikTok/Google, ghi lần thấy đầu/cuối, số lần thấy, quảng cáo mới ra (theo ngày bắt đầu), quảng cáo không còn thấy |
| `creative-tag` | chuẩn bị video + ảnh 8 khung hình, Claude xem và gắn nhãn hook/gameplay/visual/CTA/cảm xúc vào `creative_tags.csv`, rồi thống kê |
| `report --from DIR` | một file HTML tự chứa: biểu đồ, bảng sắp xếp được, khung hình, nhãn creative, lịch sử ads-watch |

Giới hạn: không thư viện nào công bố chi tiêu; TikTok Ad Library chủ yếu có QC ở châu Âu; Google Trends Explore thường không trả số cho phiên tự động; Facebook không hiện lượt xem trong kết quả tìm kiếm; TikTok cần profile đã đăng nhập TikTok.

## Quy tắc dọn tab

Skill ghi nhớ mọi tab và file tạm nó tạo (ảnh chụp để phân tích, video, khung hình). Khi xong việc Claude phải chạy `cua.py cleanup` (hoặc `stop`): đóng hết tab đã mở và xóa file tạm. Tab cần giữ thì `mark deliverable|handoff`; file cần giữ thì lưu bằng `--out`/`--csv` hoặc `artifacts keep`. Tab bạn mở sẵn không bao giờ bị đóng. File tạm quá 24 giờ và log quá 7 ngày tự xóa. Các thao tác rủi ro (gửi, thanh toán, xóa, upload, xem lịch sử, đóng tab của bạn) bị chặn nếu chưa có `--confirmed`. Tiến trình nền chạy ẩn, không hiện cửa sổ.

Dữ liệu trạng thái (profile mặc định, tab hiện tại, danh sách tab đã mở, ảnh chụp mặc định) nằm ở `~/.cua-chrome/`.
