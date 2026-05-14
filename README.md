# my-skills-gitlab-flow

Bộ skills GitLab workflow cho Claude Code và các AI coding harness khác. Tách ra từ [`nguyenvanchiens/my-skills`](https://github.com/nguyenvanchiens/my-skills) để gọn install khi project chỉ cần workflow GitLab.

## Skills

| Skill | Mô tả | License |
|---|---|---|
| [`gitlab-flow`](skills/gitlab-flow/) | Quy trình end-to-end Jira → branch → commit → MR → review → fix → merge dùng `glab`. Chuẩn hoá branch naming, commit format và safety rules cho team GitLab. Đã tích hợp toàn bộ spec của `commit` và `review-branch` qua các trigger `commit and push` và `review the whole branch`. | MIT |
| [`commit`](skills/commit/) | Tạo commit theo Conventional Commits, Jira ID ở cuối subject trong ngoặc đơn `(WRA-9)` (`/commit WRA-9`). Tự phân tích diff, chọn `type`/`scope`, có `--quick` mode, partial-staging guard, đọc `.commit-scopes` allowlist. **Standalone** — nếu đã cài `gitlab-flow` thì không cần cài thêm. | MIT |
| [`review-branch`](skills/review-branch/) | Review toàn bộ thay đổi của branch hiện tại so với `main` (committed + uncommitted) qua 3 agent song song: reuse, quality, efficiency — rồi tự fix issue. Standalone — đã tích hợp trong `gitlab-flow`. | MIT |

> **Recommendation**: chỉ cần cài `gitlab-flow` là đủ cho cả 3 vai trò (workflow + commit + review). Cài `commit` hoặc `review-branch` riêng chỉ khi không dùng GitLab/`glab`.

## Cài đặt

Yêu cầu: Node.js (để dùng `npx`).

### Cài `gitlab-flow` (recommended — đã bao gồm `commit` + `review-branch`)

```bash
npx skills add nguyenvanchiens/my-skills-gitlab-flow -s gitlab-flow -y -a claude-code --copy
```

### Cài tất cả 3 skills

```bash
npx skills add nguyenvanchiens/my-skills-gitlab-flow --all -a claude-code --copy
```

### Cài standalone `commit` hoặc `review-branch`

Chỉ dùng nếu KHÔNG dùng GitLab/`glab`:

```bash
npx skills add nguyenvanchiens/my-skills-gitlab-flow -s commit         -y -a claude-code --copy
npx skills add nguyenvanchiens/my-skills-gitlab-flow -s review-branch  -y -a claude-code --copy
```

### Cài global (dùng cho mọi project)

Thêm `-g` vào lệnh, ví dụ:

```bash
npx skills add nguyenvanchiens/my-skills-gitlab-flow -s gitlab-flow -y -g -a claude-code --copy
```

### Cập nhật

```bash
npx skills update         # cập nhật tất cả
npx skills update -g      # chỉ global
```

### Gỡ cài đặt

```bash
npx skills remove
```

## Hỗ trợ harness khác

Cờ `-a` chấp nhận: `claude-code`, `cursor`, `gemini-cli`, `codex`, `opencode`, `windsurf`, `copilot`, ... (xem `npx skills --help` cho danh sách đầy đủ).

```bash
# Cursor
npx skills add nguyenvanchiens/my-skills-gitlab-flow -s gitlab-flow -a cursor --copy

# Tất cả harness phát hiện được
npx skills add nguyenvanchiens/my-skills-gitlab-flow --all -a "*" --copy
```

## Sử dụng `gitlab-flow`

Skill này không phải `/slash command` mà kích hoạt bằng **trigger phrase tiếng Anh** trong prompt thường. Claude tự match phrase và chạy procedure tương ứng.

### Yêu cầu trước khi dùng

- `git` (luôn có)
- [`glab`](https://gitlab.com/gitlab-org/cli) — GitLab CLI. Trên Windows: `winget install GLab.GLab`
- Đăng nhập 1 lần: `glab auth login --hostname <gitlab-host>` (token scope `api` + `write_repository`)

### Bảng trigger

> **Lưu ý copy-paste**: cột Prompt không dùng backtick để tránh GitHub auto-wrap thêm xuống dòng khi copy. Cứ chọn nguyên dòng prompt rồi paste vào Claude Code.

| Prompt | Hành động |
|---|---|
| **create branch from task &lt;TASK-ID&gt;** | Bóc tách task title → đề xuất 1-2 branch ngắn (2-4 từ key, ≤50 chars), **DỪNG hỏi user pick**, pull `main`, rồi mới `git checkout -b feature/<TASK-ID>-<short-desc>` |
| **rename branch &lt;new-name&gt;** | Đổi tên branch hiện tại đồng bộ local + remote. Detect upstream → nếu chưa push: rename thuần. Đã push: rename local + push tên mới + hỏi xóa branch cũ trên remote. Tránh tình trạng local≠remote name làm hỏng push/MR sau đó |
| (paste mô tả task Jira) | Đọc scope, sinh code theo convention project |
| **review the last change** | Chạy `git diff`, list issues `#1`, `#2`... |
| **review the whole branch** | Review cumulative branch vs `main` qua 3 agent song song (Reuse / Quality / Efficiency), tự fix issues. **Macro review** trước khi commit cuối / mở MR. |
| **commit and push** (kèm `--quick` nếu cần) | Self-contained — kế thừa toàn bộ spec của `/commit`: probe repo, partial-staging guard, atomic check, `.commit-scopes` allowlist, 11 types, footer (`Closes`/`Refs`...), Quick mode, WIP/Spike, revert format. TASK-ID tự lấy từ tên nhánh. Commit local xong **HỎI user** có push không (không tự push). Detect upstream tracking — nếu local branch khác upstream (rename scenario) → STOP, hướng user qua `rename branch`. Không cần cài skill `commit` riêng. |
| **create a merge request** | `glab mr create` với title/description chuẩn |
| **review the MR !&lt;N&gt;** | Lấy `glab mr diff <N>` + comment đã có. **MR chưa có comment** → review mới, list issues + verdict. **MR đã có comment** → review tiếp nối: đối chiếu issue cũ (`✓ Resolved` / `❌ Still open` / `⚠️ Partially`) + chỉ review commit mới push thêm |
| **post review result to the MR** | `glab mr note` đăng comment Markdown |
| **fix all issues** / **fix issue #&lt;N&gt;** | Fix các issue → tóm tắt + đề xuất commit message `fix(<scope>): address review issues #N (<TASK-ID>)` → **HỎI user xác nhận** trước khi commit/push (không tự động) |
| **merge the request** | Check approve + CI pass → `glab mr merge --squash --remove-source-branch` |

### Flow điển hình end-to-end

```
1. create branch from task WRA-40 giới hạn domain account
2. (paste mô tả task)              → Claude code
3. review the last change          → fix nếu cần (lặp 2↔3 nhiều lần)
4. commit and push                 (lặp 2-4 cho từng đoạn)
   ...
5. review the whole branch         → macro review + auto-fix, trước MR
6. commit and push                 → commit fix nếu /review-branch sửa gì
7. create a merge request

   --- chuyển sang vai Reviewer ---

8.  review the MR !21              → Claude in review ra terminal (chưa lên GitLab)
9.  (đọc, chỉnh nếu cần)
10. post review result to the MR   → mới đẩy comment lên GitLab

    --- quay lại vai Developer ---

11. fix all issues                 → fix xong, đợi user xác nhận
12. (xác nhận) commit and push
13. merge the request
```

> **Lưu ý**: bước 8 và 10 là **2 prompt riêng**, không tự động nối. Mục đích để reviewer xem trước nội dung review, có thể yêu cầu Claude bổ sung/sửa, mới quyết định post lên MR.

### Convention

- **Branch**: **default `feature/<TASK-ID>-<short-desc>`** cho mọi loại thay đổi (kể cả bug fix). User override bằng cách tự gõ `bugfix/...` hoặc `hotfix/...` (Mode A — skill respect nguyên si). Desc 2-4 từ key, kebab-case, không dấu, **tổng ≤50 chars**. Drop **type filler** (`Cai-tien`, `Improve`, `Fix`, `Sua`, `Tao`, `Add`, `Create`, `Them`, `Bo-sung`) nhưng **KEEP direction marker** (`Allow`, `Validate`, `Block`, `Duplicate`, `Stale`, `Missing`) **và context marker** (`Show`/`Display`, `Filter`/`Sort`, `Sync`/`Migrate`). Vd `feature/SMT-460-Allow-qty-0-checkin-checkout` (43), bug fix: `feature/HNCW-311-Duplicate-survey-log` (37)
- **Commit**: `<type>(<scope>): <subject> (<TASK-ID>)` (vd `feat(auth): restrict login to allowed domains (WRA-40)`)
- **Target branch**: mặc định `main` — nếu repo dùng `master`, thêm dòng vào `CLAUDE.md` của project: `Default branch: master (not main)`

### Safety rules

- KHÔNG force push vào nhánh đã có MR mở
- KHÔNG merge thẳng vào `main` từ local — luôn qua MR
- KHÔNG bypass hooks (`--no-verify`) trừ khi user yêu cầu rõ
- KHÔNG commit secrets (`.env`, key, token, password)

Xem chi tiết đầy đủ ở [`skills/gitlab-flow/SKILL.md`](skills/gitlab-flow/SKILL.md).

## Add-on: `gitlab-sync` (deploy QA cho monorepo multi-app)

Nếu team dùng convention `main → builds/dev/<app>` để trigger CI/CD deploy QA và project là monorepo multi-app, cài thêm `gitlab-sync` từ repo gốc:

```bash
npx skills add nguyenvanchiens/my-skills -s gitlab-sync -y -a claude-code --copy
```

Xem chi tiết tại [`nguyenvanchiens/my-skills`](https://github.com/nguyenvanchiens/my-skills).

## Cấu trúc repo

```
my-skills-gitlab-flow/
├── README.md
└── skills/
    ├── gitlab-flow/
    │   └── SKILL.md
    ├── commit/
    │   └── SKILL.md
    └── review-branch/
        └── SKILL.md
```

Mỗi `SKILL.md` có frontmatter `name` + `description` để CLI `npx skills` tự nhận diện.

## License

MIT.
