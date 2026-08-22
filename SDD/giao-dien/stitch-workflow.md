# Workflow Stitch MCP

Sinh UI trên [Google Stitch](https://stitch.withgoogle.com) từ Cursor, rồi agent implement vào `mobile/` (Expo) hoặc `admin/` (Next.js).

Skill: `.claude/skills/stitch-ui/` · Lệnh: `/stitch-screen` · Prompt sẵn: `prompts.md` trong skill.

## Một lần trên máy dev

1. Đăng nhập [Stitch](https://stitch.withgoogle.com) → ảnh hồ sơ → **Stitch settings** → **Create API key**. Copy ngay.
2. Gán biến **user** Windows (không commit):

   ```powershell
   setx STITCH_API_KEY "dán-key-vào-đây"
   ```

   Đóng hẳn Cursor rồi mở lại (MCP không nhận `setx` của session cũ).

3. Nếu `${env:STITCH_API_KEY}` không nạp: copy `.cursor/mcp.json.example` → `.cursor/mcp.json`, dán key vào `env`. File `.cursor/mcp.json` đã gitignore nếu chứa key thật — dùng example đã commit với interpolations thì không cần dán key vào repo.
4. Cursor: **Settings → Tools & MCP**. Server `stitch` phải **xanh**. Nếu 0 tool: dùng proxy `npx @_davideast/stitch-mcp proxy` (đã ghi trong `mcp.json`), đừng chỉ trỏ HTTP `stitch.googleapis.com/mcp` (Cursor hay nuốt `tools/list` vì schema quá lớn).
5. Kiểm tra CLI (tuỳ chọn): `npx @_davideast/stitch-mcp doctor`

Cấu hình đã commit: [`.cursor/mcp.json`](../../../.cursor/mcp.json) — chỉ interpolations, không có secret.

## Mỗi màn hình

Trong chat Cursor:

```
/stitch-screen tab bản đồ
```

hoặc: *prompt Stitch màn chi tiết quán theo SDD*.

Agent: nối MCP → project **FoodMap** → generate một màn → lấy ảnh/HTML → code RN/Next → i18n.

Quota Stitch (free) giới hạn số lần generate — làm Must Have trước: Bản đồ, Chi tiết, Login, Khám phá, Chat, Dashboard admin.

## Mapping màn → code

| Stitch | Code |
|---|---|
| Map tab | `mobile/src/app/(tabs)/index.tsx` |
| Explore | `mobile/src/app/(tabs)/explore.tsx` |
| Place detail | `mobile/src/app/place/[id].tsx` |
| Login | `mobile/src/app/(auth)/login.tsx` |
| Admin home | `admin/src/app/page.tsx` |

Ghi `projectId` / `screenId` vào [`stitch-screens.md`](./stitch-screens.md) sau khi generate xong.
