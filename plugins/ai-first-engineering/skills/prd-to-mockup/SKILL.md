---
name: prd-to-mockup
description: Tạo wireframe/mockup từ Lean PRD hoặc Screen States description. Use when user has a PRD with Screen States section, hoặc nói "vẽ mockup từ PRD", "tạo wireframe", "mockup cho feature", "vẽ màn hình", "UI từ spec", "render screen states". Nhận input từ lean-prd (Screen States + Workflow) và render thành HTML wireframe tương tác được. Không dùng cho high-fidelity design — dùng Figma/design tool cho mục đích đó. MANDATORY TRIGGERS: wireframe, mockup, vẽ màn hình, UI từ spec, screen states, prd to mockup.
---

# PRD to Mockup — Wireframe Generator

Chuyển Screen States từ Lean PRD thành **low-fidelity wireframe** dạng HTML. Mỗi screen state = 1 wireframe, liên kết theo workflow diagram.

## Triết lý

> **Wireframe là visual validation cho Business Rules, không phải pixel-perfect design.**

Mục đích:
1. PM/Stakeholder validate: "đúng, user sẽ thấy như vậy"
2. Gen engineer hiểu UI structure trước khi code
3. prd-to-gherkin biết `@web` scenario cần verify element gì

Wireframe KHÔNG quyết định: màu sắc, font, spacing cụ thể, responsive behavior → đó là việc của designer.

---

## Workflow: Parse → Render → Link

### Step 1 — Parse Screen States từ PRD

Khi nhận PRD (lean-prd format), extract:

1. **Danh sách Screen States** — mỗi state = 1 screen
2. **Workflow diagram** — thứ tự screens, navigation flow
3. **Business Rules per screen** — rule nào active tại screen đó
4. **Data elements** — field hiển thị, editable, read-only
5. **Actions** — buttons/links, rule guard cho mỗi action
6. **Error states** — user thấy gì khi rule bị vi phạm

Nếu PRD không có Screen States, hỏi:
> "PRD này chưa có Screen States. Bạn mô tả sơ: user nhìn thấy gì ở mỗi bước trong workflow?"

### Step 2 — Render Wireframe

Tạo **1 file HTML duy nhất** chứa tất cả screens, navigate giữa chúng được.

#### Nguyên tắc render

**Layout:**
- Dùng gray boxes, borders, placeholder text
- Không dùng màu sắc (trừ: đỏ cho error state, xanh lá cho success)
- Font: system default, 1 size cho content, 1 size lớn hơn cho heading
- Max width 800px, centered

**Elements mapping:**

```
Screen State field        → Wireframe element
─────────────────────────────────────────────
Hiển thị (read-only data) → Gray box với label + placeholder value
Có thể sửa (editable)    → Input field với border + label
Chỉ đọc (from source)    → Gray background input, disabled look
Actions (buttons)         → Button với label, annotate guard rule
Khi lỗi (error)          → Red border + error message text
```

**Annotations (quan trọng!):**
- Mỗi element có annotation nhỏ ghi Rule ID liên quan: `[BR-M01]`
- Mỗi button ghi guard condition: `[Submit] → requires BR-M01, BR-M02`
- Mỗi screen có title = State name từ workflow

**Navigation:**
- Mỗi action button link sang screen tiếp theo (theo workflow)
- Có breadcrumb/flow indicator ở top: `Nháp → Đang kiểm tra → Đã gửi → Đã duyệt`
- Highlight current state trong flow

