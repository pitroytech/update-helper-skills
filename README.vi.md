<p align="right">
  <a href="README.md">🇺🇸 English</a> &nbsp;|&nbsp;
  <b>🇻🇳 Tiếng Việt</b>
</p>

# 🛠️ Update Helper v5 — by PitroyTech

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-5.0-blue.svg)](SKILL.md)

> **Cách an toàn và hiệu quả để AI agent cập nhật code có sẵn.**
> Không patch mù. Không vỡ build. Không đốt context window.

**Tìm trước. Đọc ít hơn. Patch gọn. Verify kỹ. Rollback sạch.**

---

## 🤔 Vấn đề là gì?

AI agent viết code mới thì nhanh. Nhưng tìm và sửa code có sẵn lại là câu chuyện khác.

**Không có protocol, agent rất dễ:**

* ❌ Đọc cả codebase lớn (15,000–30,000 dòng) để tìm một function → context window cháy, chưa patch gì đã hết token
* ❌ Xóa một nút trên UI nhưng quên mất đoạn code phía sau vẫn đang lắng nghe sự kiện của nút đó → app chạy nhưng lỗi âm thầm
* ❌ Sửa file build output thay vì file source gốc → test thấy OK, nhưng build lại là mất hết
* ❌ API đã trả về dữ liệu đúng, nhưng agent vẫn báo lỗi và đi đổi model, đổi key → mất hàng giờ truy tìm sai chỗ
* ❌ Ghi file bằng lệnh sai → BOM bị mất → tiếng Việt/CJK/emoji vỡ hết mà không có cảnh báo
* ❌ Áp dụng mô tả của user vào code, nhưng code thực tế đã được refactor từ lâu → patch sai logic

**Với Update Helper:**

* ✅ Tìm anchor → đọc đúng 40–160 dòng xung quanh, không đụng đến phần còn lại
* ✅ Map toàn bộ render → handler → state → config → dist trước khi xóa bất cứ thứ gì
* ✅ Phân biệt SOURCE vs GENERATED → chỉ patch source, rebuild artifact sau
* ✅ Tách rõ 3 tầng: request gửi đi / response nhận về / xử lý sau khi nhận → tìm đúng chỗ fail
* ✅ Kiểm tra BOM/encoding trước khi ghi → ghi đúng encoding → verify sau khi ghi
* ✅ So sánh mô tả của user với code thực tế → hỏi trước khi apply nếu khác nhau

---

## 🔄 Nó đổi cách agent làm việc thế nào?

```
User: "fix this" / "xóa UI cũ" / "patch vừa rồi làm hỏng rồi"
        ↓
Update Helper
        ↓
[Lite]  Tìm anchor → đọc range → xác định owner →
        nói rõ sẽ sửa gì → backup nếu cần → patch →
        syntax check → invariant search → báo cáo

[Full]  + phân loại source-of-truth → trace data flow →
        đánh giá blast radius → ghi đúng encoding →
        checklist UI symmetry → cleanup backup
        ↓
Kết quả: đúng file, đúng luồng, build pass, invariant sạch, còn đường rollback
```

Điểm khác biệt là agent không cố "nhìn toàn cảnh" ngay từ đầu. Nó tìm một anchor đủ tốt, đọc đúng vùng liên quan, rồi mở rộng chỉ khi blast radius thật sự lớn.

**Lite** — khi task nhỏ, rõ ràng, 1 file, không có encoding risk hay generated/source nhầm lẫn.

**Full Protocol** — khi file lớn, nhiều module liên quan, encoding risk, refactor, mô tả có thể stale, hoặc agent/người khác đã sửa repo.

---

## 🧭 Agent sẽ làm gì khác?

| Case | Agent thường làm | Với Update Helper |
|---|---|---|
| Bug UI | Sửa phần đang nhìn thấy | Trace render → handler → state → config trước khi patch |
| Bug provider/API | Đổi model, đổi key | Tách request gửi đi / response nhận về / xử lý sau response |
| File generated | Sửa file đang chạy | Tìm source-of-truth (nguồn thật), patch source, rebuild output |
| File lớn | Đọc quá nhiều | Search anchor, đọc bounded range (vùng nhỏ quanh điểm nghi vấn) |
| File có tiếng Việt/CJK | Replace bằng tool tiện tay | Check encoding/BOM, dùng anchor ổn định, preserve encoding |
| Patch hỏng | Chồng thêm workaround | Restore `.bak2`, đọc lại range, patch nhỏ hơn |

---

## ⚡ Cài đặt

### Antigravity / OpenClaw (thư mục skills)

Clone repo:

```bash
git clone https://github.com/pitroytech/update-helper-skills.git
```

Copy vào thư mục skills của agent:

```bash
xcopy /E /I update-helper-skills\skills\update-helper %USERPROFILE%\.gemini\antigravity\skills\update-helper
```

### Claude Code / Cursor / Cline (file `.skill`)

