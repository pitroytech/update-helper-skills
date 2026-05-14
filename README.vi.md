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

AI agent viết code mới rất nhanh. Nhưng cập nhật code có sẵn lại khó hơn nhiều.

Các dự án thực tế chứa đầy những "khế ước ngầm" (hidden contracts): state cũ, các UI handler, file config, dữ liệu cache, build artifact, quy tắc encoding, giả định trong test, và những quyết định được đưa ra bởi những người không còn làm trong dự án. Nếu không có một giao thức chuẩn, agent có thể tự tin tung ra một bản patch trông có vẻ đúng ở hiện tại nhưng lại làm hỏng toàn bộ luồng làm việc thực tế.

Nếu không có Update Helper, agent thường sẽ:

❌ Patch nhầm nguồn chân lý (source of truth): sửa file đang nhìn thấy, file output được copy, reference lỗi thời, artifact tự sinh (generated), hoặc tầng wrapper trong khi hành vi thực sự lại nằm ở chỗ khác

❌ Truy đuổi sai tầng: triệu chứng nhìn thấy ở một nơi, nhưng lỗi thực sự nằm ở khâu thu thập đầu vào (input collection), biến đổi dữ liệu, kiểm tra hợp lệ (validation), cập nhật state, render, lưu trữ (persistence), hoặc xử lý hậu kỳ (post-processing)

❌ Làm hỏng các file nhạy cảm với văn bản do ghi bằng lệnh hoặc encoding sai, âm thầm phá hủy BOM, tiếng Việt, CJK, emoji, hoặc nội dung đa ngôn ngữ

❌ Để lại các file backup, các đoạn patch thất bại, bản copy tạm thời, hoặc các artifact cập nhật dang dở rải rác khắp repo cho đến khi không ai biết cái nào an toàn để xóa

❌ Đọc từ 15.000 đến 30.000 dòng code chỉ để tìm một hàm, đốt cháy context window, và rốt cuộc vẫn bỏ lỡ luồng xử lý chính

❌ Sửa đổi UI mà không trace theo luồng render -> handler -> state -> config, để lại những hành vi lỗi ngầm bên trong

❌ Áp dụng một "mental model" (mô hình tư duy) lỗi thời: tài liệu cũ, comment cũ, kế hoạch cũ, ghi chú của agent trước đó, hoặc cấu trúc nhớ trong đầu không còn khớp với code hiện tại

Với Update Helper, agent làm việc hoàn toàn khác:

✅ Tìm đúng nguồn chân lý thực sự trước khi sửa: file source, output tự sinh, config, runtime state, wrapper, hoặc file reference

✅ Xác định chính xác tầng bị lỗi trước khi patch: collect -> transform -> validate -> call -> parse -> apply -> persist -> render

✅ Bảo toàn encoding và verify kỹ các file nhạy cảm với văn bản sau khi ghi

✅ Giữ một đường lui (rollback path) trong khi patch, và chỉ dọn dẹp backup sau khi quá trình verify thành công

✅ Tìm các anchor (điểm neo), sau đó chỉ đọc 40-160 dòng thực sự quan trọng thay vì nuốt chửng cả repo

✅ Map lại toàn bộ hành vi trước khi xóa hoặc đổi tên bất cứ thứ gì: render -> handler -> state -> storage -> verification

✅ So sánh những giả định với code hiện tại và coi sự sai lệch là bằng chứng, chứ không phải là nhiễu

---

## 🔄 Agent làm việc khác thế nào?

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

**Lite** — task nhỏ, rõ ràng, 1 file, không có encoding risk hay generated/source nhầm lẫn.

**Full Protocol** — file lớn, nhiều module, encoding risk, refactor, spec có thể stale, hoặc agent/người khác đã sửa repo.

| Tình huống | Agent thường làm | Với Update Helper |
|---|---|---|
| Bug UI | Sửa phần đang nhìn thấy | Trace render → handler → state → config trước khi patch |
| Bug provider/API | Đổi model, đổi key | Tách request / response / xử lý sau → tìm đúng chỗ fail |
| File generated | Sửa file đang chạy | Tìm source-of-truth, patch source, rebuild output |
| File lớn | Đọc cả file | Search anchor → đọc bounded range |
| File có tiếng Việt/CJK | Replace bằng tool tiện tay | Detect BOM trước khi ghi, verify sau khi ghi |
| Patch hỏng | Chồng thêm workaround | Restore `.bak2`, đọc lại range, patch nhỏ hơn |
| Agent mới vào | Đọc lại toàn bộ từ đầu | Đọc map/backup/KI hiện có → làm tiếp ngay |
| Spec cũ + code mới | Áp spec cũ vào code đã refactor | So sánh spec vs thực tế, hỏi trước khi merge |

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
