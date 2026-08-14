# ADR-0004: Repo cha + git submodules thay vì monorepo

| | |
|---|---|
| Trạng thái | Chấp nhận |
| Ngày | 2026-08-14 |

## Bối cảnh

FoodMap có bốn phần độc lập về công nghệ và vòng đời: tài liệu, backend (Java/Gradle),
mobile (Expo/npm), admin (Next.js/npm). Cần quyết định cách tổ chức mã nguồn trên Git.

Ba mối quan tâm:
- Bốn phần có chu kỳ phát hành khác nhau — mobile phải qua app store, backend deploy liên tục.
- Muốn tách được quyền truy cập theo phần (ví dụ cộng tác viên chỉ đụng vào `docs`).
- Cần một chỗ duy nhất chứa hạ tầng dev, script và hướng dẫn cho AI agent.

## Quyết định

Repo cha `foodmap` chứa cấu hình chung, hạ tầng dev, script và thư mục `.claude/`.
Bốn repo con gắn vào bằng **git submodule**:

```
foodmap/
├─ docs/     → foodmap-docs
├─ backend/  → foodmap-backend
├─ mobile/   → foodmap-mobile
├─ admin/    → foodmap-admin
├─ infra/    (thuộc repo cha)
├─ scripts/  (thuộc repo cha)
└─ .claude/  (thuộc repo cha)
```

Quy trình làm việc: commit và push ở repo con trước, rồi cập nhật con trỏ ở repo cha.

## Phương án đã cân nhắc

**Monorepo một repo duy nhất (pnpm workspace + Turborepo).**
Đơn giản nhất: một lần clone, một lần commit cho thay đổi xuyên nhiều phần, CI một chỗ,
không có lớp submodule để hiểu. Loại vì không tách được quyền truy cập và lịch sử theo phần,
và vì Gradle không hoà vào workspace JS một cách tự nhiên — công cụ monorepo JS
không quản lý được backend Java.

**Bốn repo hoàn toàn rời, không có repo cha.**
Đơn giản về Git. Loại vì không có chỗ nào chứa `docker-compose.yml`, script bootstrap,
hay hướng dẫn AI dùng chung; người mới vào phải clone bốn chỗ và tự ghép lại.
Cũng không ghi lại được "phiên bản nào của bốn phần đi cùng nhau".

**Google repo tool / git subtree.**
`subtree` không cần lệnh đặc biệt khi clone. Loại vì lịch sử bị trộn vào repo cha,
làm repo cha phình to và khiến việc tách quyền mất ý nghĩa. `repo` tool thì thêm một
công cụ ngoài Git mà đội chưa dùng.

## Hệ quả

### Tích cực
- Mỗi phần có lịch sử, CI, issue và quyền truy cập riêng.
- Con trỏ submodule ở repo cha ghi lại **chính xác** tổ hợp phiên bản đang hoạt động
  cùng nhau — checkout repo cha ở một commit cũ là dựng lại được đúng trạng thái đó.
- Hạ tầng dev, script và `.claude/` nằm một chỗ, không bị trùng lặp ở bốn nơi.
- Repo con nhẹ; clone nhanh; người chỉ làm tài liệu không phải tải cả mã nguồn.

### Tiêu cực
- **Submodule khó hơn monorepo, và cái bẫy quen thuộc là quên cập nhật con trỏ:**
  commit ở repo con nhưng không commit gitlink ở repo cha → người khác clone về thấy code cũ.
- Thay đổi xuyên nhiều phần (ví dụ thêm một endpoint) cần commit ở 2–3 repo,
  không có một PR duy nhất thể hiện toàn bộ thay đổi.
- Phải nhớ `git clone --recurse-submodules`, hoặc chạy `git submodule update --init` sau đó.
- CI phức tạp hơn: cần một workflow ở repo cha để bump con trỏ.
- Không share code trực tiếp giữa mobile và admin được — nhưng điều này đã được giải quyết
  bằng client sinh từ OpenAPI ([ADR-0002](./ADR-0002-contract-first-openapi.md)).

### Giảm nhẹ
- `scripts/bootstrap` chạy `git submodule update --init --recursive`, người mới không cần
  biết chi tiết.
- Slash command `/sync` kiểm tra và báo submodule nào đang lệch con trỏ.
- `AGENTS.md` có riêng một mục cảnh báo về cái bẫy quên cập nhật con trỏ.

### Cần theo dõi
- Tần suất "quên cập nhật con trỏ" trong thực tế. Xảy ra thường xuyên thì thêm bước
  kiểm tra vào CI của repo cha, hoặc xem lại quyết định này bằng một ADR mới.
- Chi phí của thay đổi xuyên nhiều repo. Nếu phần lớn thay đổi đều chạm cả ba repo
  thì lợi ích của việc tách đang không đủ bù chi phí.