1. Download [`update-helper.skill`](update-helper.skill) từ repo này.
2. Import vào skill manager của agent.

### Thủ công (bất kỳ agent nào)

Copy nội dung [`skills/update-helper/SKILL.md`](skills/update-helper/SKILL.md) vào system prompt hoặc `AGENTS.md` của project.

Khi skill được nạp đúng cách, agent sẽ tự kích hoạt protocol khi gặp trigger: `fix this`, `remove old UI`, `refactor`, `port this`, `last patch broke it`.

---

## 📊 So sánh

| Không có Update Helper | Có Update Helper |
|---|---|
| Đọc cả file lớn để tìm 1 chỗ sửa | Tìm anchor → đọc bounded range |
| Xóa UI, nhưng code xử lý sự kiện vẫn còn | Checklist đầy đủ: render + CSS + binding + config + dist |
| Sửa file output vì đó là file đang chạy | Phân loại source-of-truth → patch source → rebuild |
| Thấy lỗi → đổi model, đổi key | Tách tầng request/response/xử lý → tìm đúng chỗ fail |
| Encoding bị hỏng, không biết tại sao | Detect BOM trước khi ghi, verify sau khi ghi |
| Agent mới vào → đọc lại toàn bộ từ đầu | Đọc map/backup/KI hiện có → làm tiếp ngay |
| Áp spec cũ vào code đã refactor | So sánh spec vs thực tế, hỏi trước khi merge |
| Backup lúc thì quên, lúc thì để đầy rác | `.bak2` mỗi lần ghi + session backup + cleanup rõ ràng |

---

## 🎯 Các tình huống thực tế

### Tình huống 1: Xóa tính năng cũ trên UI

> Task: bỏ nhóm tùy chỉnh Batch khỏi trang cài đặt API.

Agent không có skill chỉ xóa phần HTML nhìn thấy. Kết quả: ô input biến mất trên màn hình, nhưng code phía sau vẫn đọc giá trị cũ, gây lỗi khi lưu cài đặt.

Agent có Update Helper chạy checklist đầy đủ:

```
Tìm: batch-gemini, batch-zhipu, api-advanced
Đọc: block render, CSS, handler Gemini, handler Zhipu, hàm lưu config chung
Patch: xóa control hiển thị + nút mặc định + đọc giá trị trong save + handler null-crash
Verify: npm run verify
Kiểm tra: rg "batch-gemini|batch-zhipu|api-advanced" src dist → 0 kết quả
```

### Tình huống 2: Sửa nhầm file output thay vì file source

> Task: thay đổi cài đặt nhưng userscript không phản ánh thay đổi.

Agent không có skill tìm file đang chạy rồi sửa thẳng vào đó. Test thấy OK. Build lại — mất hết.

Agent có Update Helper tìm build script trước, phân loại file:

```
src/module.js          → SOURCE — đây là file cần sửa
dist/app.user.js       → GENERATED — build output, không sửa trực tiếp
src/app.user.js        → REFERENCE — bản monolith cũ, không động vào

→ Patch source → chạy npm run build → kiểm tra dist
```

### Tình huống 3: Dropdown mất model sau khi test provider

> Task: test Groq xong rồi chạy RACE với Gemini, dropdown chỉ còn hiện model của Gemini.

Agent không có skill nhảy vào sửa label dropdown. Không có tác dụng vì label không phải nguyên nhân.

Agent có Update Helper trace theo luồng dữ liệu:

```
RACE → providerModelPool[provider] → sync flat modelPool → dropdown → scheduler

→ Phát hiện: RACE đang overwrite toàn bộ modelPool thay vì chỉ cập nhật provider đó
→ Fix đúng chỗ: RACE chỉ update providerModelPool[provider], dropdown hydrate từ aggregate store
```

### Tình huống 4: API trả dữ liệu đúng nhưng vẫn báo lỗi

> Task: log đã thấy response hợp lệ, nhưng scheduler vẫn báo failed.

Agent không có skill tiếp tục đổi model, test key, đổi provider. Không giải quyết được.

Agent có Update Helper tách 3 tầng:

```
Request gửi đi?  → Có ✅
Response nhận về? → Có, JSON hợp lệ ✅
Xử lý sau khi nhận? → Crash ở bước apply translation ❌

→ Root cause: hàm apply bị xóa trong lần refactor trước, chưa cập nhật call site
```

---

## 📸 Kết quả thực tế

### ❌ Trước — agent không có skill (12 lần tìm kiếm thất bại liên tiếp)

<img src="images/before-12-searches.png" width="33%" alt="Agent không có skill: 12 lần tìm kiếm thất bại">

Agent dùng `grep_search` không xử lý được file UTF-8 BOM 14,000 dòng. Kết luận: *"file có vấn đề encoding, thử cách khác"* → đoán mò, đốt token.

### ✅ Sau — agent có Update Helper (3 lần tìm kiếm, ra root cause)

