# Update-Helper Skills

A collection of AI Agent skills and workflows for safely modifying massive source files without breaking context windows or destroying formatting. 
Inspired by [andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills), but focused on hardcore file manipulation safety.

## 📦 Skills Included

### 1. `update-helper` (The Field Manual)
**Universal Large File Patcher — Complete Field Manual v3.3**
The complete, battle-tested survival manual for AI agents when patching large codebases (1,000–30,000+ lines). It covers:
- The 3-Tier Backup Protocol.
- Code Comprehension Layer (Intent Mapping, Data Flow Tracing).
- Smart Patching Workflows and Multi-Patch Offset Strategies.
- Strict Encoding Safety (BOM preservation for CJK/Vietnamese characters).

### 2. `update-helper-guidelines` (The Behavioral Rules)
A concise set of Karpathy-style behavioral rules (in Vietnamese) that enforce the core philosophies of `update-helper`. Perfect for quick system prompt inclusion to ensure AI agents behave safely before touching any code.
- **Không Tự Bịa Code (Never Assume Target Content)**
- **Khảo Sát Kỹ Trước Khi Hành Động (Read Before You Patch)**
- **Bảo Vệ Dữ Liệu Tuyệt Đối (Safety First: Backups & Encoding)**
- **Vá Lỗi Phẫu Thuật, Sửa Từ Dưới Lên (Surgical, Bottom-Up Execution)**

## 🚀 Installation & Usage

You can use these skills with supported AI Agent platforms (like Cline, Cursor, Antigravity, OpenClaw, etc.) by placing them in your `skills` or `.cursorrules` directory.

```bash
# Clone this repository
git clone https://github.com/pitroytech/update-helper-skills.git

# Copy the skills to your AI agent's skills directory
cp -r update-helper-skills/skills/update-helper ~/.gemini/antigravity/skills/
cp -r update-helper-skills/skills/update-helper-guidelines ~/.gemini/antigravity/skills/
```

## 📜 License
MIT License. Created by PitroyTech.