#### HTML Template structure

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <title>Wireframe: [Feature Name]</title>
  <style>
    /* Minimal wireframe styles - gray, borders, system font */
    * { font-family: system-ui; box-sizing: border-box; }
    body { max-width: 800px; margin: 0 auto; padding: 20px; background: #f5f5f5; }

    .screen { background: white; border: 2px solid #333; padding: 20px; margin: 20px 0; display: none; }
    .screen.active { display: block; }
    .screen-title { font-size: 18px; font-weight: bold; border-bottom: 1px solid #ccc; padding-bottom: 10px; }

    .flow-nav { display: flex; gap: 8px; margin: 20px 0; flex-wrap: wrap; }
    .flow-step { padding: 8px 16px; border: 1px solid #999; cursor: pointer; background: #eee; }
    .flow-step.current { background: #333; color: white; }

    .field-group { margin: 12px 0; }
    .field-label { font-size: 12px; color: #666; margin-bottom: 4px; }
    .field-input { border: 1px solid #999; padding: 8px; width: 100%; }
    .field-readonly { background: #e9e9e9; border: 1px solid #ccc; padding: 8px; width: 100%; }
    .field-display { background: #f0f0f0; padding: 8px; border: 1px dashed #ccc; }

    .btn { padding: 10px 20px; margin: 4px; cursor: pointer; border: 2px solid #333; background: white; }
    .btn-primary { background: #333; color: white; }
    .btn:hover { opacity: 0.8; }

    .annotation { font-size: 10px; color: #888; font-style: italic; }
    .error-state { border-color: red; }
    .error-msg { color: red; font-size: 12px; margin-top: 4px; }
    .success-msg { color: green; font-size: 12px; }

    .actions { margin-top: 20px; padding-top: 12px; border-top: 1px solid #eee; }

    /* Error variant toggle */
    .error-variant { display: none; border: 2px solid red; background: #fff5f5; padding: 12px; margin: 8px 0; }
    .show-error .error-variant { display: block; }
    .toggle-error { font-size: 11px; color: red; cursor: pointer; text-decoration: underline; }
  </style>
</head>
<body>
  <h1>Wireframe: [Feature Name]</h1>
  <p class="annotation">Generated from Lean PRD — Screen States section</p>

  <!-- Flow navigation -->
  <div class="flow-nav">
    <!-- One step per screen state, linked to workflow -->
  </div>

  <!-- Screens -->
  <div class="screen active" id="screen-1">
    <div class="screen-title">[State Name]</div>
    <p class="annotation">Trigger: [khi nào user vào screen này]</p>

    <!-- Fields -->
    <div class="field-group">
      <div class="field-label">[Label] <span class="annotation">[BR-Mxx]</span></div>
      <input class="field-input" placeholder="[placeholder]" />
    </div>

    <!-- Error variant (toggle) -->
    <span class="toggle-error" onclick="this.closest('.screen').classList.toggle('show-error')">
      Xem error state
    </span>
    <div class="error-variant">
      <div class="error-msg">[Error message khi rule vi phạm]</div>
    </div>

    <!-- Actions -->
    <div class="actions">
      <button class="btn btn-primary" onclick="showScreen('screen-2')">[Action] [BR-Mxx]</button>
      <button class="btn">[Secondary action]</button>
    </div>
  </div>

  <script>
    function showScreen(id) {
      document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
      document.getElementById(id).classList.add('active');
      // Update flow nav
      document.querySelectorAll('.flow-step').forEach(s => s.classList.remove('current'));
      document.querySelector(`[data-screen="${id}"]`)?.classList.add('current');
    }
  </script>
</body>
</html>
```

### Step 3 — Output

1. **File HTML wireframe** — 1 file duy nhất, tất cả screens, navigate được
2. **Screen inventory table:**

```
## Screen Inventory

| # | Screen Name | Workflow State | Rules Active | Elements | Actions |
|---|-------------|---------------|-------------|----------| --------|
| 1 | [name] | [state] | BR-M01, BR-M02 | 5 fields, 2 display | 2 buttons |
| 2 | [name] | [state] | BR-M03 | 3 fields | 1 button |
```

3. **Annotation legend:**

```
## Annotations

Wireframe sử dụng annotations sau:
- [BR-Mxx] trên field/button = Business Rule guard cho element đó
- Xem error state = click để toggle hiển thị error variant
- Gray background = read-only field (data từ nguồn khác)
- Red border = error state khi rule bị vi phạm
```

---

## Quality Checklist

- [ ] Mỗi Screen State trong PRD có 1 screen tương ứng trong wireframe
- [ ] Mỗi editable field trong Screen State là input field trong wireframe
- [ ] Mỗi action button có annotation Rule ID guard
- [ ] Error states có thể toggle xem được
- [ ] Navigation giữa screens follow workflow diagram
- [ ] Wireframe không có styling/color (trừ error red, success green)
- [ ] Annotations đủ để map ngược về PRD Business Rules
- [ ] Tất cả label và text hiển thị bằng tiếng Việt

---

## Lưu ý

- Wireframe là **throwaway artifact** — khi design chính thức được tạo, wireframe hết vai trò
- Không optimize cho mobile/responsive — đây là validation tool, không phải production UI
- Nếu PRD không có Screen States, hỏi user mô tả trước rồi mới render
- Nếu feature quá phức tạp (>10 screens), chia thành nhiều wireframe files theo workflow phase