<img src="images/after-trace-success.png" width="33%" alt="Agent có skill: tìm đúng bug ngay lập tức">

Chi tiết quá trình Data Flow Tracing:

<img src="images/comparison-table.png" width="33%" alt="Quá trình Data Flow Tracing chi tiết">

```
Tìm "Reset-LocalAccountPassword"     → line 6732  ✅
Trace caller                          → line 12273 ✅
Root cause: $acc.Username rỗng → ConvertMsa crash ✅
```

### 📋 Bảng so sánh nhanh

| | Không có skill | Có Update Helper |
|---|---|---|
| Công cụ | `grep_search` (IDE) | `Select-String` (PowerShell native) |
| Số lần tìm | 12+ lần thất bại | 3 lần → tìm đúng bug |
| Phương pháp | Mò mẫm random | Search → Read → Understand → Trace |
| Kết quả | "encoding issue" | Root cause + 3 bugs identified |
| Encoding | Không handle UTF-8 BOM | BOM preserved correctly |

---

## 💰 Tiết kiệm token thực tế

<img src="images/token-savings.png" width="33%" alt="Ước tính tiết kiệm token với Update Helper v5">

| Loại việc | Không có protocol | Có Update Helper |
|---|---|---|
| File JS 500KB+ | Dễ đọc 100k–180k token | Anchor + bounded range → ~15k–30k token |
| Xóa UI | 2–4 vòng vì sót handler | 1 vòng nếu checklist đủ |
| Source/dist nhầm | Patch sai file, test pass, build mất fix | Phân loại trước → patch đúng |
| Encoding hỏng | Có thể mất cả phiên để recover | Detect trước, restore path rõ |
| Multi-agent handoff | Agent sau đọc lại từ đầu | Tiếp từ map/backup hiện có |

Agent mạnh vẫn hưởng lợi — không phải vì không biết code, mà vì protocol giữ nó khỏi những sai nhỏ nhưng đắt.

---

## ✅ Đã kiểm chứng trên

* Userscript 15,000+ dòng: provider routing, scheduler, model pool, floating settings UI
* Công cụ dịch thuật dùng Gemini, Groq, OpenRouter, SambaNova
* File PowerShell/JS có tiếng Việt, CJK, emoji — UTF-8 BOM và no-BOM
* Repo có `src/`, `dist/`, generated userscript, monolith reference cũ
* Multi-agent session: một agent map, một agent patch, một agent recover

Mỗi rule trong skill này tồn tại vì đã từng có một session thiếu nó và đi sai.

---

## 🧩 Trong `SKILL.md` có gì?

README này là bản giới thiệu cho con người. `SKILL.md` là bản agent đọc khi làm việc thật.

| Tính năng | Mô tả |
|---|---|
| **Tool Cheat Sheet** | Bảng lệnh nhanh dual-platform: Linux/Claude Code và Windows/PowerShell |
| **Lite Workflow** | 9 bước cho task rõ ràng, có ví dụ command từng bước |
| **Full Protocol** | Dùng khi file lớn, nhiều module, encoding risk, refactor, hoặc patch trước bị hỏng |
| **Source-of-Truth Detection** | Phân biệt source, generated output, reference cũ |
| **Pre-Submit Checklist** | 11 mục kiểm tra bắt buộc trước khi báo patch done |
| **UI Removal Checklist** | Render, CSS, handler, config, status, consumers, dist |
| **Refactor & Port Protocol** | Leaf → caller → parent → entry point; port workflow 5 bước |
| **Backup Cleanup** | Quy trình riêng cho git workspace và non-git workspace |
| **Failure Recovery** | Restore `.bak2`, đọc lại range, patch nhỏ hơn |
| **Multi-Agent Handoff** | Đọc map/backup/KI trước khi tiếp tục, không ghi đè backup |
| **Final Report** | File đổi, behavior, verification, backup state, risk còn lại |

---

## 🔖 Lịch sử phiên bản

| Phiên bản | Thay đổi |
|---|---|
| **v5.0** | Tách Lite/Full. Source-of-truth detection. Tool cheat sheet + chẩn đoán str_replace. Pre-submit checklist. UI removal checklist. Refactor/port protocol. Backup cleanup cho git và non-git. Kiểm tra feature_map/KI trong multi-agent. Ví dụ command dual-platform toàn bộ. |
| v4.0 | Self-contained. Encoding-safe write .NET. Multi-agent onboarding. Cascade analysis. Proactive bug hunt. |
| v3.4 | Phát hiện spec stale. Cải thiện BOM write pattern. |
| v3.3 | Data-flow tracing. Architecture summary. JS UI patterns. |
| v3.x | Các bản public đầu tiên. |

---

## 📄 Giấy phép

MIT License. **Ý tưởng, thiết kế và nội dung © PitroyTech.**

Xây dựng từ thực chiến — không phải từ lý thuyết về cách agent nên làm việc.
